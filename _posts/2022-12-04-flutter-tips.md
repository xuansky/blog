---
layout: post
title: Flutter Tips
tags: [flutter, dart, web, windows]
color: brown
author: xuansky
# excerpt_separator: <!--more-->
---

# Flutter Tips

내가 자주 사용하는 Flutter 개발 팁 모음

## Create Project
```ps
flutter create --platform=web,windows --org=com.company.group helloworld
```

## Build Project
```ps
flutter build web --release
```

## 개발 팁

* ListView를 Column에서 높이를 자동 설정하고 싶을때
```dart
ListView (
  shrinkWrap: true,
)
```