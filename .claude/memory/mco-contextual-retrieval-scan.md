---
name: mco-contextual-retrieval-scan
description: "2026-07-12 컨텍스추얼 검색 동향 점검 — 원문(2024-09) 기법은 기존 설계가 흡수·초과(원자=강한 컨텍스추얼화); 적용(기능 한정) = §8 실험 케이스 2(최종 주입 k 스윕·원자 임베딩 입력 A/B) + 임베딩 벤치 후보 Qwen3-Embedding; 보류 = 캐시 친화 팩(스키마 v0.1 입력); 이관 = 롱컨텍스트+캐싱 반론(차터 v0.3 후보); 관망 = voyage-context-4·Claude memory tool GA"
metadata:
  type: project
---

발단: Ethan 공유 유튜브 리뷰 영상(Anthropic [Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval), 2024-09-19 해설) → 원문 검증 + 2026-07 지형 조사 → 채팅 판정 정렬 → Ethan **"MCO 기능에 필요한 것만 적용"** 트리거. 정본 리포트: `docs/리서치/MCO_컨텍스추얼검색동향_v0_1.md`.

**총평 판정**: 영상·원문 기법(하이브리드·리랭킹·쓰기 시점 문맥 보강·상시 평가)은 아키텍처 v0.1이 이미 반영·초과 — 기억 원자의 추출 시점 자기완결 구조화(주체·시점·출처·스코프)가 컨텍스추얼 리트리벌의 강한 형태. 적용 가치는 원문이 아니라 이후 2년의 반증 데이터에서 나옴.

**적용(2026-07-12, 아키텍처 정본 §6·§8·§9 개정 이력 반영)**:
- §8-2 **최종 주입 k 스윕(5/10/20, 리랭커 후)** — 원문 top-20 권고 vs [Chroma context rot](https://www.trychroma.com/research/context-rot)(집중 ~300tok ≫ 전체 113k, 방해 청크 1개도 유해)의 충돌을 실측으로 판정.
- §8-4 **원자 임베딩 입력 A/B** — 본문 단독 vs 스코프·엔티티 메타 프리픽스 결합. gen_queries 중복 여부 판정 목적, 무효과면 "원자 스키마+gen_queries로 충분" 증거로 기록.
- §6·§8-4 **임베딩 벤치 후보 +Qwen3-Embedding-0.6B/4B**(Apache-2.0·MRL, 한국어 공개 실측 얇음 → 2중 게이트 동일, 기대치 낮게). KURE-v1은 2024-12 이후 갱신 정체 병기.

**보류·이관·관망 (기각 아님 — 시점 이동)**:
- 보류: **캐시 친화 팩 직렬화**(정렬 결정론·안정 블록 전치·변동 필드 후치) → 스키마 v0.1 설계 입력. 근거: 캐시 읽기 0.1x·1h TTL GA로 팩 포맷이 사용자 비용 직결, 단 캐싱 주체 = MCP 클라이언트라 우리 제어 밖 + 팩 스펙 부재 상태의 노트 선행은 문서 선행.
- 이관: **"롱컨텍스트+캐싱이면 메모리 레이어 불필요?" 반론 방어**(context rot 실증 + 함대 스케일 + 벤더 횡단 + 정밀도=안전) → 차터 v0.3 후보(포지셔닝이라 이번 '기능' 범위 외).
- 관망: **voyage-context-4**(2026-06-29, 문맥화 청크 임베딩 — LLM 문맥 생성 대체 계열) = 서류 컴포저(경로 B)가 문서 인제스트 갖는 시점 재평가 / **Claude memory tool GA·context editing**(파일 기반·단일 벤더 — 벤더 횡단 공백 유지, 위협 1티어 재확인) = 하네스 플러그인 표면 호환만 주시.

**서사 재료 3(외부 설득용, 1차 출처)**: ① 읽기 2티어 = [Anthropic이 2025-09 공식화한 just-in-time 하이브리드](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)와 동형(브리핑 팩=CLAUDE.md 선로드, 딥서치=grep) ② context rot = '저장<선택'의 벤치마크 실증 ③ memory tool GA·Gemini File Search 무료 = "벤더는 담장 안에서 전부 만들었다" 최신 증거.

Why: 외부 콘텐츠 유입 시 "이미 흡수됐나 → 실험을 바꾸나 → 서사에 쓰나" 3단 판정으로 처리한 선례. 같은 원문/영상 재론 방지.

How to apply: 정밀도 실험 설계 시 §8 추가 케이스 2건 포함. 스키마 v0.1 세션에서 캐시 친화 직렬화 1줄 규칙 반영. 차터 v0.3 세션에서 반론 방어 등재 판단. [[mco-architecture-v01]] [[mco-selection-over-storage]] [[mco-positioning-bigtech-gap]]
