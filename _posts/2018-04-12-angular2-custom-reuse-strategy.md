---
layout: post
title: "angular2 - 특정 조건 때만 custom reuse strategy이 동작하도록 구현하기"
date: 2018-04-12
categories: frontend
tags: [angular, angular2, CustomReuseStrategy, RouteReuseStrategy]
comment: yes
---

앵귤러 프로젝트에서 어떤 조건(컴포넌트 별로, 특정 상황)에 따라 Custom Reuse Strategy를 적용 여부를 결정할 수 있도록 구현하면서 어려웠던 점을 적어보았다.

## 조건에 따라 이전 화면 상태가 유지되야하는 요구사항
개발 중인 프로젝트는 다음과 같은 구조로 되어있었다.
- ListComponent: 전체 글 목록 데이터테이블 확인, 카테고리 선택시 해당 카테고리 글만 확인, 키워드 검색, 등록 화면 이동
- RegisterComponent: 글 등록 화면, 등록 시 서버로 post를 쏘고 이후 ListComponent로 이동

angular는 SPA이므로 화면 이동 때마다 컴포넌트가 메모리에서 폐기 됐다 새로 생성되기 때문에  새로운 글을 등록하든 뒤로가기를 누르든, 목록화면으로 돌아올때 새로 화면을 그리게 됐다.
프로젝트 특성상 목록 화면과 등록 화면을 자주 왔다갔다해야 했고 가장 불편한점은 **등록 화면에서 목록화면으로 돌아 왔을 때 데이터테이블의 페이지네이션 정보, 선택 된 카테고리 정보 등 모두 초기화 되는 점**이 가장 불편했다.

직접 메멘토 패턴을 구현해야하나 싶다가 이전에 쓰려다가 Lazy Module Loading와 동시에 동작이 잘 안되는 문제 때문에 사용하지 않았던 Custom Reuse Strategy를 다시 써보기로 했다.

