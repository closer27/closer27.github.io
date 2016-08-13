---
layout: post
title: "Sublime Text의 Package Control을 통한 TypeScript 플러그인 설치"
date: 2016-08-13
tags: [typescript, sublimetext, plugin]
comment: yes
---

sublime text의 package control을 이용해서 typescript를 위한 서브라임 플러그인을 설치해보자

### Package Control 설치

우선 서브라임의 패키지 컨트롤을 설치해보자.

서브라임텍스트를 실행한뒤 ctrl + `을 눌러 콘솔창을 켠다 (또는 메뉴 - View - Show Console)
그리고 커맨트 창에 다음과 같은 명령어를 복붙하여 입력한다

Sublime Text 3 기준
```
import urllib.request,os,hashlib; h = '2915d1851351e5ee549c20394736b442' + '8bc59f460fa1548d1514676163dafc88'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```

이후 설치 이후 다시 시작하라는 지시에 따른다

### TypeScript 플러그인 설치

이제 typescript 플러그인을 설치해보자
https://github.com/Microsoft/TypeScript-Sublime-Plugin 참조

1. 메뉴에서 Sublime Text -> Preference -> Package Control -> Install Package를 눌러
“TypeScript”로 검색하면 이용 가능한 플러그인들이 검색 된다
2. TypeScript Completion과 TypeScript-Sublime-Plugin 두가지를 설치한다

껐다 키면 끝
