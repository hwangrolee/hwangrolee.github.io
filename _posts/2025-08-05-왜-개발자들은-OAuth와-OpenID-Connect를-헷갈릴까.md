---
layout: post
title: 왜 개발자들은 OAuth와 OpenID Connect를 헷갈릴까?
date: 2025-08-05 00:00:00
description: 개발자들이 흔히 혼동하는 OAuth 2.0과 OpenID Connect(OIDC)의 차이점을 명확히 설명합니다. 소셜 로그인이 OAuth가 아닌 OIDC인 이유, 인증(Authentication)과 권한 위임(Authorization)의 핵심 개념을 알아보고 정확한 기술 용어를 사용하세요.
tags: [Authentication, Authorization, OAuth, OIDC, OpenID Connect, 소셜로그인]
categories: [인증/인가]
giscus_comments: true
author: hwangrolee
image:
  path: /assets/img/oauth-vs-openid-connect.png
  alt: "왜 개발자들은 OAuth와 OpenID Connect를 헷갈릴까?"
---

> "구글 로그인은 OAuth 2.0으로 만들었어요."
>
> "카카오 로그인은 OAuth API 써서 붙였어요."

많은 개발자들이 ‘소셜 로그인 = OAuth 2.0’ 이라고 이야기합니다. 하지만 사실 소셜 로그인은 OAuth가 아니라 OpenID Connect(OIDC) 덕분에 가능한 기능입니다.

이 글에서는 개발자들이 OAuth와 OIDC를 혼용하는 이유와 왜 구글/카카오 로그인은 OIDC라고 말해야 하는지를 정리해봅니다.

## OAuth 2.0과 OpenID Connect(OIDC)는 뭐가 다를까?

### OAuth 2.0

- 목적: 권한 위임(Authorization)
- 예: “내 구글 드라이브 파일에 접근할 수 있는 권한을 앱에 줄게”

## OpenID Connect (OIDC)

- 목적: 인증(Authentication)
- OAuth 2.0 기반 프로토콜 + ID Token 개념 추가
- 예: “이 사용자가 ‘홍길동’인지 확인하고 로그인 처리해줘”
- 정리하면 OAuth는 ‘무엇을 할 수 있는지’, OIDC는 ‘누구인지’ 를 해결하는 프로토콜입니다.

## 개발자들이 혼용하는 대표 케이스

> "구글 로그인은 OAuth 2.0을 사용했다."

왜 혼동될까? 구글 개발자 문서에 ‘Google OAuth 2.0’이라고 적혀 있어서. 실제 구현도 OAuth Authorization Code Flow를 사용하기 때문.

### 정확한 설명

- 구글 API 호출 -> OAuth
- 구글 로그인 -> OpenID Connect
- 구글 로그인을 구현했다면 “OIDC를 사용했다”고 말하는 게 정확합니다.
- "OAuth로 로그인 기능을 만들었다."

### OAuth Flow를 사용하니까 로그인도 OAuth로 가능하다고 착각.

- OAuth는 권한 위임만 담당합니다.
- 로그인 기능을 가능하게 해주는 건 OIDC에서 발급하는 ID Token 덕분입니다.
- OAuth만 쓰면 사용자 ‘신원’은 확인할 수 없습니다.
- "ID Token도 Access Token 중 하나다."

### 왜 혼동될까? 둘 다 ‘토큰’이니까 같은 개념이라고 생각.

- JWT 구조도 비슷해서 더 헷갈립니다.
- Access Token -> API 자원 접근 용도
- ID Token -> 로그인 인증 용도 (이메일, 이름, 사용자 ID 포함)

### 그렇다면, 구글/카카오 로그인은 OAuth일까 OIDC일까?

- 정확히 말하면 -> OIDC를 활용한 것입니다.
- OAuth 2.0 -> 권한 위임 (예: 내 구글 드라이브 파일 접근 허용)
- OpenID Connect(OIDC) -> OAuth 2.0 기반 + "사용자 인증(로그인)" 기능 추가

구글 로그인 / 카카오 로그인은 “로그인(= 인증)”이 목적이라 OIDC를 사용했다고 해야 정확합니다.

### 면접이나 블로그에서 이렇게 설명할 수 있습니다

구글, 카카오 로그인 기능을 내 서비스에 붙일 때 OAuth 2.0 Flow(Authorization Code)를 사용하긴 했지만, 실제로 로그인 기능을 가능하게 한 건 OpenID Connect(OIDC)입니다.

OAuth만 쓰면 단순 권한 위임만 가능하지만, OIDC는 ID Token을 통해 사용자 정보를 확인할 수 있기 때문에 로그인 구현이 가능합니다.

## 결론: 이렇게 기억하면 안 헷갈림

- OAuth 2.0 -> 권한 위임 (내 API를 대신 호출하게 권한 줌)
- OIDC -> OAuth 2.0 + 로그인 기능 (이 사용자가 누구인지 알려줌)

## 마무리

다음에 누군가 “구글 로그인은 OAuth로 만들었어?” 라고 물으면

> “정확히는 OAuth 위에 인증 기능을 추가한 OpenID Connect로 구현했어”

라고 말해보세요. OAuth는 ‘무엇을 할 수 있나’, OIDC는 ‘누구인가’. 이 한 줄만 기억하면 더 이상 헷갈리지 않을 겁니다.
