---
layout: post
title: "Grand Centered Dispatch(GCD) 개요"
date: 2012-11-03
tags: [ios]
---

- 다중 스레드 프로그래밍

- 파일이나 네트워크 연결 같은 외부 리소스를 기다리면서 프로그램의 실행이 중단되면 전체 이벤트의 흐름이 완전이 멈춘다. 현대 OS에서는 한개의 스레드가 특정 이벤트를 기다리느라 멈추더라도 나머지 스레드를 계속 실행할 수 있게끔 프로그램 내에서 다중 스레드의 실행을 지원한다.
- 애플에서는 시스템의 스레드 레이어를 잘 모르더라도 코드를 동기적으로 실행되는 여러 조각으로 나누고 싶어 하는 사람들을 위해 그랜드 센트럴 디스패치(GCD)를 만들었다.
- 다중 cpu를 활용해 어플릴케이션이 수행할 작업을 여러 스레드로 할당해 나눠서 실행할 수 있게 해주는 전혀 새로운 API를 제공한다.
- 새 API의 대부분은 블록을 통해 접근할 수 있다. 메서드 내에서 관련 코드를 한데 모아두는 동시에 서로 다른 객체와의 연동을 구조화하는 새로운 코딩 방식을 제공한다.

스레딩의 기본

- 한 프로세스 내의 모든 스레드는 실행 가능 프로그램과 전역 데이터를 공유한다. 또 개별 스레드는 해당 스레드에서만 단독으로 사용하는 데이터도 가질 수 있다.
- 스레드는 뮤텍스(mutual exclusion의 약어)라는 특수 구조 또는 락을 활용함으로써 특정 코드 조각을 여러 스레드에서 동시에 실행하지 못하게 할 수 있다. 이는 한 스레드가 (코드의 핵심 영역에 속한) 값을 업데이트할 때 다른 스레드에 락을 걸게 해주므로 여러 스레드가 동일 데이터를 동시에 접근하더라도 항상 올바른 결과가 나오게 해준다.
- 코코아 터치에서 파운데이션 프레임워크(NSString, NSArray 등)는 보통 다중 스레드로부터 안전하다고 할 수 있다. 그러나 UIKit (UIApplication, UIView 등) 프레임워크는 대부분 다중 스레드로부터 안전하지 않다. 이 말은 실행 중인 iOS 어플리케이션에서 UIKit 객체를 처리하는 메서드 호출을 모두 메인 스레드라고 부르는 동일 스레드 내에서 실행해야 한다는 뜻이다. 만일 다른 스레드에서 UIKit 객체에 접근하면 위험한 결과가 생길 수 있다.
- 기본적으로 메인 스레드는 iOS 어플리케이션의 모든 행동(사용자 이벤트를 통해 호출되는 액션 처리 등)이 일어나는 공간이므로 간단한 어플리케이션에서는 메인 스레드에 굳이 신경 쓰지 않아도 된다.
- 애플에서는 오랜 시간이 걸리는 작업을 작업 단위로 나누고 이들 단위를 큐에 넣어서 실행하는 방식을 스레딩 해결책으로 권장한다.

GCD : 저수준 큐

- GCD의 핵심 개념 중 하나는 큐다. 시스템에서는 항상 메인 스레드에서의 실행이 보장된 큐를 비롯해 미리 정의된 큐를 여러 개 제공한다. 이런 큐는 다중 스레드로부터 안전하지 않은 UIKit과 완벽하게 연동할 수 있다. 큐는 원하는 만큼 만들 수 있다. GCD 큐는 엄격한 FIFO다.

블록헤드 되기

- CGD를 잘 활용하려면 블록이 매우 중요하다.
- 메서드나 함수처럼 블록은 하나 이상의 매개변수를 받을 수 있고 반환값을 지정할 수 있다. 블록 변수를 선언하려면 캐럿(^) 기호와 더불어 매개변수 및 반환값을 선언할 괄호를 추가로 사용해야 한다.

{% highlight Objective-C %}
// 매개변수와 반환값 없이 "loggerBlock" 블록 변수를 정의.
void (^loggerBlock)(void);

// 블록을 위에 선언한 변수에 대입. 이 블록처럼 매개변수와 반환값이 없는 블록은 앞의 변수 선언처럼 void를 사용한 '장식'이 필요 없다.
loggerBlock = ^{ NSLog(@"I'm just glad they didn't call it a lambda"); };

// 함수를 호출하는 것처럼 블록을 실행한다.
loggerBlock();     // 이렇게 호출하면 결과가 콘솔에 나타난다.

// 블록에서 수정할 수 있는 변수를 정의
__block int a = 0;

// 자신의 스코프 내에서 변수를 수정할 블록을 정의
void (^sillyBlcok)(void) = ^{ a = 47; };

