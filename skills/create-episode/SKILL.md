---
name: books-create-episode
description: Neo4j GraphRAG 기반 책 에피소드 자동 생성 — 전체 파이프라인
disable-model-invocation: true
allowed-tools: Bash(curl *)
argument-hint: [documentId=<doc>]
---

## 용도
Neo4j에서 책 청크를 가져와 ContentPlanner로 시나리오 생성, 이미지/수식 렌더링, TTS, FFmpeg 영상 합성, YouTube 업로드까지 E2E 파이프라인.

## 핵심 파일
- `src/YTB-books-project/src/services/BooksVideoService.ts` — 메인 오케스트레이터, `BOOKS_PROJECT_CONFIG`
- `src/YTB-books-project/src/services/ContentPlannerService.ts` — AI 시나리오 생성
- `src/YTB-books-project/src/services/GhibliImageService.ts` — NanoBanana 이미지 생성
- `src/YTB-books-project/src/services/MathFormulaService.ts` — LaTeX → PNG 렌더링
- `src/YTB-books-project/src/services/Neo4jService.ts` — GraphDB CRUD

## 의존성
- Neo4j (bolt://YOUR_NEO4J_HOST:7687)
- NanoBanana API (이미지 생성)
- FFmpeg (영상 합성)
- Gemini TTS (음성 합성, 스타일별 voice 자동 선택)

## API / 인터페이스
```
POST /api/books/generate-next-episode
  → 다음 미처리 청크로 에피소드 자동 생성

Pipeline: Neo4j chunks → ContentPlanner → SceneImage → BooksVideo → FFmpeg → YouTube
```

## Style System
Strategy Pattern으로 5개 스타일 지원. 새 스타일 = 1 프로파일 + registry 등록.
- `math_character` (PRIMARY), `dark_academia`, `watercolor`, `minimalist`, `retro_comic`

## 다른 프로젝트에 이식하기
1. `YTB-books-project/` 전체 복사
2. Neo4j 인스턴스 + 스키마 세팅 (Document → Episode → Scene)
3. NanoBanana API 키 또는 대체 이미지 생성기
4. FFmpeg + TTS 설정
5. `BOOKS_PROJECT_CONFIG`에서 책/스타일/TTS 설정 변경

## 주의사항
- 14개 서비스, 11개 핸들러로 구성 — 부분 이식 어려움
- TTS: Gemini TTS REST API 직접 호출 (SDK는 404 에러)
- Image: NanoBanana only (GPT-4o 코드 보존되어 있으나 미사용)
- `BOOKS_PROJECT_CONFIG`가 모든 설정의 진입점
