---
layout: post
title: "Typescript의 Variables"
date: 2017-01-31
categories: frontend
tags: [typescript]
comment: yes
---

## let
let은 기존의 var의 hoisting 문제를 해결하기 위해 나옴

``` typescript
var emotion = 'happy';
{
  var emotion = 'sad';
}
console.log(emotion)
```

실행 결과
```
sad
```

let은 위의 문제를 해결해준다.
``` typescript
var emotion2 = 'happy';
{
  let emotion2: string = 'sad';
  console.log(emotion2)
}
console.log(emotion2)
```

실행 결과
```
happy
sad
```

## Array

``` typescript
let fruits: string[] = ['banana', 'apple'];
let fruits2: Array<String> = ['banana', 'apple'];
```

두가지 타입 가능

## Union
유니언 타입은 둘중 하나의 타입만 유효하면 할당이 이뤄진다. 허용 타입을 구체적으로 명시해서 제한할 수 있다는 장점이 있음.

``` typescript
var unionX: string | number = 1;
console.log(typeof unionX, unionX)
```

실행 결과

```
number 1
```

## Destructuring
선택적으로 배열이나 객체에서 데이터를 추출하는 방법

``` typescript
var params2 = ['happy 동물원', 100];
let [m_name2, m_num2] = params2
console.log(`이름 : ${m_name2} 만족도: ${m_num2}%`);
```

실행 결과

```
이름 :  happy 동물원 만족도 : 100%
```
