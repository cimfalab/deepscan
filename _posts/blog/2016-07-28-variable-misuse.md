---
layout: post
title: JavaScript에서 변수 사용 시 주의할 점
excerpt: "JavaScript에서 변수 사용 시 중복 선언, 재정의, 선언 전 사용 등의 주의할 점을 알아봅니다."
date: 2016-07-28 11:30:00 +0900
share: true
categories: blog
tags:
- JavaScript
- JavaScript 정적 분석
- JavaScript static analysis
- 변수
- variable
- 호이스팅
- hoisting
---

세 번째 시간입니다. 오늘은 JavaScript 변수에 관해 알아보겠습니다.

JavaScript는 다른 언어와 다르게 변수 범위가 블럭이 아닌 함수 단위이고, 호이스팅 특성 때문에 변수 사용 시 특히 주의가 요구됩니다.

오늘은 검출된 사례 중 잘못된 변수 사용에 대해 공유하겠습니다.

* Table of Contents
{:toc}

## 변수 선언 전 사용

변수 선언 전에 변수를 사용하여 변수가 `undefined` 값을 갖는 경우입니다.
일반적인 언어에서는 변수를 선언하기 전에 사용하면 컴파일 에러가 발생하지만, JavaScript에서는 [호이스팅(hoisting)](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting)이라고 해서 변수나 함수 선언이 인터프리터에 의해 최상위로 끌어올려집니다.

그래서 다음과 같은 실행이 가능해지죠.

```javascript
foo();
function foo() {
   console.log('hello');
}
```

마찬가지로 변수를 선언 전에 먼저 사용하는 것도 가능합니다.

```javascript
for (var i = 0; i < rows.length; i++) {
    var obj = {
        patternId: patternId,
        sortOrder: idx++
    };

    rowOrderArr.push(obj);
}

var patternId = $("#patternNameLabel").attr("patternId");
$.post('/delete.do', { order: JSON.stringify(rowOrderArr) });
```

위 코드에서 `obj` 객체의 `patternId` 속성은 항상 `undefined` 값으로 설정됩니다.

하지만 에러는 발생하지 않기 때문에 실제로 돌려 보는 시점에서야 문제를 알 수 있게 됩니다. 가령 `rowOrderArr` 객체로 서버에 AJAX 호출을 했을 때에야 알 수 있죠. ("클라이언트에서 patternId 값을 안 채워줘서 서버 에러가 나요!")

## 변수 중복 선언

함수 내에 중복된 변수 선언이 존재하는 경우입니다.

개발자들은 일반적으로 변수가 블럭 범위(block scope)를 갖는다고 알기 때문에 아래와 같이 작성을 하고 `pjtCode` 변수가 각 블럭에 한정된다고 생각합니다.

```javascript
var idx = nextUrl.toUpperCase().indexOf('&PJTCODE=');
if (idx > -1) {
    var pjtCode = nextUrl.substr(idx + 9, 15);
    if (idx > 9) {
        var pjtCode = 'BAD_CODE';
    }
    console.log(pjtCode); // 1)
}
console.log(pjtCode); // 2)
```

하지만 JavaScript에서는 변수가 **함수 범위(function scope)**를 갖고 호이스팅에 의해 선언이 끌어올려지므로, 실제로 `pjtCode` 변수는 함수 내에서 하나로 유지됩니다.
즉 1)과 2)에서 `pjtCode` 값은 모두 `BAD_CODE`로 출력됩니다. (`idx`가 9보다 클 경우)

이와 같이 호이스팅은 JavaScript 코드 해석을 비직관적으로 만드는 측면이 있기 때문에 보통 코드 컨벤션에서 **선언문을 항상 최상위에 작성**하도록 권고합니다.

## 변수 재정의

변수를 재정의함으로써 이전에 정의한 변수가 사용되지 않는 경우(dead variable)입니다.

```javascript
if ($.browser.msie == true) {
    target = url + "userName" + userName; // 1)
    target = url.replace(/\.|\?|\&|\/|\=|\:|\-|\s/gi,""); // 2)
}
```

위 코드에서 1)에서 정의된 `target` 변수가 2)에서 재정의되면서 1)에서 할당한 `userName` 파라미터가 무시됩니다.
따라서 다음과 같이 수정되어야 합니다.

```javascript
if ($.browser.msie == true) {
    target = url + "userName" + userName;
    target = target.replace(/\.|\?|\&|\/|\=|\:|\-|\s/gi,"");
}
```

## Wrap-Up

이상 JavaScript 변수 사용에 대해 알아보았습니다.

다른 언어와 다른 JavaScript 특성을 이해하고 변수 사용에 주의하세요.

* 변수 선언은 함수 최상위에 한번만 한다.
* 해당 변수 정의가 누락되는 일 없이 사용되도록 한다.
