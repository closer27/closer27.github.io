---
layout: post
title: "typescript에서 null 체크의 기본"
date: 2017-06-21
tags: [typescript, validation]
comment: yes
---

```
if ( value ) {
}
```

typescript에서 위와 같이 value를 조건문으로 사용한다는 것은 `value`가

- null
- undefined
- NaN
- empty string (“”)
- 0
- false

에 모두 해당하지 않는다는 뜻이다.

—
Referencing

: StackOverflow. (2015). "Is there a standard function to check for null, undefined, or blank variables in JavaScript?", [online] Available at:
[https://stackoverflow.com/questions/5515310/is-there-a-standard-function-to-check-for-null-undefined-or-blank-variables-in](https://stackoverflow.com/questions/5515310/is-there-a-standard-function-to-check-for-null-undefined-or-blank-variables-in)
[Accessed June 21, 2017].