// 블록을 호출하기 전 변수 값을 확인
NSLog(@"a == %d", a);     // 출력 결과 "a == 0"

// 블록을 실행
sillyBlock();

// 블록 호출 후 변수 값을 다시 확인
NSLog(@"a == %d", a);     // 출력 결과 "a == 47"
{% endhighlight %}

- 블록을 GCD와 함께 사용하면 한 번에 코드 블록을 큐에 추가할 수 있기 때문에 진가를 발휘 할 수 있다.

{% highlight Objective-C %}
- (void)doWork:(id)sender {
    NSDate *startTime = [NSDatedate];
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
    // 항상존재하는전역큐를호출한다. 첫째인자는우선순위를지정하는인자고두번째는지금사용하지않으므로항상 0으로지정해야한다.
        NSString *fetchedData = [selffetchSomethingFromServer];
        NSString *processedData = [selfprocessData:fetchedData];
        NSString *firstResult = [selfcalculateFirstResult:processedData];
        NSString *secondResult = [selfcalculateSecondResult:processedData];
        NSString *resultsSummary = [NSStringstringWithFormat:@"First: [%@]\nSecond: [%@]", firstResult, secondResult];
        _resultsTextView.text = resultsSummary;
        NSDate *endTime = [NSDatedate];
        NSLog(@"Completed in %f seconds", [endTime timeIntervalSinceDate:startTime]);
    });
}
{% endhighlight %}

이렇게 가져온 큐는 이후 등장하는 코드 블록과 더불어 dispatch_async()함수의 인자로 넘겨준다. 그럼 GCD는 전체 블록을 받아서 이를 백그라운드 스레드에게 넘겨주고, 이 코드는 메인 스레드에서 실행될 때처럼 한번에 한단계씩 처리된다.
이 코드에서는 블록이 시작하기 전에 startTime이라는 변수를 정의하고 이 값을 블록 끝에서 사용하는 것을 볼 수 있다. 직관적으로 해석하면 이런 코드는 말이 안 된다. 블록이 실행될 시점에서는 doWork: 메서드를 이미 빠져나간 상태이므로 startTime 변수가 가리키는 NSDate 인스턴스도 이미 릴리즈 돼있을 것이기 때문이다. 바로 이 점이 블록 사용법의 핵심이다. 블록이 실행 도중 '밖에 있는' 변수에 접근할 경우 블록이 생성될 때 특수 설정이 일어나 블록이 이들 변수에 접근할 수 있게해준다. 이런 변수에서 갖고있는 값은 블록 내에서 변수 값을 사용할 수 있게 (int나 flaot 같은 일반 C 타입인 경우) 복제 되거나 (객체 포인터인 경우) 유지된다. doWork:의 두번째 줄에서 dispatch_async가 호출되고 코드의 블록이 생성될 때 startTime에는 실제로 retain 메세지가 전달되고, 반환값이 블록 내에 같은 이름(startTime)을 갖고 있는 새로운 스태틱 상수에 대입된다.

{% highlight Objective-C %}
//  NSString *firstResult = [self calculateFirstResult:processedData];
//  NSString *secondResult = [self calculateSecondResult:processedData];
__block NSString *firstResult;
__block NSString *secondResult;
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, dispatch_get_global_queue(0, 0), ^{
    firstResult = [selfcalculateFirstResult:processedData];
});
dispatch_group_async(group, dispatch_get_global_queue(0, 0), ^{
    secondResult = [selfcalculateSecondResult:processedData];
});
dispatch_group_notify(group, dispatch_get_global_queue(0, 0), ^{
    NSString *resultsSummary = [NSStringstringWithFormat:@"First: [%@]\nSecond: [%@]", firstResult, secondResult];
    dispatch_async(dispatch_get_main_queue(), ^{
        _startButton.enabled = YES;
        _startButton.alpha = 1.0f;
        [_spinnerstopAnimating];
        _resultsTextView.text = resultsSummary;
    });
    NSDate *endTime = [NSDatedate];
    NSLog(@"Completed in %f seconds", [endTime timeIntervalSinceDate:startTime]);
});
{% endhighlight %}

디스패치 그룹을 통해 동시성 작업을 구현한다. 그룹 컨텍스트 내에서 dispatch_group_async() 함수를 통해 비동기적으로 디스패치한 모든 블록은 동시 실행이 가능한 경우 동시 실행이 이뤄지도록 여러 스레드로 분산하는 등의 수단을 통해 최대한 빨리 실행되게끔 설정된다. 또 dispatch_group_notify() 를 사용하면 그룹 내의 모든 블록이 완전히 실행된 후 호출할 추가 블록도 지정할 수 있다.
