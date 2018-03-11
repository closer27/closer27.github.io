---
layout: post
title: "앵귤러 컴포넌트 생명 주기 - Lifecycle Hooks"
date: 2017-07-06
categories: frontend
tags:
- angular
- lifecycle
comment: yes
---

![Lifecycle Hooks](https://angular.io/generated/images/guide/lifecycle-hooks/hooks-in-sequence.png "Lifecycle Hooks")

- ngOnchanges
: 컴포넌트 구현에 따라 초기에 선택적으로 호출되는 생명 주기

- ngOnInit
: 실질적으로 컴포넌트가 초기화를 마치고 완전한 상태를 갖춘 시점
constructor는 ES6 클래스 문법 표준으로서 앵귤러가 초기화 작업을 수행하기 전이므로 컴포넌트의 속성 가운데 템플릿과 바인딩한 속성이나 부모 컴포넌트로 받은 속성 등의 초기화를 보장하지 않음
: => API 호출이나 앵귤러가 제공하는 기능은 ngOnInit 이후에 사용해라

- ngOnDestroy
: 컴포넌트가 뷰에서 제거되기 직전 호출

- ngAfterContentInit
: 컴포넌트의 뷰가 초기화되는 시점에 호출, Content Projection으로 전달받은 템플릿의 초기화 완료 시점에 호출 됨

- ngAfterViewInit
: 컴포넌트의 템플릿이 완전히 초기화된 시점에 호출; 이 시점에는 컴포넌트의 모든 속성이 정상적으로 바인딩되었고 뷰도 렌더링 되었음을 의미. 예를 들어 부모 컴포넌트로부터 프로퍼티 바인딩으로 받은 속성 a를 템플릿에서 뷰로 노출하도록 구현했을 때, ngAfterViewInit이 호출 될 때 a 속성은 부모 컴포넌트로부터 전달됐음을 보장하고 렌더링 됐다는 것도 보장.

- ngOnChanges
: 어떠한 이벤트에 의해서 컴포넌트의 상태가 변경된다면 앵귤러는 ngOnChanges나 ngDoCheck를 호출; 구현한다고 반드시 호출되는 것은 아님 => 프로퍼티 바인딩을 통해 부모 컴포넌트에게 상태를 전달받은 경우에만 호출되는 메서드.
SimpleChanges라는 인자와 함께 전달; SimpleChange 안에는 previousValue와 currentValue라는 속성과 isFirstChange라는 메서드로 이루어진 타입.
최초 호출 시: isFirstChange는 true, previousValue는 UNINITIALIZED, currentValue는 최초 바인딩 값.

- ngDoCheck
: ngDoCheck는 앵귤러가 컴포넌트의 상태 변경을 감지한 후에 항상 호출
: 기본적으로는 최초 컴포넌트 초기화 후 ngOnInit 이후 바로 호출
: 이후에는 외부 이벤트에 따라 상태 변경 감지가 내부에서 실행된 후에 항상 호출 됨.
앵귤러가 변경 사항을 직접 관리하기 어려운 경우 구현해야 함, 가능하면 DOCheck를 구현하지 않거나 구현하더라도 무거운 작업을 포함시키면 안됨. 구현된 컴포넌트와 관계없이 애플리케이션에서 일어나는 모든 비동기 이벤트마다 호출되기 때문에 성능에 무리를 줄 수 있음.

- ngAfterContentChecked, ngAfterViewChecked
: ngAfterContentInit, ngAfterViewChecked 이후 바로 호출 되는 메서드, 뷰의 상태가 변경된 이후 처리할게 있을 때 사용

> #### 지시자의 생명 주기
> 기본적으로 컴포넌트와 동일, 지시자는 컴포넌트와 달리 뷰가 없기 때문에 뷰의 상태와 관련된 메서드는 사용하지 않음.

—
Referencing

: 조우진 (2017), Angular 첫 걸음, 한빛미디어, 서울

: Angular.io. (2017). Angular Docs. [online] Available at: [https://angular.io/guide/lifecycle-hooks](https://angular.io/guide/lifecycle-hooks) [Accessed 13 Jul. 2017].
