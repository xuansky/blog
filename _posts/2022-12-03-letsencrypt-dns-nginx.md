---
layout: post
title: Ubuntu Nginx에 let's encrypt (DNS)
tags: [ubuntu, nginx, let's encrypt, ssl, https]
color: grey
author: xuansky
# excerpt_separator: <!--more-->
---

# 우분투에서 Let's Encrypt로 ssl 발급하고 Nginx로 https 서비스 하기

Let's Encrypt로 ssl 인증서 발급하는 방법은 인터넷을 찾으면 쉽게 알 수 있다. 하지만 DNS를 이용한 발급 방식은 잘 없고 더욱이 자동 연장하는 방법이 함께 설명된 자료는 더 찾기 힘들다. 이 내용은 DigitalOcean[^1] 에서 참고하여 정리하였다.

<!--more-->

Lets' Encrypt는 웹 환경에 정말 좋은 영향을 준다고 생각한다. 웹 보안을 위해 SSL을 의무화 하고 강하게 권유하고 있지만 대 부분 SSL인증서는 고가로 서비스 되고 있다. 일반적으로 1년 사용 비용이 도메인 비용의 몇배에서 몇십배에 달하는데 Let's Encrypt는 3개월 갱신만 지키면 영원히 무료이다.

## 1. Certbot 설치

최신 버전의 Certbot을 받기 위해서 apt 저장소를 등록한다.
```bash
sudo apt-add-repository ppa:certbot/certbot
```

그 다음에 certbot을 설치한다.
```bash
sudo apt install certbot
```
> 중간에 나오는 질문은 전부 `y` 또는 `Y`를 입력하면 된다.


설치가 완료되면 버전확인을 해서 정상으로 설치되었는지 점검한다.
```bash
certbot --version
```

```bash
<출력>
certbot 1.21.0
```

## 2. acme-dns-certbot 설치

기본 certbot이 설치되었다면 이제는 DNS 검증 모드를 사용하기 위해서 acme-dns-certbot을 설치할 차례이다.

```bash
wget https://github.com/joohoi/acme-dns-certbot-joohoi/raw/master/acme-dns-auth.py
```

설치 후에 실행 권한을 부여 합니다.
```bash
chmod +x acme-dns-auth.py
```

파일을 수정하기 위해 다음 명령을 실행해 보자
```
vi acme-dns-auth.py
```

파일의 첫번째 줄에서 `python`을 `python3`로 변경한다.  
`acme-dns-auth.py`
```py
#!/usr/bin/env python3
```
수정후 저장하고 vi를 종료한다.

수정한 파일을 letsencrypt로 이동한다.
```bash
sudo mv acme-dns-auth.py /etc/letsencrypt/
```

## 3. acme-dns-certbot 설정하기

`acme-dns-certbot`을 사용하려면 최소한 한번은 certbot을 설정해서 실행해야 한다.

다음과 같이 초기 설정을 한다.
```bash
sudo certbot certonly --manual --manual-auth-hook /etc/letsencrypt/acme-dns-auth.py --preferred-challenges dns --debug-challenges -d \*.your-domain -d your-domain
```
> `--manual`는 모든 자동 발급 기능은 무효화 한다. 지정한 발급 기능을 사용하기 위해 기본적인 자동발급 기능은 사용하지 않는다.  
> `--manual-auth-hook`은 acme-dns-certbot 으로 hook 기능을 사용하기 위해 설정한다.  
> `--preferred-challenges`는 DNS방식으로 발급한다는 설정이다.  
> `--debug-challenges`는 인증서의 유효성 검사하기 전 Certbot을 일시 중지하도록 해야 하기 때문에 사용한다. 이는 acme-dns-certbot에서 요구하는 DNS CNAME 레코드를 설정할 수 있도록 하기 위한 것이며, 중지하지 않게 되면 DNS 설정을 변경할 시간이 없을 것이다.  
> `-d`는 사용할 도메인 이름을 지정할 때 사용한다.  
> 만약에 와일드카드 인증을 하려면 별표(*)를 사용하는데 앞에 backslash(\\)를 붙여야 한다.

위의 명령을 실행하고 각 질문에는 적절한 답을 해 주면 된다.
```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): 
```
위과 같은 질문에는 인증서가 만료되기전 미리 알림 메일 주소를 입력하면 된다.

```
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.3-September-21-2022.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: 
```
서비스를 위해 약관에 동의 하는지 묻는다. 사용하려면 `Y` 또는 `y`를 입력해야 한다.

```
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o:
```
인증서가 발급이 되면, 무료 디지털을 위해 Certbot을 개발하는 Let's Encrypt 파트너 재단, 비영리 단체 등에 메일 주소를 공유해도 되겠냐는 질문이다.
무료로 사용하려면 `Y`를 눌러준다.

```
Account registered.
Requesting a certificate for *.your-domain and your-domain
Hook '--manual-auth-hook' for your-domain ran with output:
 Please add the following CNAME record to your main DNS zone:
 _acme-challenge.your-domain CNAME baad1234-11x1-49x1-b345-c3ff52692x53.auth.acme-dns.io.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Challenges loaded. Press continue to submit to CA. Pass "-v" for more info about
challenges.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

위와 같은 문구가 나오면 이제 도메인 관리 사이트 (후이즈, 가비아, ...)에서 CNAME 설정을 위의 내용을 아래와 같이 해 주면 된다.

| Type | Hostname | Value |
|----|----|----|
| CNAME | _acme-challenge.your-domain | baad1234-11x1-49x1-b345-c3ff52692x53.auth.acme-dns.io. |

도메인 설정이 완료되면 `Enter`키를 눌러서 완료 한다.

```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/your-domain/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/your-domain/privkey.pem
This certificate expires on 2023-03-03.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
발급이 성공한다면 위와 같이 표시된다.

자동으로 renew가 가능한지 확인하려면 다음과 같이 테스트 가능하다.
```bash
sudo certbot renew --dry-run
```

재발급을 하려면 아래와 같이 하면 된다.
```
sudo certbot renew
```

자동으로 재발급 하도록 하기 위해서는 crontab을 이용하면 된다. 다음과 같이 입력해 본다.
```bash
sudo crontab -e
```
vi 편집 화면이 뜨고 설명이 나와 있다. 사용법은 검색해 보면 되고 우리는 맨 아래줄에 다음과 같은 내용만 입력하면 된다.

```
0 1 1 * * /usr/bin/certbot renew >> /var/log/letsencrypt/renew.log
```
설명하자면 매월 1일 01:00에 갱신 한다는 것이다.

추가로 certbot으로 renew하고 nginx를 재시작 하려면 다음과 같이 한다.
```
0 0 1 * * certbot renew --renew-hook "sudo service nginx restart" >> /var/log/letsencrypt/renew.log
```

이제 거의다 왔다.

## 4. Nginx에 인증서 등록하기

인증서가 발급이 되었으면 웹페이지에 적용하기 위해 nginx에 적용해 본다.

다음 명령으로 nginx 설정을 연다.
```bash
sudo vi /etc/nginx/sites-available/default
```

```conf
server {
  listen 80;

  # (중략) 기존의 http 설정 부분
}

server {
  # https 설정은 여기에 한다.
  listen 443 ssl http2;
  server_name api.user-domain www.user-domain user-domain;

  ssl_certificate /etc/letsencrypt/live/your-domain/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/your-domain/privkey.pem;
  
  # 필요하면 로그를 남긴다.
  access_log /var/log/nginx/proxy/access.log;
  error_log /var/log/nginx/proxy/error.log;

  location / {
    include /etc/nginx/proxy_params;
    root /var/www/html;
  }

  location / {
    include /etc/nginx/proxy_params;
    proxy_pass http://127.0.0.1:4001;
  }

}
```
> /var/log/nginx/proxy 폴더가 없으면 생성해 주어야 한다.

새로 설정한 환경을 적용하기 위해 nginx를 재시작 한다.
```bash
sudo service nginx restart
```

이제 https://your-domain 페이지에 접속해서 인정서가 표시되는지 확인해 보면 된다.

페이지가 표시되지 않는다면 다음 절차를 확인해 보자.

## 5. 보안설정 - 포트 접근 설정

서버에서 80포트는 열려 있으나 443포트가 허용이 되지 않았을지 모른다.

```bash
sudo ufw status
```
위 명령으로 혀용된 포트를 확인하고 443이 허용이 되어 있지 않으면 다음 명령을 실행해서 허용시킨다.

```bash
sudo ufw allow 443
```

여기 까지가 설정 마무리 이다.
잘 작동이 되는지 알고 싶으면 몇일 후 `certbot renew`를 하던지 자동 갱신 명령까지 잘 되는지 확인하려면 crontab의 갱신 주기를 변경해서 작동 테스트를 해 보면 된다.

> 같은날 동일한 도메인으로 너무 많은 인증서를 발급받으면 해당 도메인이 차단 되거나 블랙리스트로 관리 될수 있으니 테스트는 적당히 해야 한다.

---

[^1]: https://www.digitalocean.com/community/tutorials/how-to-acquire-a-let-s-encrypt-certificate-using-dns-validation-with-acme-dns-certbot-on-ubuntu-18-04
