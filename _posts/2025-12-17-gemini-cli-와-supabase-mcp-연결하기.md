---
layout: post
title: Gemini CLI 와 Supabase MCP 연결하기
date: 2025-12-17 01:00:00
description: Gemini CLI와 Supabase를 MCP로 연결하는 방법을 단계별로 설명합니다. PostgreSQL 스키마 조회, 마이그레이션 파일 관리 자동화, 실무 활용 시 주의사항까지 실제 사용 경험을 바탕으로 정리했습니다.
tags: [AI]
categories: [AI]
giscus_comments: true
author: hwangrolee
image:
  path: /assets/img/gemini-cli-supabase-mcp/gemini-cli-supabase-mcp1.png
  alt: "Gemini CLI 와 Supabase MCP 연결하기"
---

백엔드 개발자로서 데이터베이스를 수정할 때마다 마이그레이션 파일을 별도로 관리하며 개발하고 있습니다. Expo + Supabase + Gemini CLI 조합으로 혼자 개발하면서 Supabase PostgreSQL 구조가 변경될 때마다 마이그레이션 파일을 업데이트해야 했습니다.

하지만 매번 PostgreSQL 스키마 정보를 AI에게 알려주는 것도 번거롭고, 마이그레이션 파일을 만들 때 여러 번 대화를 주고받아야 하는 것도 귀찮았습니다. MCP를 연결하면 빠르게 해결될 것을 알고 있었지만, MCP 연결 자체도 귀찮아서 계속 미루고 있었습니다. 그러나 더 이상 미루면 혼자 개발하는 시간 자체가 아까워질 것 같아 결심하게 되었습니다.

그래서 이번에 Gemini CLI와 Supabase를 MCP로 연결해보기로 했습니다.

---

## Supabase MCP 가이드

Supabase 공식 문서의 MCP 가이드를 참고하여 연결을 시도했습니다. 직접 해보니 생각보다 어렵지 않게 연결할 수 있었습니다.
다른 개발 문서에 비해 Supabase는 가이드 문서가 상당히 잘 정리되어 있는 편입니다.

가이드 링크: https://supabase.com/docs/guides/getting-started/mcp

---

## Supabase MCP 설정 방법

가이드 문서의 "[Step 2: Configure your AI tool](https://supabase.com/docs/guides/getting-started/mcp#step-2-configure-your-ai-tool)" 섹션으로 이동하여 다음 단계를 진행했습니다.



<img src="/assets/img/gemini-cli-supabase-mcp/gemini-cli-supabase-mcp2.png" width="100%" alt="image2">

1. MCP로 연결할 프로젝트를 선택합니다.
2. Supabase 내에서 연결할 서비스를 선택합니다.
3. Client는 Gemini CLI를 선택합니다.
4. Installation 파트에 안내된 대로 MCP를 설정합니다.
5. Gemini CLI에 접근한 후 /mcp auth supabase를 실행하면 Supabase에 연결을 시도합니다.
   - <img src="/assets/img/gemini-cli-supabase-mcp/gemini-cli-supabase-mcp3.png" width="100%" alt="image3">

---

## MCP 테스트 결과

#### 스키마 조회 테스트

미리 Supabase PostgreSQL에 products 테이블을 생성해둔 상태에서 조회 테스트를 진행했습니다.

"supabase PostgreSQL에서 products 테이블의 스키마 정보를 마크다운 형태로 작성해줘. 그리고 테이블 생성 쿼리 작성해줘"라는 프롬프트를 실행했을 때, 스키마 정보와 생성 쿼리가 정상적으로 출력되었습니다.

<img src="/assets/img/gemini-cli-supabase-mcp/gemini-cli-supabase-mcp4.png" width="100%" alt="image4">

#### 스키마 변경 요청 테스트

