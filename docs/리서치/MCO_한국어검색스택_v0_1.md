# MCO 한국어 검색 스택 검증 v0.1 (2026-07-11)

> **정본.** 검증 리포트 P1(Supabase×ParadeDB 비호환)의 후속 심층 조사 — "그래서 한국어(+영어 혼용) 하이브리드 검색을 뭘로 구현하나"의 확정안. 방법: 3트랙 병렬(① 관리형 PG 한국어 렉시컬 전수 ② 외부 엔진 + BM25 필요성 right-sizing ③ 지능층 한국어 준비도). 한국어 소스 포함. **결과: 검색 스택 확정 + 지능층 교정 2건 추가 발견(리랭커·임베딩 기본값) — 패치 P8, "적용" 대기.**

---

## 0. 심각도 재정의

- **무엇이 부러졌었나**: 코드 이전, 설계 단계의 통합 가정 1건(관리형 프로바이더의 확장 지원 여부 미확인). 코어 설계(텍스트 정본·2계층·2티어·pgvector·비용 모델)는 검증에서 전부 생존.
- **이번 조사가 추가로 찾은 것**: 같은 오류 클래스 2건 더 — 리랭커 1순위(Voyage)와 임베딩 후보(Gemini)가 **한국어 실측 데이터 0인 상태로 핀**되어 있었다. 한국어가 우리 시장인 이상 이건 P1과 동급의 문제.
- **프로세스 교정(재발 방지)**: 모든 인프라·모델 선택에 2중 게이트 — ⓐ 관리형 호환성 매트릭스 확인 ⓑ 한국어 실측 근거 확인(없으면 자체 벤치 게이트 뒤로). 이 규칙을 표준 검증 단계로 채택.

## 1. 확정안 — 검색 스택 (5채널, 전부 Postgres 안)

