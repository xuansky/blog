---
layout: post
title: Windows의 모든 콘솔에서 유니코드 적용 (codepage 65001)
author: xuansky
color: "#7777"
tags: [windows, console, unicode]
---

Windows에서 개발을 하거나 작업을 하다보면 한글이 깨져서 나오는 경우가 있는데, cmd.exe에서는 단순히 변경할 수 있지만 통제권이 없는 외부 프로그램일 경우 답답할 때가 있다.

이런경우 Windows의 모든 콘솔을 `유니코드` 로 바꾸는 방법이 있다.

일반 적인 `cmd.exe`나 `powershell`이라면
아래와 같이 코드 페이지를 변경할 수 있을 것이다.

```
chcp 65001
```

하지만 이 경우는 사용중인 콘솔만 바뀌고 지속적인 것도 아니다. 새 창을 열면 기본값으로 돌아가기 때문이다.

그러면 영구적으로 전체콘솔에 적용해보자.
먼저 텍스트 파일을 하나 만들어서 파일명을 `windows-console-unicode.reg`라고 만들자

그리고 아래와 같이 작성한다.

```
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Console\%SystemRoot%_System32_cmd.exe]
"CodePage"=dword:0000fde9
```

파일을 저장하고 해당 파일을 더블클릭해서 실행해 준다.

그리고 재부팅 하면 이제 모든 콘솔 화면은 유니코드가 적용되었을 것이다.
