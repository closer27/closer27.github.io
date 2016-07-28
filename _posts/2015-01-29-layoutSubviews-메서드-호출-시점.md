---
layout: post
title: "UIView의 layoutSubviews 메서드 호출 시점"
date: 2015-01-29
tags: [iOS, UIView]
---

UIView 의 크기가 변경되면, 크기가 변경된 UIView 의 서브뷰들은 위치와 크기가 조정되어야 한다. UIView는 이를 위해 자동과 수동으로 UIView의 layout을 조정하는 방법을 제공한다.

다음의 이벤트가 발생할 때 레이아웃 변경이 발생한다.

- UIView의 bounds 사이즈 변경
- root view의 변화를 유발하는 Interface orientation(세로모드, 가로모드 등) 변화
- UIView의 view layer 변화 유발 또는 layout을 요청하는 Core Animation sublayers의 설정
- UIView의 setNeedsLayout 또는 layoutIfNeeded 메소드가 호출될 경우
- UIView의 layer에서 setNeedsLayout이 호출되 경우. (UIView는 Cora Animation Layer 인 CALayer 와 결합하여 화면에 컨텐츠/에니메이션을 표시. UIView 에는 layer 속성이 존재)
