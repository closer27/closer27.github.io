---
layout: post
title: "Objective-C 코딩 지침"
date: 2014-06-01
tags: [iOS, objective-c, coding, tip]
---

- 단어 조합 규칙
    - 메서드명 유형
        - 접두어를 붙이지 않습니다. 첫 글자는 소문자로 시작합니다.
        - 예: `strings`, `anElement`, `stringByExpandingTideInPath`
    - 함수명 유형
        - 접두어를 붙입니다. 접두어 다음의 글자는 대문자로 시작합니다.
        - `NSData`, `CFString`, `NSRunAlertPanel`
    - 예외는 잘 알려진 약어가 첫 단어일 때, 예: URLWithString:
- 접두어(prefix)
    - 언더스코어(`_`)로 시작하는 이름은 금지
    - 애플 사가 API를 구성하는 동안에 사용하는 인스턴스 변수나 비공개 메서드명으로 사용한다고 명시 됨.
- 이름에 사용하는 단어
    - 간결한 이름이 좋다고 단어를 줄이는 건 피해야 함.
    - 단, 널리 사용되고 역사가 오래 된 줄임말은 허용.
    - | alloc = Allocate | init = initialize |
| max = Maximum | max = Maximum |
| max = Maximum | app = Application |
| min = Minimum | cacl = Calculate |
| msg = Message | dealloc = Deallocate |
| rect = Rectangle | func = Function |
| temp = Temporary | info = Information|
  - 같은 의미의 단어가 몇가지 있을 때는 Cocoa API와 같은 단어 사용
      - 예: 요소의 개수는 `count`이지, `number`가 아님.
      - 잘모를 때는 Cocoa 레퍼런스 참고
  - 전치사 사용법
      - 초기화할 때 파라미터를 지정할 때는 `with`
      - 위치를 표시하는 `at`
      - 출력 방향을 나타내는 `to`
  - 복수형 사용법 주의
      - 단수형일 때 하나만
      - 복수형일 때 배열로
- 클래스명과 프로토콜명
    - 클래스 명은 `<함수명 형식>`으로 그 클래스 내용을 잘 표현하는 명사 구문 사용
    - 프로토콜 명도 클래스명과 같은데 클래스명과 혼동할 때가 많으므로 프로토콜 명에는 ‘~ing’를 붙여서 구별하기도 함.
- 카테고리를 포함한 파일명
    - `<클래스명+카테고리명.m>`
- 메서드명 관련 일반 규칙
    - 리시버 객체에 어떤 동작을 시키는 메서드는 이름에 동사를 사용한다. 하지만 <do>는 어떤 일을 하는지 모르므로 피해야 함.
    - 리시버에 어떤 값을 문의하는 메서드라면 그 값을 나타내는 이름을 명서드명으로 사용한다 - getXXX는 쓰지 않는다.
    - 참/거짓 문의는 `isEqualToData:`와 같이 `is~`로 하던지, `fileExistsAtPath:`처럼 참에 해당하는 상태를 기술한다.
    - 인수가 여러 개일 때 모든 인수에 키워드를 대응시킨다. 또한 각 인수가 무엇인지 나타내는 말을 붙인다.
        - `setValue:forKey:`
    - 상속 등으로 어떤 메서드 기능을 특화시킨 메서드는 새로운 키워드를 끝에 덧붙이는 방법이 권장.
        - `compare:`
        - `compare:options:`
        - `compare:options:range:`
    - 인수의 키워드에는 `and`를 붙이지 않는다.
    - 메서드 두 개가 서로 다른 동작을 한다면 `and`로 묶는다
        - `openFile:withApplication:andDeactivate:`
- 접근자 규칙
    - 객체 속성을 참조, 설정하는 메서드를 접근자 메서드라고 부른다.
    - 속성명이 명사일 때
        - 참조 : `- (NSColor *)color;`
        - 설정 : `- (void)setColor:(NSColor *)aColor;`
    - 속성명이 형용사일 때
        - 참조 : `- (BOOL)isEditable;`
        - 설정 : `- (void)setEditable:(BOOL)flag;`
    - 속성명이 참,거짓의 명사일 때
        - 참조 : `- (BOOL)showsHelp;`
        - 설정 : `- (void)setShowsHelp:(BOOL)showsHelp;`
    - 키-값 코딩에서 사용하는 접근자(프로퍼티명은 name)일 때
        - 참조 `- (elementType)name;` 또는 `- (BOOL)isName;`
        - 설정 `- (void)setName:(elementType)newName;`
