---
title: "`limitedParallelism(1)`을 이용한 안전한 작업 큐 만들기"
categories:
  - Concurrency
tags:
  - Concurrency
  - Coroutines
  - Kotlin
toc: true
---

Kotlin Coroutines 1.6부터 도입된 `limitedParallelism`을 사용하면 복잡한 락 없이도 안전한 동시성 제어가 가능합니다.

## 문제: 코루틴 환경의 경쟁 상태(Race Condition)

코틀린 코루틴은 기본적으로 **동시 실행(Concurrency)**을 지원하기 때문에, 여러 코루틴이 공유된 가변 상태를 동시에 수정하면 **경쟁 상태(Race Condition)**가 발생할 수 있습니다.

{% highlight kotlin linenos %}
var counter = 0

suspend fun increment() {
    // 여러 코루틴이 동시에 접근하면 읽기-수정-쓰기 과정에서 데이터 유실 발생
    counter++ 
}
{% endhighlight %}

이를 방지하기 위해 보통 `Mutex`나 `synchronized` 블록 같은 잠금 메커니즘을 사용하지만, 이는 코드 복잡도를 높이고 데드락 같은 잠재적 위험을 수반합니다. 

또한, `newSingleThreadContext`를 사용하여 전용 스레드를 만들 수도 있지만, 이는 스레드 생성 비용이 크고 리소스 관리가 필요하다는 단점이 있습니다.

---

## 해결: limitedParallelism(1)을 이용한 순차 실행

`CoroutineDispatcher.limitedParallelism(1)`은 해당 디스패처가 동시에 실행할 수 있는 코루틴의 수를 1개로 제한합니다. 

즉, **한 번에 하나의 작업만 실행됨을 보장**하므로, 락 없이도 순차적인 실행을 달성할 수 있습니다.

{% highlight kotlin linenos %}
// Default 디스패처의 스레드 풀을 공유하되, 동시 실행은 1개로 제한하는 뷰(View) 생성
val confined = Dispatchers.Default.limitedParallelism(1)
var counter = 0

suspend fun increment() = withContext(confined) {
    counter++ // 항상 순차적으로 실행되므로 Race Condition이 발생하지 않음
}
{% endhighlight %}

### 왜 newSingleThreadContext보다 좋은가?

`newSingleThreadContext`는 실제 운영체제 스레드를 하나 할당하므로 비용이 비싸고, 사용 후 반드시 `close()`해야 합니다. 

반면 `limitedParallelism(1)`은 기존 디스패처(예: `Dispatchers.Default`)의 스레드 풀을 **공유**하면서 동시 실행 수만 제한하므로 훨씬 가볍고 효율적입니다.

---

## 심화: 스레드가 바뀌어도 안전한 이유

공식 문서에 따르면 `limitedParallelism(1)`은 작업이 **항상 같은 스레드에서 실행된다는 것을 보장하지 않습니다.** 

기존 스레드 풀의 남는 스레드를 활용하기 때문에, A 작업은 1번 스레드에서, B 작업은 2번 스레드에서 실행될 수 있습니다.

그렇다면 **서로 다른 스레드에서 실행되는데 메모리 가시성 문제는 없을까요?**

### Happens-Before 관계 보장

정답은 **"안전하다"** 입니다. 코루틴 라이브러리는 내부적으로 작업 간의 **happens-before** 관계를 보장합니다.

`limitedParallelism(1)`은 내부적으로 `LockFreeTaskQueue`와 원자적 연산(Atomic Operation)을 사용하여 작업 순서를 제어합니다.

1. **Release**: 앞선 작업(A)이 끝날 때 원자적으로 "다음 작업 실행 가능" 상태를 기록합니다.
2. **Acquire**: 다음 작업(B)이 시작될 때 이 상태를 확인하고 실행합니다.

이 과정에서 **메모리 장벽(Memory Barrier)**이 동작하여, A 작업에서 변경한 메모리 내용이 B 작업 시작 시점에 반드시 보이도록 강제합니다. 

따라서 물리적인 스레드가 달라져도 데이터의 일관성은 깨지지 않습니다.

---

## 결론

`limitedParallelism(1)`은 단순한 병렬도 제한을 넘어, **락 없는 안전한 작업 큐**를 구현하는 가장 코틀린스러운 방법입니다.

공유 상태를 보호해야 한다면, 복잡한 Mutex 대신 `limitedParallelism(1)`을 먼저 고려해보세요. 가독성과 성능, 안전성을 모두 잡을 수 있습니다.
