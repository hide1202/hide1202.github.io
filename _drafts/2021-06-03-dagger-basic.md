---
title: "Dagger2 기초: 의존에 대하여"
categories:
  - Android
tags:
  - Android
  - Dagger2
toc: true
---

## 개체의 의존성
우리는 개체 지향 프로그래밍을 할 때 필연적으로 개체에 의존을 하게 된다. 예를 들면 아래와 같은 코드를 보자.

{% highlight kotlin linenos %}
class CentralProcessingUnit {
  private val memory: Memory = DramMemory(GigaBytes(4))

  fun process(instruction: Instruction) : Result {
    val data = memory.fetchData()
    return execute(instruction, data)
  }
}

fun main() {
  val cpu = CentralProcessingUnit()
  println(cpu.process(Instructions.Add(1, 1)))
}
{% endhighlight %}

`CentralProcessingUnit`(이하 CPU) 라는 개체는 `Memory` 에 의존해서 명령어를 수행한다. 물론 개체 자체만으로 무언가할 수는 있겠지만, 보통은 이렇게 여러 개체들의 협력으로 우리 프로그램은 수행된다.

갑자기 컴퓨터를 업그레이드하고 싶어졌다. 메모리를 좀 더 올려보도록 하자.

### 의존성 주입
위 코드로는 도저히 메모리를 교체할 수가 없다. 하고 싶다면 CPU 를 분해해서 메모리를 교체해아하는데, 불가능에 가까울 것 같다. 이런 CPU는 굉장히 확장성이 떨어진다. 메모리를 올리고 싶어도 올리기 매우 어렵고 까다롭기 때문이다.

그렇다면 이제 아래 코드를 보자.

{% highlight kotlin linenos %}
class CentralProcessingUnit(
  private val memory: Memory
) {
  fun process(instruction: Instruction) : Result {
    val data = memory.fetchData()
    return execute(instruction, data)
  }
}

fun main() {
  val memory = DramMemory(GigaBytes(8))
  val cpu = CentralProcessingUnit(memory)
  println(cpu.process(Instructions.Add(1, 1)))
}
{% endhighlight %}

CPU가 메모리 개체를 받아들이게 함으로서, 메모리를 확장할 수 있게되었다! 

CPU는 자신이 하는 일에 영향을 받지 않으면서, 좀 더 크거나 혹은 더 빠른 메모리를 사용할 수 있다.

위 코드는 동일한 개체간 의존성이라도 어떻게 구현하느냐에 따라서 확장성있게 만들 수 있느냐를 보여주는 대표적인 예이다.

그리고 후자를 `외부에서 의존성을 받는다` 라고 하여 _**`의존성 주입`**_ 이라고 표현한다.

위키피디아의 `Dependency injection` 정의도 아래에 첨부했다.

> In software engineering, `dependency injection` is a technique in which an object receives other objects that it depends on. These other objects are called dependencies.

참고로 첫 번째 코드처럼 의존이 개체의 라이프사이클과 동일(생성과 함께 의존도 생성되고, 파괴될 때 함께 파괴된다)한 것을 `구성(Composition)` 이라고 한다.

**그렇다고해서 구성 이 나쁜 것은 아니다.** 

캡슐화를 위해 의존 개체를 개체 안으로 얼마든지 숨길 수 있다. 무조건적으로 의존성 주입을 사용하는게 아니라, 확장성이 필요한 의존을 선택적으로 주입받는게 중요하다.

## Dependency Injection Framework
위 예제 코드처럼 개체를 확장성있게 만들었다고 하더라도, 의존 변경시 번거로운건 여전하다. 개체 밖에선 여전히 어떤 개체를 의존으로 줄지 결정하고 생성해야 하기 때문이다.

코드가 복잡해지면 관리도 잘 안되고 실수를 할 여지가 많아진다. 그럼 확장성 있게 만들 수 있다는걸 알더라도 다소 복잡해진(혹은 귀찮)다는 이유로 의존을 내부로 숨기게 된다.

인간은 도구를 사용하는 동물이라고 했던가. 다양한 프레임워크(혹은 라이브러리)가 의존성 주입을 도와주고 있고, 다음 포스팅부터 그 중 안드로이드 환경에서 많이 쓰고 있는 Dagger2 에 대해서 알아볼까 한다.

(안드로이드 환경에서 Dagger2 를 더 편하게 사용할 수 있는 `Hilt` 도 있다. 이에 대해선 나아중에 알아보도록 하자)

## 참조
- Dagger2: [https://dagger.dev/dev-guide/](https://dagger.dev/dev-guide/)
- Dependency injection(Wikipedia): [https://en.wikipedia.org/wiki/Dependency_injection](https://en.wikipedia.org/wiki/Dependency_injection)