- 객체 집합 조작
    - 객체 집합에 요소를 추가, 삭제하는 메세지는 다음과 같다.
    - 또한 요소가 된 객체를 포함한 배열을 돌려주는 메서드는 객체의 종류를 복수형으로 한 메서드명을 사용
    - `- (void)addElement:(elementType)obj;`
    - `- (void)removeElement:(elementType)obj;`
    - `- (NSArray *)elements;`
- 인수명에 대한 주석
    - 인수명은 소문자로 시작하는 `<메서드명 형식>`으로 엮는다. 알기 어려운 줄임말은 피한다.
    - 단순히 부정관사를 형식명에 붙인 `anObject`나 `aSelector` 같은 이름을 사용하거나, 역할에 따른 친숙한 인수명이 정해진 경우도 많다.
- 인스턴스 변수명
    - 인스턴스 변수명은 소문자로 시작하는 `<메서드명 형식>`으로 엮는다. 밑줄로 시작하지 않는다.
    - 객체의 속성을 명확하게 나타낼 수 있어야 하며, 객체 외부에서 접근 가능한 것은 `@public`이 아니라 접근자 메서드를 작성하거나 선언 프로퍼티로 만든다.
- 함수명
    - `<함수명 형식>`으로 엮는다. 관련 클래스나 상수와 같은 접두어를 사용. 이름에는 함수의 동작을 나타내는 동사나 반환값을 나타내는 명사를 사용
    - `NSLog()`, `NSLocalizedString()`, `NSCreateZone()`
- 형식명, 상수명, 문자열 상수명
    - 구조체나 열거형 같은 새로 정의한 형식명, 매크로 상수, 열거 상수, 수식어 const를 사용하여 정의한 상수명 등은 `<함수명 형식>`으로 엮는다.
    - 조건부 컴파일을 위해 `#if`와 함께 사용하는 매크로는 모두 대문자로 쓴 이름(`NS_BLOCK_ASSERTIONS` 같은)을 쓴다.
    - 알림명, 예외명, 여러 도메인명, 사전 키 등에서 사용하는 문자열은 매크로나 전역 변수로 정의해 둔다.
    - 예외명 선언 예: `FOUNDATION_EXPORT`는 extern을 나타내는 매크로이다.
        - `FOUNDATION_EXPORT NSString * const NSRangeException;`
    - 상수명은 종류에 따라 몇가지 관습이 있다.
        - 예외명 : 끝에 `Exception`을 붙인다
        - 에러명 : 끝에 `Error`를 붙인다.
        - 알림명 : `‘관련 클래스명’ + ‘Will’` 또는 `‘Did’ + ‘동사원형 + 문자열’ + Notification`
            - 무엇이 이미 발생한 것을 알리는 이름은 `‘Did + 동사원형’`
            - 지금부터 무엇이 일어난다는 것을 알리는 이름에는 `‘Will + 동사원형’`
- 델리게이트의 메서드명과 알림명
    - 델리게이트가 구현하는 메서드는 원칙적으로 델리게이트에게 메세지를 위임하는 객체의 클래스명을 메서드명 앞에 붙인다.
    - 접두어는 붙이지 않고 소문자로 시작.
        - 예: `- (BOOL)textView:(NSTextView *)aTextView clickedOnLink:(id)link;`
    - 델리게이트에 대한 알림 메세지명은 알림명과 비슷하지만 접두어와 끝의 Notification을 붙이지 않는다.
        - 예: `windowDidExpose:`
    - 델리게이트에 대해 ‘~를 해도 되는가’라고 문의하는 메세지는 `‘should + 동사원형’`을 사용.
        - 예: `windowShouldClose:`
- 전역이자 유일한 이름
    - 애플리케이션 식별명이나 도메인명처럼 전역인 이름을 하나밖에 없는 것으로 하려면 자바 패키지명과 같은 명명 규칙을 추천한다.
    - `com.company.애플리케이션명`
