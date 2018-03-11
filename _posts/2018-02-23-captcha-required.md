---
layout: post
title: "fatal: remote error: CAPTCHA required 에러가 뜰 때 해결 방법"
date: 2018-02-23
categories: note
tags:
- git
- keychain
comment: yes
---

회사에서 보안상의 이유로 현재 사용하고 있는 비밀번호를 변경하라고 권고가 내려왔다. 비밀번호를 변경한 이후
개발 중 git commit과 같은 명령을 수행할 때 다음과 같은 에러가 떴다.

```
fatal: remote error: CAPTCHA required
Your Bitbucket account has been locked. To unlock it and log in again you must
solve a CAPTCHA. This is typically caused by too many attempts to login with an
incorrect password. The account lock prevents your SCM client from accessing
Bitbucket and its mirrors until it is solved, even if you enter your password
correctly.

If you are currently logged in to Bitbucket via a browser you may need to
logout and then log back in in order to solve the CAPTCHA.

Visit Bitbucket at https://bitbucket.pearson.com for more details.
```

git이 계속 위와같은 CAPTCHA 메세지를 던지면서 fetch나 push가 전혀 안될때는

> 브라우저에서 Bitbucket 접속해 로그아웃한 뒤 로그인 다시한 후 재시도

----
여기까지가 보통의 방법인데
`Authenticated Error`가 뜨면서 마찬가지로 안되고 또 다시 CAPTCHA 에러가 나타났다.

소스트리, 터미널 명령어 모두 문제가 발생해서 인텔리제이 git 설정을 봤더니

Intellij에서 preference -> appearance & behavior system settings -> passwords

![](https://d.pr/i/CJg4En+)

이처럼 나의 빗버킷의 계정 정보는 macOS의 keychain에 저장 중인 것을 알 수 있었다.
다음과 같이 수행한다.

> 1. macOS의 Keychain Access 앱을 켠 뒤 검색창에 bitbucket을 찾아 기존에 저장돼있는 키를 지우기
> 2. 다시 git 명령어 수행

새로 비밀번호를 입력하는 창이 뜨면서 다시 입력할 수 있었고 이후 정상적으로 동작했다.

계속 os에 저장된 keychain의 자동으로 읽어와서 발생했던 문제였다.
