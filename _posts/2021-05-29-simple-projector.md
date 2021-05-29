---
title: "Projector를 이용해 원격으로 IntelliJ 실행해보기"
categories:
  - Tools
tags:
  - Tools
  - IntelliJ
create_vm:
  - url: /assets/images/posts/simple-projector/create-azure-vm.png
    image_path: /assets/images/posts/simple-projector/create-azure-vm.png
    alt: "Create-Azure-VM"
run_projector:
  - url: /assets/images/posts/simple-projector/run-projector.png
    image_path: /assets/images/posts/simple-projector/run-projector.png
    alt: "Run-Projector"
toc: true
---

# Projector ?!
Projector 는 Swing GUI 어플리케이션을 원격으로 실행해주는 기술로, Jetbrains 에서 개발했으며 현재 오픈소스로 관리되고 있다. 
이를 이용해 Swing GUI 로 개발된 IntelliJ 플랫폼을 원격으로 실행할 수 있고, 현재 대부분의 Jetbrains IDE 들을 실행해볼 수 있다.

원격 실행은 웹에서 할 수 있고, Electron 기반의 Projector-Client 를 이용해서도 가능하다.

# 그럼 한번 해보자!
한번 구축해놓으면 언제나 어디서나 이용할 수 있고, 무엇보다 아이패드에서도 개발환경을 어느정도 구축할 수 있을 것으로 기대된다.

복잡한 개발은 좀 힘들지도 모르겠지만, 알고리즘 문제를 풀거나 간단한 테스트를 할 때 유용하게 사용할 수 있다.

## Azure VM 생성
집에 남는 PC가 있다면 구축해서 테스트해볼 수 있겠지만, 집에 남는 PC가 없기도 하고 어디서나 원격으로 접속할 수 있게 Azure VM 위에 올려볼까 한다.

Azure 사용과 관련된 내용은 이 글에서 다루지 않으며, 다른 클라우드나 여분의 PC를 이용해도 된다.

현재 Projector-Server 는 Linux 만을 지원하지만, Docker 를 이용하면 어디든 설치할 수 있다. (테스트는 우분투 기준으로 진행했다)

그럼 먼저 Azure VM 을 생성해주자. 필자는 아래와 같이 생성했다.

{% include gallery id="create_vm" %}

## Docker 설치
빠른 설치 및 확인을 위해 Projector-Docker 기준으로 설치할 예정이다. Docker 를 빠르게 설치해보자.

Docker 는 [공식 홈페이지에 설치법](https://docs.docker.com/engine/install/)이 아주 자세히 적혀있으므로 참고하자!

## Projector-Docker 받기
[이 곳](https://github.com/JetBrains/projector-docker#run-jetbrains-ide-in-docker)을 참고해서 설치를 원하는 IDE 를 선택해서 pull 받도록 하자.

이 글에서는 IntelliJ 설치 위주의 글이므로 아래와 같이 명령어를 실행했다.

```
docker pull registry.jetbrains.team/p/prj/containers/projector-idea-c
```

## 네트워크 포트 설정
Projector 는 기본적으로 8887 포트를 사용한다. Azure VM 에서는 (당연하게도) 해당 포트가 열려있지 않기 때문에 인바운드 규칙에 8887 을 열어줘야 한다.

(주의) 테스트를 위해 public 으로 열여서 확인한다. Projector 를 기본 설치할 경우 별다른 보안 설정이 되어있지 않으므로 보안에 유의해야한다.
설치/실행을 위한 테스트 목적이라면 public 으로 열어서 확인만 해보고, 실사용시에는 LAN 환경(혹은 VPN을 구축해서 사용)을 이용하거나 token 파라미터 등을 이용하는게 좋다.

## 이미지 실행
아래 커맨드를 통해서 Docker 이미지를 실행한다. 가장 기본적인 명령어를 사용했는데, 기타 파라미터는 목적에 맞게 사용하면 된다.

(참고로 mount 설정을 안했기 때문에 서버가 내려가면 코드들이 그대로 날아간다. 실사용 목적이라면 최소 mount 설정은 하고 실행하자)

```
sudo docker run --rm -p 8887:8887 -it registry.jetbrains.team/p/prj/containers/projector-idea-c
```

## 실행
이제 웹브라우저를 통해 구축한 해당 IP 로 접근해보자. 웹브라우저에서 실행되는 IntelliJ 를 볼 수 있다!!

{% include gallery id="run_projector" %}

## (참고)
접근하는 모든 클라이언트는 동일한 화면을 보게된다. 즉, 다수의 사람이 읽고 쓸 수 있게 된다는 의미다.

추가로 쓰기 권한을 가진 사람이 보고 있는 화면에 따라 윈도우 크기가 변경된다. (빈 곳은 초록색으로 채워진다)

# 결론
가볍게 설치해보고 잠깐 실행만 해봤는데, 실사용을 위해선 보안을 어떻게 해야할지 고민해봐야한다.

앞에서 언급했지만 접근시 모든 사용자가 동일한 화면을 보고 쓸 수 있게 된다. 따라서, Public IP 로 접근가능하게 하고 토큰 등을 활용하는 것보다는 네트워크를 격리하는게 더 옳은 방법이다.

어떻게 활용할지 아직은 미지수인데, 아이패드에서 조금씩 써볼 생각이다. ~그러려면 얼른 매직 키보드를 사야...~

# References
- Projector-Server : [https://github.com/JetBrains/projector-server](https://github.com/JetBrains/projector-server)
- Projector-Client : [https://github.com/JetBrains/projector-client](https://github.com/JetBrains/projector-client)
- Projector-Docker : [https://github.com/JetBrains/projector-docker](https://github.com/JetBrains/projector-docker)

