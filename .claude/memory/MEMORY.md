# Memory index

> MCO 프로젝트 지속 메모리 (단일 진실원천). 세션 시작 시 이 인덱스를 읽고 관련 메모리 파일을 로드한다. 한 파일 = 한 사실. 새 사실은 `.claude/memory/`에 파일로 추가하고 여기에 한 줄 등록한다.

## 컨셉 코어 — 현재 합의

- [MCO concept core](mco-concept-core.md) — MCO = 메모리 오케스트레이션 레이어(LLM 앞단); 스몰 모델 5역할(가치 판단·범위 결정·충돌 감지·갱신·선택); 세션 기억=LLM 몫, MCO=세션 밖 지속 메모리; MCP+OAuth 계정 바인딩; 단순 메모리 RAG 아님.
- [Fleet reframe 2.0](mco-fleet-reframe.md) — 2026-07-10: 디자인 센터 = 에이전트 함대를 지휘하는 사람; 에이전트 = 기억의 소비자→생산자(게이트 환류); 사람 채팅 = 함대 크기 1 특수해; 가드레일 = 오케스트레이션·관측 아님.
- [Positioning: big-tech gap](mco-positioning-bigtech-gap.md) — v0.2: '짜쳐서 안 함' → '벤더 횡단 중립이라 못 함'; 차별점 = 조합 해자(소유+횡단 스코프+정밀 선택+게이트 환류+브리핑 팩).
- [Selection over storage](mco-selection-over-storage.md) — 코어 가치 = 저장이 아니라 상황에 필요한 기억만 꺼내는 선택(노이즈 필터); 실패 기준 = 애플워치 논의에 Setto 기억 주입; recall보다 precision 우선.
- [Memory structure 3-layer](mco-memory-structure.md) — 파이프라인 구상: 수집(런 배기가스 포함) → 구조화 3층 → 선택. v0.2: 출처(provenance)·TTL 필드 코어 승격(차터 §4).

## 경쟁·시장

- [Competitive landscape 2026-07](mco-competitive-landscape.md) — 이동성 단독 차별점 사망(혼잡); 사용자 소유·벤더 횡단 스코프 공백; 조합 해자 전부 하는 곳 없음; 위협 1티어 = mem0·Anthropic·AWS AgentCore. 정본: `docs/리서치/MCO_경쟁지형_v0_1.md`.
- [Watchlist](mco-watchlist.md) — 분기 점검 단일 취합: 킬 트리거 3(정밀도+10 실패·OpenAI Memory API·Nous 번들) + 감시 항목(빅테크 메모리 확장·메모리 오픈표준·경쟁사 행보). 출처 = viability·competitive·사업성 §8·경쟁지형 §6.
- [Hermes verdict](mco-hermes-verdict.md) — 2026-07-11: Hermes = 경쟁자 아닌 3역(수요 증명·커모디티화·유통 채널); 정면 경쟁 금지; "기억 가진 에이전트 하나 vs 모든 에이전트가 공유하는 기억 하나"; Hermes Cloud(07-07) 주시.
- [Viability verdict](mco-viability-verdict.md) — 2026-07-11: 저장 스택 커모디티(NO-GO), 경쟁력=게이트·정밀 선택·강제 스코프·라벨 축적; GB 저장 구독 기각; 경로 A(슬롯)·B(서류 컴포저) 병렬 실험; 킬 트리거 3. 정본: `docs/리서치/MCO_사업성분석_v0_1.md`.
- [Term collision: Passport](mco-term-collision-passport.md) — 'Memory Passport' 외부 충돌(MemoryLake 제품명·mem0 비전 문구); 정본 유지하되 재검토 안건, 대외 헤드라인 사용 금지.

## 프로덕트 방향

