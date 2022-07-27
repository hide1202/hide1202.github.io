---
title: "번호 매기기 목록에서 들여쓰기를 표현해보자!"
categories:
  - Android
  - UI
tags:
  - Android
  - UI
toc: true
---

## 열거식 문장
안드로이드로 문자열을 표현하다보면 가끔 열거식으로 문장들을 표현해야할 일이 있습니다.

그 중에서도 번호 매기기 목록(Numbered list) 형태로 문장을 나열해야할 때도 있는데요. 

```
1. 첫 번째 요소입니다.
2. 두 번째 요소입니다.
3. 세 번째 요소입니다.
```

단순히 한 줄로된 문장들을 표현하는걸 넘어서 꽤 긴 문장들을 표현할 때는 어떻게 해야할까요?

## 들여쓰기
우리는 긴 문장을 작성할 때 들여쓰기를 활용합니다. 보통은 문단의 첫 번째 줄을 들여쓰기하죠. 하지만 번호 매기기 목록이라면 어떨까요?

이 경우 우리는 아래 글과 같이 번호가 표현되는 부분은 제외하고 두 번째 줄부터 들여쓰기가 되기를 기대합니다.

```
1. 첫 번째 요소인데, 이건 생각보
   다 긴 문장입니다.
2. 두 번째 요소 또한, 생각보다는
   긴 문장이네요.
3. 세 번째 요소도 당연히 길었으면
   하는 문장입니다.
```

그렇다면 이걸 안드로이드에서는 어떻게 표현할까요?

## LeadingMarginSpan
최근에 이런 요구사항이 생겼고, 이를 해결하기 위해 `LeadingMarginSpan.Standard` 를 활용했습니다.

{% highlight kotlin linenos %}
private fun generateMessage(): CharSequence {
    val leadingMargin = 14.dp // 글자 크기와 동일한 크기라고 가정합니다.
    return sequenceOf(
            R.string.help_1,
            R.string.help_2,
            R.string.help_3,
    )
            .map {
                SpannableString(getString(it))
                        .apply {
                            val span = LeadingMarginSpan.Standard(/* first */ 0, /* rest */ leadingMargin)
                            setSpan(span, 0, length, 0)
                        }
            }
            .joinTo(SpannableStringBuilder(), "\n\n", transform = { it })
}
{% endhighlight %}

위와 같은 코드를 작성하게 되었고, 코드의 의미는 매우 직관적입니다.

1. 도움말인 세 문장을 이용해서 하나의 문장을 생성합니다.
2. 이 때, 각 문장에 `LeadingMarginSpan.Standard` 인 Span 을 설정합니다.
3. `LeadingMarginSpan.Standard` 의 매개변수의 의미는 아래와 같습니다.
	1. 첫 번째 매개변수는 첫 줄의 들여쓰기 픽셀 크기입니다. 첫 줄은 들여쓰기를 하지 않습니다.
	2. 두 번째 매개변수는 나머지 들여쓰기 픽셀 크기입니다. 숫자가 포함되지 않는 두 번째 줄부터 모두 들여쓰기를 합니다.
	3. 들여쓰기의 픽셀 크기는 글자 크기와 동일하게 설정합니다.
4. 각 `SpannableString` 를 line feed 문자를 separator 로 설정해서 병합합니다.

여기서 `LeadingMarginSpan.Standard` 를 `setSpan` 할 때 길이를 꼭 length 만큼 하지 않아도 원하는 결과를 얻을 수도 있는데요. `TextView` 의 `movementMethod` 에 `ScrollingMovementMethod` 가 설정되어 있다면, 스크롤하는 중에 들여쓰기가 사라질 수도 있습니다.

## 결론
문자열을 사용하게 되는 경우, 특수한 요구사항이 아니라면 대부분 Span 으로 요구사항을 만족할 수 있습니다.

문자열 표현과 관련된 요구사항이 나온다면 꼭 각종 Span 들을 검색해보면 왠만한 요구사항들은 해결할 수 있을 거라고 생각합니다.