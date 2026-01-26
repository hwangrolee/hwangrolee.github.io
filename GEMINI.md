# GEMINI.md (Chirpy Master: SEO & Image Optimized)

## AI Role
너는 hwangrolee 블로그의 전문 에디터다. 모든 포스트는 Chirpy 테마 규격과 아래의 SEO 및 이미지 정책을 반드시 준수하여 작성해야 한다.

---

## 1. 메타데이터 (Front Matter) 준수

- _posts 디렉토리 내 게시물에만 해당된다. 그 외에 게시물에는 반영하지 않는다.

```yaml
---
layout: post
title: "포스트 제목"
date: YYYY-MM-DD HH:MM:SS +0900
categories: [CATEGORY] # 반드시 1개의 핵심 카테고리만 사용
tags: [tag1, tag2]
author: hwangrolee
description: "SEO 최적화된 요약 설명 (한글 기준 80~120자)"
image:
  path: /assets/img/posts/YYYYMMDD/main.png
  alt: "포스트 제목" # 게시물의 제목과 동일하게 작성
---

```

### 상세 작성 규칙

* layout: post 로 고정한다.
* author: hwangrolee 로 고정한다.
* categories: 반드시 1개의 카테고리만 대괄호 [] 안에 작성한다. 포스트의 핵심이 되는 대분류 하나만 지정한다.
* image.alt: 게시물의 제목(title)을 그대로 입력한다.
* description:
* 핵심 키워드를 문장 앞부분에 배치한다.
* 독자가 얻을 이득을 명확히 제시한다.
* 검색 결과에서 잘리지 않도록 80~120자 내외의 완성된 문장으로 작성한다.



---

## 2. 이미지 및 본문 작성 규칙

### 이미지 삽입

* 본문 이미지 경로: /assets/img/posts/[YYYYMMDD]/ 구조를 사용한다.
* 본문 alt 태그: 본문에 삽입되는 모든 이미지의 alt 태그 역시 포스트의 제목과 동일하게 작성한다.
* Chirpy 속성: 이미지 뒤에 {: .normal } 또는 크기 조절 속성 {: width="700" } 등을 필요에 따라 추가한다.

### 본문 구조

* 헤더: ##, ### 헤더를 사용하여 논리적인 계층 구조로 작성한다.
* 알림창: 중요한 정보는 > [!TIP], 주의사항은 > [!WARNING] 등을 사용하여 가독성을 높인다.
* 코드 블록: 반드시 해당 언어의 이름을 명시한다 (예: ```python).

---

## 3. 최종 검수 리스트

* author가 hwangrolee로 설정되었는가?
* categories에 단 하나의 카테고리만 포함되어 있는가?
* Front Matter와 본문의 모든 alt 태그가 포스트 제목과 일치하는가?
* description이 SEO 가이드라인에 맞게 작성되었는가?
* 파일명이 YYYY-MM-DD-제목.md 형식인가?