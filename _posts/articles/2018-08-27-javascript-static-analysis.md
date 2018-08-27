---
layout: post
title: "정적 분석으로 자바스크립트 코드의 오류와 코드 스멜 찾기"
excerpt: "JavaScript 언어에 정적 분석 도구를 적용해 예방할 수 있는 문제에 대해 알아보고 정적 분석 도구 도입 시 고려할 사항들을 제안합니다."
date: 2018-08-27 14:00:00 +0900
categories: articles
tags:
- JavaScript
- coding practices
- 정적 분석
- 오픈 소스
comments: true
share: true
---

JavaScript의 강세가 여전히 지속되고 있습니다.

2016년 이후 GitHub 통계에서 (pull request 기준) 가장 인기 있는 언어의 위치를 계속 차지하고 있고,
많이들 아시는 Martin Fowler의 <리팩토링(Refactoring)>이 올해 개정판이 나오는데 예제 언어가 Java에서 JavaScript로 바뀌는 것도 하나의 징후인 것 같습니다.

JavaScript에도 정적 분석 도구를 통한 코드 품질 관리가 필요하다는 나름의 생각으로 운영하고 있는 [DeepScan 서비스](https://deepscan.io/)도 좀 더 힘을 얻을 것 같고요.

몇 달 전에 DeepScan에서 축적된 데이터 기반으로 정적 분석 도구의 JavaScript 적용에 관한 백서를 썼었는데요,
이번에 한글 번역이 돼서 링크 공유합니다.

내용을 요약하면 아래와 같습니다.
* JavaScript 활용이 늘어나지만 개발 및 유지보수에 어려움이 있다. 기존 언어들은 정적 분석 도구를 통해 미리 코드 에러에 대응하여 품질 비용을 낮추어 왔다. JavaScript에도 정적 분석 기술을 적용하면 어떨까?
* 정적 분석 도구의 동작 원리
* 정적 분석 서비스를 운영하면서 수집된 오류 통계와 예제를 통해 JavaScript 개발자들이 많이 실수하는 패턴을 알 수 있다. 또 수정에 걸린 시간을 통해 개발자들이 중요하게 생각하는 에러의 종류도 알 수 있다.
* 정적 분석 도구 도입을 위한 체크리스트

본문은 아래 링크에서 보실 수 있습니다.
* 한글 버전: [네이버 포스트](https://m.post.naver.com/viewer/postView.nhn?volumeNo=16517463)
* 영문 버전: [Medium](https://medium.com/deepscan/detecting-javascript-errors-and-code-smells-with-static-analysis-504787b0acad)
