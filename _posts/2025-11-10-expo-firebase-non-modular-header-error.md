---
layout: post
title: '[해결됨] Expo Firebase iOS 빌드 에러: ''non-modular header inside framework module
  RNFBApp'' 해결책 (forceStaticLinking)'
date: 2025-11-10 00:00:00
description: '이 글은 Expo 환경에서 @react-native-firebase/app을 연동할 때 발생하는 iOS 빌드 에러(include
  of non-modular header inside framework module RNFBApp.RCTConvert_FIRApp)에 대한 정확한
  해결책을 제시합니다. 일반적인 버전 조정이나 클린 빌드로 해결되지 않는 이 문제는 app.json 파일의 expo-build-properties
  플러그인 설정에 forceStaticLinking 옵션을 추가하여 해결할 수 있습니다. 특히 Expo SDK 54+ 버전과 React Native
  Firebase 사용 시 useFrameworks: static 설정을 활용할 때 발생하는 iOS 고유의 헤더 모듈 문제를 해결하는 핵심 가이드입니다.
  개발자들이 이 문제로 시간을 낭비하지 않도록 도와주는 실용적인 개발 팁입니다.'
tags: [App Development, 앱개발]
keywords: non-modular header inside framework module, RNFBApp.RCTConvert_FIRApp, include
  of non-modular header, Expo Firebase 연동, Expo iOS build error, react-native-firebase,
  Expo build properties
categories: [Mobile]
giscus_comments: true
toc:
  sidebar: left
author: hwangrolee
image:
  path: /assets/img/expo-firebase-non-modular-header-error.png
  alt: '[해결됨] Expo Firebase iOS 빌드 에러: ''non-modular header inside framework module RNFBApp'' 해결책 (forceStaticLinking)'
---

Expo 공부하면서 Firebase 연동을 시도해보고자 했습니다. 하지만 아래와 같은 에러가 발생했어요.

##### 1. 에러 사항

```bash
  18 | #import <FirebaseCore/FirebaseCore.h>
> 19 | #import <React/RCTConvert.h>
     |         ^ include of non-modular header inside framework module 'RNFBApp.RCTConvert_FIRApp': '.../ios/Pods/Headers/Public/React-Core/React/RCTConvert.h' [-Werror,-Wnon-modular-include-in-framework-module]
  20 |
  21 | @interface RCTConvert (FIRApp)
  22 | + (FIRApp *)firAppFromString:(NSString *)appName;
```

ChatGPT, Claude, Gemini 다 물어봤는데 Nodejs, React, React native, Expo 버전을 조정하라는 말만 반복할 뿐 진짜 문제는 답하지 않아 결국엔 구글링을 하게 되었습니다.

---

##### 2. 해결 방법

참고링크:

- [https://github.com/expo/expo/issues/39607?source=post_page-----3fe6f0563ee0---------------------------------------#issuecomment-3337284928](https://github.com/expo/expo/issues/39607?source=post_page-----3fe6f0563ee0---------------------------------------#issuecomment-3337284928)

app.json 에 ”forceStaticLinking”: [“RNFBApp”, “RNFBAnalytics”] 를 추가하면 됩니다.

```json
"plugins": [
    "@react-native-firebase/app",
    [
        "expo-build-properties",
        {
            "ios": {
                "useFrameworks": "static",
                "forceStaticLinking": ["RNFBApp", "RNFBAnalytics"]
            },
            "android": {}
        }
    ],
]
```

React Native 에서도 Firebase 연동했을 때 고생을 했는데 Expo 에서도 예기치 못한 이슈가 발생하네요…

React Native, Expo 모두 개발해 보면서 느낀점은 라이브러리를 연동할 때가 더 시간이 많이 소요되고 AI 로도 해결이 잘 안된다는 점입니다.
