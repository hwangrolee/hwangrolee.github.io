---
layout: post
title: Apple login 사용해보기
date: 2024-12-01
description: Apple Login을 직접 구현하는 방법을 알아봅니다.
tags: 애플로그인, AppleLogin
keywords: Apple, Apple Login, 애플로그인, OAuth
categories: 로그인
giscus_comments: true
toc:
  sidebar: left
---

### 0. 준비사항

[애플 로그인 뉴스](https://developer.apple.com/kr/news/?id=09122019b)

https://developer.apple.com/account 에서 필요한 정보를 알 수 있습니다.

### 1. Ceriticates, IDs & Profile에서 세팅정보를 알수 있습니다.

{% include figure.liquid loading="eager" path="assets/img/posts/2024-12-01-apple-login/apple_login_1.webp" class="img-fluid rounded z-depth-1"
alt="Ceriticates, IDs & Profile에서 세팅정보를 알수 있습니다."
title="Ceriticates, IDs & Profile에서 세팅정보를 알수 있습니다."
%}

### 2. Identifiers 탭에서 본인의 identifier를 선택하세요

{% include figure.liquid loading="eager" path="assets/img/posts/2024-12-01-apple-login/apple_login_2.webp" class="img-fluid rounded z-depth-1"
alt="Identifiers 탭에서 본인의 identifier를 선택하세요"
title="Identifiers 탭에서 본인의 identifier를 선택하세요"
%}

### 3. Apple ID Prefix = Team ID, Bundle ID = Client ID

Apple ID Prefix는 Team ID 입니다.

Bundle ID는 ClientId 입니다.

{% include figure.liquid loading="eager" path="assets/img/posts/2024-12-01-apple-login/apple_login_3.webp" class="img-fluid rounded z-depth-1"
alt="Apple ID Prefix = Team ID, Bundle ID = Client ID"
title="Apple ID Prefix = Team ID, Bundle ID = Client ID"
%}

### 4. 로그인 기능을 활성화하세요.

{% include figure.liquid loading="eager" path="assets/img/posts/2024-12-01-apple-login/apple_login_4.webp" class="img-fluid rounded z-depth-1"
alt="로그인 기능을 활성화하세요."
title="로그인 기능을 활성화하세요."
%}

### 5. Keys 에서 애플 로그인을 위한 서비스를 생성하세요

{% include figure.liquid loading="eager" path="assets/img/posts/2024-12-01-apple-login/apple_login_5.webp" class="img-fluid rounded z-depth-1"
alt="Keys 에서 애플 로그인을 위한 서비스를 생성하세요"
title="Keys 에서 애플 로그인을 위한 서비스를 생성하세요"
%}

### 6. key id & private key 를 확인하세요.

Download 버튼을 클릭하면 private key 를 다운받으실 수 있습니다.

{% include figure.liquid loading="eager" path="assets/img/posts/2024-12-01-apple-login/apple_login_6.webp" class="img-fluid rounded z-depth-1"
alt="key id & private key 를 확인하세요."
title="key id & private key 를 확인하세요."
%}

### 7. client_secret 을 구해보겠습니다.

jwt를 설치합니다.

```bash
sudo gem install jwt
```

아래 스크립트를 client_secret.rb 파일로 생성합니다.

```ruby
require 'jwt'
key_file = 'private_key.p8'
team_id = 'team_id'
client_id = 'client_id'
key_id = 'key_id'
ecdsa_key = OpenSSL::PKey::EC.new IO.read key_file
headers = {
  'kid' => key_id
}
claims = {
  'iss' => team_id,
  'iat' => Time.now.to_i,
  'exp' => Time.now.to_i + 86400*180,
  'aud' => 'https://appleid.apple.com',
  'sub' => client_id
}
token = JWT.encode claims, ecdsa_key, 'ES256', headers
puts token
```

client_secret.rb 실행하여 client_secret 을 구합니다.

```bash
ruby client_secret.rb
```

{: .block-tip }

> ##### TIP
>
> 테스트할 때에는 웹을 사용하는데 그럴때에는 Client ID 에 Bundle ID가 아닌 Group ID를 넣어주면 더욱 편하게 테스트를 할 수있습니다.

### 8. 토큰을 구하기 전에 먼저 사용자인증을 통해 code 값을 구합니다.

```bash
https://appleid.apple.com/auth/authorize?response_type=code&client_id={client_id}&redirect_uri={redirect_uri}
```

{% include figure.liquid loading="eager" path="assets/img/posts/2024-12-01-apple-login/apple_login_7.webp" class="img-fluid rounded z-depth-1"
alt="토큰을 구하기 전에 먼저 사용자인증을 통해 code 값을 구합니다."
title="토큰을 구하기 전에 먼저 사용자인증을 통해 code 값을 구합니다."
%}

{: .block-warning }

> ##### 확인사항 1
>
> 이 부분에서 상당히 시간소모를 했습니다. Client_id가 어떤것인지 애플쪽에서 정확히 명시하지 않아 헷갈려서 다른 값을 넣었고 에러코드가 client_id 가 틀리다고 나오질 않았기 때문에 다른 부분에서 문제가 있는줄 알고 구글서치만 수시간…
> 이 부분에서 redirect_uri를 입력했는데도 잘 안된다면 client_id 부터 의심하세요.

{: .block-warning }

> ##### 확인사항 2
>
> 만약 리디렉션됐는데도 Code가 안나온다면 response_mode, scope를 파라미터로 전달했는지 확인해보세요. response_mode와 scope를 전달하면 code를 받을 수 없습니다

{: .block-warning }

> ##### 확인사항 3
>
> 위 부분을 실행하면 xxx.com/?code={code} 형태로 리디렉션되며 code를 알 수 있습니다. 참고로 code는 5분간 유효한 키입니다.

### 9. code를 구했다면 이제는 Access Token을 구할 수 있습니다.

id_token은 jwt로 인코딩되어있으며 https://jwt.io 에서 디코딩해서 데이터를 볼 수 있습니다.

참고 : [Generate and validate tokens](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens?source=post_page-----a5b70fbf2f02--------------------------------)

```bash
curl -XPOST "https://appleid.apple.com/auth/token" -H "Content-Type: application/x-www-form-urlencoded" -d "client_id={client_id}&client_secret={client_secret}&grant_type=authorization_code&code={code}" | json_pp
{
    "refresh_token" : "",
    "expires_in" : 3600,
    "id_token" : "",
    "access_token" : "",
    "token_type" : "Bearer"
}
```

이메일은 이메일 공유하기로 로그인한 경우 애플ID 로 보이고, 가리기를 선택한 경우 @private.appleid.com 으로 이메일이 생성되어집니다.

sub라는 필드는 하나의 애플계정의 유니크한 값이니 이메일로 유저를 구분하지 않고 sub필드로 유저를 구분할 수 있습니다.

### 참고

- [sign-in-with-apple-4](https://sarunw.com/posts/sign-in-with-apple-4/)
- [generate_and_validate_tokens](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens)
- [authenticating_users_with_sign_in_with_apple](https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_rest_api/authenticating_users_with_sign_in_with_apple)
