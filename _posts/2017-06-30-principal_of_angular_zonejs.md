---
layout: post
title: "Angular의 동작원리 2 - Zone.js"
date: 2017-06-30
categories: frontend
tags:
- angular
- zone.js
comment: yes
---

Angular의 동작원리 2 - Zone.js

Zone.js와 변화 감지
최초 부트스트랩 과정을 진행하여 초기 로드를 마친 후 애플리케이션은 정적인 상태로 대기
다음과 같은 세가지 이벤트가 변화를 발생시킨다.

- 사용자 액션
- Ajax와 같은 외부 호출
- setTimeout, interval 함수

비동기 호출 이벤트라는게 공통점
: 언제 호출 될지 모르는 비동기 코드를 예의 주시해야함 => zone.js 사용

zone.js는 앵귤러의 필수 의존 패키지; 특정 영역에서 일어나는 비동기 코드의 실행 맥락을 후킹할 수 있는 기능 제공

부트스트랩 과정에서 NgZone를 미리 생성해 의존성 등록을 했던 과정은 이를 위함.

클릭 이벤트가 발생하면 클릭 이벤트에 등록한 콜뱅 등 VM의 큐에 쌓여 있는 비동기 코드를 모두 실행
: 그리고나서 앵귤러는 내부적으로 후처리가 필요한 로직 실행; 후처리 중 가장 중요한 것은 값의 변경 여부를 감지하는 것.

앵귤러 컴포넌트마다 자신만의 change detector가 있다. 이에 따라 컴포넌트 트리에 대응하는 변화 감지트리 change detection tree도 있음.

앵귤러는 비동기 이벤트가 발생할때마다 모든 컴포넌트가 한번씩 변화감지 로직을 수행한다. 비효율적인 것 같지만 앵귤러는 VM 친화적코드를 통해 짧은 시간안에 단방향으로 확인.
=> 효율적인 변화감지 전략을 위한 옵션 ChangeDetectionStretegy를 사용할 수 있다

```
@Component({
    changeDetcion: ChangeDetectionStrategy.OnPush
})
export calls TestOne { … }
```

이렇게 선언하면 TestOne 컴포넌트의 자식 컴포넌트는 앞으로 다른 컴포넌트에서 발생한 변경 사항에서 변경을 감지하지 않음 => 자기 자신 컴포넌트의 이벤트나 상위 컴포넌트에서 바인딩한 객체의 레퍼런스가 바뀐 경우에만 감지

—
Referencing

: 조우진 (2017), Angular 첫 걸음, 한빛미디어, 서울