- [MVP direction](mco-mvp-direction.md) — v0.3: 1차 표면 = Claude 커넥터+웹 대시보드, 하네스·Hermes 병행 심화; 페르소나 = 컨텍스트가 돈인 프로슈머(코딩 파워유저 = 경로 A 채널); 경로 A·B 병렬; 최우선 검증 = write-back 정밀도.
- [Wedge: web connector first](mco-wedge-web-connector.md) — 2026-07-11 재개정(차터 §7 반영 v0.3): 일반 사용자 웹 딸깍 우선 — Claude 커넥터가 소거법상 유일 열린 문(무료 1커넥터 슬롯); 하네스·Hermes = 심화 표면 병행; 코어 서버 하나.
- [Architecture v0.1](mco-architecture-v01.md) — 2026-07-11(P1~P8 반영 완료): 텍스트 정본·벡터 인덱스; 검색 5채널(Kiwi 형태소 FTS·pg_trgm·정확키·pgvector·gen_queries); 2계층 기억; 읽기 2티어; 자동 기록+undo; 모델 확정 = 2.5 Flash-Lite 핀·**bge-m3**·**Qwen3-Reranker**(전부 한국어 벤치 승격제); 관문 = 부품 벤치 3건+정밀도 실험+호출률(전부 구현 전). 정본: `Product/MCO_아키텍처_v0_1.md` · 근거: `docs/리서치/MCO_검증리포트_v0_1.md`·`MCO_한국어검색스택_v0_1.md`.
- [Contextual retrieval scan](mco-contextual-retrieval-scan.md) — 2026-07-12: 원문(2024-09) 기법은 기존 설계가 흡수·초과; 적용(기능 한정) = §8 최종 주입 k 스윕·원자 임베딩 입력 A/B + 임베딩 벤치 후보 Qwen3-Emb; 보류 = 캐시 친화 팩(스키마 입력); 이관 = 롱컨텍스트 반론(차터 v0.3); 관망 = voyage-context-4·memory tool GA. 정본: `docs/리서치/MCO_컨텍스추얼검색동향_v0_1.md`.
- [Agent failure scan](mco-agent-failure-scan.md) — 2026-07-12: AWS 5+1 환각 방지 기법(Strands+AgentCore) 3단 판정; 4/5 흡수·초과(Semantic=역할⑤·Guardrails=보안 통제①②·Multi-agent=게이트); 넷-뉴 = 집계·전수 질의 조작(§8-2 슬라이스); AgentCore = 담장 안 풀스택(경쟁 감시 등재); 툴 라우터 확장 기각·벤더 수치(73%) 대외 인용 금지.
- [Security model: trifecta controls](mco-security-model.md) — 전제: MCO = lethal trifecta 정중앙(민감 메모리 × 비신뢰 유래 write-back × 외부 통신 주입), 방어 본체 = 아키텍처 제약; 통제 6항(신뢰 티어링·write-back 스코프 고정·스캔=보조·대시보드 IAP 게이트·신뢰 헤더 규율·제로 상태 테스트) = 차터 §9 #6·인프라 §9 #5 확정 입력.
- [Ideation 2026-07-11](mco-ideation-2026-07-11.md) — 페인 서열(오염>무영향>부재); 발산 14 재배치(⑨⑩=슬롯 자산, ②⑥=출력 앱); 기각 기록(회의 수확기→회의 전 브리핑 반전, GB 구독, 에스크로 보류).
- [Manufacturing positioning](mco-manufacturing-positioning.md) — 제조 GTM: 'LLM 독립형 메모리 레이어' 표현 금지 → '설비·품질·작업자 이력을 기억하고 상황 맞는 과거 사례를 꺼내주는 제조 메모리 컨텍스트'(Factory Memory Context).

## 워크플로

- [Workflow: 작성 gate](workflow-write-gate.md) — 큰 결정은 채팅 정렬 → Ethan "작성" 트리거로만 생성; 임의 선행 생성 금지; 브레인스톰 단계 결정은 '잠김' 아닌 '현재 합의'로 취급.
- [Thinking order](mco-thinking-order.md) — 2026-07-11: ①needs ②UX ③아키텍처 ④구현기술 → 상품성 결론 후에만 pricing; 비판은 사실 기반+완화책; **인프라·모델 선택 2중 게이트(관리형 호환성+한국어 실측 — P1·P8 재발 방지)**.

## 빌드 상태

- [Build state & next steps](build-state-next-steps.md) — 2026-07-12 최종 산출물 3종 문서화 DONE(`Concept/MCO_시스템개요` · `Product/MCO_아키텍처설명` · `Product/MCO_인프라구축`, mermaid 7 검증); next = 한국어 부품 벤치 3건+정밀도 실험(A)·서류 컴포저 시뮬레이션(B) 병렬 → 차터 v0.3 → 스키마·첫 5분 화면 → MVP 스펙.
