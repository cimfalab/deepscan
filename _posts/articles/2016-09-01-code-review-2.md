---
layout: post
title: JavaScript 코드 리뷰 - 코드 리뷰 사례로 알아보는 좋은 JavaScript 코드
excerpt: "코드 리뷰 문화에 관해 설명하고 JavaScript 코드의 실제 리뷰 사례를 통해 좋은 JavaScript 코드에 대해 알아봅니다."
date: 2016-09-01 19:00:00 +0900
share: true
categories: articles
tags:
- JavaScript
- JavaScript 정적 분석
- Code Review
- 코드 리뷰
related:
- JavaScript 코드 리뷰 - 코드 리뷰 문화
---

오늘은 [코드 리뷰 문화]({{ site.baseurl }}{% post_url 2016-08-22-code-review-1 %})에 이어 실제로 코드 리뷰를 하면서 개선된 코드 사례를 몇 가지 공유합니다.

라이브러리의 활용, 좋은 코드를 위한 리팩토링과 같이 작지만 유용하다고 생각했던 사례들을 코드 및 실제 코멘트와 함께 정리해 보도록 하겠습니다.

* Table of Contents
{:toc}

## 라이브러리의 활용

### Underscore

Underscore나 Lodash 같은 라이브러리를 잘 활용하면 같은 기능을 더 간결한 코드로 구현할 수 있습니다.
팀에서 "Underscore 라이브러리를 잘 쓰면 점수(score)를 받을 수 있다"는 우스갯소리를 한 적도 있을 정도로요.

