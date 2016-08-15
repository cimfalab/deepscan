---
layout: post
title: "오픈 소스 검증 사례 - Intel HTML5 Web Apps (2)"
excerpt: "Intel의 오픈 소스 프로젝트인 HTML5 Web Apps에서 발견된 코드 오류를 통해 잘못된 JavaScript 코딩 사례를 알아봅니다."
date: 2016-08-10 21:00:00 +0900
modified: 2016-08-15 15:00:00 +0900
share: true
categories: articles
tags:
- JavaScript
- JavaScript 정적 분석
- JavaScript static analysis
- Intel HTML5 Web Apps
- 오픈 소스
- 코드 오류
related:
- 오픈 소스 검증 사례 - Intel HTML5 Web Apps (1)
---

> 오픈 소스 프로젝트의 코드 오류에 관한 시리즈입니다.
>
> 개발 중인 정적 분석기의 유용성 검증, 코드 오류 패턴 수집 등의 목적으로 JavaScript 기반의 오픈 소스 프로젝트를 분석기로 진단해 보게 되었고 공유할 만한 내용을 추려 시리즈로 연재합니다.
{:.preface}

오늘은 지난 시간에 이어 Intel의 HTML5 Web Apps에 대해 3가지 사례를 추가적으로 소개하고 마무리하겠습니다.

* 프로젝트명: HTML5 Web Apps
* 프로젝트 사이트: https://01.org/html5webapps
* 프로젝트 설명: 최신 웹 기술 전시용으로 HTML5, JavaScript 그리고 CSS3만으로 작성된 순수 웹앱 샘플을 제공합니다. 샘플은 게임, 유틸리티 등 다양한 종류로서 20개가 제공됩니다.

## Annex

