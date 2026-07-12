---
name: mco-architecture-v01
description: "아키텍처 코어 합의(2026-07-11, P1~P8 반영 완료) — 정본=텍스트 레코드·벡터=파생 인덱스; Postgres 한 채; 검색 5채널(Kiwi 형태소 FTS·pg_trgm·정확키·pgvector·gen_queries)+RRF; 2계층 기억+승격·강등; 읽기 2티어(쿼리시 LLM 금지); 쓰기=자동 기록+undo; 모델 확정: 2.5 Flash-Lite 핀·bge-m3·Qwen3-Reranker(전부 한국어 벤치 승격제); 관문=정밀도 실험+호출률+부품 벤치 3건"
metadata:
  type: project
---

2026-07-11 정렬 + 같은 날 검증 패치 P1~P6·P8a~f 반영 완료 (정본: `Product/MCO_아키텍처_v0_1.md`, 근거: `docs/리서치/MCO_검증리포트_v0_1.md` · `MCO_한국어검색스택_v0_1.md`):

- **대원칙**: 정본 = 구조화 텍스트 레코드(DB 행), 벡터·FTS·그래프 = 재생성 가능한 파생 인덱스(우리 임베딩 모델 교체·자체 파인튜닝 대비). 저장(커모디티)/지능(투자처)/정본 규칙(보호 장치)의 3층.
- **데이터층·검색(P8 확정)**: Postgres 한 채 + pgvector 0.8. **검색 5채널** — ① 앱사이드 Kiwi 형태소 정규화→tsvector('simple') 한국어 랭킹 FTS(확장 의존 0, 전 관리형 이식) ② pg_trgm 정확·퍼지(한글 trigram 1일차 검증 필수) ③ 정확 키 조회 ④ pgvector dense ⑤ gen_queries·엔티티 임베딩 → RRF(k=60). ParadeDB 기각 유지, PGroonga=Supabase 한정 보조 백스톱만, 외부 엔진 이주 임계값 3개(50만 행+/검색 UX 기능화/리랭커 후 리콜 실패 계측). **한국어 하이브리드 검색 = 경쟁 공백**(mem0 BM25 영어 하드코딩).
- **2계층 기억**: 설정성(상시 주입 T0/T1, 상한 강제) vs 사건성(T2 검색·T3 아카이브). "확실히 필요한 건 주입, 아마 필요한 건 검색." 승격/강등 순환.
- **쓰기(P2)**: 게이트→스코프→충돌검사→**자동 커밋+즉시 undo**. 승인은 충돌·교차 스코프만, 미드태스크 인터럽트 금지, 주간 changelog, 인박스=예외 처리함. ADD-only+invalid_at.
- **읽기 2티어**: 사전 조립 브리핑 팩(쿼리 시점 LLM 금지, ~50–200ms) + 딥서치(5채널→RRF→리랭커). 안티패턴은 적시 주입.
- **모델 확정(P8)**: 추출 = **Gemini 2.5 Flash-Lite 핀**(한국어+코드스위칭 벤치 게이트 조건부, 스왑 후보 GPT-5-mini KMMLU 76.47) / 임베딩 기본값 = **bge-m3**(DeepInfra $0.01/1M — Gemini 대비 15분의 1, Ko-MTEB IR 79.30 실증) / 리랭커 1순위 = **Qwen3-Reranker**(DeepInfra $0.01/1M, 한국어 실측 1위 4B 0.8324), 폴백 bge-reranker-v2-m3. Gemini·Voyage·KURE는 자체 벤치 승격제.
- **연결층**: 무상태 MCP(07-28 개정 대응, Tasks API 금지), OAuth DCR+CIMD, Workers+Hyperdrive 풀링 필수, 디렉토리 등재=출시 요건, 호출 의존 3중 완화(스니펫 원클릭 설치·self-test·호출률 텔레메트리).
- **비용**: 헤비 유저 월 $0.75–1.5, 웜스타트 $1–2(bge-m3 시 임베딩분 ~$0.05).
- **관문(전부 구현 전 API 수준)**: ⓐ 부품 벤치 3건(리랭커·임베딩·추출 — 공개 하네스: instructkr·AutoRAGRetrieval·Ko-StrategyQA·KURE) ⓑ 게이트·선택 정밀도 실험(베이스라인 full-context·BM25 +10점, 기권 클래스, 한국어 슬라이스) ⓒ 호출률 A/B. 라이브 라벨 = undo·제외·수정·충돌 판정.

Why: "여러 모델을 오가도 일관 추론" = 입력 일관성(같은 브리핑 팩). UX 예산에서 역산 + 2회 적대적 검증으로 교정 완료.

How to apply: 문서화 세션(3개 산출물)과 스키마·MVP 스펙·실험 설계는 이 결정에서 출발. 변경 시 이 메모리와 정본 문서 동기 갱신. [[mco-hermes-verdict]] [[mco-viability-verdict]] [[mco-wedge-web-connector]] [[mco-thinking-order]]
