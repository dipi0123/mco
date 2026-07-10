# Memory index

> MCO 프로젝트 지속 메모리 (단일 진실원천). 세션 시작 시 이 인덱스를 읽고 관련 파일을 로드한다. 한 파일 = 한 사실. 새 사실은 `.claude/memory/`에 파일로 추가하고 여기에 한 줄 등록한다.

## 컨셉 코어 — 현재 합의

- [MCO concept core](mco-concept-core.md) — MCO = 메모리 오케스트레이션 레이어(LLM 앞단); 스몰 모델 5역할(가치 판단·범위 결정·충돌 감지·갱신·선택); 세션 기억=LLM 몫, MCO=세션 밖 지속 메모리; MCP+OAuth 계정 바인딩; 단순 메모리 RAG 아님.
- [Positioning: big-tech gap](mco-positioning-bigtech-gap.md) — 개인화 메모리 자체는 빅테크가 이미 함 → 차별점 = 크로스플랫폼 이동성 + 계정 소유 + 에이전트별 권한 스코프; "빅테크가 하기엔 짜치고 사용자는 절실한" 빈틈; 앵커 = Personal Memory Passport.
- [Selection over storage](mco-selection-over-storage.md) — 코어 가치 = 저장이 아니라 상황에 필요한 기억만 꺼내는 선택(노이즈 필터); 실패 기준 = 애플워치 논의에 Setto 기억 주입; recall보다 precision 우선.
- [Memory structure 3-layer](mco-memory-structure.md) — 파이프라인 구상: 수집(지속 기억 후보 리스트업) → 구조화 3층(명시적 저장 정보 · 맥락 추출 의미 · 사건 조각) → 선택.

## 프로덕트 방향

- [MVP direction](mco-mvp-direction.md) — MVP = MCP 메모리 서버 + 브라우저 확장 + 대시보드; 초기 타깃 = 파워유저(개발자·컨설턴트·변호사).
- [Manufacturing positioning](mco-manufacturing-positioning.md) — 제조 GTM: 'LLM 독립형 메모리 레이어' 표현 금지 → '설비·품질·작업자 이력을 기억하고 상황 맞는 과거 사례를 꺼내주는 제조 메모리 컨텍스트'(Factory Memory Context).

## 워크플로

- [Workflow: 작성 gate](workflow-write-gate.md) — 큰 결정은 채팅 정렬 → Ethan "작성" 트리거로만 생성; 임의 선행 생성 금지; 브레인스톰 단계 결정은 '잠김' 아닌 '현재 합의'로 취급.

## 빌드 상태

- [Build state & next steps](build-state-next-steps.md) — 2026-07-09 하네스 구성 DONE(CLAUDE.md·메모리·차터 v0.1 draft); next 후보 = 경쟁 리서치·스몰 모델 실체·메모리 스키마·MVP 스펙·제조 시나리오 (차터 §9).
