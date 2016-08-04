---
layout: post
title: 사용되지 않는 코드 (1)
excerpt: "JavaScript에서의 중복 속성 정의, 불필요한 인자 전달 등 사용되지 않는 코드에 대해 알아봅니다."
date: 2016-07-29 13:30:00 +0900
share: true
categories: blog
tags:
- JavaScript
- 중복 함수
- 중복 속성
- 불필요한 인자
- 사용되지 않는 코드
- unused codes
---

* Table of Contents
{:toc}

네 번째 시간입니다.

오늘은 지난 [포스팅]({{ site.baseurl }}{% post_url 2016-07-28-variable-misuse %})에서 알아본 변수의 잘못된 사용 사례 중 변수 재정의와 연관해서 사용되지 않는 코드에 대한 얘기를 해 보려고 합니다.

변수 재정의란 변수에 할당한 값을 사용하지 않은 상태에서 새로운 값을 할당하는 경우였는데요, 이와 같이 사용되지 않는 코드는 처음부터 필요가 없거나 혹은 개발자의 의도와 다른 결과를 유도할 수 있어서 체크가 필요합니다.

```javascript
var repeatMethod = this.options.repeatMethod;
if (repeatCondition != null) {
    repeatMethod = repeatCondition.substr(0, 5);
} else {
    repeatMethod = "DAY";
}
```

위 코드에서 `repeatMethod`에 할당된 값 `this.options.repeatMethod`이 사용되지 않습니다.
애초부터 필요 없는 할당이었거나 아니면 원래 개발자의 의도가 옵션에 설정된 `repeatMethod`를 기본 값으로 사용하려는 것이었을 수도 있습니다. 후자의 경우라면 버그성 상황이 되는 것이죠.

## 함수 중복 정의

중복 정의된 함수가 존재하는 경우입니다.

```javascript
function loadingClose(obj, finalize) { // 1)
}

function loadingClose(obj) { // 2)
}
```

위 코드에서 `loadingClose` 함수가 중복 정의되어 있는데 JavaScript에는 함수 오버로딩이 없으므로 **마지막에 정의된 함수만 유효**하게 됩니다.

이전에 정의된 함수 1)은 사용되지 않으면서 코드만 크게 만들고 유지보수를 어렵게 합니다. 개발자가 함수 1)을 열심히 수정했는데, 실제로는 함수 2)를 수정했어야 한다는 걸 나중에 알게 될 수도 있다는 거죠. :)

## 객체 속성 중복 정의

중복 정의된 객체 속성이 존재하는 경우입니다.

```javascript
var obj = {
    page: $('#pageIndex').val(),
    rowNum: $('#pageSize').val(),
    sortname: '',
    sortorder: 'asc',
    rowNum: 15
}
```

위 코드에서는 `rowNum` 속성이 중복 정의되고 있습니다. 역시 **마지막 속성만 유효**하므로 `rowNum`은 `#pageSize` 엘리먼트에 설정된 값과 관계 없이 항상 15가 됩니다. 가령 사용자가 어떤 목록을 30개씩 끊어서 보겠다고 설정 후 조회해도 15개씩 보여진다는 것이죠.

## 불필요한 인자 전달

함수에서 사용되지 않는 인자를 호출자(caller)가 전달하는 경우입니다.

```javascript
ConditionDialog = function (divId, options) {
    $("#" + divId).html(buildHtml(divId, options));
}

ConditionDialog.prototype.buildHtml = function () {
    var html = "";
    html += '<div class="view">' + this.options.messages.repeatPeriod + '</div>';
    return html;
}
```

`buildHtml` 함수는 인자 없이 `this` 객체의 속성을 사용하므로 호출 인자가 필요 없습니다. 즉, 호출자가 전달한 `divId` 및 `options`는 사용되지 않고 무시됩니다.

이런 상황은 다음과 같은 리팩토링을 고려할 필요가 있음을 나타내기도 합니다. (Code smell이라고 하죠?)

* `buildHtml` 함수 호출 전 혹은 함수 내부에서 `this.options` 속성이 잘 설정되어 있는지 체크
* 함수의 독립적인 동작이 가능하도록 두 개의 인자를 갖도록 수정

## Wrap-Up

이상 사용되지 않는 코드 패턴에 대해 알아보았습니다.

* 함수 중복 정의
* 객체 속성 중복 정의
* 불필요한 인자 전달

다음 포스팅에서는 중복 조건, 사용되지 않는 표현식 등 추가적인 패턴에 대해 알아보도록 하겠습니다.
