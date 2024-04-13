---
title: WebSocket Jmeter 테스트

categories:
- test

tags:
- websocket
- jmeter
- test

toc: true
toc_sticky: true
toc_label: Contents
---

> Websocket을 테스트하는 툴들이 여러 있긴 하지만,
Jmeter의 도입 사례나 레퍼런스가 가장 많고 활발한 것 같아 테스트를 적용해본 내용을 정리하고자 합니다.
> 

> 기본적으로 Websocket 환경과 Jmeter가 설치된 환경이 되어있어야 합니다.
> 

---

## 1️⃣ Jmeter 플러그인 사용하기

웹소켓을 테스트하기 위해선, Jmeter에서 기본적으로 제공하는 sampler로는 부족합니다. 그렇기에 websocket 플러그인을 추가해서 사용했습니다.

플러그인 목록 검색에 websocket을 입력하면 아래와 같은 2가지 항목이 나올텐데, Peter Doornbosch 로 사용하시면 됩니다.

- WebSocket Sampler by Maciej Zaleski
- **WebSocket Samplers by Peter Doornbosch**

## 2️⃣ 기본적인 테스트 설정하기

테스트를 반복적으로 돌리기 위해서 Counter를 설정합니다. 

Counter는 반복적으로 진행하는 테스트 케이스들을 구분하기 위한 일종의 `unique id`로 보시면 됩니다.

!https://imgur.com/QJK3Qw9.png

위 설정에서 *Exported Variable Name*을 지정한대로, Counter 변수를 내부 로직에 넣을 수 있습니다

(ex. WebSocket Open - ${counter}, Send Message ${counter} …)

다음으로 Thread Group을 설정합니다.

Thread Group은 테스트 케이스 그룹이기에, 우리는 이 그룹 안에서 동작하기 위한 로직을 설정하면 됩니다.

!https://imgur.com/tdlQh0M.png

!https://imgur.com/undefined.png

- Number of Threads: 동시에 사용할 사용자(스레드) 수
- Ramp-up period: 사용자 생성마다 걸리는 시간
- Loop Count: 반복 횟수

## 3️⃣ 웹소켓 테스트 설정하기

일부 차이가 있을 수 있지만 stomp 기반의 웹소켓은 아래와 같은 순서로 동작합니다.

1. 웹소켓 Open
2. 소켓 연결
3. 소켓 구독
4. 메시지 전달 및 수신 (반복) 🔄
5. 소켓 구독 해제
6. 소켓 연결 해제
7. 웹소켓 Close

이런 순차적인 과정을 Jmeter에 등록하여 테스트를 진행할 수 있습니다.

### WebSocket Open Connection

Thread Group → Add → Sampler → WebSocket Open Connection

웹소켓을 오픈할 서버, 포트, 경로를 설정합니다.

!https://i.imgur.com/LiZlGo6.png

Name의 ${counter}에는 Counter에서 지정한 Exported Variable Name을 넣습니다. 

Path는 `SockJs`에서 사용되는 경로를 넣습니다. 

(`/{end-point}/${counter}/${__RandomString(8,abcdefghijklnmopqrstuvwxyz,sessionId)}/websocket`)

> SockJs에서 웹소켓은 `{server IP}/{end-point}/{random number}/{random string}/websocket` 와 같은 구조로 지정됩니다. 
그래서 random number에는 counter를 지정하고, random string에는 랜덤한 문자열 8자리를 제너레이팅하는 값으로 넣었습니다.
> 

### Connect

Thread Group → Add → Sampler → WebSocket Write Sampler

웹소켓이 오픈되었으니, 이를 연결하기 위한 일종의 핸드쉐이킹을 *Connect* 부분에서 처리합니다.

!https://imgur.com/5YD531m.png

Name의 ${counter}는 Open할 때와 마찬가지로 unique한 구분값을 두기위한 변수입니다.

신경써야할 부분은, Data 영역입니다.

Data의 Request data 타입을 지정할 수 있는데, WebSocket 전용 플러그인 이다보니 STOMP Text 항목이 있습니다. STOMP 항목을 선택하시면 됩니다. (근데 Text 항목을 선택해도 테스트는 정상적으로 진행 되는 것 같습니다)

Request Data 값은, 기존 SockJS에서 보내는 **CONNECT** 메시지를 넣어줬습니다. 

```
["CONNECT\naccept-version:1.1,1.0\nheart-beat:10000,10000\n\n\u0000"]
```

### Response 받기

Thread Group → Add → Sampler → WebSocket Read Sampler