Underscore나 Lodash는 객체나 컬렉션을 다룰 때 유용하니 [API 레퍼런스](https://lodash.com/docs)를 참고해서 잘 활용하면 좋습니다. 특히 컬렉션을 loop 할 일이 많은 경우 [_.forIn, _.map 등을 적절히 사용](http://joelhooks.com/blog/2014/02/06/stop-writing-for-loops-start-using-underscorejs/)하면 좋겠습니다.

#### pluck

아래와 같은 json 데이터로부터 "mobile"과 "wearable"이 포함된 배열을 반환하는 `getProfiles` 함수의 리뷰를 보죠.

```javascript
{
    "mobile-2.4.0": {
        "profile": "mobile"
    },
    "mobile-2.3.1": {
        "profile": "mobile"
    },
    "wearable-2.3.1": {
        "profile": "wearable"
    },
    "wearable-2.3.0": {
        "profile": "wearable"
    }
}
```

JavaScript 객체를 순환하면서 'profile' 키의 값을 얻어야 하고, 배열 중 중복되는 값은 제외해야 하죠.
이렇게 구현한 최초 코드가 왼쪽이고 해당 커밋에 달린 리뷰 코멘트입니다. ("Done"은 요청자가 리뷰에 따라 수정을 완료했을 경우 올리는 코멘트)

[pluck](https://lodash.com/docs#map)[^1]은 지정된 속성의 값만 쏙 뽑아주므로 리뷰대로 수정하면 원하는 기능이 단 한 줄로 구현됩니다.

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="3-7"><code class="language-javascript">function getProfiles() {
    return getPlatforms().then(function (platforms) {
        var profiles = _.reduce(platforms, function (result, platform) {
            result.push(platform.profile);
            return result;
        }, []);
        return _.uniq(profiles);
    });
}
</code></pre>
<blockquote class="quotes reviewer">
  <p>return _.uniq(_.pluck(platforms, ‘profile’));</p>
</blockquote>
<blockquote class="quotes">
  <p>Done</p>
</blockquote>
</td><td>
<pre><code class="language-javascript">function getProfiles() {
    return getPlatforms().then(function (platforms) {
        return _.uniq(_.pluck(platforms, 'profile'));
    });
}
</code></pre>
</td></tr>
</table>

#### last

배열의 마지막 요소를 얻을 때 사용할 수 있습니다.

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="2,9"><code class="language-javascript">function getLastAnalysis(project) {
    if (project && project.analyses && project.analyses.length >= 0) {
        return project.analyses[project.analyses.length - 1];
    }

    return null;
}

var lastAnalysis = getLastAnalysis(this.content);
</code></pre>
<blockquote class="quotes reviewer">
  <p>[-1]인 경우가 생길 것 같은데요?</p>
</blockquote>
<blockquote class="quotes reviewer">
  <p>_.last(this.content)</p>
</blockquote>
</td><td>
<pre><code class="language-javascript">var lastAnalysis, project = this.content;
if (project && project.analyses) {
    lastAnalysis = _.last(project.analyses);
}
</code></pre>
</td></tr>
</table>

단순한 로직이지만 경계값 체크를 놓치기 쉽죠. 소위 '바퀴를 재발명'하지 말고 검증된 라이브러리를 활용하는 것이 코드의 간결성과 예외 처리에 도움이 됩니다.
참고로 [last](https://lodash.com/docs#last)의 코드는 아래와 같습니다.

```javascript
function last(array) {
  var length = array ? array.length : 0;
  return length ? array[length - 1] : undefined;
}
```

#### some

배열에서 특정 요소의 존재 여부를 파악할 때 유용한 [some](https://lodash.com/docs#some)입니다.

아래는 배열에 어떤 값이 존재하지 않을 때만 배열에 추가하는 코드인데, 보통 많이 하는 loop을 통한 존재 여부 체크 없이도 배열의 각 요소를 인자로 받아 true/false를 반환하는 함수(predicate)만 넘겨 줌으로써 더 간결하게 구현됩니다.

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="2-11"><code class="language-javascript">addModel: function (modelName, model) {
    var exist = false;

    for (i = 0, length = cModels.length; i < length; i++) {
        cModel = cModels[i];

        if (cModel.isEqual(model)) {
            exist = true;
            break;
        }
    }

    if (!exist) {
        cModels.push(model);
    }
}
</code></pre>
</td><td>
<pre><code class="language-javascript">addModel: function (modelName, model) {
    var exist = _.some(cModels, model.isEqual.bind(model));

    if (!exist) {
        cModels.push(model);
    }
}
</code></pre>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>아래와 같이 하면 더 깔끔하겠네요
<pre>
var exist = _.some(cModels, model.isEqual.bind(model));
</pre>
  </p>
</blockquote>
</td></tr>
</table>

#### debounce

비용이 많이 드는 함수(서버 요청, 느린 I/O 등)의 빈번한 호출은 [debounce](https://lodash.com/docs#debounce)를 통해 호출을 제한시킬 수 있는지 검토할 필요가 있습니다.

입력 값에 따라 서버 파일을 검색하는 기능의 구현을 보죠.

  ![]({{ site.baseurl }}/assets/images/code-review-18.png)

아래 코드의 `search` 함수가 서버의 파일 목록을 요청하는데 REST API를 통한 네트워크 요청이면서 서버 I/O가 필요하므로 꽤 무겁습니다. 이를 오른쪽과 같이 `debounce`를 적용해 수정하면 사용자 키 입력마다 매번 호출되지 않고 0.3초 동안에는 한 번만(마지막 함수 호출만) 호출되므로 보다 효율적입니다.

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="2"><code class="language-javascript">this.$input.on('keyup', function (e) {
    search();
});
</code></pre>
</td><td>
<pre><code class="language-javascript">// to avoid costly search call for each user input
var searchDebounced = _.debounce(search, 300);

this.$input.on('keyup', function (e) {
    searchDebounced();
});
</code></pre>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>입력할 때마다 search api가 너무 많이 불릴 수 있을 것 같네요. debounce를 쓰면 좋겠습니다.</p>
</blockquote>
<blockquote class="quotes">
  <p>debounce 적용했습니다. 좋은 함수네요^^</p>
</blockquote>
</td></tr>
</table>

### URI.js

URL을 다룰 때는 [URI.js](https://medialize.github.io/URI.js/) 모듈을 사용합니다.
프로토콜, 호스트, 경로 정보를 쉽게 얻을 수 있고 파싱 같이 URL 처리에 필요한 기능들이 모두 제공되므로 URI.js 객체를 생성하고 이를 이용하는 것이 좋습니다.

<table class="compare">
<tr><td>
<pre><code class="language-javascript">var protocol = 'http://';
var hostName = app.getHost();
if (hostName.startsWith('fs')) {
    hostName = hostName.split('.');
    hostName.shift();
    hostName = hostName.join('.');
}

this.definitionUrl = protocol + hostName + '/sample-type-info.json';
</code></pre>
</td><td>
<pre><code class="language-javascript">function getSampleServerAddress() {
    var uriObj = URI(conf.appServer);
    uriObj.pathname('sample-type-info.json');

    return uriObj.toString();
}

this.definitionUrl = getSampleServerAddress();
</code></pre>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>
함수로 빼서 로직에 이름을 주세요. 그리고 url을 다룰 때는 항상 URIjs 모듈을 이용하세요.<br>
protocol도 하드코딩 되어서는 안 됩니다.
  </p>
</blockquote>
</td></tr>
</table>

## Promise

Promise를 활용하면 비동기 프로그래밍에서의 어지러운 콜백 흐름을 동기 프로그래밍처럼 간결하게 표현할 수 있습니다.
최신 브라우저에서 대부분 지원되기도 하지만, [bluebird](http://bluebirdjs.com/docs/getting-started.html) 라이브러리를 통해서도 바로 사용할 수 있습니다.

Promise 사용이 익숙지 않아 자주 리뷰된 내용 위주로 정리해 봅니다.

### return과 throw의 사용

Promise의 장점은 비동기 프로그래밍을 개발자들에게 익숙한 동기 프로그래밍처럼 할 수 있게 해 주는 것이라고 생각합니다.
기존의 비동기 콜백을 정상적인 return과 에러 상황에서의 throw로 쓸 수 있다는 것이죠.

Promise에서 어떤 값을 반환하거나 예외를 던지면 그렇게 resolved 혹은 rejected 되므로 굳이 `Promise.reject`를 사용할 이유가 없습니다.

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="5"><code class="language-javascript">return dbProject.$saveAsync(data)
    .then(function (context) {
        return ...;
    }).catch(function (error) {
        return Promise.reject(new Error(error));
    });
</code></pre>
</td><td>
<pre><code class="language-javascript">return dbProject.$saveAsync(data)
    .then(function (context) {
        return ...;
    }).catch(function (error) {
        throw new Error(error);
    });
</code></pre>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>return Promise.reject() 보다 throw 직접 사용</p>
</blockquote>
</td></tr>
</table>

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="2-5"><code class="language-javascript">return new Promise(function (resolve, reject) {
    if (widgetElement.nodeName.toLowerCase() !== TAG_NAME.WIDGET) {
        errorInfos.push(ERROR_MESSSAGE.WRONG_ROOT);
        return reject(errorInfos);
    }

    if (profileElements.length < 1) {
        errorInfos.push(ERROR_MESSSAGE.NOT_EXIST_PROFILE);
        return reject(errorInfos);
    }
</code></pre>
</td><td>
<pre><code class="language-javascript">function checkWrongRootNode(node) {
    if (node.nodeName.toLowerCase() !== TAG_NAME.WIDGET) {
        errorInfos.push(ERROR_MESSSAGE.WRONG_ROOT);
        throw errorInfos;
    }
}

return new Promise(function (resolve, reject) {
    checkWrongRootNode(widgetElement);
    checkWrongProfileNode(widgetElement);
</code></pre>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>reject를 쓰지 않고 throw를 쓰고 promise 함수 바깥으로 빼면 아래와 같이 promise 함수를 작게 만들 수 있겠습니다.
<pre>
return new Promise(function (resolve, reject) {
  checkXX1();
  checkXX2();
  ...
</pre>
  </p>
</blockquote>
</td></tr>
</table>

마찬가지로 then에 주는 함수가 그냥 값을 반환하면 그 값으로 resolve 되는 Promise가 만들어집니다.

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="2-5"><code class="language-javascript">function getDevicePath(result) {
    return new Promise(function (resolve) {
        result.devicePath = ...;
        resolve(result);
    });
}

return packageCommands.packageProject(path)
                      .then(getDevicePath)
                      .catch(handlePackageError);
</code></pre>
</td><td>
<pre><code class="language-javascript">function getDevicePath(result) {
    result.devicePath = ...;
    return result;
}

return packageCommands.packageProject(path)
                      .then(getDevicePath)
                      .catch(handlePackageError);
</code></pre>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>onFulfilled에서 promise를 만들어서 리턴해도 되지만 그냥 값을 리턴하면 그 값으로 resolve 된 promise가 만들어집니다. 따라서 아래와 같이 쓰면 됩니다.
<pre>
  function getDP(result) {
    ...
    return result;
  }
</pre>
  </p>
</blockquote>
<blockquote class="quotes">
  <p>회의 때 들은 내용인데 막상 Promise를 쓰려니 잘 적용이 안 되네요, then의 첫 번째 파라미터는 그냥 값을 리턴하는 함수여도 chaining이 되는 거군요. 처리했습니다.</p>
</blockquote>
</td></tr>
</table>

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="2-18"><code class="language-javascript">Validator.validate = function (profile) {
    return new Promise(function (resolve) {
        Platform.getPlatforms()
                .then(function (platforms) {
                    var matched = _.filter(platforms, function (p) {
                        return p.getProfile() === profile;
                    });

                    if (matched.length > 0) {
                        resolve(true);
                    } else {
                        resolve(false);
                    }
                })
                .catch(function (err) {
                    resolve(false);
                });
    });
};
</code></pre>
</td><td>
<pre><code class="language-javascript">Validator.validate = function (profile) {
    return Platform.getPlatforms()
                   .then(function (platforms) {
                       var matched = _.filter(platforms, function (p) {
                           return p.getProfile() === profile;
                       });

                       if (matched.length > 0) {
                           return true;
                       } else {
                           return false;
                       }
                    });
};
</code></pre>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>굳이 new Promise를 할 필요가 없습니다.</p>
</blockquote>
</td></tr>
</table>

### reject 처리

Promise나 Deferred를 사용할 때 성공 콜백만 정의할 경우 문제가 발생할 수 있습니다.

```javascript
function getViableItems() {
    var menuItems = {};
    var deferred = new Deferred();

    function updateViableItems(isWebProject) {
        menuItems = ...;
        deferred.resolve(menuItems);
    }

    common.isWebProject(projectPath)
          .then(updateViableItems);

    return deferred.promise;
}
```

위 코드는 dojo의 Deferred 객체를 사용하는데, reject 상황에서 `menuItems` 배열이 반환되지 않아 메뉴 표시에 문제가 있었습니다.
Promise라면 finally를 써야 할 상황이고, Deferred에서는 실패 콜백을 항상 같이 쓰거나 then 다음에 또 then을 써서 최종 반환 값이 유실되지 않도록 할 필요가 있겠습니다.[^2]

```javascript
function getViableItems() {
    var menuItems = {};
    var deferred = new Deferred();

    function updateViableItems(isWebProject) {
        menuItems = ...;
        deferred.resolve(menuItems);
    }

    common.isWebProject(projectPath)
          .then(updateViableItems, function () {
              // Should return items although in case of reject (invalid configuration, etc.)
              deferred.resolve(itemsForProject);
          });

    return deferred.promise;
}
```

### Promisify 및 bind

기존의 콜백을 Promise로 변환하기 위한 new Promise() 코드는 대부분 필요 없습니다.
bluebird 등의 라이브러리를 사용하면 promisify()를 이용해 쉽게 변환되기 때문입니다.

예를 들어, 아래와 같은 콜백 함수가 이미 있다면 `Promise.promisify(fs.getQuotaLimit)` 같이 Promise로 변환해서 사용할 수 있습니다.

```javascript
FileSystem.prototype.getQuotaLimit = function (callback) {
    var self = this;
    function restApi() {
        ajaxCall({
            url: conf.fsApiBaseUrl + '/limit/' + self.fsid,
            data: null,
            callback: callback
        });
    }
    ensureAuthorize(restApi);
};
```

하지만, 이 경우 bind를 고려해야 합니다. `self.fsid`(즉 `this` 객체의 `fsid`)를 사용하는 위 코드를 그냥 promisify 하면 `this`가 호출자가 되므로 정상적으로 동작하지 않습니다. 따라서 오른쪽 코드와 같이 적절한 객체로 bind 해서 promisify 해야 합니다.

<table class="compare">
<tr><td>
<pre><code class="language-javascript">var getQuotaLimit = Promise.promisify(fs.getQuotaLimit);
getQuotaLimit().then(function (limit) {
});
</code></pre>
</td><td>
<pre><code class="language-javascript">var getQuotaLimit = Promise.promisify(fs.getQuotaLimit.bind(fs));
getQuotaLimit().then(function (limit) {
});
</code></pre>
</td></tr>
</table>

### all

순서대로 실행할 필요가 없는 일은 [all](http://bluebirdjs.com/docs/api/promise.all.html)을 통해 병렬 수행하면 성능 향상이 있을 수 있습니다.

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="1-3"><code class="language-javascript">readFileAsync(path, 'blob')
    .then(_pushPackageFileStep)
    .then(_pushCertFileStep);
</code></pre>
</td><td>
<pre><code class="language-javascript">readFileAsync(path, 'blob')
    .then(_uploadStep);

function _uploadStep() {
    return Promise.all([_pushPackageFileStep(), _pushCertFileStep()]);
}
</code></pre>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>패키지 파일을 올리는 것과 Cert 파일을 올리는 것은 동시에 진행해도 되므로 all([pushPackage, pushCert])로 하면 약간의 성능 향상이 있을 수도 있겠습니다.</p>
</blockquote>
</td></tr>
</table>

### 문서화

Promise는 하나의 값을 갖고 성공하거나(fulfilled with a single fulfillment value) 혹은 하나의 예외를 갖고 실패하거나(rejected with a single rejection reason) 둘 중 하나이므로 문서(jsdoc)에 이를 기록하는 것이 좋습니다.

적어도 아래와 같이 성공 시에 어떤 값을 반환하는지 적어 주는 것이 좋은 습관이라 할 수 있습니다.

```javascript
    /**
     * Get supported profiles
     *
     * @return {Promise} - a promise that is resolved with the array of supported profile names
     */
    function getProfiles() {
        return getPlatforms().then(function (platforms) {
            return _.uniq(_.pluck(platforms, 'profile'));
        });
    }
```

## JavaScript 일반

### 이벤트 처리

동적으로 생성되는 DOM 엘리먼트에 대해 이벤트 핸들러를 추가해야 할 경우가 있습니다.

패널(panel) 내에 리스트로 하나씩 내용과 버튼이 추가되고 이 버튼에 이벤트를 붙이는 경우를 생각해 보죠.
보통 버튼마다 이벤트 핸들러를 추가하는 식으로 구현하는데, 이것보다는 event delegation을 이용하는 것이 좋습니다.

* 부모 엘리먼트가 event를 처리하므로 추가되는 버튼마다 이벤트 리스너를 추가할 필요가 없습니다.

* 이벤트 핸들러의 수가 감소하므로 메모리 효율적입니다. 버튼 삭제 시 리스너의 unbind를 신경 쓰지 않아도 됩니다.

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="4-6"><code class="language-javascript">_.forEach(pages, function (page) {
    var $button = $(ownerPageButtonTemplate());
    $page.append($button);
    $button.on('click', function () {
        topic.publish('editor/open', filePath);
    });
}
</code></pre>
</td><td>
<pre><code class="language-javascript">panel.elements.$panel.on('click', 'button', function () {
    topic.publish('editor/open', $(this).attr('data-path'));
});

_.forEach(pages, function (page) {
    var $button = $(ownerPageButtonTemplate());
    $page.append($button);
    $button.attr('data-path', filePath);
}
</code></pre>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>
동적으로 생성/삭제되는 element의 event handler는 delegated event로 처리하는 게 낫겠습니다.<br>
http://api.jquery.com/on/<br>
ex)
<pre>
$button.attr('data-path', filePath); // 버튼마다
</pre>
<pre>
$(pagesPanel).on('click', 'button', function () { // 한 번만
  topic.publish('editor/open', $(this).attr('data-path');
});
</pre>
  </p>
</blockquote>
</td></tr>
</table>

### 배열의 join

배열의 값들을 문자열로 만들 필요가 있을 때 배열의 기본(built-in) 함수인 [join](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/join)을 활용하는 것이 좋습니다.

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="1-4"><code class="language-javascript">_.forEach(params.uids, function (uid) {
    configContents += '"URI:uid=' + uid + '",';
});
configContents = configContents.substring(0, configContents.length - 1) + '\n';
</code></pre>
</td><td>
<pre><code class="language-javascript">var uids = _.map(params.uids, function (uid) {
    return '"URI:uid=' + uid + '"';
}).join(',');
configContents += uids + '\n';
</code></pre>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>_.map(duids, function (d) { return "URI.." + duid; }).join(',') 하면 더 깔끔할 것 같습니다.</p>
</blockquote>
</td></tr>
</table>

### JSON.parse 에러 처리

서버로부터 받은 JSON 문자열의 파싱을 위해 `JSON.parse`를 자주 사용하는데 이때 try/catch 사용을 권장합니다.

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="3"><code class="language-javascript">$.ajax({
    success: function (data) {
        data = JSON.parse(data);
        if (data.result === 'ok') {
        }
    }
});
</code></pre>
</td><td>
<pre><code class="language-javascript">$.ajax({
    success: function (data) {
        try {
            data = JSON.parse(data);
            if (data.result === 'ok') {
            }
        } catch (e) {
            console.error('Failed to parse a response', e);
        }
    }
});
</code></pre>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>JSON.parse error 처리가 필요해 보입니다.</p>
</blockquote>
</td></tr>
</table>

## UX

보통은 리뷰를 통해 로직을 체크하게 되지만 UI나 사용성도 대상이 됩니다.
저는 해당 커밋을 cherry-pick 해서 실제로 돌려 보고 화면 구성이나 메시지/레이블에 대해서도 리뷰를 하는 편인데요, 아예 [미리보기 서버](http://tech.kakao.com/2016/02/04/code-review/)를 도입해서 개발 중이거나 개발이 막 끝난 기능을 실제로 사용하면서 기획자나 product owner에게 사용성에 대한 피드백을 받는 것이 유용한 것 같습니다.

일관된 사용성을 위해 몇 가지 규칙을 정하고 이에 따라 리뷰를 하면 됩니다.

* 어떤 대상을 선택해서 액션을 취하는 경우 더블클릭으로 해당 액션이 실행되어야 한다

* 라디오 버튼 같은 form element는 레이블(label)을 눌러도 선택되게 한다

* 다이얼로그는 Esc 키로 닫을 수 있어야 한다

* 다이얼로그의 버튼 배치는 OK, Cancel 순서로 한다.

  참고로 웹 개발과는 조금 다르지만 데스크톱 어플리케이션의 경우 OS 플랫폼마다 버튼 배치에 대한 UX 철학이 다르다고 합니다. Windows는 OK가 먼저 나오고([Right-align the buttons and use this order (from left to right): OK, Cancel, and Apply.](https://msdn.microsoft.com/en-us/library/dn742500.aspx)) Mac과 Ubuntu에서는 실제 액션을 유발하는 버튼을 나중에 둔다고([A button that initiates an action is furthest to the right.](https://developer.apple.com/library/mac/documentation/UserExperience/Conceptual/OSXHIGuidelines/WindowDialogs.html)) 하네요.

  ![](http://uxmovement.com/wp-content/uploads/2011/05/efficient-task-flow.png)

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="1-3"><code class="language-javascript">grid.on('.dgrid-row:click', function (event) {
    dialog.targetObject = grid.row(event).data;
}),
</code></pre>
</td><td>
<pre><code class="language-javascript">grid.on('.dgrid-row:dblclick', function () {
    dialog.onClickOkButton();
}),
</code></pre>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>더블클릭하면 바로 실행까지 되어야 합니다. 사용성 관점에서 매우 중요합니다.</p>
</blockquote>
</td></tr>
</table>

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="5"><code class="language-markup">&lt;div class='rcw-content-table-label rcw-table-col'>
    &lt;input type='radio'/>
&lt;/div>
&lt;div class='rcw-content-table-value rcw-table-col'>
    &lt;span>Target dialog&lt;/span>
&lt;/div>
</code></pre>
</td><td>
<pre><code class="language-markup">&lt;div class='rcw-content-table-label rcw-table-col'>
    &lt;input type='radio' id='radio-targetdialog'/>
&lt;/div>
&lt;div class='rcw-content-table-value rcw-table-col'>
    &lt;label for='radio-targetdialog'>Target dialog</label>
&lt;/div>
</code></pre>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>&lt;label for>로는 안 되는 건가요?</p>
</blockquote>
</td></tr>
</table>

## linter 적용

팀에서 jshint, eslint 등의 linter를 적용하기로 했다면 개발자들이 사용하는 에디터에서 바로 체크되어야 하고, [커밋된 코드에서도 자동으로 수행되어]({{ site.baseurl }}{% post_url 2016-08-22-code-review-1 %}#section-17) 그 결과를 리뷰어가 볼 수 있어야 합니다.

아래는 개발자가 리팩토링 과정 중에 실수로 require의 alias를 `WizardPage`에서 `WizardPAge`로 변경하면서 jshint 에러가 발생한 상황이고, 단순 컨벤션 에러를 넘어 기능 수행이 되지 않았을 심각한 버그입니다.

{% include image.html img="/assets/images/code-review-20.png" title="jshint 에러" caption="jshint 에러: 실행에 문제가 있는 상황" %}

<table class="compare">
<tr><td>
<pre class="line-numbers" data-start="468" data-line="1"><code class="language-javascript">page = new WizardPage(pageParams);

return page;
</code></pre>
</td><td>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>
Convention check 바랍니다. line 468, char 20: 'WizardPage' is not defined.<br>
이거 verify하고 올리신 건가요?
  </p>
</blockquote>
<blockquote class="quotes">
  <p>패치 셋 올리기 전에 제대로 실행해서 확인을 안 했네요 죄송합니다. 이번 패치 셋 올릴 때는 실행해보고 올렸습니다.</p>
</blockquote>
</td></tr>
</table>

## 읽기 쉬운 코드

### 작은 함수

함수는 작게, 한 가지 일만 잘하게 그리고 이름을 서술적으로 써야 합니다.

아래 두 코드를 비교해 보면 오른쪽 코드가 더 쉽게 읽히죠? 명시적인 함수 이름을 써서 주석도 제거된 것을 알 수 있습니다.

<table class="compare">
<tr><td>
<pre class="line-numbers" data-line="5-14"><code class="language-javascript">if (model) {
    manifest = model.getContents();

    if (manifest) {
        // Diff with formatted xml string and saved one
        if (model.serialize() === persistenceContents) {
            return false;
        } else {
            if (!isContentChanged()) {
                return false;
            }

            return true;
        }
    } else {
        return false;
    }
} 
</code></pre>
</td><td>
<pre><code class="language-javascript">if (model) {
    if (isFormatContentChanged()) {
        return isNonFormatContentChanged();
    } else {
        return false;
    }
}
</code></pre>
</td></tr>
<tr><td colspan="2">
<blockquote class="quotes reviewer">
  <p>뭔가 좀 더 의미 있는 이름을 붙여서 빼면 좋겠습니다.</p>
</blockquote>
<blockquote class="quotes">
  <p>Done</p>
</blockquote>
<blockquote class="quotes reviewer">
  <p>
아주 좋아졌네요.<br>
return isFCC() && isNFCC(); 하는 게 좀 더 보기 쉬운 것 같네요. 일단 이 change는 여기서 마무리하도록 하죠.
  </p>
</blockquote>
</td></tr>
</table>

### 커밋 메시지

커밋 메시지는 코드를 왜 변경했는지 알 수 있게 구체적으로 쓰여야 합니다.
하지만 무엇을 위한 커밋인지 전혀 드러내지 못하는 메시지들이 많았죠.

`server: statistics 버그 수정`, `server: fix a missing column bug.`, ...

* statistics 버그는 무엇을 의미하는지: SQL 문제로 통계 데이터가 아예 나오지 않거나 일부 누락되었나? 아예 잘못된 통계 데이터를 주고 있었나? 통계 페이지에서 사용되는 export 기능이 동작하지 않았나? 심각도 관점에서 이 버그는 당장 반영이 필요한 hot-fix 성격인가 아니면 특정 환경에서만 발생하는 예외적인 버그인가?
* column이란 무엇을 의미하는지: DB 칼럼? 테이블 칼럼?
* DB 칼럼이라면 해당 칼럼이 빠져서 어떤 문제가 있었는지 그리고 왜 빠졌는지
* DB 칼럼을 추가하기 위해 무엇을 수정했는지

리뷰어와 다시 그 코드를 보게 될 미래의 나를 위해서는 버그 원인, 수정 내용과 패치 영향 등의 사항을 구체적으로 적는 습관을 갖는 것이 좋습니다.

그리고 [영어로 쓰는 게 원칙]({{ site.baseurl }}{% post_url 2016-08-22-code-review-1 %}#section-18)이었는데 리뷰하다 보면 아무래도 영 어색한 표현들이나 개발자 개인의 잘못된 영어 습관이 보이게 됩니다.

팀 리뷰에서 수집된 잘못된 습관들을 분류해 보았습니다. 누구는 전치사 다음에 동사가 올 수 없다는 것도 몰랐다나 뭐라나^^;

(잘못된 수동태; 웬만하면 능동태로)

| Before | After |
| -------------- |
| Error results _are print_ in popup dialog. | Error results _are printed_ in popup dialog. |
| Datasource _not unregist_ when file deleted | _Unregister_ datasource when file is deleted |
| This commit _is resolve to_ problem. | This commit _resolves_ problem. |

(과도한 명사 연결)

| Before | After |
| -------------- |
| If the configuration is changed, _reset message output_. | If the configuration is changed, _reset message is printed in Output view_. |
| Tooltip added in _non schema validation target widget_ | Tooltip, _displaying non-schema validation result_, added to widget |
| fix _to editor content sync logic_ | fix _a problem (that) sync logic between editors does not work properly_ |

(잘못된 동사 연결/사용)

| Before | After |
| -------------- |
| _add enable_ import wizard plugins | _enable_ Import Wizard plugins |
| _fix get github-token_ in setting page | fix _to get GitHub token_ in setting page |
| _don't changing_ the device view _force_ | _don't change_ the device view _forcefully_ |
| support _for open_ the simulator | support _for opening_ the simulator |
| basic UI _define_ | basic UI _defined_ 혹은 _define_ basic UI |
| 1. _project file copy_ to temporary directory<br>2. _project select_ | 1. _copy project file_ to temporary directory<br>2. _select project_ |

(인칭, 단수/복수)

| Before | After |
| -------------- |
| Because import logic _create_ temporary files, | Because import logic _creates_ temporary files, |
| so _these temporary resource are_ removed | so _these temporary resources are_ removed |

## Wrap-Up

코드 리뷰에 의한 개선 사례를 유용하다고 느꼈던 것들 위주로 정리해 보았습니다.

정리를 다시 해 보니, 컨텍스트 전환이 자주 일어나고 다른 사람의 코드를 읽는 것이 생각보다 어려워 부담이 되기도 하는 코드 리뷰이지만 리뷰를 통해 전보다 지식이 쌓이고 다양한 관점에서 생각해 볼 수 있다는 것이 큰 장점이라는 생각이 다시 한 번 듭니다.

또 사람이 수행해야 하는 리뷰의 부담을 줄여 주는 도구가 더 유용해질 것이므로 현행 linter보다 더 좋은 JavaScript 정적 분석기를 개발하고자 하는 제 목표도 좀 더 확고해지는 것 같습니다. :)

## References

  * [Promises, Promises](http://www.slideshare.net/domenicdenicola/promises-promises)
  * [lodash documentation](https://lodash.com/docs)

[^1]: Lodash 최신 버전(4.x)에서는 map으로 대치되었네요.
[^2]: [How to execute common code after a Dojo Deferred object is resolved or rejected?](http://stackoverflow.com/questions/17346506/how-to-execute-common-code-after-a-dojo-deferred-object-is-resolved-or-rejected)
