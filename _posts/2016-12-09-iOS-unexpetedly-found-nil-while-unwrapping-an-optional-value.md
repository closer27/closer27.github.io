---
layout: post
title: "segue 사용 시 unexpectedly found nil while unwrapping an Optional value 에러 발생"
date: 2016-12-09
categories: ios
tags: [iOS, segue, tableview]
comment: yes
---


segue 통해서 뷰컨트롤러에서 다른 뷰컨트롤러로 데이터를 전달하려고하는데 엉뚱한데서 문제가 발생했다.

```
fatal error: unexpectedly found nil while unwrapping an Optional value
```

`tableView.reloadData()`에서 저런 에러로 앱이 죽어버리는데 원인을 도통 알수가 없다

디버깅해보았더니 tableView가 nil인 상태였고 그 상태에서 `tableView.reloadData()`를 부르니 에러가 발생하는 것인데
왜 `viewWillAppear()`가 다시 불리는지도 잘모르겠고 왜 nil이 되는지도 모르겠다

---

아…
문제는 스토리보드에서 뷰컨트롤러의 뷰를 없애고 대신 tableview를 올려놓고는
따로 `@IBOutlet tableView` 를 선언하고 또다시 연결한 뒤에 tableView로 접근해서 생기는 문제다.

그러니까 다음 화면으로 넘길 때 tableView는 nil이 될수 있는데 (여기서 왜 그러는지 잘모르겠음)
이때 잘못 메서드가 불려서 죽는 것..
