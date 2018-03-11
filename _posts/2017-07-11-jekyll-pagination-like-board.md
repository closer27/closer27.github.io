---
layout: post
title: "jekyll 블로그 pagination 방식을 게시판 형태로 변경하기"
date: 2017-07-11
categories: note
tags:
- jekyll
- pagination
comment: yes
---

# Modify the way of pagination to like a board in jekyll
---

현재 블로그에서 사용하고 있는 [jekyll](http://jekyllrb.com/) 블로그 테마 [lanyon](https://github.com/poole/lanyon)에서는 pagination 방식이 '이전 페이지', '다음 페이지' 이렇게만 제공하고 있었는데 보통의 게시판처럼(< 1,2,3,4,5 >)형식으로 변경하고자 했다. 카카오 기술블로그에서 쓰고있는 로직을 베이스로 일부만 변경하였다.

지킬의 paginator 라이브러리에서 제공하는 두가지 값을 활용한다.

- `paginator.total_pages` : 총 페이지 수
- `paginator.page` : 현재 페이지 숫자

구현해야할 로직은 다음과 같다.

1. 현재 페이지에서 표시해야할 첫 페이지와 끝 페이지 번호를 계산하여 표시한다.
2. 현재 페이지 번호는 별도로 표시한다.
3. 이전, 다음 버튼은 다음 단위의 페이지의 첫페이지로 이동시킨다(1~5페이지면 6 페이지로 이동)

1번 로직을 구현하기 위해서는 현재 페이지(paginator.page)를 기준으로 first와 last를 계산해줘야한다.
즉, 현재 페이지를 기준으로 현재 페이지에서 이동할 수 있는 첫페이지와 끝페이지를 계산해야하기 때문. 가령 현재 페이지가

> 1~5 사이인 경우 => first는 1, last는 5여야 한다.

> 6~10 사이인 경우 => first는 6, last는 10이여야한다.

결국 paginator.page 가 1~5 중 어느 값이어도 first는 1이 나와야한다.

```
first = ( page / 5 ) * 5 + 1
// 5로 나누고 다시 곱한 이유는 몫만 취하기 위해서
last = first + 4
```
이 경우에는 현재 페이지가 5페이지일 때 문제가 된다. 5페이지일 때는 결과값이 2가 나와 첫페이지가 2페이지부터가 되버린다.

변경된 로직

```
if (page % 5 != 0) {
  first = ( page / 5 ) * 5 + 1
  // 5로 나누고 다시 곱한 이유는 몫만 취하기 위해서
  last = first + 4
} else if (page % 5 == 0) {
  first = (page -1) / 5 * 5 + 1
  // 현재 5페이지일 때는 -1을 빼고 계산한다.
}
````

이를 jekyll에서 사용하고있는 템플릿 라이브러리인 `liquid`에서 사용하기 위해선 다음과 같이 작성 된다.

```liquid
{ % assign first = paginator.page | divided_by: 5 | floor | times: 5 | plus: 1 % }
```

이후 first와 last를 기반으로 for loop을 돌면서 계산 된 페이지 숫자가 표시된다.

그런데 막상 돌려보니 숫자가 기대한 값대로 표시되지 않는다. 찾아보니 first의 결과가 계속 1로 표시되는 문제가 발생한다. paginator.page가 6일때도 왜 1로 계산이 되는거지?
원래대로라면 6 / 5 = 1.2 -> 1 * 5 = 5 + 1 = 6이 나와야되는데.

알고보니 if 문에 수식들이 정확하게 연산이 되지 않아 조건문을 잘못타고 있었다.
liquid의 버그인건지는 잘모르겠지만

조건문에 사용하기 위한 변수를 미리 계산해두고 조건만 비교할 수 있도록 한다


이렇게해서 각 페이지에서 표시할 첫 페이지와 끝페이지 계산하는 수식은 다음과 같다.

```liquid
 { % assign remainder = paginator.page | modulo:5 %}
 { % if remainder != 0 %}
     { % assign first = paginator.page | divided_by: 5 | floor | times: 5 | plus: 1 %}
 { % else %}
     { % assign first = paginator.page | divided_by: 5 %}
 { % endif %}
```

2번 로직은 for loop 내에서 표시되는 페이지 숫자가 현재 page와 같으면 별도의 스타일을 적용하도록 설정.

3번 로직은 다음버튼의 링크 경로를 다음과 같이 설정(마지막 페이지일 경우에는 disabled 처리)

```liquid
<a href="/page/{{ last | plus: 1 }}">
// last 페이지 값의 + 1
```


그렇게 완성된 페이지네이션 로직

```liquid
<ul id="pagination" role="navigation">
    { % if paginator %}
        { % if paginator.total_pages <= 5 %}
            { % assign first = 1 %}
            { % assign last = paginator.total_pages %}
        { % else %}
            { % assign remainder = paginator.page | modulo:5 %}
            { % if remainder != 0 %}
                { % assign first = paginator.page | divided_by: 5 | floor | times: 5 | plus: 1 %}
            { % else %}
                { % assign first = paginator.page | divided_by: 5 %}
            { % endif %}
            { % if first < 1 %}
                { % assign first = 1 %}
            { % endif %}
            { % assign last = first | plus: 4 %}
            { % if last > paginator.total_pages %}
                { % assign last = paginator.total_pages %}
            { % endif %}
        { % endif %}
        { % assign previous_page = first | minus: 5 %}
        { % if previous_page > 1 %}
            <li id="page-prev"><a href="/page/{{ first | minus: 5 }}"><span class="sr-only"><</span></a>
            </li>
        { % elsif previous_page == 1 %}
            <li id="page-prev"><a href="/"><span class="sr-only"><</span></a>
            </li>
        { % else %}
            <li id="page-prev" class="disabled"><span class="sr-only"><</span></li>
        { % endif %}
        { % for p in (first..last) %}
            { % if p == paginator.page %}
                <li class="page-number active">{{ p }}</li>
                { % elsif p == 1 %}
                <li class="page-number"><a href="{{ site.baseurl }}/">{{ p }}</a></li>
            { % else %}
                <li class="page-number"><a
                            href="{{ site.baseurl }}{{ site.paginate_path | replace: ':num', p }}">{{ p }}</a></li>
            { % endif %}
        { % endfor %}
        { % if last < paginator.total_pages %}
            <li id="page-next"><a href="/page/{{ last | plus: 1 }}"><span class="sr-only">></span></a>
            </li>
        { % else %}
            <li id="page-next" class="disabled"><span class="sr-only">></span></li>
        { % endif %}
    { % endif %}
</ul>
```

-
Referencing

: kakao 기술 블로그. (2017). “kakao 기술 블로그가 GitHub Pages로 간 까닭은”, [online] Available at: [http://tech.kakao.com/2016/07/07/tech-blog-story/](http://tech.kakao.com/2016/07/07/tech-blog-story/) [Accessed July 11, 2017].
