# 🧃 쥬스 메이커 프로젝트 저장소

## Flow chart

![3B 쥬스메이커](https://user-images.githubusercontent.com/39452092/121300084-50bbd680-c931-11eb-93fb-f5d7edb60c2c.jpg)

&nbsp;

## UML

![쥬스메이커UML](https://user-images.githubusercontent.com/39452092/121300099-56192100-c931-11eb-84b6-c75c9c119bd9.png)

### 🤩 Details

- miro를 통해 flow chart를 그림

  - 함수별로 상세히 작성해볼까 고민하였지만 UI가 개입되는 부분이 많았기 때문에 너무 복잡해질 것 같아서 생략하였다.

- Step1~Step3까지 종합하여 작성함

- UML을 처음으로 써보았다...! 

  - Coden - 다른 것보다도 `관계를 나타내는 선`이 너무 어려웠다. 아직도 헷갈린다... 프로젝트가 끝난 지금 '코드가 저 UML대로 만들어졌냐' 라고 누군가 묻는다면 당당히 '아니오...'할 것 같다. (달라진 부분은 많지만 뼈대는 거의 비슷!)
  - Jiss - 무작정 코드를 짜는 것보다 이렇게 미리 한번 설계를 하고 이 것대로 코드를 작성하는 것이 확실히 이해가 더 쉬워졌던 것 같다. 하지만 처음 써보아서 STEP3로 갈수록 수정되는 부분은 많았다. 

  &nbsp;

## STEP1

### 📖 주요 학습 개념

* `reduce`

  ```swift
  func reduce<Result>(_ initialResult: Result, _ nextPartialResult: (Result, Element) throws -> Result) rethrows -> Result
  ```

  * 선언은 위와 같이 되어있다. reduce는 한마디로 말하자면 `주어진 클로저를 이용하여 시퀀스의 요소들을 조합하는 고차함수`이다.
  * `initialResult` 매개변수에는 초기 값이 들어간다. (모든 연산의 base값)
  * `nextPartialResult`에는 클로저가 들어가는데, 이 클로저의 매개변수로는 이전 연산의 결과 값인 `Result`와 sequence의 `Element`가 하나씩 들어온다. 클로저의 실행 횟수는 sequence의 요소 개수와 같다.
  * 누산기라고 보면 조금 이해가 더 쉽다. 이전 연산의 결과값이 다음 연산의 결과값에 영향을 미친다는 점에서 `피보나치 수열`을 생각하면 더 쉬울 것 같다.

  [Swift reduce document](https://developer.apple.com/documentation/swift/array/2298686-reduce)

&nbsp;

* `Error Handling`
  * `Error protocol`을 custom enum에 채택시켜 우리만의 에러 case를 만들어보았다.
  * 함수 내에서 에러가 throw되는 경우 이를 내부에서 처리해주지 않으면 호출한 곳에서 에러를 반드시 처리해주어야 한다. (물론 호출한 곳에서도 에러를 propagating할 수는 있지만 어디에선가는 꼭 처리해주어야 한다.)
  * 함수가 에러를 던질 수 있는 경우 함수 선언부 중 매개변수 부분과 반환 값 타입 부분 사이에 `throws`키워드를 넣어주어야 한다.
  * 에러를 던지는 구문은 `throw`이며 이는 guard 구문의 else 내에서도 사용 가능하다. 
  * Error는 `do-catch` 구문으로 처리할 수 있다. 
  * Error case에도 rawValue나 associatedValue를 부여할 수 있다. 
  * Error에 대한 상세 설명을 부여하고 싶다면 `LocalizedError`를 사용해보자.

&nbsp;

* `Nested Type`
  * class나 struct, enum의 내부에 다른 class, struct, enum을 정의하고 사용할 수 있다.
  * 관련있는 타입들끼리 중첩해놓을 때 사용하는 경우가 많다.
  * *다만 언제 이 방식을 쓰고 쓰지 말아야할지 아직은 감이 잡히지 않는다.*

&nbsp;

* **KVO**

  * Key-value observing의 약어로 object가 다른 object의 프로퍼티 변경을 관찰하고자 할 때 사용한다.
  * 두 인스턴스가 직접적으로 맞닿아 있는 경우 사용한다.
  * 관찰받는 object는 NSObject를 상속받아야 하며 관찰받을 프로퍼티 앞에는 `@objc dynamic`을 작성해주어야 한다. (해당 annotation들은 objc와 관련이 있다.)
  * 관찰자 object는 NSObject를 상속받아도 되고 받지 않아도 된다. 
  * *objc나 NSObject같은 것들 때문에 쓰기가 좀 어려웠다.*

  

  *아래는 coden이 채팅에서 주고받았던 내용이다.*

오늘 KVO를 보고 엄청 헷갈리고 멘붕이 와서 정리해봤습니다.. 정리해본 내용이 맞는지 궁금하여 이곳에 올려봅니다.. 정리한 내용에 대해 다른 의견이 있다면 말씀해주세요 ㅠㅠ  

### KVO 구현하기 KVO를 구현하는 방식은 크게 3가지로 구분된다고 본다.  

1. **Objective C방식**    
   - 피관찰자 object에서는 addObserver를 통해 관찰자 object를 등록한다.    
   - 관찰자 object는 내부에 observeValueForKeyPath:ofObject:change:context를 구현하여 변경이 발생했을 시 어떻게 대응할 것인지 작성한다. (콜백) 

2. **Swift방식 - 관찰자가 NSObject를 상속받지 않아도 되는 방식**    

   - 피관찰자 클래스는 `NSObject`를 상속받고 관찰받을 프로퍼티에 `@objc dynamic` 키워드를 붙여준다.    
   - 이후 관찰자 클래스는 내부 프로퍼티로 이 피관찰자 인스턴스를 참조한다.    
   - 관찰자 클래스 내부에서 `[피관찰자 인스턴스 명].observe(_:options:changeHandler:)`를 호출한다. 이 때 "피관찰자 인스턴스 명"을 통해 observe를 호출하므로 첫번째 인자 keyPath에는 `\.[프로퍼티명]`만 작성해줘도 된다. 

3. **Swift방식 - 관찰자도 NSObject를 상속받아야 하는 방식**    

   - 피관찰자 클래스는 `NSObject`를 상속받고 관찰받을 프로퍼티에 `@objc dynamic` 키워드를 붙여준다.   

   - 이후 관찰자 클래스도 `NSObject`를 상속받고 내부 프로퍼티로 이 피관찰자 인스턴스를 참조한다. 이 때 피관찰자에 대한 내부 프로퍼티 앞에는 `@objc`키워드를 적어주어야 한다.   

   - 관찰자 클래스 내부에서 바로 `self.observe(_:options:changeHandler:)`를 호출한다. 이 때 첫번째 인자 keyPath에는 `\.[피관찰자 인스턴스명].[프로퍼티명]`으로 전체 경로를 명확히 작성해준다.    

   - 주의할 점은 생성자 작성 시 (생성자에서 바로)`self.oberve`를 호출하기 위해서는 superclass에 대한 부분이 먼저 존재해야 하므로 `super.init()`을 호출해줘야 한다는 것이다.     

     

   →Swift에서는 `observe(_:options:changeHandler:)`메소드를 피관찰자의 메소드로 호출할 것이냐 관찰자의 메소드로 호출할 것이냐에 따라 구현방식이 조금 달라진다.     

   → 클래스 내부에서는 반드시 Observation에 대한 부분을 어딘가에 받아 저장해야한다. 그렇게 하지 않으면 observing이 바로 해제된다. - (정확하지는 않지만) 관찰 시작 시 관찰에 대한 인스턴스가 생성이 되는데 이 인스턴스를 변수에 받아두지 않으면 이에 대한 참조카운트(ARC)가 바로 0이 되어 인스턴스가 해제되고 관찰이 중지된다.(invalidate)

=============================== **테스트 해본 코드는 아래와 같습니다.** =============================== 

```swift
import Foundation

class Camper: NSObject {
    @objc dynamic var point: Int = 0
}

class CampSupporter {
    var myCamper: Camper
    var observerContainer: NSKeyValueObservation?
    
    init(myCamper: Camper) {
        self.myCamper = myCamper
        observerContainer = myCamper.observe(\.point, options: [.old, .new]) { _, _ in
            print("내가 도와주고 있는 캠퍼의 포인트가 변경됐군!")
        }
    }
}

class CampAdmin: NSObject {
    @objc var myCamper: Camper
    var observerContainer: NSKeyValueObservation?

    init(myCamper: Camper) {
        self.myCamper = myCamper
        super.init()
        observerContainer = self.observe(\.myCamper.point, options: [.old, .new]) { _, _ in
            print("내가 지켜보고 있는 캠퍼의 포인트가 변경됐군!")
        }
    }
}

let coden = Camper()
let jake = CampSupporter(myCamper: coden)
let yagom = CampAdmin(myCamper: coden)

coden.point = 100
coden.point = 1000

/* 결과
내가 지켜보고 있는 캠퍼의 포인트가 변경됐군!
내가 도와주고 있는 캠퍼의 포인트가 변경됐군!
내가 지켜보고 있는 캠퍼의 포인트가 변경됐군!
내가 도와주고 있는 캠퍼의 포인트가 변경됐군!
*/
```

observe를 시작할 때 `NSKeyValueObservation`이 반환되는데 이를 프로퍼티로 받아주지 않으면 관찰이 제대로 되지 않았습니다.. 위에서 제가 생각한 내용이 맞는지 궁금합니다... (인스턴스가 바로 해제되서 관찰이 중지되는 것이 맞는지)

&nbsp;

### 위의 질문에 대한 답변

https://jcsoohwancho.github.io/2019-11-30-KVO(Key-Value-Observing)/

```
Swift 4 이후로는 이 KVO 문법도 Swift스럽게 바뀌었습니다. 변경점은 클로저를 활용함으로써, 오버라이딩이나 context등이 사라진 것입니다. 대신, addObservser 대신 사용하는 observe 메소드가 NSKeyValueObservation이라는 객체를 반환하고, 해당 객체가 힙에 유지되고 있어야만 감시가 이루어진다는 점이 차이점입니다. 즉, Observer를 등록( add)하는 방식에서, Observation 객체를 소유하는 객체가 Observer 역할을 암시적으로 수행하는 방식으로 바뀐 것입니다.
```

라고 하네요! 혹시나 저 말고도 궁금하셨을 분들을 위해 링크 남깁니다!

&nbsp;

* `NotificationCenter`
  * KVO처럼 다른 object로부터 알림을 받을 수 있는 communication pattern 중 하나이다. 
  * 다수의 object에게 동시에 이벤트 발생을 알려줄 수 있다. `aka. 광역도발`
  * 알림을 전달하는 자와 받는 자가 직접적으로 맞닿아 있지 않아도 된다.
  * NotificationCenter default 싱글톤 인스턴스가 중간에서 메시지를 받고 이를 브로드캐스팅하는 역할을 수행한다.
  * post 시 userInfo에 데이터를 담아서 보낼 수 있다. (object에는 메시지를 보내는 당사자가 들어갈 수도 있고 안들어갈 수도 있다.)
  * post 시 지정하는 Notification.Name은 `채널`과 비슷하다.
  * 알림을 받는 쪽에서는 addObserver로 관찰자를 등록해야 한다. (적절한 때에 removeObserver로 관찰자 해제를 해주자)

&nbsp;

* `Access Control`
  * 객체지향언어에서 `캡슐화`와 `은닉화`를 달성할 수 있는 수단
  * 모듈 또는 클래스/인스턴스 등의 특정 항목을 외부에서 볼수 있게 할 것이냐 아니냐를 결정할 수 있다.
  * 작성할 수 있는 키워드에는 public, open, internal, fileprivate, private이 있다.
  * private(set)이라는 특이한 키워드도 존재하는데, 이는 해당 프로퍼티를 read-only로 읽을 수만 있게 하려 할 때 사용한다.

&nbsp;

* `Associated Value`
  * 연관값은 기존에도 배웠던 개념이기는 하지만, Error enum case와 한번 같이 사용해보았다.
  * `switch-case`에서 연관값을 받을 수 있었던 것처럼 `do-catch`의 catch구문에서도 연관값을 받아서 쓸 수 있었다. (이 두곳 외에도 연관값을 받아서 쓸 수 있는 곳이 있을까?)

&nbsp;

## STEP2

### 📖 주요 학습 개념

* **Singleton**
  * Design pattern 중 하나이다. 
  * 특정 클래스의 인스턴스를 하나만 만들어서 사용하는 방식이다.
  * static let 형태로 만들며 프로퍼티명은 보통 `shared`를 사용한다.
  * initializer는 일반적으로 private으로 만들어 둔다.

&nbsp;

* **ViewController LifeCycle**
  * 뷰컨트롤러의 생명주기
  * 헷갈리지 말자! **뷰가 아니라 뷰컨트롤러의 라이프사이클**이다. (뷰는 drawing cycle이라는 그리는 부분에 대한 사이클같은 것은 있지만 별도의 lifecycle이 존재하지는 않는다.)
  * viewDidLoad()
    * 뷰컨트롤러 및 내부 뷰 인스턴스들이 모두 메모리에 올라간 직후 호출되는 메소드
    * 이곳에서 뷰에 대한 추가적인 초기화 작업들을 수행할 수 있다.
    * 뷰 컨트롤러 생애 중 단 한번만 호출된다.
  * viewWillAppear(_:)
    * 뷰컨트롤러가 나타나기 바로 전에 호출되는 메소드
    * 뷰컨트롤러가 사라졌다 나타났다 하면서 여러번 호출 될 수 있다.
  * viewDidAppear(_:)
    * 뷰컨트롤러가 나타난 직후 호출되는 메소드
  * viewWillDisappear(_:)
    * 뷰컨트롤러가 사라지기 직전에 호출되는 메소드
  * viewDidDisappear(_:)
    * 뷰컨트롤러가 사라진 직후 호출되는 메소드
  * ***super메소드를 호출해주는 것에 대하여***
    * 정말 피해야하는 경우가 아니라면 super의 원본 메소드를 먼저 호출해주는 것이 좋다.
    * 동작 상 다른 점이 없다고 하더라도 부모 메소드에 critical한 무언가가 존재할 수 있음. 특히 loadView()

&nbsp;

- Segue를 이용한 화면 전환 및 prepare 메소드를 이용한 데이터 넘겨주기.
  - Storyboard를 통해 segue를 설정하고, 이를 통해 특정 버튼에 화면 전환 동작을 구현해보았다.
  - segue에 identifier를 부여할 수 있었고 이를 이용하여 코드에서 segue를 불러다가(?) 쓸 수 있었다.
  - segue가 동작할 때 데이터를 같이 넘겨주고 싶으면 `prepare(for segue:sender:)`를 사용하자.
  - segue를 코드로 동작시키고 싶다면 `performSegue(withIdentifier identifier:sender:)`를 사용하자

&nbsp;

* **Target-Action**
  * 버튼과 같은 사용자 interaction 수단에 이벤트에 대한 처리를 지정할 수 있는 패턴
  * 버튼들에 대해 각각의 Action메소드를 만들어줄 필요 없이 하나의 메소드로 처리되도록 구현할 수 있다.
  * addTarget을 할 때 메소드 지정은 #selector()로 하며 해당 메소드는 @objc 어노테이션이 존재해야 한다.

&nbsp;

* UIAlertController
  * UIViewController를 상속받은 알럿 창이다.
  * UIAlertAction으로 버튼들을 추가할 수 있다.
  * UIAlertAction들에 대해 handler들을 지정할 수 있다. (눌렀을 시 어떤 동작을 해야하는 경우)

&nbsp;

* Extension, MARK
  * 이번 프로젝트를 구현할 때 코드들을 extension으로 구분하고 MARK 주석을 달아보았다.
  * 코드의 가독성이 높아졌고 원하는 부분을 빠르게 찾을 수 있었다.
  * Extension을 하는 경우 파일 이름은 보통 `UIViewController+Extension`형식으로 작성한다.
  * ViewController의 경우 Extension으로 코드를 나누는 기준은 '저장 프로퍼티와 라이프사이클', '기능별 분리', '접근지정자별 분리'이다.

&nbsp;

-> STEP3에서 별도로 학습한 개념은 존재하지 않는다. 

&nbsp;

## 용어 정리 - 결합도, 응집도, 의존성

* 결합도

  * 소프트웨어 공학에서, 결합도(coupling) 또는 의존도는 어떤 모듈이 다른 모듈에 의존하는 정도를 나타내는 것이다.

    > [위키백과 - 결합도](https://ko.wikipedia.org/wiki/결합도)

* 응집도

  * 응집도란, 한 클래스 또는 모듈이 특정 목적 또는 역할을 얼마나 일관되게 지원하는지를 나타내는 척도이다.

    > [tistory blog - javaiyagi](https://javaiyagi.tistory.com/69)

* 의존성
  * [의존성이란](https://velog.io/@huttels/의존성이란)

&nbsp;

결합도와 응집도에 대해 잘 정리한 블로그를 소개한다.

[[설계 용어\] 응집도와 결합도](https://medium.com/@jang.wangsu/설계-용어-응집도와-결합도-b5e2b7b210ff)

*말꼬가 올려준 참고 링크*

[Software Engineering | Differences between Coupling and Cohesion - GeeksforGeeks](https://www.geeksforgeeks.org/software-engineering-differences-between-coupling-and-cohesion/)

&nbsp;

**2주동안 탈도많았지만 배운게 정말 많았던 시간이었다.** 😀👍 앞으로도 화이팅하자.







