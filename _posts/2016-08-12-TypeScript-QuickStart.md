---
layout: post
title: "TypeScript Quick Start"
date: 2016-08-12
tags: [typescript, quickstart]
comment: yes
---

타입스크립트의 공식 사이트에 있는

### Quickstart

quickstart.ts 파일을 작성한다

```typescript
function greeter(person) {
	return "Hello, " + person;
}

var user = "Jane User";

document.body.innerHTML = greeter(user);
```

이후 다음의 명령어로 변환하면 같은 이름의 .js 파일로 컴파일 된다.

$ tsc quickstart.ts

만약 다음과 같이 잘못된 자료구조를 써서 오류가 나는 상황이면

```typescript
function greeter(person: String) {
	return "Hello, " + person;
}

// var user = "Jane User";
var user = [0, 1, 2];         // 문자열이 아닌 배열을 대입하게 되는 상황.

document.body.innerHTML = greeter(user);
```

컴파일 시 다음과 같이 에러 메세지를 표시해준다

```
quickstart.ts(8,35): error TS2345: Argument of type 'number[]' is not assignable to parameter of type 'String'.
  Property 'charAt' is missing in type 'number[]'.
```

TypeScript can offer static analysis based on both the structure of your code, and the type annotations you provide.
타입스크립트는 코드의 구조와 타입 어노테이션을 기반으로 정적분석을 제공한다.

Notice that although there were errors, the greeter.js file is still created.
그러나 에러가 있어도 js 파일을 생성 된다. 다만 타입스크립트는 개발자의 의도대로 동작하지 않을 수 있다는 경고를 알려준다.

### Interface
two types are compatible if their internal structure is compatible. This allows us to implement an interface just by having the shape the interface requires, without an explicit implements clause.
내부 구조가 비교 가능하다면 두 타입은 비교할 수 있다. 이러한 방식은 `implements`  구문 없이도 인터페이스가 필요한 모양새를 갖는 것만으로 인터페이스를 구현할 수 있도록 해준다.

```typescript
interface Person {
	firstName: string;
	lastName: string;
}

function greeter(person: Person) {
	return "Hello, " + person.firstName + " " + person.lastName;
}

var user = { firstName: "Jane", lastName: "User" };

document.body.innerHTML = greeter(user);
```

user는 딕셔너리 형태지만 내부적으로 firstName과 lastName을 가지고 있으므로 Person 인스턴스로 간주하여 비교가 가능하다.

컴파일을 해보면 실제로 js 파일에는 Person 인터페이스와 관련 된 코드는 없다.

### Classes

TypeScript supports new features in JavaScript, like support for class-based object-oriented programming.
타입스크립트는 클래스 기반의 OOP를 지원하는 것처럼 자바스크립트에 새 기능을 제공한다.

Also of note, the use of public on arguments to the constructor is a shorthand that allows us to automatically create properties with that name.
생성자의 인자에 대한 public 사용은 그 이름에 맞는 프로퍼티들을 자동으로 생성해주는 간소화 표기이다.

```typescript
class Student {
	fullName: string;
	constructor(public firstName, public middleName, public lastName) {
		this.fullName = firstName + " " + middleName + " " + lastName;
	}
}

interface Person {
	firstName: string;
	lastName: string;
}

function greeter(person: Person) {
	return "Hello, " + person.firstName + " " + person.lastName;
}

var user =  new Student("Jane", "M.", "User");

document.body.innerHTML = greeter(user);
```

컴파일을 해보면 다음과 같은 js파일이 생성된다.

```javascript
var Student = (function () {
    function Student(firstName, middleName, lastName) {
        this.firstName = firstName;
        this.middleName = middleName;
        this.lastName = lastName;
        this.fullName = firstName + " " + middleName + " " + lastName;
    }
    return Student;
}());
function greeter(person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}
var user = new Student("Jane", "M.", "User");
document.body.innerHTML = greeter(user);
```

Classes in TypeScript are just a shorthand for the same prototype-based OO that is frequently used in JavaScript.
타입스크립트의 Class는 위와 같이 자바스크립트에서 자주 쓰이는 prototype-based OO에 대한 간편한 표기 방식이다.

### Ruuning your TypeScript web app

```html
<!DOCTYPE html>
<html>
    <head><title>TypeScript Greeter</title></head>
    <body>
        <script src="quickstart.js"></script>
    </body>
</html>
```

위와 같이 quickstart.html 파일을 작성하고 브라우저에서 실행해보면 스크립트에서 작성한 문자열이 표시 된다.

참조 : https://www.typescriptlang.org/docs/tutorial.html
