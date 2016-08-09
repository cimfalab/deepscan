---
layout: post
title: 사용되지 않는 코드 (2)
excerpt: "JavaScript에서의 중복 조건, 사용되지 않는 표현식 등 사용되지 않는 코드의 사례에 대해 알아봅니다."
date: 2016-08-01 22:00:00 +0900
share: true
categories: blog
tags:
- JavaScript
- JavaScript 정적 분석
- JavaScript static analysis
- 중복 조건
- 동일 결과 조건
- 사용되지 않는 표현식
- 사용되지 않는 코드
related:
- 사용되지 않는 코드 (1)
---

* Table of Contents
{:toc}

어느새 다섯 번째 시간이네요^^

지난 [포스팅]({{ site.baseurl }}{% post_url 2016-07-29-unused-codes-1 %})에 이어 사용되지 않는 코드에 대한 얘기를 해보겠습니다.

## 중복 조건

if/else if 문에서 선행 조건에 의해 후행 조건이 항상 참 혹은 거짓으로 만족되는 경우입니다.

### null/undefined에 의한 중복 조건

아래 코드에서 후행 조건 `boardId != ""`는 `boardId` 값이 null일 때만 체크되며 `null != ""`는 true이므로 항상 참이 됩니다.

```javascript
if (boardId != null || boardId != "") {
     params += '&boardId=' + boardId;
}
```

개발자의 의도는 변수가 null이나 빈 값이 아닐 때만 파라미터를 추가하려는 것으로 보이며, `boardId != null && boardId != ""`로 코딩해야 합니다.

비슷한 사례를 하나 더 보죠.

<pre class="line-numbers" data-start="66" data-line="2"><code class="language-javascript">  validateInteger : function(str, min, max) {
    if (str==null || str==undefined || (str + "").trim().length < 1) {
      return Em.I18n.t('number.validate.empty');
    } else {
</code></pre>
-- Source: Apache Ambari 2.2.2 `/app/utils/number_utils.js`
{: .right}

위 코드에서는 `str==undefined`가 항상 거짓이 되는데, 그 이유는 `str`이 `undefined`일 경우 선행 조건 `str==null`에 의해 체크되기 때문입니다. 선행 조건에서 `undefined==null`이 참인 이유는 == 연산자가 타입 체크 없이 비교를 하기 때문이구요. 따라서 `str===null || str===undefined`가 올바른 코드입니다. 헷갈리죠? :)

### 비교 연산자에 의한 중복 조건

아래 코드에서는 선행 조건 `page <= 0`에 의해 null 및 빈 문자열("")이 체크되기 때문에 `page == ""`과 `page === null`이 항상 거짓이 됩니다.

```javascript
var page = $(this).data('page');
if (page <= 0 || page == "" || page === null || typeof page === "undefined") {
```

선행 조건에서 참이 되는 이유는 <= 같은 [비교 연산자로 null이나 빈 문자열을 숫자와 비교](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Comparison_Operators)할 때 null이나 빈 문자열이 0으로 변환되기 때문입니다.

이 경우 간단하게 `if (!page || page < 0)`로 체크하면 됩니다. :)

### 중첩 if에서의 중복 조건

아래 코드는 중첩된(nested) if 문에서 중복 조건에 의해 로직이 실행되지 않는 예입니다.

<pre class="line-numbers" data-start="863" data-line="3,7"><code class="language-javascript">  bulkOperationForHostComponentsDecommissionCallBack: function (operationData, data) {
    ...
    if (turn_off) { // 1)
      ...
    } else {
        ...
        if (turn_off) { // 2)
          parameters['included_hosts'] = hostsWithComponentInProperState.join(',') // 3)
        }
        else {
          parameters['excluded_hosts'] = hostsWithComponentInProperState.join(',');
        }
</code></pre>
-- Source: Apache Ambari 2.2.2 `/app/controllers/main/host.js`
{: .right}

2)의 `turn_off` 체크는 1)의 `else` 블럭 안에 포함되어 있으므로 항상 false입니다.
따라서 3)의 코드는 전혀 실행되지 않으므로 필요 없는 코드인지 실행 경로가 잘못된 것인지 체크할 필요가 있습니다.

## 동일 결과 조건

조건식의 결과가 항상 동일한 경우입니다.

### 미정의 변수 사용

```javascript
.closest('.header_inner').off().on('mouseleave', function() {
    if (schWrap.is('.active')) {
        var closeSch;

        if (closeSch) { // 1)
            clearTimeout(closeSch);
        }
        closeSch = setTimeout(function() {
            ...
```

위 코드에서 1)의 조건식 `if (closeSch)`는 항상 false가 됩니다. `closeSch`가 지역 변수이면서 나중에야 값이 할당되기 때문이죠.

`closeSch` 변수를 전역 변수로 선언해야 `setTimeout`에서 반환된 핸들을 통해 정상적으로 `clearTimeout` 호출이 가능합니다.

### 값이 변하지 않는 변수 사용

