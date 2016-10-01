---
layout: post
title: JavaScript에서 함수 사용 시 주의할 점
excerpt: "JavaScript에서 함수 사용 시 인자 누락 등의 주의할 점을 알아봅니다."
date: 2016-10-01 16:00:00 +0900
share: true
categories: blog
tags:
- JavaScript
- JavaScript 정적 분석
- JavaScript static analysis
- 함수
- 인자 누락
related:
- JavaScript에서 변수 사용 시 주의할 점
- 사용되지 않는 코드 (1)
---

* Table of Contents
{:toc}

오늘은 JavaScript에서 함수 사용 시 주의할 부분에 대해 알아보겠습니다.

JavaScript는 동적 타입이면서 함수로의 인자 전달이 유연하기 때문에 사용에 주의가 요구됩니다.

## 중복 정의

Java에서는 함수를 중복 정의하면 에러가 발생합니다.

![Duplicate method in Java]({{ site.baseurl }}/assets/images/duplicate-method-in-java.png)

하지만 JavaScript에서는 함수를 중복으로 정의해도 에러가 발생하지 않고 대신 마지막 함수만 유효하게 사용됩니다.
이에 대한 사례는 [사용되지 않는 코드]({{ site.baseurl }}{% post_url 2016-07-29-unused-codes-1 %})를 참고하세요.

## 호출 인자 누락

호출자(caller)가 함수에 인자를 전달하지 않았는데 함수 내에서 해당 인자를 사용하는 경우입니다.

다음의 예를 보죠.

1. `checkDateFormat` 함수를 두 개의 인자 `date`와 `datePattern`으로 호출합니다.

   ```javascript
   function setCondition(rowId) {
       try {
           ...
           var date = getCellEditingValue("input", rowId, "startDate", $tr);

           var result = checkDateFormat(date, datePattern);
           if (!result.success) {
               throw new Error("date is not vaild.");
           }
   ```

2. 호출된 `checkDateFormat` 함수에서는 위에서 전달되지 않은 `dateName` 인자를 사용합니다.

   ```javascript
   function checkDateFormat(dt, dateFormat, dateName) {
       var isValid = ValidatorFactory.getDateFormatValidator(dateFormat);

       if (!isValid(dt)) {
           return {
               success: false,
               message: dateName + ' : 일자 형식이 잘못되었습니다.'
           };
       }
   ```

그 결과 undefined 값과 문자열이 결합하여 유효하지 않은 문자열이 사용자에게 노출되므로 해당 인자에 대한 처리(null 체크나 기본값 설정)가 필요합니다.

## 호출 인자의 속성 누락

호출자가 함수에 전달한 인자에 없는 속성을 함수 내에서 사용하는 경우입니다.

다음의 예를 보죠.

1. `fn_detailPopup` 함수를 호출하는 두 개의 실행 경로 1)과 2)가 있습니다. 1)의 경우 `argsVo.statusCodeId`가 설정되지 않은 채로 호출합니다.

   ```javascript
   if (colName.indexOf("testSeverity") != -1) {
       argsVo.flag = "SEVERITY";
       argsVo.severityCodeId = colName.replace("testSeverity", "");

       fn_detailPopup(argsVo); // 1)
   }

   if (colName.indexOf("statusCodeId") != -1) {
       argsVo.flag = "STATUS";
       argsVo.statusCodeId = colName.replace("statusCodeId", "");

       fn_detailPopup(argsVo); // 2)
   }
   ```

2. 호출된 `fn_detailPopup` 함수에서는 `argvVo` 인자의 설정되지 않은 `statusCodeId` 속성을 사용합니다.

   ```javascript
   function fn_detailPopup(argsVo) {
       ...
       url += '&severityCode=' + argsVo.severityCodeId;
       url += '&statusCodeId=' + argsVo.statusCodeId;
   ```

그 결과 `statusCodeId` 파라미터가 "undefined"라는 문자열로 서버에 전달되므로 서버에서 이에 대한 처리가 필요할 수 있습니다.
따라서 클라이언트 측에서 해당 속성 존재 여부를 서버 전달 전에 명확하게 체크하는 것이 좋습니다.

## API 호출 시 인자 타입 오류

JavaScript built-in 함수나 DOM API 함수는 정해진 타입의 인자가 있어서 잘못된 타입의 인자로 호출하는 경우 TypeError가 발생합니다.

가령 아래 코드 실행 시 `TypeError: Argument 1 of Node.appendChild is not an object.` 에러가 발생합니다.

```javascript
document.body.appendChild('foo');
```

제가 개발 중인 솔루션에서는 built-in 함수와 DOM API 함수 모델링을 통해 인자 타입 정보를 내장하여 **실행 전에 소스 레벨에서** 아래와 같이 검출할 수 있습니다.
`first argument type of Node.prototype.appendChild() API must be Node DOM instance object.`

## API 호출 시 인자 개수 오류

JavaScript built-in 함수나 DOM API 함수는 잘못된 개수의 인자로 호출하는 경우 TypeError가 발생할 수 있습니다.

가령 아래 코드 실행 시 `TypeError: Not enough arguments to Document.getElementById.` 에러가 발생합니다.

```javascript
var elem = document.getElementById();
```

개발 중인 솔루션에서는 built-in 함수와 DOM API 함수 모델링을 통해 아래와 같이 검출됩니다.
`number of arguments of Document.prototype.getElementById() API must be 1`

## Wrap-Up

이상 JavaScript 함수 사용에 대해 알아보았습니다.

다른 언어와 다른 JavaScript 특성을 이해하고 함수 사용에 주의하세요.

* JavaScript는 함수 오버로딩을 지원하지 않으므로 함수 정의를 중복하지 않도록 한다.
* 다양한 실행 경로에서 인자 및 인자 속성 정의가 잘 전달되는지 점검한다. 필요한 경우 함수 내에서 인자 및 속성에 대한 유효성 검사를 하는 것이 좋다.
