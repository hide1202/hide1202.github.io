---
title: (1) F# 시작하기
categories:
  - Fsharp
tags:
  - Fsharp
---

# 왜 F# 인가?
최근에 함수형 프로그래밍에 대한 관심이 늘어나고 있는 추세다. Scala, Haskell, Clojure 등 다양한 함수형 프로그래밍 언어(혹은 멀티 패러다임 언어)들의 관심 또한 많이 늘어났다. 나 또한 팀에서 Scala 스터디를 진행했었다. 하지만 닷넷을 좋아하던 나로서는 항상 F#이 눈에 밟히곤 했다. 

상황에 따라 언어를 선택하는건 어쩔 수 없는 현실이긴 하지만, 한번 해보고 싶은 언어를 공부해보는건 손해는 아니라는 생각이 들었다. 그래서 영어 장벽을 무릅쓰고 F#을 차근차근 보고자 한다.

우선 F#에 대해서 간단한 소개를 보고, 왜 F#인지에 대해서 다시 논해보도록 하자.

# F# 이란?
F#은 복잡한 문제들을 간단한 코드를 작성해서 해결하기 위한 강한 타입(strongly-typed)의 함수형 우선(functional-first) 프로그래밍 언어이다. `함수형 우선` 이란 말에서 알 수 있듯이 F#은 함수형 프로그래밍을 지원하면서 기존의 객체 지향과 절차 프로그래밍 또한 지원한다. F#은 닷넷 프레임워크의 일급 멤버이며 ML 계열의 함수형 언어와 매우 흡사하다고 한다.

## 멀티 패러다임 언어
- 일급 함수 : 유연한 함수 조작이 가능하다.
- 함수 합성, 파이프라이닝 : 새로운 함수들을 만들기 위해서 함수들을 합성할 수 있고, 데이터의 연속적인 연산들의 코딩을 단순화할 수 있다.
- 타입 추론 : 타입 안정성을 유지하면서 명시적인 타입 호출을 줄일 수 있다.
- 자동 일반화(제네릭) : 별도의 노력 없이 다양한 유형의 코드를 쉽게 작성하여 코드 재사용을 할 수 있다.
- 패턴 매칭 지원 : 복잡한 조건의 코드를 단순화한다. (참고. [Discriminated unions](https://docs.microsoft.com/ko-kr/dotnet/fsharp/language-reference/discriminated-unions))
- 불변 데이터를 위한 컬렉션 타입들
- 익명 함수 표현(Lambda expressions)
- 함수 인수의 부분 적용 : 인수를 부분 적용함으로서 나머지 인수를 요구하는 새로운 함수를 만들 수 있다.
- Code Quotations, a feature that enables you to manipulate F# expressions programmatically. 

F#은 아래와 같이 객체 지향 프로그래밍과 닷넷 프레임워크 기능을 지원합니다.
- 닷넷 프레임워크 객체 모델 : 프로퍼티, 메소드, 이벤트을 가진 객체들; 다형성 또는 가상 함수들; 상속; 인터페이스;
- 데이터 캡슐화, 혹은 구현으로부터 타입의 공개된 인터페이스의 분리할 수 있다.
- 연산자 오버로딩 : 제네릭과 기탑재된 기본 타입들에도 잘 동작한다.
- 타입 익스텐션 : 새로운 파생 타입을 만드는 추가적인 오버헤드 없이 이미 존재하는 타입을 쉽게 확장가능합니다.
- 객체 표현식 : 새로운 타입을 선언하고 객체를 만드는거 대신, 필요할때 표현식으로 암시적으로 작은 객체들을 정의할 수 있다.
- 닷넷 프레임워크와 어떤 관리 코드 어셈블리에도 접근할 수 있습니다.
- 플랫폼 호출을 통해서 네이티브 코드에 접근할 수 있습니다.

# 왜 F# 인가? (continue)
## 간결함
- F#은 중괄호, 세미콜론 등과 같은 "노이즈" 를 쌓아 두지 않습니다.
- 강력한 유형 추론 시스템 덕분에 객체 유형을 지정할 필요가 거의 없습니다.
- C#과 비교하면 동일한 문제를 해결하는 데 필요한 코드 줄이 더 적습니다.

{% highlight fsharp linenos %}
// one-liners
[1..100] |> List.sum |> printfn "sum=%d"

// no curly braces, semicolons or parentheses
let square x = x * x
let sq = square 42 

// simple types in one line
type Person = {First:string; Last:string}

// complex types in a few lines
type Employee = 
  | Worker of Person
  | Manager of Employee list

// type inference
let jdoe = {First="John";Last="Doe"}
let worker = Worker jdoe
{% endhighlight %}

# References
- About F# : [http://fsharp.org/about/index.html](http://fsharp.org/about/index.html)
- Why use F# : [http://fsharpforfunandprofit.com/why-use-fsharp/](http://fsharpforfunandprofit.com/why-use-fsharp/)
- Visual F# : [https://msdn.microsoft.com/visualfsharpdocs/conceptual/visual-fsharp](https://msdn.microsoft.com/visualfsharpdocs/conceptual/visual-fsharp)
- The F# Core Engineering Group : [http://fsharp.github.io/](http://fsharp.github.io/)