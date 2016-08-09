---
layout: post
title: 조건 체크 및 코드 실행
excerpt: "if/else 문, switch 문에서의 조건 체크 및 코드 실행에 대해 알아봅니다."
date: 2016-08-09 19:00:00 +0900
share: true
categories: blog
tags:
- JavaScript
- JavaScript 정적 분석
- JavaScript static analysis
- 중복 함수
- 중복 속성
- 불필요한 인자
- 사용되지 않는 코드
---

* Table of Contents
{:toc}

여섯 번째 시간입니다.

지난 [포스팅]({{ site.baseurl }}{% post_url 2016-08-01-unused-codes-2 %})에서 살펴본 중복 조건과 동일 결과 조건에 이어 오늘은 if/else 문과 switch 문에서의 조건 체크 및 코드 실행에 대해 정리합니다.

## 조건문의 실행 코드 동일

if/else 문에서 각 분기(branch)의 실행 코드가 동일한 경우입니다.

```javascript
function caseStudy(device) {
    var setVal;

    if (device == 'mobile') {
        setVal = 4;
    } else {
        setVal = 4;
    }
```

위 코드는 접속 디바이스가 모바일인 경우(`if (device == 'mobile')`) 별도 처리를 하기 위한 것인데, 실제로는 이에 관계 없이 동일한 값으로 설정되고 있습니다. 아마도 개발자가 복사해서 붙여넣기 후 수정을 하지 않았을 것이라고 생각됩니다. :)

따라서 모바일 및 PC에서 접속했을 때 의도대로 페이지 레이아웃이나 동작이 되는지 체크할 필요가 있습니다.

## null 체크 없이 사용되는 변수

<pre class="line-numbers" data-start="517" data-line="3,8"><code class="language-javascript">  checkRepoURL: function (wizardStep1Controller) {
    var selectedStack = this.get('content.stacks').findProperty('isSelected', true);
    selectedStack.set('reload', true); // 1)
    var nameVersionCombo = selectedStack.get('id');
    var stackName = nameVersionCombo.split('-')[0];
    var stackVersion = nameVersionCombo.split('-')[1];
    var dfd = $.Deferred();
    if (selectedStack && selectedStack.get('operatingSystems')) { // 2)
</code></pre>
-- Source: Apache Ambari `/app/controllers/installer.js`
{: .right}

위 코드에서는 `selectedStack` 변수에 대한 일관적인 null 체크가 필요합니다.

* 1)에서 먼저 `selectedStack` 변수를 null 체크 없이 사용함
* 2)에서는 `selectedStack` 변수를 null 체크와 함께 사용함

1)에서 null 체크 후 사용하도록 수정하거나 해당 변수가 null일 수 없는 경우에는 일관성 있게 null 체크를 제거하는 것이 권장됩니다.

아래의 예는 더 좋지 않은 코드입니다.

<pre class="line-numbers" data-start="176" data-line="5-6"><code class="language-javascript">  loadRecommendationsSuccess: function(data) {
    if (!data) {
      console.warn('error while loading default config values');
    }
    this._saveRecommendedValues(data); // 1)
    var configObject = data.resources[0].recommendations.blueprint.configurations; // 2)
    if (configObject) this.updateInitialValue(configObject);
    this.set("recommendationsConfigs", Em.get(data.resources[0] , "recommendations.blueprint.configurations"));
  },
</code></pre>
-- Source: Apache Ambari `/app/mixins/common/serverValidator.js`
{: .right}

`data` 인자 체크를 통해 기본 설정 정보가 로딩되지 않은 상황을 아는데도 이후 로직을 진행하고 있습니다. 따라서 아래와 같이 트랜잭션이 보장되지 않는 불안정한 동작을 하게 되므로 체크 후 바로 return 하는 것이 바람직합니다.

* 1)에서 비정상적인 값으로 저장
* 2)에서 `data`를 참조하면서 TypeError 발생

## Switch 문에서의 break 미사용

switch/case 문에서 `break`를 사용하지 않는 경우입니다.

<pre class="line-numbers" data-start="338" data-line="12,15,17,19"><code class="language-javascript">  loadAllPriorSteps: function () {
    var step = this.get('currentStep');
    switch (step) {
      case '7':
      case '6':
      case '5':
        this.loadServiceConfigProperties();
        this.getServiceConfigGroups();
      case '4':
      case '3':
        this.loadClients();
        this.loadServices();
        this.loadMasterComponentHosts();
        this.loadSlaveComponentHosts();
        this.load('hosts');
      case '2':
        this.loadServices();
      case '1':
        this.load('hosts');
        this.load('installOptions');
        this.load('cluster');
    }
  },
</code></pre>
-- Source: Apache Ambari `/app/controllers/main/host/add_controller.js`
{: .right}

switch 문에서 이전 case의 실행 흐름을 다음으로 이어지게 하면 가독성이 떨어지고 실수로 `break`를 누락한 경우와 구분이 어려워 사용에 주의가 요구됩니다.

위 코드는 특정 단계에서 이전 단계의 코드들을 모두 실행하기 위한 의도로 보이는데, `loadServices()`와 `load('hosts')` 호출이 중복되므로 체크가 필요합니다.

## Wrap-Up

좋은 코드 작성을 위해 조건문 사용 시 다음을 체크하시면 좋겠습니다.

* 각 조건의 분기가 어느 상황에 수행되는지 또 각 분기는 해당 상황에 따른 로직을 실행하는지 점검
* switch/case 문에서 `break`가 사용되지 않았을 경우 맞는 로직인지 재점검
