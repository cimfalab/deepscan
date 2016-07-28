---
layout: post
title: "ECMAScript 6 소개"
excerpt: "새롭게 확정된 JavaScript 명세인 ECMAScript 6의 새로운 기능들을 소개합니다."
date: 2016-07-28 13:00:00 +0900
categories: articles
tags:
- JavaScript
- ECMAScript 6
- ES6
author: wcho
comments: true
share: true
---

> 해당 글은 동료 개발자 우영님이 기여해 주었습니다.
>
> 저는 아래 자료에서 소개된 기능들 중 클래스 정의, `super` 키워드로 부모 클래스의 함수 호출, `let` 키워드로 변수 정의가 우선 와 닿더군요. 특히 블럭 범위 변수를 정의하는 `let`은 [JavaScript에서 변수 사용 시 주의할 점]({% post_url 2016-07-28-variable-misuse %}) 포스팅에서 다뤘던 함수 범위 변수에 대한 대안이 되겠습니다.
{:.preface}

현재 사용하는 대부분의 JavaScript는 2009년에 처음 제정되어 2011년에 개정된 [ECMAScript 5.1](http://www.ecma-international.org/ecma-262/5.1/) 표준에 기반하고 있습니다.

이후 클래스 기반 상속, 데이터 바인딩(`Object.observe`), Promise 등 다양한 요구사항들이 도출되었고 그 결과 2015년 6월에 대대적으로 업데이트된 [ECMAScript 6 ](http://www.ecma-international.org/ecma-262/6.0/)가 발표되었고, 매년 표준을 업데이트하는 정책에 따라 올해 6월에 [ECMAScript 7 ](http://www.ecma-international.org/ecma-262/7.0/)까지 발표되었습니다.

ECMAScript 6가 제정된지도 일년이 지났고 Internet Explorer를 제외한 [대부분의 브라우저가 표준을 지원하는 상황](http://kangax.github.io/compat-table/es6/)에서 웹 개발자들이 이제는 ECMAScript 6에 대해 관심을 가질 필요가 있다고 생각합니다.

그래서 제가 팀 내부에 발표했던 ECMAScript 6 소개 자료를 공유합니다.
새로운 기능들에 대해 코드와 함께 정리되어 있어 ECMAScript 6 이해에 도움이 될 것이라고 생각합니다.

<iframe src="//www.slideshare.net/slideshow/embed_code/key/d6HOlv2E0FNHNX" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/WooyoungCho/ecmascript-6-64456124" title="ECMAScript 6의 새로운 것들!" target="_blank">ECMAScript 6의 새로운 것들!</a> </strong> from <strong><a href="//www.slideshare.net/WooyoungCho" target="_blank">WooYoung Cho</a></strong> </div>

## References

위 발표 자료에 언급된 주요 참고 자료입니다.

* [ES6 In Depth](http://hacks.mozilla.or.kr/category/es6-in-depth/) (Mozilla 블로그에 게재된 시리즈로 재미있고 각 기능에 대한 깊이 있는 설명을 담고 있어 발표에서 많이 참조)
* [ECMAScript 6 — New Features: Overview & Comparison](http://es6-features.org) (ECMAScript 5와의 코드 비교를 통해 기능별 설명)
* [Overview of ECMAScript 6 features](http://git.io/es6features) (ECMAScript 6 전체에 대한 개괄적인 소개)
* [Exploring ES6](http://exploringjs.com/) (ES6 전체에 대한 상세하고 친절한 설명을 담은 무료 e-book)

ECMAScript 6 기반 개발 환경에 대해서는 다음을 참고하세요.

* [ES6 기반 프론트엔드 개발 환경 알아보기](http://readme.skplanet.com/?p=12185)
* [ES2015 단위 테스트 환경 구축하기](http://huns.me/development/1913)
