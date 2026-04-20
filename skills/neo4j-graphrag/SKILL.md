---
name: books-neo4j-graphrag
description: Neo4j GraphDB 서비스 — 책 문서/에피소드/씬 CRUD + 청크 관리
user-invocable: false
paths: short-video-maker/src/YTB-books-project/src/services/Neo4jService.ts
---

## 용도
책 콘텐츠를 Neo4j 그래프DB에 저장하고, 에피소드/씬 단위로 관리하며, GraphRAG 패턴으로 다음 에피소드 생성에 필요한 청크를 조회.

## 핵심 파일
- `src/YTB-books-project/src/services/Neo4jService.ts` — 전체 Neo4j CRUD

## 의존성
- `neo4j-driver` (npm)
- Neo4j 인스턴스: `bolt://YOUR_NEO4J_HOST:7687`, user: `YOUR_NEO4J_USER`

## API / 인터페이스
```typescript
class Neo4jService {
  // Document
  createDocument(title: string, metadata: object): Promise<string>

  // Episode
  createEpisode(documentId: string, episodeNum: number, data: object): Promise<string>
  linkEpisodes(prevId: string, nextId: string): Promise<void>

  // Scene
  createScene(episodeId: string, sceneData: object): Promise<string>

  // Chunk
  getChunks(documentId: string, startIndex: number, count: number): Promise<BookChunk[]>
}
```

## Graph Schema
```
(Document) -[:HAS_EPISODE]-> (Episode) -[:HAS_SCENE]-> (Scene)
(Episode) -[:NEXT]-> (Episode)
(Document) -[:HAS_CHUNK]-> (BookChunk)
```
- Episode 노드: episodeNumber, title, summary, latexFormulas
- BookChunk: index, text, embedding (optional)

## 다른 프로젝트에 이식하기
1. `Neo4jService.ts` 복사
2. `npm install neo4j-driver`
3. Neo4j 연결 정보 환경변수로 변경 (현재 하드코딩)
4. 스키마에 맞게 Cypher 쿼리 수정

## 주의사항
- 연결 정보가 코드에 하드코딩되어 있음 — 이식 시 환경변수로 전환 필요
- linkEpisodes로 에피소드 간 순서 보장 (NEXT 관계)
- 청크 인덱스는 0부터 시작, 순차 처리 기반
- 트랜잭션 사용: session.writeTransaction / readTransaction
