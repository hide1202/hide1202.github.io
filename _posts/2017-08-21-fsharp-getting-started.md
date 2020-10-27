---
title: "F# beginning : (2) F# 시작하기"
categories:
  - Fsharp
tags:
  - Fsharp
install_ionide:
  - url: /assets/images/posts/fsharp-getting-started/install-ionide.png
    image_path: /assets/images/posts/fsharp-getting-started/install-ionide.png
    alt: "Install-Ionide"
fsi_command:
  - url: /assets/images/posts/fsharp-getting-started/fsi-command.png
    image_path: /assets/images/posts/fsharp-getting-started/fsi-command.png
    alt: "FSI-Command"
fsi_result:
  - url: /assets/images/posts/fsharp-getting-started/fsi-result.png
    image_path: /assets/images/posts/fsharp-getting-started/fsi-result.png
    alt: "FSI-Result"
toc: true
---

# 차례
1. [Visual Studio Code 설치하기](./#1-visual-studio-code-설치하기)
2. [Ionide 설치하기](./#2-ionide-설치하기)
3. [Hello, World!](./#3-hello-world)

# 1. Visual Studio Code 설치하기
F#을 실습하기 위해선 IDE 혹은 에디터가 필요하다. Visual Studio(+for Mac), Monodevelop, Atom 등 다양한 IDE 혹은 에디터를 사용할 수 있지만, 여기선 `Visual Studio Code`를 이용해서 실습을 해볼 예정이다.

Visual Studio Code는 아래 사이트를 통해서 쉽게 다운로드 받아서 설치할 수 있다. 플랫폼별로 설치파일을 제공하고 있기 때문에 손쉽게 다운로드 받을 수 있다.

[Visual Studio Code 공식 사이트](https://code.visualstudio.com/)

> 2017/08/21 기준으로 macOS의 경우, Zip 파일을 제공한다. 압축을 해제해서 파인더의 애플리케이션 폴더로 이동하면 런치패드에서 실행할 수 있다.

# 2. Ionide 설치하기
Visual Studio Code를 설치했다고 해서 F#을 곧바로 사용할 수 있는 것은 아니다. F#을 사용하기 위해서는 `Ionide` 확장을 설치해야 한다.

{% include gallery id="install_ionide" %}

설치가 완료되면 다시 로드 가 활성화 된다. 다시 로드 를 눌러서 VS Code를 재시작 하자.

# 3. Hello, World!
이제 F#을 사용할 준비가 완료되었다. `HelloWorld.fs` 파일을 하나 만들어보자.

만들어진 파일에 다음과 같이 코드를 작성하자.

{% highlight fsharp linenos %}
let prefix prefixStr baseStr =
    prefixStr + ", " + baseStr

printfn "%s" (prefix "Hello" "World")
{% endhighlight %}

그리고 Command + Shift + P (윈도우는 Ctrl + Shift + P) 를 눌러서 커맨드 창을 띄우자. `fsi` (F# interactive) 를 입력하면 다음과 같은 명령들을 볼 수 있다.

{% include gallery id="fsi_command" %}

명령 중에 `Send File` 을 선택하면 FSI로 파일에 입력된 코드를 보내게 된다. 그리고 아래와 같은 결과값이 터미널을 통해서 표시된다.

{% include gallery id="fsi_result" %}

# 결론
F# 실습에 필요한 Visual Studio Code와 Ionide 확장을 설치해보고, 간단한 함수를 통해 Hello, World를 출력해보았다. 컴파일을 통해서 프로그램 실행도 가능하지만, 당분간은 F# Interactive 을 통해서 실습을 진행할 예정이다. 다음 시간에는 다른 간단한 예제를 통해서 F#의 기본 문법에 대해서 조금 더 자세히 알아보도록 하자.