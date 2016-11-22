---
layout: post
title: "JavaScript 정적 분석기로 살펴본 좋은 JavaScript 코딩"
excerpt: "JavaScript 언어에 대해 좋은 코드 작성과 높은 코드 품질을 위해 알아두면 좋을 것들을 공유합니다. JavaScript 정적 분석기(static analyzer)를 통해 다양한 사이트와 오픈 소스를 분석하면서 수집된 사례들을 패턴으로 정리하여 알아봅니다."
date: 2016-11-22 10:00:00 +0900
categories: articles
tags:
- JavaScript
- coding practices
- 정적 분석
- 오픈 소스
comments: true
share: true
---

최근 발표된 GitHub의 [2016년 통계](https://octoverse.github.com/)에서도 알 수 있듯이 JavaScript는 가장 인기 있는 언어로서 웹 UI를 넘어 서버, 모바일, 데스크톱, IoT 등 다양한 영역으로 확대되고 있습니다.

하지만 JavaScript는 그 언어적인 특성으로 인해 개발을 진행할수록 유지보수가 어렵고 코드 품질이 저하될 가능성이 크기 때문에 개발 생산성과 코드 품질 향상에 도움을 줄 도구가 필요하다고 생각합니다.

이에 JavaScript 정적 분석 기술을 통해 코드를 실행하지 않고도 잠재적인 오류 가능성이 있는 부분을 찾는 분석기를 개발하면서 ECMAScript의 예외와 개발자들이 자주 실수하는 패턴들(pitfalls)로부터 분석기의 규칙(분석기가 찾아낼 오류 패턴)을 발굴하고 웹 사이트와 GitHub의 오픈 소스를 대상으로 분석해 보고 있습니다.

얼마 전 해당 오류 패턴과 정적 분석기 도입 고려 사항에 대해 발표했던 자료가 있는데, 좋은 JavaScript 코드 작성을 위해 도움이 될 만한 내용이라고 생각되어 공유합니다.

[Swipe](https://swipe.to/)로 작성된 웹 슬라이드이므로 PC나 모바일에서 아래 링크를 클릭하세요:
<div markdown="0"><a href="https://swipe.to/1599c" class="btn">Go to slide</a></div>
