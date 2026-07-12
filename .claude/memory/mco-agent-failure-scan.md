---
name: mco-agent-failure-scan
description: "2026-07-12 AWS '에이전트 실패 5+1 기법'(Strands+AgentCore, Elizabeth Fuentes) 3단 판정 — 4/5 아키텍처·보안 흡수·초과; 넷-뉴 = 집계·전수 질의 조작(§8-2 슬라이스); AgentCore = 담장 안 풀스택(경쟁 감시 등재·서사); 툴 라우터 확장 기각; 벤더 수치(73% 등) 대외 인용 금지"
metadata:
  type: project
---

발단: Ethan 공유 유튜브([why-agents-fail](https://github.com/elizabethfuentes12/why-agents-fail-sample-for-amazon-agentcore), AWS 데브렐 Elizabeth Fuentes) + 참조 파일 3종(AWS 홍보 톤 — 데모 수치를 확정처럼 다듬음) → [[mco-contextual-retrieval-scan]]의 '흡수/실험/서사' 3단 판정 재적용 → Ethan "목표에 맞는 것만 적용" 트리거.

**총평 판정**: 5기법 중 4개는 아키텍처 v0.1·보안 모델이 이미 흡수·초과. 영상 종착점 AgentCore는 MCO의 모델이 아니라 **대조군(담장 안)** — 서사·경쟁 재료. 넷-뉴 기술 항목은 1건(집계·전수 질의 조작).

**적용(2026-07-12, 정본 아키텍처 §8·§9 + 경쟁지형 메모리 반영)**:
- §8-2 **집계·전수(aggregation/whole-set) 질의 슬라이스** — 정답이 관련 기억 전체 집계·카운트·"모든 X"·교차 스코프 합산을 요구할 때 벡터 top-k 샘플링의 조작률 측정. 구조적 질의 경로(pg 집계·관계 조인·엔티티 엣지, **Neo4j 도입 아님** — 정본=Postgres 단일 소스 유지) A/B, 전수 불가 시 §8-3 기권(정직한 제로)과 연결. ※ 충돌③은 이미 §8-3 '모순 페어 합성 F1'로 커버 — 집계/전수는 별 실패 모드라 중복 아님.
- **경쟁 감시 등재**: AWS Bedrock AgentCore(장·단기 메모리+Gateway 시맨틱 툴 선택+가드레일, Strands 결합) → [[mco-competitive-landscape]] 위협·감시. 담장 안 풀스택 = **수요 증거**([[mco-positioning-bigtech-gap]]), 구조적으로 못 하는 것 = 벤더 횡단 중립. 정본 `docs/리서치/MCO_경쟁지형_v0_1.md`는 다음 분기 리프레시 때 흡수.

**흡수(무반영 — 기록만)**:
- Semantic Tool Selection = '저장<선택'의 툴판(역할 ⑤·브리핑 팩). [[mco-selection-over-storage]] 초과 흡수. 토큰 3,000→300(≈89%)은 데모 한정 정당.
- Neurosymbolic Guardrails = 보안 통제 ①②(아키텍처 제약 본체, before_tool_call 하드블록 = write-back 스코프 강제)와 동형. "코드=법칙, 프롬프트=제안"은 Willison에 이은 **독립 외부 검증재** — 차터 §9 보안 서사 보강용. [[mco-security-model]]
- Multi-agent Validation(Executor-Validator-Critic 스웜) = write-back 게이트(①)+결정 인박스의 풀에이전트판. 스웜 전면 채택은 비용·지연으로 **기각**, 스몰모델 트리아지+사람 인박스 노선 유지.
- Runtime Steering = 결정 인박스의 '차단 말고 화해안 제시' UX 노트(교차 스코프 충돌 시 분할·버전 제안).

**기각(명시)**:
- **툴 라우터/툴 스코프 확장**(브리핑 팩이 툴까지 선택) → 오케스트레이션 가드레일(차터 §5: 스케줄링·라우팅 안 함) 위반. 유혹만 기록, 채택 금지.
- **별도 docs/리서치 정본 리포트** → 대부분 흡수라 과함, 이 메모로 대체(원하면 승격).
- **벤더 수치 대외 인용 금지**: "73% 환각 감소"는 영상 원문에 없음 — 참조 파일이 붙인 벤더 인용(웹 확인 시 62/73/89% 편차, 피어리뷰 앵커 부재). 대외 자료엔 자체 한국어 실측만 사용.

Why: 외부 콘텐츠 유입 처리 선례([[mco-contextual-retrieval-scan]]) 재적용 — 같은 영상 재론 방지 + 흡수/실험/서사 분리로 스코프 규율 유지.

How to apply: 정밀도 실험 설계 시 §8-2 집계·전수 슬라이스 포함. 경쟁 분기 리프레시 때 AgentCore를 정본 경쟁지형 doc에 흡수. 대외 문서에 벤더 수치 인용 금지. [[mco-architecture-v01]] [[mco-competitive-landscape]] [[mco-security-model]] [[mco-selection-over-storage]] [[mco-fleet-reframe]]