[Annex](https://01.org/html5webapps/online/annex/)는 바둑돌 게임입니다.
DOM 조작과 JavaScript로 구현된 AI 엔진을 사용하고 있습니다.

![](https://01.org/sites/default/files/styles/webapp_screenshots/public/webapps/annex2.png?itok=y8Fl0cY_)

제가 개발 중인 JavaScript 정적 분석기로 해당 앱의 소스를 크롤링해서 분석해 보았습니다.
브라우저 호환성과 false alarm을 제외하면 다음과 같은 코드 오류가 있다고 나왔네요.

| For array, consider using for loop instead of for-in. | js/annex.js:49 |
| 'return ret' of Asynchronous Callback $.getJSON("_locales/en/messages.json", function (data) { ... }).error has no effect. | js/dialogs.js:609 |
| The value assigned to variable board is never used. | js/annex.js:454 |

### 배열에 대한 for-in loop 사용
**js/annex.js:** For array, consider using for loop instead of for-in.
{: .notice}

배열에 for-in loop을 사용하는 것에 대해 경고하고 있습니다.

<pre class="line-numbers" data-start="47" data-line="3"><code class="language-javascript">		init: function(play){
			this.playerNum = play || 1;
			for (var i in this.board){
				for (var j=0; j<this.bounder; j++){
					this.board[i][j] =  'board';
				}
			}
</code></pre>

배열에 대해 for-in loop 사용을 권장하지 않는 이유는 다음과 같습니다.

* for-in loop는 `for`나 배열의 `forEach` 함수 대비 성능이 느립니다.
* for-in loop에서 사용되는 변수(위 코드의 `i`)는 배열의 인덱스를 의미하는데 배열의 실제 아이템으로 혼동하기 쉽습니다.
* `hasOwnProperty` 체크를 항상 사용하는 것이 좋습니다.

아래 코드는 `hasOwnProperty` 체크가 필요한 상황을 보여줍니다.

<pre class="line-numbers"><code class="language-javascript">Array.prototype.contains = function(value) {
    for (var key in this) {
        if (this[key] === value) {
            return true;
        }
    }
    return false;
};

var arr = [1, 2];
for (var i in arr) {
    console.log(i);
}
</code></pre>

for-in loop는 객체가 갖고 있는 prototype 객체의 속성들(위 코드의 경우 `contains` 함수)까지 포함하기 때문에 출력 결과가 다음과 같습니다.

```
0
1
contains
```

인덱스 0, 1 외에 contains가 포함되는 것을 볼 수 있습니다. 배열의 실제 값인 1, 2가 아닌 인덱스가 나온다는 것도 혼동하기 쉬운 부분이죠.

따라서 해당 속성이 객체에 직접적으로 존재하는지 `hasOwnProperty`를 통해 체크할 필요가 있습니다.

<pre class="line-numbers" data-start="11" data-line="2"><code class="language-javascript">for (var i in arr) {
    if (arr.hasOwnProperty(i))
        console.log(arr[i]);
}
</code></pre>

참고로, ECMAScript 6에 새로 추가된 for-of는 객체 속성만 순회하면서 값을 반환하기 때문에 배열의 값인 1과 2가 바로 출력됩니다.

```javascript
for (var i of arr) {
    console.log(i);
}
```

ECMAScript 6에 대해서는 [ECMAScript 6 소개]({{ site.baseurl }}{% post_url 2016-07-28-ecmascript-6 %}) 포스팅을 참고하세요.

### AJAX 콜백 함수에서의 return
**js/annex.js:** 'return ret' of Asynchronous Callback $.getJSON("_locales/en/messages.json", function (data) { ... }).error has no effect.
{: .notice}

AJAX 콜백(callback) 함수에서 불필요한 반환 값을 사용하는 경우입니다.

<pre class="line-numbers" data-start="603" data-line="10,17"><code class="language-javascript">function getMessage(key, alter) {
	var ret = alter || '';
	if (window.chrome && window.chrome.i18n && window.chrome.i18n.getMessage) {
		ret = chrome.i18n.getMessage(key);
	} else {
		if (typeof this.messages == 'undefined') {
			$.getJSON("_locales/en/messages.json", function(data){
				this.messages = data;
			}).error(function(){
					return ret;
			});
		}
        if (this.messages && (this.messages.hasOwnProperty(key)) && (this.messages[key].hasOwnProperty('message'))) {
			ret = this.messages[key].message;
		}
	}
	return ret;
}
</code></pre>

AJAX 호출 시 인자로 넘기는 콜백 함수는 비동기이므로 그 반환 값이 사용되지 않습니다. 위의 코드에서 612 라인의 `return`과는 상관 없이 `getMessage` 함수는 619 라인에서 바로 반환됩니다.

이런 코드는 개발자가 콜백 함수의 반환 값이 의미 있는 것으로 잘못 가정하고 있다는 신호일 것입니다.

## Slider puzzle

[Slider puzzle](https://01.org/html5webapps/online/slider-puzzle/)은 그림을 맞추는 퍼즐 게임입니다.
DOM 및 CSS3를 활용하고 있습니다.

JavaScript 정적 분석기 수행 결과입니다.

| Firefox, Internet Explorer 8, Internet Explorer 9, Internet Explorer 10, Internet Explorer 11 browsers do not support Window.openDatabase() method. | js/dbmanager.js:53 |
| Result of expression self._currMoves is unused. | js/finishpopout.js:23 |
| The value assigned to variable isAndroid is never used. | js/iscroll-lite.js:14 |
| Multiple variable declaration of "canvas". | js/leaderboardpage.js:93 |

### Web SQL DB의 사용
**js/dbmanager.js:** Firefox, Internet Explorer 8, Internet Explorer 9, Internet Explorer 10, Internet Explorer 11 browsers do not support Window.openDatabase() method.
{: .notice}

Firefox와 Internet Explorer에서는 `openDatabase` 함수가 지원되지 않는다고 하네요.

<pre class="line-numbers" data-start="53" data-line="1"><code class="language-javascript">  self.db = openDatabase("SliderPuzzleDb", "1.1", "Slider Puzzle DB", 200000);
</code></pre>

`openDatabase`는 Web SQL DB에서 제공되는 함수로 브라우저에서 관계형 DB를 사용할 수 있도록 해 줍니다.
하지만 [스펙](https://www.w3.org/TR/webdatabase/)이 중단되어 Chrome을 제외한 대부분의 브라우저에서는 지원하지 않고 있습니다.

실제로 Firefox에서 실행해 보면 `ReferenceError: openDatabase is not defined` 에러가 발생하면서 게임을 시작할 수 없습니다.

따라서 어떤 값을 브라우저에 저장하고 싶을 때는 Web Storage나 IndexedDB를 사용해야 합니다.
IndexedDB로의 마이그레이션은 [Migrating your WebSQL DB to IndexedDB](http://www.html5rocks.com/en/tutorials/webdatabase/websql-indexeddb/)를 참고하세요.

## Tenframe

[Tenframe](https://01.org/html5webapps/online/tenframe/)은 산수 학습을 위한 게임입니다.
DOM 조작과 CSS 애니메이션으로 구현되어 있습니다.

![](https://01.org/sites/default/files/styles/webapp_screenshots/public/webapps/tenframe_ss2.png?itok=N4bbYJmQ)

JavaScript 정적 분석기 수행 결과입니다.

| Firefox, Internet Explorer 8, Internet Explorer 9, Internet Explorer 10, Internet Explorer 11 browsers do not support ['WebKitCSSMatrix'] global property. | js/pirates.js:353 |
| The value assigned to variable title is never used. | js/help.js:15 |
| Multiple variable declaration of "tgt". | js/rockets.js:147 |

### WebKitCSSMatrix의 사용
**js/pirates.js:** Firefox, Internet Explorer 8, Internet Explorer 9, Internet Explorer 10, Internet Explorer 11 browsers do not support ['WebKitCSSMatrix'] global property.
{: .notice}

Firefox와 Internet Explorer에서는 `WebKitCSSMatrix` 속성이 지원되지 않는다고 하네요.

<pre class="line-numbers" data-start="343" data-line="11"><code class="language-javascript">        $("#pirates_page .draggable_pirate").mousedown(function(e){
            if(!data.input)
                return;

            var id = "#"+e.currentTarget.id;
            if($(id).hasClass("flying"))
                return;

            sounds.pirate_arr.play();
            accel = new acceleration;
            curTransform = new WebKitCSSMatrix(window.getComputedStyle(document.body).webkitTransform);
</code></pre>

그 결과, Firefox에서는 `ReferenceError: WebKitCSSMatrix is not defined` 에러가 발생하면서 마우스에 의한 캐릭터 이동이 불가능합니다.

참고로, Firefox 최신 버전(46+)에서 `about:config`를 통해 `layout.css.prefixes.webkit` 설정을 true로 변경하면 동작합니다.
하지만 크로스 브라우저 지원을 고려한다면 해당 속성 사용 시 주의가 필요하고 [CSSMatrix](https://github.com/arian/CSSMatrix) 같은 polyfill의 사용을 검토해야 합니다.

## Wrap-Up

2편에 걸쳐 Intel의 오픈 소스 프로젝트 HTML5 Web Apps에 대한 코드 오류를 살펴보았습니다.

웹 기술이 빠르게 발전하고 변화하면서 웹앱으로 많은 것을 할 수 있게 된 만큼 웹 개발자는 실행 오류, 브라우저 호환성 같은 기능 요건과 확장성, 유지보수성, 코드 품질 같은 비기능 요건 모두를 만족시킬 필요가 늘어나고 있습니다.

이에 따라 개발에 도움을 주는 적절한 웹 개발 도구의 필요성도 따라서 커질 것이라 생각하고 있습니다. 개발 중인 정적 분석기를 통해 전세계의 개발자들이 보다 좋은 JavaScript 코드를 작성하는 데 기여할 수 있으면 좋겠습니다.