## Custom Reuse Strategy
기본적인 클래스 구현은 아래 글을 참고
[\[angular2 custom reuse strategy 사용\] 확인하기](http://closer27.github.io/frontend/2017/02/22/angular2-custom-reuse-strategy/)

### ReuseStrategy 오버라이드 할 때 각 메서드 설명
- shouldDetach : 스냅 샷을 라우터에서 분리해야하는지 여부를 묻는다. 즉, 라우터는 store 메서드를 호출하여 저장 한 후에 더 이상이 스냅 샷을 처리하지 않는다.
- store : 라우터가 shouldDetach 메서드를 사용하여 물어 본 후 true를 반환하면 store 메서드가 호출(즉시는 아니지만 나중에 시간이 다름). 라우터가 null 값을 보내면 저장소에서이 항목을 삭제할 수 있다. 메모리는 신경 쓸 필요가 없다(앵귤러가 알아서함).
- shouldAttach : 현재 경로의 스냅 샷이 이미 저장되었는지 여부를 물음. 저장소에 올바른 스냅 샷이 포함되어 있고 라우터가이 스냅 샷을 라우팅에 다시 첨부해야하는 경우 true를 반환.
검색 : 저장소에서 스냅 샷을 로드. shouldAttach 메서드가 true를 반환하는 경우에만 호출.
- shouldReuseRoute : 현재 라우팅의 스냅 샷을 향후 라우팅에 사용할 수 있는지 여부를 확인.

간단하게 말해서 앵귤러의 Router가 navigate와 같은 메서드를 동작해 화면 전환이 이루어질 때 현재 컴포넌트의 상태를 메모리에 저장할지 여부, 이동하는 화면의 컴포넌트에 저장된 상태가 있는지를 확인, 이를 로드해 표시해야하는지 여부 등을 커스터마이징을 하여 이를 개발자가 직접 판단하고 구현하는 것이다.

### ActivatedRouteSnapshot 인스턴스
각 메서드에서 넘어오는 ActivatedRouteSnapshot 인스턴스는 다음과 같은 멤벼변수들을 가지고 있다.
```
ActivatedRouteSnapshot {
	children:(...)
	component:ƒ ListComponent(route, location)
	data:{bye: "o"}
	firstChild:(...)
	fragment:undefined
	outlet:"primary"
	paramMap:(...)
	params:{id: "15", hello: "true"}
	parent:(...)
	pathFromRoot:(...)
	queryParamMap:ParamsAsMap
	queryParams:{}
	root:(...)
	routeConfig:{path: “register”, component: ƒ, data: {…}}
	url:(2) [UrlSegment, UrlSegment]
	_lastPathIndex:2
	_resolve:{}
	_resolvedData:{}
	_routerState:RouterStateSnapshot {_root: TreeNode, url: “/register“}
	_urlSegment:UrlSegmentGroup {segments: Array(3), children: {…}, parent: UrlSegmentGroup}
	__proto__:Object
```

이상하게도 프로젝트 콘솔로그를 찍도록 해놨는데 CustomReuseStrategy 클래스가 정상 동작해도 콘솔에 찍히지가 않는다. 그래서 https://plnkr.co/edit/kvFQ2u6lQDjIOsVP3dOU?p=preview 이곳을 참조해서 디버깅해가며 개발했다.

## 실제 구현하기

### 첫번째 시도 - 라우터모듈에서 특정 값을 지정해 이를 통해 저장
우선 가장 쉬운 방법은 라우터를 구성할 때 `data` 파라미터로 플래그값을 넘겨 shouldDetach 메서드에 넘어오는 ActivatedRouteSnapshot 인스턴스에서 읽어들여 처리하는 것이다.

```
// app.routing.module.ts
const appRoutes: Routes = [
  { path: ‘list’, component: ListComponent, data: { shouldDetach: true } },
  ...
];
```
```typescript
// custom-reuse-strategy.ts
shouldDetach(route: ActivatedRouteSnapshot): boolean {
  return route.data[‘shouldDetach’] == true;
}
...
```
shouldDetach가 true일 때 store 메서드를 호출하여 스냅샷을 저장하도록 한다. (나머지 메서드는 건드릴 필요가 없음)

이처럼 구현한 경우에는 listComponent는 항상 스냅샷을 저장하도록 처리가 되는데, 문제는 신규 등록을 했을 경우에도 데이터를 새로 불러오지 않고 항상 기존 데이터만 표시하는 문제가 발생한다. 요구사항은 신규 저장 없이 목록으로 돌아갔을 때만 스냅샷을 저장하는 것이다.

### 두번째 시도 - RegisterComponent의 라우터를 통해 파라미터 전달하기
어떻게 처리할까 고민하다 라우터모듈이 아닌 컴포넌트에서 파라미터를 넘기도록 수정했다.

```typescript
// register.component.ts
...
this.router.navigate([‘list’, { needsRefresh: true }]);
// needsRefresh라는 플래그 값을 파라미터로 함께 보낸다.
...
```

```typescript
shouldDetach(route: ActivatedRouteSnapshot): boolean {
  return route.routeConfig.path.includes('list') && !route.params['needsRefresh'];
}
```

위와 같은 로직은 다음의 조건을 만족하는 경우에 스냅샷을 분리한다는 것을 의미한다.
- route의 path가 ‘list’라는 문자열을 포함하고
- route의 params로 ‘needsRefresh’라는 키의 값이 존재하지 않는 경우

```typescript
// registerComponent
public postArticle() {
  this.router.navigate([‘list’, { needsRefresh: true }]);
  // needsRefresh true로 설정; shouldDetach는 true
}

public goToBack() {
  this.router.navigate([‘list’]);
  // needsRefresh 파라미터 없이 라우팅; shouldDetach는 false
}
```

아 그런데 뭔가 동작이 이상하다.
정확히 파라미터는 전달이 잘 됐는데, listComponent는 의도한대로 동작하지 않고 여전히 새로 초기화가 되고 있었다.

RouterReuseStrategy의 메서드들이 호출되는 사이클을 다시 살펴봤더니 `shouldDetach`는 컴포넌트가 폐기 되는 시점에 컴포넌트의 상태를 저장할 것이냐(스냅샷을 분리할 것이냐)를 확인하기 위해 호출 된다. true면 저장하면서 ngOnDestroy는 호출되지 않고 false면 스냅샷을 저장하지 않고 ngOnDestroy이 호출된다.

따라서 register 화면에서 navigate를 통해 list로 파라미터를 전달한다 한들, list 컴포넌트의 스냅샷 저장 여부를 판단하는 것은 애초에 list에서 register로 화면 전환이 이루어질때 이미 결정된다는 것이다.

1. list -> register 이동 시 : listComponent의 스냅샷 저장 여부 확인 & 저장
2. register -> list 이동 시 : registerComponent의 스냅샷 저장 여부 확인 & 저장 -> listComponent의 스냅샷을 붙여야하는지 확인 & 붙이기(재사용)

list를 초기화할지 재사용해야할지 여부는 스냅샷을 분리할 때 결정하는 것이 아니라 저장된 스냅샷을 붙일 때 결정해야한다는 것을 깨달았다.

### 세번째 시도 - route.parent를 참조하여 이전 컴포넌트의 상태를 읽어오기
그럼 어떻게 해야할까? 정답은 activatedRouteSnapshot 내부의 parent 인스턴스를 참조하는 것이다. parent에는 이전 화면의 activatedRouteSnapshot 인스턴스가 들어있다.

처음 의도한 로직은 다음과 같다.
- shouldDetach가 아니라 retrieve 메서드에서 분기처리를 하고 참이면 activatedRouteSnapshot의 parent 인스턴스를 this.handlers에서 삭제시켜 null을 반환하도록 한다.
- 그렇다면 listComponent에서는 저장된 스냅샷이 없기 때문에 새로 ngOnInit을 호출하며 뷰를 그릴 것

그런데 실행을 해보니 다음과 같은 에러가 뜨면서 제대로 된 동작을 안했다.
```
ERROR Error: Uncaught (in promise): TypeError: Cannot read property 'contexts' of null
```
충격적이게도 null이면 안된단다. 아마도 ActivatedRouteSnapshot 인스턴스는 화면 상태 값 뿐만 아니라 기존 뷰 자체를 그리기 위한 인스턴스들도 들어가있나보다 (이 부분은 확인이 필요)

음 근데 로그를 찍어보니 저거 null로 반환하기도 하는데?..
shouldAttach 메서드를 보면 routeConfig가 있으면 true 조건이 되는게 있는데 아마 여기서 true가 걸렸을 때 retrieve 메서드에서 값이 없으면 문제가 생기는게 아닐까?
또는 shouldReuseRoute 메서드에서라던지..

확인을 해보니 shouldAttach가 true일 때는 retrieve 메서드에서 null이 나오는 경우는 contexts 에러가 나타난다. 따라서 기존 로직인
```return !!route.routeConfig && !!this.handlers[route.routeConfig.path]```
즉 routeConfig와 저장된 snapshot 인스턴스가 둘다 존재해야하는 것은 기본적인 전제이다.

### 네번째 시도 - 결국 shouldAttach 메서드에서 결정 되는 것
retrieve 메서드에서 activatedRouteSnapshot 강제로 null을 반환할 것이 아니라 shouldAttach에서 저장된 스냅샷을 적용할 것인지를 결정하기 때문에 이 부분에서 분기처리를 해주면 되는 것이다.

### 완성된 코드
```typescript
import {RouteReuseStrategy, ActivatedRouteSnapshot, DetachedRouteHandle} from "@angular/router";

export class CustomReuseStrategy implements RouteReuseStrategy {

  handlers: {[key: string]: DetachedRouteHandle} = {};

  shouldDetach(route: ActivatedRouteSnapshot): boolean {
    return route.routeConfig.path.includes('list');
  }

  store(route: ActivatedRouteSnapshot, handle: DetachedRouteHandle): void {
    this.handlers[route.routeConfig.path] = handle;
  }

  shouldAttach(route: ActivatedRouteSnapshot): boolean {
    if (route.routeConfig.path.includes('list') && !route.params['noRefresh']) {
      // list 메뉴에서 noRefresh 값이 존재하는 경우에는 reuse strategy를 적용하지 않음.
      return false;
    }
    return !!route.routeConfig && !!this.handlers[route.routeConfig.path];
  }

  retrieve(route: ActivatedRouteSnapshot): DetachedRouteHandle {
    if (!route.routeConfig) return null;
    return this.handlers[route.routeConfig.path];
  }

  shouldReuseRoute(future: ActivatedRouteSnapshot, curr: ActivatedRouteSnapshot): boolean {
    return future.routeConfig === curr.routeConfig;
  }
}
```

--
Referencing
: [https://stackoverflow.com/questions/41483187/conditionally-apply-router-reuse-strategy-for-angular2-routes?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa](https://stackoverflow.com/questions/41483187/conditionally-apply-router-reuse-strategy-for-angular2-routes?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)
: [https://github.com/angular/angular/issues/16713](https://github.com/angular/angular/issues/16713)
: [https://stackoverflow.com/questions/36835123/how-do-i-pass-data-to-angular-routed-components](https://stackoverflow.com/questions/36835123/how-do-i-pass-data-to-angular-routed-components)
