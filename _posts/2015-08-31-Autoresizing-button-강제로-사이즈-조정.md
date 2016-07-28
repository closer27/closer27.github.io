---
layout: post
title: "오토레이아웃을 사용할때 버튼 사이즈 강제로 변경하기"
date: 2015-05-19
tags: [iOS, Autolayout, Autoresizing, size, tip]
---

Autolayout이나 Autoresizing 때문에 button의 위치나 사이즈가 코드로 변경해줘도 실제로 적용이 안될 떄,

```objc
[_button setTranslatesAutoresizingMaskIntoConstraints:YES];
```

이걸로하면 해당하는 프로퍼티의 frame을 강제로 변경할 수 있다.
