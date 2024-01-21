---
title: rocky 환경에서 mysql 서버 만들기
categories:
  - server
tags:
  - server
  - mysql
  - rocky
toc: true
toc_sticky: true
toc_label: Contents
---

메인 개발 서버에서 연동할 DB가 필요했고 rocky OS로 구축한 서버에서 테스트로 한번 mysql DB 서버를 구축해보고자 했습니다. ~~근데 생각보다 많이 막혔음;;~~

이번 테스트를 거치고 신규로 띄우는 가상 시스템에서는 본격적인 세팅을 해볼까합니다.  
참고로 이건 내가 혹시 모르니 기록해 놓으려고 하는거라, 간단하게 텍스트 위주로 갈거 같지만  
그래도 누군가에겐 참고할만한 정보가 되었으면 합니다. (진심)

---
## 1. MySQL 세팅
일단 가장 기본적으로 mysql을 설치해야합니다.
```sh
sudo dnf install mysql-server -y
```
> 명령은 굉장히 간단함. 이래서 리눅스 리눅스하는 것 같음ㅎㅎ

그리고 mysql을 활성화.
```sh
sudo systemctl enable --now mysqld
```

오케이. 여기까지 상태 점검
```sh
sudo systemctl status mysqld
```

![](https://i.imgur.com/AezVjrW.png)
- Loaded 항목에 service\:**enabled**  으로 나온다 (preset은 disabled 되어도 무관)
- Active 항목에 **active** 상태로 나온다.

> 여러번 테스트하다, 간혹 mysql 활성화를 시켜도 service 항목이 disabled로 나오는 경우가 있었는데, mysql 경로가 달라서 시스템에서 심볼릭을 못찾고 나오는 상태라고함.. (재설치가 맘편한 듯)

## 2. MySQL 접근 설정
이 접근 설정은 사실 옵션인데, 만약 mysql을 로컬에서만 접근하고 사용하면 위의 설치만하고 다음으로 넘어가도 됩니다. (localhost 혹은 127.0.0.1 주소로 사용하면 되니까)  
다만,  mysql을 DB 서버로 구축하고 외부 서버에서 접근하게 하려면 해당 과정이 **필수**.

저는 마음만 급하게 빨리 구축하고 싶어서 해당 과정을 넘어갔다가 고생했습니다  
mysql8 버전이랑 rocky9는 지금 기준으로 제법 최신버전이라 보안강화가 잘되어서 꼭 해주셔야 정상접근 됩니다

### 2-1. 방화벽 뚫기
저는 이 부분은 생각지도 못했습니다. 다른 OS는 모르겠는데, rocky는 설치하자마자 방화벽을 다 막아논 상태였습니다.  
그래서, 수동으로 사용할 포트를 뚫어줘야 합니다.

```sh
sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp
sudo firewall-cmd --reload
```
db 국룰 포트인 3306 포트를 뚫어 줬고 반영될 수 있게 리로드했습니다.
> rocky의 방화벽은 `firewall-cmd` 로 관리할 수 있는데 생각보다 옵션이 많아서, 이 부분은 한번 더 정리할 생각입니다.  
> 추가로 source 옵션을 통한 같은 IP 대역을 열어두면 더 안전하게 사용할 수 있습니다.

### 2-2. mysql 보안 설정
처음엔 그냥, "어차피 테스트 해보는건데 root 계정으로 해보지 뭐" 라고 생각했는데,  
설정안하면 안되는거 같았습니다. (뭐 다른 이유가 있을 수도 있는데,,,)

```sh
sudo mysql_secure_installation
```
위 명령을 입력하면 MySQL의 기본 보안을 설정할 수 있게 해줍니다.

## 3. mysql 설정

```sh
vi /etc/my.cnf.d/mysql-server.cnf
```

```
bind-address = 0.0.0.0
```

```sh
sudo systemctl restart mysqld
```

위 명령을 통해서 mysql을 접근하는 IP를 설정해놓으면 외부에서 접근가능하게 됩니다.  
기본적으로 127.0.0.1 (localhost)로 설정되어 있습니다. 즉, DB가 설치된 네트워크에서만 DB를 접근가능하도록 되어있습니다.  
`bind-address` 를 통해 열어줍니다.  
**다만, 저렇게 설정하면 모든 외부에서 접근할 수 있게 허용해두는 것이니 방화벽을 꼭 설정하거나 접근할 IP를 지정하게 해주는게 좋습니다.**

---

이렇게까지 하면 구축한 DB를 외부에서 접근할 수 있게 됩니다.  

> 만약, 이렇게 했는데도 외부에서 접근 이슈가 있다면 해당 서버에서 신규 계정을 생성하여 권한을 줘야합니다.  
> 그 이유는, rocky OS 9.xx 기준으로 외부에서 root 계정을 접근하는 것을 기본적으로 막아두었기 때문에 별도의 설정이 필요하나, 하위 계정을 생성해서 사용하면 해결됩니다.

