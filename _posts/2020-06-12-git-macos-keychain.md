---
title: "SourceTree(macOS) 에서 Github 비밀번호를 계속 입력해야할 때"
categories:
  - Tips
tags:
  - Tips
---

## 문제
Mac OS 에서 SourceTree 를 잘 이용하고 있다. 최근에 안드로이드 공부를 위해서 private repository 를 만들어서 이것저것 해보고 있는데, SourceTree 에서 fetch/pull/push 등을 하려고 하니 계속 Github 비밀번호를 입력해야하는 불편함이 있다.

키체인에 기억하도록 체크했는데도 불구하고 계속 입력하라고 나온다. push 하는게 귀찮아질 정도다.

## 해결
구글링을 열심히 해보았다. `git config` 를 이용해서 키체인을 활용하도록 설정하면 해결된다고 한다.

아래 커맨드를 입력하면 이제 remote 에 무언가를 하려고 하면 로그인 비밀번호(macOS 진입시 입력하는 비밀번호)를 입력하라고 나온다.

이제 편하게 fetch/pull/push 등을 할 수 있게 되었다!!!

```bash
git config --global credential.helper osxkeychain
```
