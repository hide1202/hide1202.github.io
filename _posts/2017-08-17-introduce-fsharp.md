---
title: "F# beginning : (1) F# 소개"
categories:
  - Fsharp
tags:
  - Fsharp
toc: true
---

# 왜 F# 인가?
최근에 함수형 프로그래밍에 대한 관심이 늘어나고 있는 추세다. Scala, Haskell, Clojure 등 다양한 함수형 프로그래밍 언어(혹은 멀티 패러다임 언어)들의 관심 또한 많이 늘어났다. 나 또한 팀에서 Scala 스터디를 진행했었다. 하지만 닷넷을 좋아하던 나로서는 항상 F#이 눈에 밟히곤 했다. 

상황에 따라 언어를 선택하는건 어쩔 수 없는 현실이긴 하지만, 해보고 싶은 언어를 공부해보는건 손해는 아니라는 생각이 들었다. 그래서 영어 장벽을 무릅쓰고 F#을 차근차근 보고자 한다.

우선 F#에 대해서 간단한 소개를 보고, 왜 F#인지에 대해서 다시 논해보도록 하자. 참고로 이 문서는 대부분이 References의 페이지의 글을 번역한 글이다.

# F# 이란?
F#은 복잡한 문제들을 간단한 코드를 작성해서 해결하기 위한 `강한 타입(strongly-typed)의 함수형 우선(functional-first) 프로그래밍 언어`이다. '함수형 우선' 이란 말에서 알 수 있듯이 F#은 함수형 프로그래밍을 지원하면서 기존의 객체 지향과 절차 프로그래밍 또한 지원한다. F#은 닷넷 프레임워크의 일급 멤버이며 ML 계열의 함수형 언어와 매우 흡사하다고 한다.