MCP 연결 시 read only 모드로 설정했기 때문에, 스키마 변경 요청을 할 경우 어떻게 동작하는지 확인해보았습니다. 예상대로 실제 스키마 변경은 실패했으며, 대신 스키마 변경에 필요한 쿼리를 전달해주었습니다.

<img src="/assets/img/gemini-cli-supabase-mcp/gemini-cli-supabase-mcp5.png" width="100%" alt="image5">

#### 데이터 조회 테스트

실제 데이터를 조회했을 때도 정상적으로 데이터가 출력되는 것을 확인했습니다.

<img src="/assets/img/gemini-cli-supabase-mcp/gemini-cli-supabase-mcp6.png" width="100%" alt="image6">

---

## 마무리 및 주의사항

지금까지 MCP에 대해서는 많이 들어보았고 사례 글도 많이 읽어보았지만, 실제로 다뤄본 것은 이번이 처음이었습니다. 생각보다 간단하게 연결이 가능했고 유용하게 활용할 수 있는 기능이라고 느꼈습니다.

다만 몇 가지 주의할 점이 있습니다. 인프라와 연결할 경우 가능하다면 쓰기 기능은 사용하지 않는 것이 좋으며, 데이터베이스와 연결하는 계정에는 Read 권한만 부여해야 합니다. 프롬프트를 아무리 잘 작성해도 AI에는 할루시네이션이 존재하고 의도한 바와 다르게 동작할 수 있기 때문에 무조건 믿어서는 안 됩니다.

실무에서 활용할 경우, 원하는 데이터를 추출하기 위한 쿼리를 생성하도록 프롬프트를 작성하고, 답변으로 나온 쿼리를 직접 확인한 후 개발 데이터베이스에서 먼저 테스트하고 라이브 데이터베이스에 적용할 것을 권장합니다.

---

## 사용하면서 느낀 MCP의 한계점

MCP를 실제로 사용해보면서 몇 가지 제약사항을 발견했습니다.

**토큰 소비 문제**: MCP는 외부 시스템과 통신하기 위해 추가적인 컨텍스트를 전송하므로 일반적인 대화보다 토큰을 더 많이 소비합니다. AI 서비스에서 토큰은 곧 비용이기 때문에, 빈번한 MCP 사용은 API 비용 증가로 이어질 수 있습니다.

**응답 속도 문제**: 외부 데이터베이스나 서비스와 실시간으로 통신하다 보니 응답 속도가 생각보다 느립니다. 백그라운드에서 작업을 시켜놓고 다른 일을 하면 되지만, 개발 흐름이 끊기면서 오히려 집중력이 떨어지는 경우도 있었습니다. 특히 여러 작업을 동시에 진행하다 보면 컨텍스트 스위칭으로 인해 생산성이 저하되기도 합니다.

---

## MCP의 전망과 실무 활용 시 고려사항

MCP는 AI가 외부 시스템과 안전하게 통합될 수 있는 표준화된 방식을 제시했다는 점에서 의미가 있습니다. 앞으로 기술이 성숙해지면 더 효율적으로 활용될 것으로 기대됩니다.

다만 개발자들이 MCP를 실무에서 신중하게 접근할 가능성이 높습니다. 그 이유는 MCP로 연결하는 대상이 주로 데이터베이스, 클라우드 인프라 등 서비스의 핵심 자원이기 때문입니다. 코드는 버전 관리와 리뷰를 통해 검증할 수 있지만, 인프라에 대한 잘못된 조작은 즉시 서비스 장애로 이어질 수 있습니다.

따라서 MCP를 활용할 때는 다음과 같은 안전장치가 필요합니다.

- 읽기 전용 권한으로 연결하기
- AI가 생성한 쿼리나 명령어를 직접 검토한 후 실행하기
- 개발 환경에서 충분히 테스트한 후 프로덕션에 적용하기

이러한 제약사항에도 불구하고, MCP는 반복적인 데이터 조회나 스키마 분석 작업에서 충분히 생산성 향상에 기여할 수 있는 도구라고 생각합니다.
