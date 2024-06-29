---
title: NPM(Nginx Proxy Manager) SSL 설정 이슈
categories:
  - server
tags:
  - npm
  - ssl
  - cloudflare
toc: true
toc_sticky: true
toc_label: Contents
---


도커에서 Nginx Proxy Manager(이하 NPM) 이미지를 재빌드하면서 클라우드플레어에 SSL을 다시 등록해야하는 상황이 있었는데, 갑자기 아래와 같은 이슈가 발생했습니다.

```
CommandError: The 'certbot dns cloudflare._internal.dns cloudflare' plugin errored while loading: No module named 'CloudFlare'. 
You may need to remove or update this plugin. 
The Certbot log will contain the full error details and this should be reported to the plugin developer.
```


기존에는 lastest 버전을 사용했기에 버전 문제인 줄 알았는데, 그건 또 아니었습니다.

2.11.2 → 2.11.1 → 2.11.0 으로 순차적으로 리빌드 했는데도 문제가 사라지지 않았습니다.

우연치 않게, `reddit` 에서 비슷한 증상을 겪는 게시글이 보였고 이를 참고해서 해결하였습니다.

1️⃣. 도커로 올라간 컨테이너에 접속한다.

`docker exec -it <컨테이너 명> /bin/bash`  

2️⃣. `certbot`을 입력해서 Cloudflare-DNS 플러그인에서 이슈가 발생하는지 확인한다.

이슈가 발생한다면 아래와 같은 텍스트가 포함되어 출력된다.

```
The 'certbot_dns_cloudflare._internal.dns_cloudflare' plugin errored while loading: No module named 'CloudFlare'. 
You may need to remove or update this plugin. The Certbot log will contain the full error details and this should be reported to the plugin developer.
```

만약 보였다면 아래 순서를 이어서 진행하면 된다.  


3️⃣. `pip uninstall certbot-dns-cloudflare` 명령을 입력하여 `certbot-dns-cloudflare` 패키지를 삭제한다.  


4️⃣. 다시 한번 `certbot`을 입력하여 오류가 출력되는지 확인한다.

저 같은 경우엔 이번엔 출력이 안되고 아래와 같은 로그가 나왔습니다.

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Certbot doesn't know how to automatically configure the web server on this system. 
However, it can still get a certificate for you. Please run "certbot certonly" to do so. 
You'll need to manually configure your web server to use the resulting certificate.
```  

5️⃣. 이번엔 방금 삭제했던 `certbot-dns-cloudflare`를 설치합니다.

`pip install certbot-dns-cloudflare`

#6️⃣. 마지막으로, certbot을 입력하여 오류 메시지를 체크한다. (`certbot`)  

이렇게 해결이 되었고 SSL 등록까지 되는 걸로 확인했습니다 :D

