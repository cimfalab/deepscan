---
layout: post
title: Firefox에서 이벤트 객체 제대로 사용하기
excerpt: "[브라우저 호환성] Firefox에서 흔히 잘못 사용하는 이벤트 객체의 사례를 공유하고, 제대로 사용하는 방법을 알아봅니다."
date: 2016-07-27 10:25:00 +0900
modified: 2016-08-05 15:00:00 +0900
share: true
categories: blog
tags:
- JavaScript
- JavaScript 정적 분석
- JavaScript static analysis
- Firefox
- 이벤트
- Event
- 브라우저 호환성
- browser compatibility
---

두 번째 시간에서는 브라우저 호환성에 관해 알아보겠습니다.

브라우저 호환성이란 Internet Explorer, Chrome, Firefox 등 다양한 브라우저에서 웹 사이트가 정상적으로 동작하는지에 대한 체크입니다.

이를 위해 [Browsersync](https://blog.outsider.ne.kr/1216) 같은 도구나 [SauceLabs](https://saucelabs.com/) 같은 서비스를 사용할 수 있는데, 이들 방식은 각 브라우저를 실제로 구동하고 웹 사이트를 로딩한 이후 동작 여부를 사용자가 눈으로 확인해야 합니다.

하지만 제가 개발 중인 솔루션에서는 JavaScript 정적 분석과 브라우저 호환성 데이터베이스를 통해 소스 레벨에서 브라우저 호환 여부를 자동 체크할 수 있습니다.

오늘은 검출된 사례 중 Firefox에서 흔히 잘못 사용하는 이벤트 객체를 공유하고 제대로 사용하는 방법을 알아보겠습니다.

* Table of Contents
{:toc}

## 글로벌 event 객체 사용

이벤트 핸들러에서 핸들러에 전달된 이벤트 객체를 사용하지 않고 글로벌 객체(`window.event` 혹은 `event`)를 사용하는 경우입니다.
Internet Explorer와 Chrome은 글로벌 객체를 지원하지만 Firefox에서는 해당 코드에서 에러가 발생합니다.

`ReferenceError: event is not defined`

글로벌 객체 `event`를 사용하는 코드를 보도록 하겠습니다.

```javascript
$(document).on( "click touchstart", function() {
    if ($(".board_view").hasClass('active')) {
        $(".board_view").removeClass('active')
        $(".board_view").find(".is-click").removeClass('active')
        activeFlag = false;
    }
});

$(".is-click").click(function() {
    ...
    } else if ($(this).parent().parent().hasClass('board_view')) { //board
        if (activeFlag == true) {
            $(this).addClass('active');
            $(this).parent().parent().addClass('active');
            event.stopPropagation(); // 1) ReferenceError: event is not defined
        } else {
            activeFlag = true;
        }
    ...
});
```

개발자의 의도는 `is-click` 클래스를 가진 엘리먼트를 클릭했을 때, `active` 클래스를 설정하고 event propagation을 멈추는 것입니다.
Event propagation을 막아야 하는 이유는 `document` 객체에 설정된 이벤트 핸들러가 있고 여기서는 해당 `active` 클래스를 제거하기 때문이죠.

하지만 Firefox에서는 1) 부분에서 에러가 발생하고 event propagation이 중단되지 않습니다.
따라서 어떤 액션을 취했을 때 설정한 `active` 상태가 `document`의 이벤트 핸들러에 의해 바로 해제되는 현상이 발생합니다.

키 입력 처리에 대한 예를 하나 보겠습니다.

```javascript
function eventUtil_blockBackspace_onkeydown() {
    if (window.event.keyCode == 8) {
        ...
    }
}
```

이 코드는 사용자가 backspace 키를 눌렀을 때 어떤 동작을 하기 위한 것인데, 역시 `window.event`가 Firefox에서 지원되지 않으므로 원하는 대로 동작하지 않습니다.

따라서 글로벌 `event` 객체를 사용하는 경우는 **이벤트 핸들러의 인자로 전달된 이벤트 객체를 사용**하도록 수정되어야 합니다.

```javascript
$(".is-click").click(function(e) {
    ...
    } else if ($(this).parent().parent().hasClass('board_view')) { //board
        if (activeFlag == true) {
            $(this).addClass('active');
            $(this).parent().parent().addClass('active');
            e.stopPropagation();
        } else {
            activeFlag = true;
        }
    ...
});
```

## Event.srcElement 속성 사용

이벤트 핸들러에서 어떤 엘리먼트로부터 이벤트가 발생했는지 알기 위해 `Event.srcElement` 속성을 사용합니다.

```javascript
function eventUtil_blockBackspace_onkeydown() {
    if (window.event.keyCode == 8) {
        if (window.event.srcElement.readOnly ||
            window.event.srcElement.disabled) {
            ...
        }
    }
}
```

하지만 [Firefox는 해당 속성을 지원하지 않기 때문에](https://developer.mozilla.org/en-US/docs/Web/API/Event/srcElement) 에러가 발생하고 로직이 수행되지 않습니다.

따라서 다음과 같이 **`Event.target` 속성을 함께 사용**해야 모든 브라우저에 대한 대응이 가능합니다. Internet Explorer 8은 또 `srcElement`만 [지원](https://developer.mozilla.org/en-US/docs/Web/API/Event/target)하기 때문에요.

```javascript
function eventUtil_blockBackspace_onkeydown(e) {
    var target = e.target || e.srcElement;
    if (e.keyCode == 8) {
        if (target.readOnly ||
            target.disabled) {
            ...
        }
    }
}
```

## Event.returnValue 속성 사용

이벤트 핸들러에서 해당 이벤트를 취소하기 위해 `Event.returnValue` 속성을 사용합니다.

```javascript
function eventUtil_blockBackspace_onkeydown() {
    if (window.event.keyCode == 8) {
        if (window.event.srcElement.readOnly ||
            window.event.srcElement.disabled) {
            window.event.returnValue = false;
            return;
        }
    }
}
```

하지만 Firefox는 해당 속성을 지원하지 않기 때문에 에러가 발생하고 로직이 수행되지 않습니다.

따라서 다음과 같이 **`preventDefault` 함수를 사용**하도록 수정되어야 합니다.

```javascript
function eventUtil_blockBackspace_onkeydown(e) {
    var target = e.target || e.srcElement;
    if (e.keyCode == 8) {
        if (target.readOnly ||
            target.disabled) {
            e.preventDefault();
            return;
        }
    }
}
```

## Wrap-Up

이상 Firefox에서의 이벤트 객체 사용에 대해 알아보았습니다.

다양한 브라우저에서 호환되는 이벤트 핸들러의 작성을 위해서 다음 두 가지는 기억하세요.

* Firefox에는 `window.event` 객체가 존재하지 않으므로 이벤트 핸들러에 전달된 이벤트 객체를 사용해야 한다.
* `Event.srcElement` 대신 `Event.target`을 통해 이벤트가 발생한 엘리먼트를 얻어야 한다.