## 멀티 패러다임 언어
- 일급 함수 : 유연한 함수 조작이 가능하다.
- 함수 합성, 파이프라이닝 : 새로운 함수들을 만들기 위해서 함수들을 합성할 수 있고, 데이터의 연속적인 연산들의 코딩을 단순화할 수 있다.
- 타입 추론 : 타입 안정성을 유지하면서 명시적인 타입 호출을 줄일 수 있다.
- 자동 일반화(제네릭) : 별도의 노력 없이 다양한 유형의 코드를 쉽게 작성하여 코드 재사용을 할 수 있다.
- 패턴 매칭 지원 : 복잡한 조건의 코드를 단순화한다. (참고. [Discriminated unions](https://docs.microsoft.com/ko-kr/dotnet/fsharp/language-reference/discriminated-unions))
- 불변 데이터를 위한 컬렉션 타입들
- 익명 함수 표현(Lambda expressions)
- 함수 인수의 부분 적용 : 인수를 부분 적용함으로서 나머지 인수를 요구하는 새로운 함수를 만들 수 있다.
- Code Quotations : F# 표현식을 프로그래밍적으로 조작할 수 있는 기능.

F#은 아래와 같이 객체 지향 프로그래밍과 닷넷 프레임워크 기능을 지원한다.
- 닷넷 프레임워크 객체 모델 : 프로퍼티, 메소드, 이벤트을 가진 객체들; 다형성 또는 가상 함수들; 상속; 인터페이스;
- 데이터 캡슐화, 혹은 구현으로부터 타입의 공개된 인터페이스의 분리할 수 있다.
- 연산자 오버로딩 : 제네릭과 기탑재된 기본 타입들에도 잘 동작한다.
- 타입 익스텐션 : 새로운 파생 타입을 만드는 추가적인 오버헤드 없이 이미 존재하는 타입을 쉽게 확장가능하다.
- 객체 표현식 : 새로운 타입을 선언하고 객체를 만드는거 대신, 필요할때 표현식으로 암시적으로 작은 객체들을 정의할 수 있다.
- 닷넷 프레임워크와 관리 코드 어셈블리에도 접근할 수 있다.
- 플랫폼 호출을 통해서 네이티브 코드에 접근할 수 있다.

# 왜 F# 인가? (continue)
## 간결함
- F#은 중괄호, 세미콜론 등이 필요없다.
- 강력한 타입 추론 시스템 덕분에 객체 유형을 지정할 필요가 거의 없다.
- C#과 비교하면 동일한 문제를 해결하는 데 필요한 코드 줄이 더 적다.

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

## 편의성
많은 프로그래밍 작업들이 F#으로 더 간단해진다. 예를 들면 복잡한 타입을 정의하고 생성하거나 리스트를 처리하거나, 비교를 하거나 상태 기계를 만드는 등이 있다.

그리고 함수가 일급 객체이기 때문에 매개변수로서 다른 함수를 넘길 수 있는 함수를 만들거나, 새로운 기능을 위해서 이미 존재하는 함수들을 합성함으로서 강력하고 재사용 가능한 코드를 쉽게 만들수 있다.

{% highlight fsharp linenos %}
// automatic equality and comparison
type Person = {First:string; Last:string}
let person1 = {First="john"; Last="Doe"}
let person2 = {First="john"; Last="Doe"}
printfn "Equal? %A"  (person1 = person2)

// easy IDisposable logic with "use" keyword
use reader = new StreamReader(..)

// easy composition of functions
let add2times3 = (+) 2 >> (*) 3
let result = add2times3 5
{% endhighlight %}

## 정확성
NullReferenceException 같은 일반적인 오류들을 막기 위해서 F#은 매우 강력한 타입 시스템을 가지고 있다.

값들은 기본적으로 불변이기 때문에 많은 오류들을 예방할 수 있다.

게다가, 잘못된 코드를 작성하거나 측정 단위를 혼합하는 것을 불가능하게 하는 방법으로 타입 시스템을 사용하여 비즈니스 논리를 표현할 수 있다. 이를 통해 단위 테스트의 필요성이 크게 줄어 든다.

{% highlight fsharp linenos %}
// strict type checking
printfn "print string %s" 123 //compile error

// all values immutable by default
person1.First <- "new name"  //assignment error 

// never have to check for nulls
let makeNewString str = 
   //str can always be appended to safely
   let newString = str + " new!"
   newString

// embed business logic into types
emptyShoppingCart.remove   // compile error!

// units of measure
let distance = 10<m> + 10<ft> // error!
{% endhighlight %}

## 동시성
F#에는 한 번에 두 가지 이상의 일이 발생할 때 도움이 되는 여러 내장 라이브러리가 있다. 또한 F#에는 액터 모델도 내장되어 있으며, 이벤트 처리 및 Functional Reactive Programming(FRP)을 훌륭하게 지원한다.

{% highlight fsharp linenos %}
// easy async logic with "async" keyword
let! result = async {something}

// easy parallelism
Async.Parallel [ for i in 0..40 -> 
      async { return fib(i) } ]

// message queues
MailboxProcessor.Start(fun inbox-> async{
	let! msg = inbox.Receive()
	printfn "message is: %s" msg
	})
{% endhighlight %}

## 완전함
F#은 함수형 언어이지만, 순수하지 않은 스타일도 지원하므로 웹사이트, 데이터베이스, 혹은 다른 응용프로그램 등과 상호작용하기 더 쉽다. 특히, F#은 하이브리드(함수형/객체지향 언어)로 설계되어서 C#으로 할 수 있는 거의 모든 것이 가능하다. (당연하게도) F#이 닷넷 생태계의 일부이므로, 모든 써드 파티 닷넷 라이브러리와 도구들을 사용할 수 있다.

{% highlight fsharp linenos %}
// impure code when needed
let mutable counter = 0

// create C# compatible classes and interfaces
type IEnumerator<'a> = 
    abstract member Current : 'a
    abstract MoveNext : unit -> bool 

// extension methods
type System.Int32 with
    member this.IsEven = this % 2 = 0

let i=20
if i.IsEven then printfn "'%i' is even" i
	
// UI code
open System.Windows.Forms 
let form = new Form(Width= 400, Height = 300, 
   Visible = true, Text = "Hello World") 
form.TopMost <- true
form.Click.Add (fun args-> printfn "clicked!")
form.Show()
{% endhighlight %}

# 결론
F#의 장점과 함께 예제도 간단히 살펴보았다. 하지만 지금으로선 딱히 와닿는 부분이 적다. 앞으로는 구체적인 예제들과 함께 간단한 프로젝트를 진행하면서 F#을 익숙하게 사용해보고자 한다.

# References
- About F# : [http://fsharp.org/about/index.html](http://fsharp.org/about/index.html)
- Why use F# : [http://fsharpforfunandprofit.com/why-use-fsharp/](http://fsharpforfunandprofit.com/why-use-fsharp/)
- Visual F# : [https://msdn.microsoft.com/visualfsharpdocs/conceptual/visual-fsharp](https://msdn.microsoft.com/visualfsharpdocs/conceptual/visual-fsharp)
- The F# Core Engineering Group : [http://fsharp.github.io/](http://fsharp.github.io/)
