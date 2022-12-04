---
layout: post
title: Ubuntu에서 dotNet 구축하기 (dotNet6)
tags: [ubuntu, nginx, dotNet, .net]
color: skyblue
author: xuansky
# excerpt_separator: <!--more-->
---

# 우분투에 c# ASP.NET API 서버 구축하기 (with nginx)

기존에 .NET5로 만든 미들웨어 API가 있었다. 지금까지 윈도우서버에서 IIS로 서비스 하고 있었는데 배보다 배꼽이 더 큰 상황이라 리눅스 서버로 교체하기로 했다.

기존에 윈도우서버로 운영할 경우 네이버 클라우드 플랫폼에서 15만/월으로 사용했지만 VULTR 리눅스 서버로 교체할 경우 7달러/월(약 9천/월) 정도로 16배 절약 가능하다고 본다.

본 내용은 [Microsoft](https://learn.microsoft.com/ko-kr/dotnet/core/install/linux-ubuntu#2004)에서 참고하여 작성한 내용이다.

## 1. Ubuntu 20.04에 dotnet 6 설치

우분투 22.10이나 22.04(LTS) 버전을 선택해도 되지만 처음 시도하기 때문에 하위 호환을 고려해서 20.04(LTS) 버전으로 선택했다.
apt-get으로 20.04에서 설치되는 최종 버전이 dotnet6 이다.

여러 가지 방식으로 설치 할 수 있는데 가장 흔하게 사용하는 apt-get을 사용할 것이기 때문에 신뢰키 등록부터 한다.
```bash
wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb

sudo dpkg -i packages-microsoft-prod.deb

rm packages-microsoft-prod.deb
```

SDK설치
```bash
sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-6.0
```

런타임 설치
```bash
sudo apt-get update && \
  sudo apt-get install -y aspnetcore-runtime-6.0
```

설치가 잘 되었는지 확인하려면
```bash
dotnet --version
```

## 2. 빌드하기

설치가 완료 되었다면 소스파일을 빌드해 보자
여기서는 프로젝트 이름을 `HelloWorld`로 가정해 본다.

소스파일에서 타깃 플랫폼 설정을 수정해 준다.
```html
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
  </PropertyGroup>
  <!-- 생략 -->
</Project>
```
저는 TargetFramework가 `net5.0`이었는데 `net6.0`으로 변경했다.
> 마이크로소프트에서는 net5.0은 더이상 지원하지 않는다고 한다.

```bash
cd HelloWorld
dotnet build --configuration Release
```
다행히 net5에서 작성한 소스코드 변경없이 net6에서도 잘 빌드 된다.
빌드가 성공하면 `HelloWorld/bin/Release/net6.0/` 디렉토리에 빌드된 파일이 만들어 진다.

## 3. asp.net api 실행해 보기

다음과 같이 실행해 본다.
```bash
# HelloWorld/bin/Release/net6.0/에서
dotnet HelloWorld.dll
```

정상적으로 실행되면 다음과 같이 출력된다.
```
# (출력)
: info: Microsoft.Hosting.Lifetime[14]
:       Now listening on: http://localhost:5000
: info: Microsoft.Hosting.Lifetime[14]
:       Now listening on: https://localhost:5001
: info: Microsoft.Hosting.Lifetime[0]
:       Application started. Press Ctrl+C to shut down.
: info: Microsoft.Hosting.Lifetime[0]
:       Hosting environment: Production
: info: Microsoft.Hosting.Lifetime[0]
:       Content root path: /
```

잘 실행 되었는지 확인 하려면 웹브라우저에서 `http://localhost:5000`을 실행해 보면 되는데, 여기는 리눅스 서버이고 웹브라우저 따윈 없다.
그렇다면 확인하기 위해서 nginx를 사용하는 수 밖에 없다.

nginx 설치를 위해서 일단 서비스 중지 : Ctrl+C

## 4. nginx설치

다음 명령으로 nginx설치
```bash
sudo apt update
sudo apt install nginx
```

nginx에서 몇가지 환경 설정이 필요하다.
```
vi /etc/nginx/sites-available/default
```

서버 구성에 맞는 로케이션 설정추가 하거나 변경한다.
```conf
# (생략)
location /api {
  proxy_pass http://127.0.0.1:5000;
}
```

수정한 내용을 반영하기 위해 nginx 재시작
```
sudo service nginx restart
```

이제 다시 접속 테스트를 위해 웹브라우저가 있는 환경에서 해당 서버 IP로 접속을 해 본다. 서버 IP가 `123.123.12.45`라고 가정하면 `http://123.123.12.45/api`에 접속해 본다.

당연히 오류가 날 것이다. dotnet을 실행하지 않았기 때문이다.
앞에서 했던 asp.net HelloWorld를 시작해 보자.

```bash
# HelloWorld/bin/Release/net6.0/에서
dotnet HelloWorld.dll
```
다시 `http://123.123.12.45`에 접속하면 잘 표시될 것이다.

그러면 매번 dotnet으로 서버를 구동 해야 한다면 불편할 것이다.
그래서 리눅스 서비스로 등록해서 사용해야 한다.

`Ctrl+C`하고 다음 단계로 넘어 가 보자.


## 5. dotnet실행 명령을 리눅스 서비스로 등록하기


서비스 등록하는 방법은 [Microsoft](https://learn.microsoft.com/ko-kr/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-7.0)를 참고하였다.

빌드 결과 파일(`HelloWorld/bin/Release/net6.0/*`)을 모두 `/var/www/helloworld/*`로 복사한다.

```bash
sudo vi /etc/systemd/system/helloworld.service
```
아래와 같이 파일을 작성한다.
```service
[Unit]
Description=Example .NET Web API App running on Ubuntu

[Service]
WorkingDirectory=/var/www/helloworld
ExecStart=/usr/bin/dotnet /var/www/helloworld/helloapp.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=hello-world-api
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```

서비스를 사용하도록 등록하고 시작한다.
```
sudo systemctl enable helloworld.service
sudo systemctl start helloworld
sudo systemctl status helloworld
```

## 기타
모든 준비가 다 되어도 웹 접근이 안될 때는 포트가 허용되었는지 확인해 보자
```
sudo ufw status
```
허용할 포트가 80이면
```
sudo ufw allow 80
```