| # | 채널 | 구현 | 근거 |
|---|---|---|---|
| 1 | **한국어 랭킹 FTS** | **앱사이드 Kiwi 형태소 정규화 → 공백 결합 morpheme 섀도 컬럼 → `to_tsvector('simple')` + GIN** | 한국어 IR 컨센서스: 형태소 토크나이저 ≫ 공백 분리([AutoRAG 벤치](https://medium.com/@autorag/making-benchmark-of-different-tokenizer-in-bm25-134f2f0e72f8) — 공백 분리가 전 지표 최하위). Kiwi = 유일하게 활발히 유지되는 분석기([0.23.2, 2026-06-11](https://pypi.org/project/kiwipiepy/); mecab-ko-dic은 2018년 동결), LGPL, 사용자 사전 API. 쿼리도 동일 토크나이즈. **확장 의존 0 = Supabase/Neon/RDS 전부 이식 가능, WAL/PITR 안전** |
| 2 | 정확/퍼지 매칭 (이름·ID·전문용어) | pg_trgm GIN | ≤10k행에선 지연 무시 가능. ⚠️ **한글 trigram은 로케일 의존** — 배포 DB에서 `show_trgm('공원')` 비어있지 않은지 1일차 검증 필수([실패 사례 실존](https://velog.io/@goatyeonje/N-gram%EC%9C%BC%EB%A1%9C-PostgreSQL-LIKE-%EA%B2%80%EC%83%89-%EC%84%B1%EB%8A%A5-%EA%B0%9C%EC%84%A0%ED%95%98%EA%B8%B0)) |
| 3 | 정확 키 조회 | fact-key/엔티티 정확 매칭 | [Cloudflare Agent Memory의 최고 가중치 채널](https://blog.cloudflare.com/introducing-agent-memory/) — 메모리 검색 실전 선례 |
| 4 | 의미(dense) | pgvector + 다국어 임베딩(§3) | 기존 확정 유지 |
| 5 | (기존) gen_queries·엔티티 임베딩 | 쓰기 시점 사전 생성 | HyDE·엔티티 채널의 우리식 구현 — 유지 |
| 융합 | RRF(k=60) → 리랭커(§3) | | ts_rank에 IDF 없는 약점은 RRF가 순위만 소비 + 리랭커가 정밀도 담당이라 수용 가능 |

**PGroonga 판정 (Supabase 한정 옵션)**: [Supabase 공식 지원 실재](https://supabase.com/docs/guides/database/extensions/pgroonga)(2026-06 로스터 생존, v4.0.6 활발). 단 한국어엔 **형태소가 아니라 bigram 엔진**(TokenMecab은 일본어 사전 전제)이고 스코어링은 **TF만, BM25 없음**, 인덱스 WAL 미기록이 기본(PITR/복구 후 REINDEX 필요), 플랫폼 업그레이드 파손 전례 1건, Supabase 락인. → **채택하면 '보조 리콜 백스톱'으로만, 1번 채널 대체 불가.**

**기각 목록**: textsearch_ko(사전 2018 동결·관리형 불가·사실상 사망) / pg_bigm(RDS 전용, 랭킹 없음) / ParadeDB(관리형 경로 부재 — 기존 P1 판정 유지).

## 2. 외부 엔진 판정 — 현 규모에서 기각, 임계값 명시

- **BM25 레그의 실제 가치(실측)**: 현대 다국어 dense(bge-m3급) 위에서 렉시컬 추가분은 한국어 MIRACL 기준 **+1.3~2.2 nDCG**([BGE-M3 논문](https://arxiv.org/html/2402.03216v5) — mDPR 시대의 +19에서 붕괴). 단 **희귀 용어·ID·숫자 쿼리에선 여전히 BM25가 dense를 이김**([T2-RAGBench, 7.3k 문서](https://arxiv.org/html/2604.01733v1) — 우리 규모·형태와 유사), 리랭커는 1차 회수가 놓친 걸 복구 못 함(정밀도≠리콜). → 렉시컬 = "평균 성능"이 아니라 **리콜 보험**, 우리 규모에선 채널 1~3이면 충분.
- **메모리 시스템 선례 수렴**: Cloudflare = Porter FTS+정확키(검색 엔진 아님) / mem0 v3 = BM25를 부스트 신호로 강등 / Hindsight = BM25 용도를 "고유명사·전문용어·정확 매칭"으로 명시. 아무도 외부 검색 엔진을 안 씀.
- **외부 엔진 비용·운영 실측**: OpenSearch Serverless 유휴 최저 $175~350/mo, 이중 저장소 운영 실패담 실존(동기화 랙으로 "방금 쓴 게 안 보임", 인덱스 파손 4시간 복구, [6개월 운영기](https://navanathjadhav.medium.com/postgresql-full-text-search-vs-elasticsearch-i-ran-both-for-6-months-a585f60c8a5d): <50만 행은 Postgres 유지 권고). 한국 업계의 OpenSearch+nori 표준은 **백만 규모 상품/문서 검색** 선례지 유저당 ≤10k 기억 검색이 아님.
- **이주 임계값(문서화)**: ⓐ 행 수 ~50만+ ⓑ 사용자 대면 검색 UX(자동완성·패싯)가 기능이 될 때 ⓒ 리랭커 후에도 한국어 리콜 실패가 계측될 때 — 그 전까지 외부 엔진 금지.

## 3. 지능층 한국어 준비도 — 교정 2건 + 게이트 1건

| 층 | 기존 핀 | 판정 | 확정 교정 |
|---|---|---|---|
| **리랭커** | Voyage rerank-2.5-lite 1순위 | **교체** — Voyage는 한국어 공개 실측 0(31개 언어 집계뿐) | **1순위 = Qwen3-Reranker-0.6B/4B(DeepInfra $0.01/1M, Apache-2.0)** — [한국어 벤치 18,945쿼리에서 4B가 실측 1위(MRR@10 0.8324), 0.6B 0.8095](https://github.com/instructkr/reranker-simple-benchmark). 셀프호스트 폴백 = bge-reranker-v2-m3(0.8113, 한국어 실전 최다 — [Allganize 실측 +10~14%](https://www.allganize.ai/ko/blog-posts-ko/reranker)). Voyage는 자체 벤치 게이트 뒤로. 참고: 한국어 파인튜닝 포크들이 오히려 베이스보다 낮음 — "한국어 특화" 브랜딩 ≠ 품질 |
| **임베딩** | Gemini embedding 또는 voyage-4-lite 시작 | **기본값 교체** — 둘 다 한국어 공개 실측 0 | **기본값 = bge-m3** — 한국어 실증(Ko-MTEB IR 79.30, recall@10 0.792) + **관리형 실재**: [DeepInfra $0.01/1M](https://deepinfra.com/BAAI/bge-m3/api)·Cloudflare Workers AI $0.012/1M — **Gemini($0.15) 대비 15분의 1 비용**. KURE-v1은 +1.5 NDCG인데 관리형 호스트 없음(셀프호스트 경제성은 규모 후 재검토). Gemini·Voyage는 자체 벤치에서 이기면 승격 |
| **추출 LLM** | Gemini 2.5 Flash-Lite 핀 | **핀 유지 + 게이트 추가** — 한국어 공개 벤치 부재(다국어 프록시뿐) | 자체 한국어 추출 벤치 통과 조건부 유지. 증거 기반 스왑 후보 = **GPT-5-mini(KMMLU 76.47 실측** vs 4o-mini 52.63, [3자 평가](https://github.com/daekeun-ml/evaluate-llm-on-korean-dataset)). 벤치에 **한·영 코드스위칭 케이스 필수**(EN-KO 혼용 입력 자체는 문제없으나 출력 언어 혼동이 실증된 실패 모드 — 출력 언어·스키마 안정성 assert) |

**한국 특화 공급자 판정**: Upstage 임베딩(솔라)은 벤더 주장뿐+bge-m3 대비 10배 가격, Solar Pro는 추출용 소형 티어 아님, Naver HCX 소형은 2025-26 공개 실측 부재 — **현 단계 글로벌 mini + bge-m3 조합이 증거 우위.** 예외: 데이터 주권 요구 시 HyperCLOVA X SEED 14B(오픈, KMMLU 66.49) 셀프호스트 옵션 기록.

**한국 프로덕션 선례**: [네이버 플레이스 RAG](https://medium.com/naver-place-dev/backoffice-ai-agent-%EA%B5%AC%EC%B6%95%EA%B8%B0-rag-mcp-%EA%B8%B0%EB%B0%98-%ED%94%8C%EB%A0%88%EC%9D%B4%EC%8A%A4ai-%ED%8A%B9%ED%99%94-%EC%A7%80%EC%8B%9D-%EA%B2%80%EC%83%89-%EC%8B%9C%EC%8A%A4%ED%85%9C-9a66b4afa1aa)도 한국어 전용 모델이 아니라 다국어 모델+한국어 정규화 조합 — 우리 확정안과 같은 방향.

## 4. 남는 자체 벤치 3건 (정밀도 실험에 통합 — 공개 하네스 재사용)

1. 리랭커: Qwen3-0.6B/4B vs bge-reranker-v2-m3 vs Voyage-lite — 우리 한국어 기억 데이터 + [instructkr 하네스](https://github.com/instructkr/reranker-simple-benchmark).
2. 임베딩: bge-m3(기본) vs Gemini vs voyage-4-lite vs KURE-v1 — KURE 하네스 + AutoRAGRetrieval·Ko-StrategyQA.
3. 추출: Flash-Lite vs GPT-5-nano/mini — 한국어+코드스위칭 트랜스크립트 구조화 추출, 출력 스키마·언어 안정성 포함.

## 5. 패치 P8 (아키텍처 v0.1 수정안 — "적용" 대기)

- **P8a (§4)**: 검색 채널 확정 — Kiwi 형태소 섀도 컬럼+tsvector('simple') 1번 채널, pg_trgm(배포 DB 한글 trigram 1일차 검증 절차 포함), 정확 키 채널 명시. PGroonga = Supabase 한정 보조 백스톱 옵션. 외부 엔진 이주 임계값 3개 명문화.
- **P8b (§5·§6)**: 리랭커 1순위 = Qwen3-Reranker(DeepInfra), 폴백 bge-reranker-v2-m3, Voyage는 벤치 게이트 뒤로.
- **P8c (§6)**: 임베딩 기본값 = bge-m3(DeepInfra/Workers AI, $0.01~0.012/1M — 비용 모델 개선 반영), Gemini·Voyage·KURE는 벤치 승격제.
- **P8d (§6·§8)**: 추출 모델 한국어+코드스위칭 자체 벤치 게이트, 스왑 후보 GPT-5-mini 명시.
- **P8e (§8)**: 한국어 공개 하네스(instructkr·AutoRAGRetrieval·Ko-StrategyQA·KURE) 명시.
- **P8f (프로세스, 메모리 반영)**: 인프라·모델 선택의 2중 게이트 — 관리형 호환성 확인 + 한국어 실측 확인.

---

*관련 정본: `Product/MCO_아키텍처_v0_1.md`(P8 반영 대상) · `docs/리서치/MCO_검증리포트_v0_1.md`(P1 발원) · 전체 출처는 본문 링크.*