<pre class="line-numbers" data-start="384" data-line="4,8"><code class="language-javascript">  validate: function () {
    ...
    var isError = false;
    var isWarn = false;

    ... <- isWarn 변수를 설정하는 부분이 없음

    if (!isWarn || isError) { // Errors get priority
      this.set('warnMessage', '');
      this.set('warn', false);
    } else {
      this.set('warn', true);
    }
</code></pre>
-- Source: Apache Ambari 2.2.2 `/app/models/configs/objects/service_config_property.js`
{: .right}

`isWarn` 변수가 초기값을 그대로 갖기 때문에 `if (!isWarn)`은 항상 true가 되고 `else` 분기는 실행되지 않습니다. 의도한 동작인지 체크할 필요가 있습니다.


## 도달 불가능한 코드

말 그대로 절대 실행되지 않는(unreachable) 코드입니다.

```javascript
try {
    //rows['startDate'] = rows['startDate'].toDate(datePattern);
} catch (e) {
    console.log('rows["startDate"] is not date format');
}
```

```javascript
function withusSave(f) {
    var d = new Date();
    ...

alert("죄송합니다. 시스템 점검중입니다.");
return;

    if ( $.trim(busiName) == "" ) {
        ...
```

위 두 경우 모두 절대 실행되지 않는 코드를 갖고 있습니다.

첫째 코드는 `try` 문 내의 코드가 주석 처리되어 실행 코드가 없으므로 `catch` 문 내의 코드가 실행될 수 없습니다.

두 번째 코드는 코드 중간에 명시적인 `return` 문이 삽입되어 있는 경우죠. 실제로 이런 코드들이 종종 발견됩니다. :)

개발자가 테스트를 위해 임시로 삽입했다가 테스트 완료 후 삭제하는 것을 잊었거나, `return` 이후의 코드를 향후 사용 목적으로 남겨 놓는 경우일 텐데 좋은 코드라고 볼 수는 없습니다.

## 사용되지 않는 표현식

표현식(expression)의 결과값을 사용하지 않는 경우입니다.

### 삼항 연산자

<pre class="line-numbers" data-start="642" data-line="7"><code class="language-javascript">  onErrorPerHost: function (actions, contentHost) {
    if (!actions) return;
    if (actions.someProperty('Tasks.status', 'FAILED') || actions.someProperty('Tasks.status', 'ABORTED') || actions.someProperty('Tasks.status', 'TIMEDOUT')) {
      contentHost.set('status', 'warning');
    }
    if ((this.get('content.cluster.status') === 'PENDING' && actions.someProperty('Tasks.status', 'FAILED')) || (this.isMasterFailed(actions))) {
      contentHost.get('status') !== 'heartbeat_lost' ? contentHost.set('status', 'failed') : '';
    }
  },
</code></pre>
-- Source: Apache Ambari 2.2.2 `/app/controllers/wizard/step9_controller.js`
{: .right}

위 코드 중 삼항 연산자의 `''`가 사용되지 않는 표현식입니다.
삼항 연산자에서 실행할 로직이 없을 경우 `''` 같은 임의의 값을 반환하는 경우가 많이 있는데, 아래처럼 작성하는 것이 보다 간결합니다.

```
contentHost.get('status') !== 'heartbeat_lost' && contentHost.set('status', 'failed');
```

### HTML 이벤트 핸들러

아래 코드는 HTML의 이벤트 핸들러에서 검출된 것입니다.

```html
<input type="text" onkeydown="if(event.keyCode==13) handled=true" onblur="handled-false"></span>
```

위 코드에서 `handled-false`가 사용되지 않는 표현식입니다.
정적 분석기 차원에서는 `handled` 변수 값에서 `false` 값을 뺀 결과를 사용하지 않아서 검출된 것이고, 실제로 `handled=false`에 대한 개발자의 명백한 오타이죠. 그 결과 `blur` 이벤트에서의 `handled` 변수 리셋이 동작하지 않습니다.


**참고:** JavaScript의 built-in 함수를 사용할 때도 결과값 사용에 주의해야 합니다.
가령 개발자가 `str.substring(0, 3);` 같이 사용하는 경우인데, `substring` 함수는 해당 문자열 객체를 직접 조작(side-effecting)하는 것이 아니라 새로운 객체를 반환하기 때문에 반환된 객체를 사용하지 않으면 함수 호출의 의미가 없습니다.
{: .notice}

## Wrap-Up

이상 사용되지 않는 코드 패턴에 대해 알아보았습니다.
깨끗한 JavaScript 코드 작성을 위해 참고하세요.

| 코드 패턴 |
| :-------- |
| [함수 중복 정의]({{ site.baseurl }}{% post_url 2016-07-29-unused-codes-1 %}#section) |
| [객체 속성 중복 정의]({{ site.baseurl }}{% post_url 2016-07-29-unused-codes-1 %}#section-1) |
| [불필요한 인자 전달]({{ site.baseurl }}{% post_url 2016-07-29-unused-codes-1 %}#section-2) |
| [중복 조건](#section) |
| [동일 결과 조건](#section-2) |
| [도달 불가능한 코드](#section-5) |
| [사용되지 않는 표현식](#section-6) |
