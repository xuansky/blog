---
layout: post
title: Ubuntu 20.04에 Redis 설치하기
auth: xuansky
tags: [linux, ubuntu, redis]
---

이 페이지 요약: 우분투에서 Redis를 간단히 설치해 본다.

### Redis 설치

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install redis-server
```

### 설치 확인하기
```bash
redis-cli
ping
exit
```

### 비밀번호 구성

먼저 `redis-cli`로 클라이언트 실행하고,

```bash
redis-cli
```

```redis
config set requirepass my-new-pass
```

설정할 암호를 `my-new-pass`대신 원하는 문자를 입력한다.

다음 명령으로 암호를 확인할 수 있다.

```redis
auth my-new-pass
```

클라이언트를 종료할 때는 `exit`를 실행한다.

```redis
exit
```

### Redis 서비스 재시작

```bash
sudo systemctl restart redis-server
```