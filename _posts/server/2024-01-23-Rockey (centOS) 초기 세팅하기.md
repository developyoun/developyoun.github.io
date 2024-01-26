---
title: Rockey (centOS) 초기 세팅하기
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

서버 접속은 로컬에서 접근하여 사용해도 되지만, 내가 사용중인 `ESXi 호스트 클라이언트` 에서 제공하는 터미널은 너무 불편합니다..  
일단 위아래 스크롤이 안될 뿐아니라, 버그인지 모르겠는데 브라우저 탭을 여러개 사용하면서 터미널을 사용하면 창이 점점 작아지는 현상이 있었습니다.  
그래서 맥에서 사용중인 Iterm2을 이용해 SSH로 접근해 사용하기 위해 몇가지 초기 설정을 하려고 합니다.

## 1. 계정 만들기
가장 먼저 Rocky를 시작하면 기본적으로 사용가능한 계정인 `root`이 있습니다. 근데 이 계정은 외부에서 사용할 수 없습니다.  
만약 접속해본다면 `Permission denied` 가 발생해요.
![](https://i.imgur.com/MVf5SFd.png)
그러면 이 `root` 계정의 권한을 가지고 새로 계정을 생성해서 사용하도록 해야합니다.
### 1. 계정 리스트 조회
```sh
sudo cat /etc/passwd
```
passwd 파일에는 `root` 계정을 포함한 모든 계정의 정보가 들어가 있습니다.  
신규 계정을 생성하면 조회된 계정리스트에 추가되어 보여집니다.

### 2. 계정 생성
```sh
sudo userradd {계정명}
```
`useradd` 명령을 이용해서 계정을 생성할 수 있습니다. (매우 간단)

만약 계정명을 잘못지어서 삭제해야한다? 그럼 아래와 같은 명령을 이용해서 삭제할 수 있습니다.
```sh
sudo userdel {계정명}
```

계정 삭제 후 같은 계정명으로 재생성하려고 하면 어딘가에서 이미 존재하여 사용할 수 없다고 나타날 수 있습니다.
![](https://i.imgur.com/Kam7zkl.png)

이럴 경우에, `-r` 옵션을 붙여서 해당 계정을 삭제 후 재생성하면 됩니당.  
이 옵션은 remove 옵션인데, home 디렉토리와 mail spool을 모두 제거해줍니다.
```sh
# 계정 삭제
sudo userdel -r {계정명}

# 계정 생성
sudo useradd {계정명}
```

### 3. 계정 비밀번호 설정
보안 때문이라도 일반 계정을 비밀번호 설정없이 사용할 순 없겠죠? 그래서 비밀번호 설정을 해줍시다.
```sh
sudo passwd {계정명}
```

비밀번호와 비밀번호 확인까지해서 입력해주면 쉽게 계정을 생성할 수 있습니다.

### 4. 신규 계정에 sudo 권한 주기
신규로 생성한 계정은 아직 완전하지 않습니다. 계정을 생성만 했지 권한은 없습니다.  
그렇기에 신규 계정에 sudo 명령을 사용할 수 있는 명령을 줘야합니다.

권한을 주기위한 sudo 설정 파일을 수정해야합니다
```sh
sudo vi /etc/sudoers
```

설정안에서 아래와 같은 설정 정보를 찾습니다.
![](https://i.imgur.com/iMtxkYx.png)
root 권한에 ALL 설정이 되어있는 부분처럼 새로 생성한 계정을 추가합니다
```
root       ALL=(ALL)    ALL
신규 계정명   ALL=(ALL)    ALL
```

이렇게 권한까지 준다면 외부에서 접속하여 sudo를 통해 서버를 제어할 수 있습니다.

> [!tip]
> 만약 root 계정으로 외부에서 접근하고 싶다면 정말 비추하지만 아래와 같이 설정하면 됩니다.
> 1. /etc/ssh/ssh_config 파일을 편집기로 엽니다.
> 2. 주석 처리되어있는 항목중에서 PermitRootLogin 값을 찾습니다. 기본적으로 해당 값은 no로 되어있기 때문에 주석처리 되어있습니다
> 3. 주석을 해제하고 prohibit-password (비밀번호 로그인은 막고, key파일 로그인 허용)을 없애고 yes로 설정하여 저장하면 됩니다.

## 2. SSH 포트 번호 변경하기
이 부분은 옵션입니다만, 대표 SSH 포트인 22번 포트가 만만하다는건 많은 일반인들도 아시는 사실이니 포트 번호를 내맘대로 바꿔서, 무차별적으로 접근하려는 경우의 수를 줄여볼까 합니다.

### 1. 현재 SSH 포트 확인
```sh
sudo netstat -tlnp | grep sshd
```

![](https://i.imgur.com/Y77CuUL.png)
tcp항목을 보면, 현재 22번 포트로 접근을 허용하고 있다.

> 만약 netstat 명령을 찾지 못한다면 net-tools를 설치해줘야 한다.
> `sudo dnf install net-tools`

### 2. SSH 설정 파일 수정
```sh
sudo vi /etc/ssh/sshd_config
```

설정에서 주석 처리된 `Port 22` 항목을 찾아 원하는 포트번호로 변경하여 수정 및 저장하면 됩니다.
> [!tip]
> ssh 디렉토리에는 ssh_config와 sshd_config 두 가지의 config 파일이 존재하는데 정의 는 아래와 같다.
> - ssh_config : 나가는 설정 (outbound)  
> - sshd_config : 들어오는 설정 (inbound)

### 3. SSHD 서비스 재시작
변경된 포트 번호를 반영하기 위해선 SSHD 서비스를 재시작해야 합니다.  
```sh
sudo service sshd restart
# 혹은
sudo systemctl restart sshd
```

근데 SELinux (Security Enhanced Linux) 정책으로 인해 SSHD 서비스를 재시작하면 에러가 발생합니다.
![](https://i.imgur.com/b7FadBm.png)

이를 해결하기 위해, 먼저 Semanage를 설치합니다.  
설치할 패키지를 찾기 위해 패키지를 찾아줍니다.
```sh
sudo dnf provides */semanage
```

![](https://i.imgur.com/7UMf8eh.png)
여기선 패키지명이 `policycoreutils-python-utils`입니다.
```sh
sudo dnf install policycoreutils-python-utils
```

그리고 아래 명령으로 SELinux에게 해당 포트를 정의합니다.
```sh
sudo semanage port -a -t ssh_port_t -p tcp {정의한 포트 번호}
```
- -a: add 추가한다는 옵션 (-d는 삭제)
- -t: 타입 정의 (다양한 타입이 있지만 ssh 관련은 ssh_port_t로 사용)
- -p: 포트 번호 (위 명령은 tcp + 포트 번호 정의)

정의한 후, 서비스를 다시 정의한 후 조회해보면 잘 설정됨을 볼 수 있습니다.  
```sh
sudo service sshd restart
sudo netstat -tlnp | grep sshd
```

### 4. 방화벽 설정
기본적으로 22번 포트는 모든 접근을 허용합니다. 하지만 커스텀한 포트는 기본적으로 방화벽으로 막혀있습니다. 그렇기 때문에 해당 포트의 방화벽을 해제 해주어야 합니다.

> 방화벽 관련 설정은 추가적으로 작성할 예정이니 여기선 세팅을 중심으로 해서 절차대로 진행하겠습니다

```sh
sudo firewall-cmd --list-all
```
해당 명령으로 현재 설정된 방화벽 세팅을 확인할 수 있습니다.

```sh
sudo firewall-cmd --permanent --add-port={포트번호}/tcp
```
그리고 방화벽에 포트를 추가하여 줍니다.

```sh
sudo firewall-cmd --reload
```
그리고 반영될 수 있게 리로드를 진행해주시면 처음 확인했던 `sudo firewall-cmd --list-all` 명령을 통해 포트번호가 추가됨을 확인할 수 있습니다.

그리고 이후부턴, 외부 접속도 본인이 설정한 포트로 SSH 접근 해야합니다.

---

Rocky가 나름 최신 OS이기도 해서 보안에 굉장히 힘을 준것 같습니다. 기본적으로 설정되어있는 보안이 강도가 있는 편이고, 보안관련된 패키지도 미리 설치된 경우가 많았습니다.  
다만 아직 설정할게 많으니 이번 설정 이후에도 틈틈히 정리해두어야 될 거 같습니다.