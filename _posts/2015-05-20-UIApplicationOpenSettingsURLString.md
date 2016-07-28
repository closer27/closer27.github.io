---
layout: post
title: "UIApplicationOpenSettingsURLString를 이용해 앱에서 설정 앱으로 전환 시키기"
date: 2015-05-19
tags: [iOS, setting, tip]
---

iOS8에서부터 새로 추가된 로직
앱에서 카메라나 GPS 권한을 얻지 못했을 때 설정 앱으로 전환시켜 권한을 설정하도록 유도할 수 있다.

openURL을 통해 `UIApplicationOpenSettingsURLString`을 써서 연결

```objc
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:UIApplicationOpenSettingsURLString]];
```
