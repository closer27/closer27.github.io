---
layout: post
title: "&lt;component&gt; is not a known element 에러가 날 때"
date: 2018-02-21
categories: frontend
tags: [angular, tip]
comment: yes
---

# &lt;component&gt; is not a known element 에러가 날 때

예를 들어 AppComponent에 TodoListComponent라는 컴포넌트를 child component로써 삽입할 때
부모 모듈인 AppModule에 TodoListModule를 imports에 추가했는데도

```
zone.js:388 Unhandled Promise rejection: Template parse errors:
'todo-list' is not a known element:
1. If 'todo-list' is an Angular component, then verify that it is part of this module.
2. If 'todo-list' is a Web Component then add "CUSTOM_ELEMENTS_SCHEMA" to the '@NgModule.schemas' of this component to suppress this message. ("lass="todo-item" *ngFor="let todo of todos" [ngClass]="{'completed': todo.completed }">-->
        [ERROR ->]<todo-list ></todo-list> <!--[todos]="todos"-->
    </section>
</main>
"): AppComponent@14:8 ; Zone: <root> ; Task: Promise.then ; Value: Error: Template parse errors:(…) Error: Template parse errors:
'todo-list' is not a known element:
1. If 'todo-list' is an Angular component, then verify that it is part of this module.
2. If 'todo-list' is a Web Component then add "CUSTOM_ELEMENTS_SCHEMA" to the '@NgModule.schemas' of this component to suppress this message. ("lass="todo-item" *ngFor="let todo of todos" [ngClass]="{'completed': todo.completed }">-->
        [ERROR ->]<todo-list ></todo-list> <!--[todos]="todos"-->
    </section>
</main>
```

위와 같은 에러가 나타난다면
TodoListModule 모듈에 다음이 구문이 빠졌는지 확인해보자.

```typescript
exports: [TodoListComponent]
```

TodoListModule을 임포트하는 다른 모듈이 exports에 포함된 TodoListComponent에 접근할 수 있도록 해주는 설정이다.
