---
title: JDK 쉽게 설정하는 법 (for mac)

categories:
  - settings

tags:
  - settings

toc: true
toc_sticky: true
---

jdk를 설치하는 방법은 다양하다. oracle사에서 제공하는 jdk을 설치하거나, 마이크로소프트사에서 제공하는 jdk를 설치하거나 등등..
하지만 특정 플랫폼(여기서는 linux환경)에서 제공하는 sdk 매니저를 통해 쉽게 그 버전을 선택하여 사용할 수 있는 방법도 있다.

여기서는 나는 **[sdk man](https://sdkman.io/)** 이라는 소프트웨어 개발 킷을 사용했다.
위의 링크에 접속하면 임팩트 있어보이는 첫 화면이 나오는데,
![](https://i.imgur.com/umJMYPq.png)

이후 오른쪽 상단의 install을 클릭해, 쉽게 설치하고 확인까지 하는 과정을 볼 수 있다.

sdkman을 먼저 설치한다.
```bash
$ curl -s "https://get.sdkman.io" | bash
```

이후, 환경 변수에 이 sdkman을 등록한다.
```bash
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
```

마지막으로, 잘 설치가 되었는지 확인하면 된다.
```bash
$ sdk version
```

![](https://i.imgur.com/BWNEyNq.png)

---
본래 목적은 jdk를 설치하는 것이었으므로, 제공하는 jdk 목록을 확인하기 위해서 아래의 명령을 이용할 수 있다.
```bash
$ sdk list java
```

그러면, 다양한 벤더사에서 제공하는 jdk 및 그 버전들을 확인할 수 있는데, 원하는 jdk의 identifier를 복사하여 사용하면 된다.
```bash
$ sdk use java 11.0.19-librca
```

그리고, 다시 한번 `sdk list java` 로 조회했을 때, **use** 라는 마킹이 된 것을 확인하면 끝이다.
하지만, 기존의 있던 default 설정이 있는경우, `sdk update`를 추가적으로 입력해야 새로운 jdk 버전으로 유지될 수 있다.
→ **이로써 다양한 jdk 버전을 유연하게 사용할 수 있다.**


