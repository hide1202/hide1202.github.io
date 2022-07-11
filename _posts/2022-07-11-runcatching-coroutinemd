---
title: "Coroutines 에서 runCatching 사용을 하지 말아야 하는 이유"
categories:
  - Kotlin
  - Coroutines
tags:
  - Kotlin
  - Coroutines
toc: true
---

## 예외 처리
코루틴을 적용하면 예외 처리가 까다로워집니다. 특히 코틀린은 Checked Exception 이 존재하지 않기 때문에 이를 강제할 수가 없습니다.

하지만 매번 `try...catch` 를 감싸는건 너무 비효율적이고 미관상 좋지도 못합니다.

여기서 사용할 수 있는 대안은 `runCatching` 입니다.

## runCatching 을 이용한 예외 처리
`runCatching` 을 이용하면 `Result<T>` 로 결과값이 나오고 아래 코드와 같이 성공/실패 처리를 할 수 있습니다.

이는 코드를 읽는 사람에게 이 메소드는 예외를 반환할 수도 있다는 것을 암시해줍니다.

{% highlight kotlin linenos %}
suspend fun <T> request(message: String): Result<ByteArray> {
    return runCatching {
        doSomething()
    }
}

suspend fun handleResponse() {
    request(message = "body")
            .onSuccess {
                handleByteArray()
            }
            .onFailure {
                handleException(it)
            }
}
{% endhighlight %}

하지만 이는 예상치 못한 결과를 초래합니다. 왜냐하면 코루틴의 취소는 예외로 전파되기 때문입니다.

## 코루틴 그리고 취소
코루틴은 스코프가 존재하고 취소될 수 있습니다. 이는 다양한 이점이 존재합니다.

특히 안드로이드처럼 라이프사이클이 존재하는 경우, 라이프사이클에 따라서 모든 작업을 일괄 취소할 수 있습니다.

코루틴의 취소는 `CancellationException` 를 전파하면서 발생합니다. 다시 말해, 이를 잘못 제어하면 취소가 되지 않을 수도 있다는걸 의미합니다.

앞서 살펴본 `runCatching` 은 모든 `Throwable` 을 잡습니다. 따라서 `CancellationException` 도 잡히게 되는데요.

보통 suspend 함수는 `CancellationException` 이 발생하면 스코프에 이를 던지고 취소되어져야합니다.

다음은 `runCatching` 을 일부 수정한 코드입니다.

{% highlight kotlin linenos %}
suspend inline fun <T, R> T.resultOf(crossinline block: suspend T.() -> R): Result<R> {
    return try {
        Result.success(block())
    } catch (e: CancellationException) {
        throw e
    } catch (t: Throwable) {
        Result.failure(t)
    }
}
{% endhighlight %}

정확한 취소 처리를 위해서는 위 처럼 `CancellationException` 발생시 곧바로 다시 던져야 합니다.

## 대표적인 잘못된 예시
대표적으로 예외를 받았을 때 에러 메시지를 토스트로 띄운다고 생각해봅시다. `runCatching` 로 이를 구현하면 아래 코드와 같습니다.

{% highlight kotlin linenos %}
suspend fun handleResponse(context: Context) {
    request(message = "body")
            .onSuccess {
                handleByteArray()
            }
            .onFailure {
                Toast.makeText(context, "에러가 발생했습니다!", Toast.LENGTH_LONG).show()
            }
}
{% endhighlight %}

보면 딱히 문제가 없어보이는 코드입니다. 하지만 마침 스코프가 종료되서 작업이 취소되었다고 생각해봅시다.

그럼 우리는 토스트가 뜨지 않기를 기대합니다. 왜냐하면 스코프가 종료되었기 때문입니다. 하지만 우리의 예상과는 다르게 토스트가 노출됩니다.

왜냐하면 `onFailure` 로 `CancellationException` 가 전파되었기 때문입니다. 하지만 여느 예외처럼 동일한 처리를 했기 때문에 우리는 알 수 없는 문제에 봉착하고 맙니다.
(저 같은 경우 취소에 대해 고려하지 못해서 이와 유사한 이슈를 맞게되었습니다. 다행히 다른 분의 도움으로 예상보다는 빠르게 문제를 해결할 수 있었습니다.)

따라서, 코루틴 처리시 취소는 매우 중요한 문제입니다. 

항상 코루틴은 취소될 수 있고, 이는 `CancellationException` 으로 전파될 수 있다는걸 잊어선 안됩니다.

## 참조
- https://www.droidcon.com/2022/04/06/resilient-use-cases-with-kotlin-result-coroutines-and-annotations