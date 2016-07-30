---
layout: post
title: "NSString의 format argument 순서를 지정하는 방법"
date: 2015-05-19
tags: [iOS, NSString, argument, tip]
comment: yes
---

NSString의 format argument 순서를 지정하는 방법

```objc
// "foo bar foo bar" 순서로 출력
NSLog(@"%@", [NSString stringWithFormat:@"%2$@ %1$@ %2$@ %1$@", @"bar", @"foo"]);

// NSLog도 지원
NSLog(@"%2$@ %1$@ %2$@ %1$@", @"bar", @"foo");
```