Connect 부분을 설명할 때, 일종의 핸드쉐이킹이라는 표현을 했는데, 정상적인 통신가능 환경에서 핸드쉐이킹을 하게 되면 통신가능하다는 응답을 받게됩니다.

즉, Connect를 요청해서 정상적으로 통신가능하게 된다면 이에 따른 응답을 수신받을 수 있습니다. 이 응답을 받도록 Sampler을 추가하겠습니다.

!https://imgur.com/vu1pY8M.png

Read Sampler는 별도로 설정할건 없습니다.

### Subscribe

Thread Group → Add → Sampler → WebSocket Write Sampler

Subscribe에서는 메시지 송신과 수신을 받을 일종의 경로를 지정합니다. 번역 그대로 구독하는 과정입니다.

Subscribe는 Connect 부분과 크게 다르지 않습니다. 요청 데이터에만 약간의 차이가 있습니다.

!https://imgur.com/LGnMVRs.png

Request Data를 보시면, **SUBSCRIBE** 메시지가 들어갔고, 전송 구분을 위한 `sub-${counter}` 및 destination 경로를 넣었습니다.

```
["SUBSCRIBE\nid:sub-${counter}\ndestination:{Destination 경로}\n\n\u0000"]
```

이 또한 SockJS에서 사용중인 메시지의 규격을 넣어준 것입니다.

### Send

Thread Group → Add → Sampler → WebSocket Write Sampler

구독까지 완료되었다면 메시지 전송을 확인하면 됩니다. 

!https://imgur.com/W9wxCgj.png

Request data에서는 **SEND** 메시지를 넣고, 발송지인 destination을 설정한 뒤, 전달할 규격에 맞는 데이터를 전달하면 됩니다.

```
["SEND\ndestination:/pub/chat/message\n\n{\"roomNo\":100,\"message\":\"최신영화 추천좀요\"}\u0000"]
```

### Receive

Thread Group → Add → Sampler → WebSocket Read Sampler

메시지 발송했을 때, 정상적으로 수신되는지 테스트하기 위해 Receive를 설정합니다.

앞서 Read Sampler를 작성했었는데, 마찬가지로 동일한 커넥션이면 크게 설정할 부분은 없습니다.

!https://imgur.com/8fJH7wd.png

### Unsubscribe

Thread Group → Add → Sampler → WebSocket Write Sampler

메시지 송수신까지 완료했다면 웹소켓의 커넥션을 끊어줘야합니다. Unsubscribe에서는 커넥션을 끊기 위한 일련의 과정 중 하나인 “구독 해지”를 진행합니다.

!https://imgur.com/EU0aWXf.png

어떤 구독을 끊을지는 Request data에 `sub-${counter}` 에 지정합니다.

이 값은, 동일한 counter에 한에서, subscribe에서 넣은 요청 데이터의 id 입니다.

```
["UNSUBSCRIBE\nid:sub-${counter}\n\n\u0000"]
```

### Disconnect

Thread Group → Add → Sampler → WebSocket Write Sampler

Disconnect는 또다른 과정인 연결을 해제합니다.

!https://imgur.com/705PW1R.png

```
["DISCONNECT\n\n\u0000"]
```

### Connection Close

Thread Group → Add → Sampler → WebSocket Close

최종적으로 웹소켓을 닫습니다. 

!https://imgur.com/vtHu8g0.png

Name을 제외하고는 기본값만을 사용했습니다.

## 4️⃣ 테스트 통계 설정하기

통계는 위와 같은 과정들을 테스트했을 때 어떤 데이터를 보내고 받았으며, 이에 대한 수치적인 정보는 어떤지 확인하기 위해 설정합니다.

### Summary Report

Thread Group → Add → Listener → Summary Report

Summary Report에서는 각 sampler들이 동작했을 때의 정보들을 수치화시켜 통계로 보여줍니다. 

다양한 정보들이 있으니 선별하여 사용자가 적절하게 선별하면 될 것 같습니다.

!https://imgur.com/HoMody7.png

### View Results Tree

Thread Group → Add → Listener → View Results Tree

View Results Tree에서는 내가 Request한 데이터의 정보 뿐 아니라, Response까지 확인할 수 있습니다. 그 외에도 많은 정보를 담고 있습니다.

저는 테스트 실패 시에 디버깅하는 용도로 많이 사용했던거 같습니다.

!https://imgur.com/yEw6kRM.png

### 5️⃣ 기타

웹소켓 커넥션 연결, 구독 뒤에 메시지 송수신만 여러 반복하고 싶다면 Loop Controller로 사용할 수 있고, 메시지 발송시 API 호출한다면 추가 Read Sampler를 활용할 수도 있습니다.
