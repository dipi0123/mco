# MCO 사업성 분석 v0.1 (2026-07-11)

> **정본.** 2026-07-11 아이디에이션·사업성 세션 산출. 질문: "MCO의 경쟁력이 사실로 확보되는가, 안 되면 빠른 피벗이 맞는가." 방법: 병렬 웹 리서치 — 페인 증거 3트랙 + 사업성 3트랙(Hermes 심층 / 아키텍처 SOTA / 빅테크·시장) + 인프라 3트랙. 모든 핵심 주장에 출처 링크. 차터 v0.2와 경쟁지형 v0.1을 갱신하는 내용 포함(차터 v0.3 반영 대기). **2026-07-11 적대적 검증(`MCO_검증리포트_v0_1.md`) P7 반영 완료.**

---

## 0. 판단 프레임 — 사고 흐름 (2026-07-11 합의)

**① 사용자 needs 존재 → ② needs에 맞는 UX → ③ ①·②를 부합하는 아키텍처 → ④ 구현 기술 — 이 4단계로 상품성이 결론된 뒤에만 pricing을 논한다.** 세션 중 pricing 선행 논의는 순서 위반으로 기각됨.

| 단계 | 상태 (2026-07-11) |
|---|---|
| ① needs 존재 | ✅ 검증 — 행동 증거 확보(§4). 남은 건 페르소나 우선순위 선택 |
| ② UX | 🔶 방향 확정(웹 딸깍·제로 인프라·첫 5분·투명 작동), 첫 5분 플로우 설계 필요 |
| ③ 아키텍처 | ✅ 정렬 — `Product/MCO_아키텍처_v0_1.md` 정본 |
| ④ 구현 기술 | 🔶 부품 전부 실존 확인, **미검증 코어 = 게이트·선택 정밀도 → 실험이 관문** |

---

## 1. 종합 판정표

| 축 | 판정 | 근거 요약 |
|---|---|---|
| 페인 실재성 | **GO** | 재설명(세션당 10–15분·재설정 ~20k 토큰), 오염(기능을 끄는 유저들), 무영향(기억 있어도 행동 불변), 함대 상태 분산 — 전부 행동·지출 증거 |
| 저장 스택(md+벡터+관계형+단기/장기+스코프태그)의 단독 경쟁력 | **NO-GO** | 전 요소 커모디티. 1인 개발자가 동일 조합 무료 복제([Memory OS, 2026-05](https://www.marktechpost.com/2026/06/01/meet-memory-os-a-6-layer-open-source-memory-stack-built-on-top-of-hermes-agent/)) |
| 비저장 요소(게이트 환류·정밀 선택·강제 스코프·TTL/출처) | **조건부 GO** | 시장 희소/부재 실측 — 단 "미해결이라 비어있는" 난제. 증명 부담 = 베이스라인 +10점 |
| 빅테크 정면 충돌 | **당장 회피 구조** | 전원 담장 안 강화, 벤더 횡단은 공백 지속. 시한폭탄 3개(§5) |
| 개인 'GB 저장' 구독 미터 | **NO-GO** | 실증 잔혹사(§6). 구독 형태 자체가 아니라 GB 가치 미터를 기각 |
| Hermes 정면 경쟁 | **NO-GO** | 속도·무료·규모에서 자살 코스. 단 '탑승'(프로바이더 슬롯)은 구조적으로 열림 |

**한 줄 결론: "존재하는가"는 GO, "무엇으로 이기는가"는 저장이 아니라 선택 — 그리고 그 선택 정밀도만이 실험 없이는 알 수 없는 단 하나의 미검증 전제다.**

---

## 2. Hermes Agent 판정 — 경쟁자가 아니라 3역

**실체** (2026-07 실측): Nous Research, 2026-02-25 출시, [GitHub ★212K / 4.5개월](https://github.com/nousresearch/hermes-agent), 릴리즈 주기 ~2주(v0.17 한 번에 커밋 1,475·기여자 245). 메모리 = MEMORY.md(~800tok)+USER.md(~500tok) **하드캡 1,300토큰** 동결 스냅샷 주입 + SQLite FTS5 세션 검색 + **외부 메모리 프로바이더 플러그인 슬롯**(mem0·Supermemory·Honcho·Hindsight 등 9개). "md 기반 히스토리 관리"라는 특성화는 부분적으로만 맞음 — 절차 기억(스킬 자동 생성)과 프로바이더 슬롯이 별도 존재. 스코프(프로젝트/채널 벽)·TTL·출처·충돌 해소 **없음**(전역 단일 기억, [셀프호스트 동기화는 P3 방치](https://github.com/NousResearch/hermes-agent/issues/20510)).

**Hermes Cloud (2026-07-07~08 출시)**: 호스티드 에이전트 + 팀/조직 권한 + 통합 과금. 카피: *"Memory lives with the agent, not the device."* — 멀티 디바이스 문제를 **에이전트 호스팅**으로 풀었다. 기억을 사용자 계정의 독립 자산으로 만드는 방향은 로드맵에 없음.

**판정 — 3역:**
1. **수요 증명**: ★212K의 훅 = "너를 기억하고 함께 자라는 에이전트" → 기억 욕구가 대중 스케일임을 증명. "개발자만 원하는 것 아니냐" 리스크 해소.
2. **런타임 커모디티화**: 에이전트 런타임 무료화($5 VPS·MIT·300+ 모델) → 지속 가치는 런타임 사이를 이동하는 기억으로.
3. **유통 채널**: 프로바이더 슬롯 = 기억·소유에 자기선별된 인구가 모인 저수지. 슬롯 실측 공백 = 정밀도([독립 벤치: Supermemory 36건 중 19건, mem0 20건이 낡거나 엉뚱한 회상](https://izzuddin8803.medium.com/i-tested-hermes-agents-third-party-memory-providers-so-you-don-t-have-to-here-are-what-i-found-94138d7b85fb); 내장은 노이즈 2/36이나 대부분 회수 실패).

**구조 대비 (포지셔닝 문장):** *Hermes = 기억을 가진 에이전트 하나. MCO = 당신의 모든 에이전트가 공유하는 기억 하나.*

**차용 목록**: 상한 강제 큐레이션(캡) · 메모리 도구 인터페이스(꽉 차면 에러, 조용히 버리지 않음) · 주입 전 보안 스캔(인젝션·시크릿). **개선 목록**: 전역→스코프 벽, 동결 스냅샷→서버사이드 라이브 갱신.

---

## 3. 아키텍처 경쟁력 판정 — "저장 스택은 바닥재, 경쟁력은 위층"

- **커모디티(전원 보유)**: md 파일(Hermes·Letta MemFS·CLAUDE.md 관행), 벡터+키워드 하이브리드(mem0 v3·Supermemory), 관계형(Cognee·Honcho·Hindsight), 단기/장기 분리(사실상 전원), 스코프 **태그**(mem0 IDs·AWS 네임스페이스·Supermemory containerTags).
- **희소(소수 보유)**: **강제되는** 에이전트별 권한(AWS IAM 조건 키·Cognee ACL 정도), bi-temporal 유효기간(Zep Graphiti·Hindsight), 일급 TTL(AWS), 리뷰 가능한 write-back(Letta git).
- **부재(아무도 못함)**: 최강 염가 베이스라인을 +10점 이기는 선택 정밀도, 신선도·충돌 해소, 기억→행동 반영, 앱 수준 평가.
- **아키텍처 우위 ≠ 상업적 승리 (실증)**: 시간 모델링 최약 mem0가 채택 1위+AWS 독점 슬롯 / 최고 아키텍처 Zep은 [OSS 중단](https://blog.getzep.com/announcing-a-new-direction-for-zeps-open-source-strategy/) / 최심층 Letta는 [메모리 제품에서 피벗](https://www.letta.com/blog/our-next-phase/) / [LoCoMo 정답지 6.4% 오염·벤치 분쟁·게이밍](https://essays.bloo-mind.ai/posts/2026-05-20-mem-eval/) / 파일시스템+grep 에이전트가 LoCoMo 74%.
- **해자의 실제 출처(실증)**: 유통(AWS·Hermes 슬롯), 기본값 지위, 축적된 사용자 데이터·라벨, 컴플라이언스. 아키텍처는 몇 주면 무료 복제.

**결론**: 경쟁력의 위치 = 커모디티 부품 위에서 **더 정확히 걸러 넣고(게이트) 더 정확히 골라 조립하는(선택) 스몰 모델의 판단력** + 그 판단력을 키우는 **라벨 축적**(결정 인박스 판정·승인/기각 루프 = 평가셋 복리).

---

## 4. 사용자 needs 증거 (요약 — 상세는 세션 로그)

- **페인 서열: 오염 > 무영향 > 부재.** 벤더들이 기억을 다 만들자 파워유저들이 그걸 **끄기 시작**한 것이 최상위 증거(직장↔취미 오염, 에이전시 클라이언트 오염, Cursor 프로젝트 누출로 전체 삭제).
- 재설명: 세션당 재발견 10–15분, 재설정 ~20k 토큰, HANDOFF.md·마스터 프롬프트 수제 생태계.
- 무영향: "기억은 쌓이는데 행동이 안 바뀐다" — [claude-code #37314](https://github.com/anthropics/claude-code/issues/37314).
- 함대(2026 신종): 병렬 세션 상호 덮어쓰기, 형제 세션의 미출하 전제 가정, agent teams는 세션 한정·지속 기억 0.
- 지불 실측: 정부과제 서류 대필 건당 30만~100만 원(1건 53h50m), 회의도구 $8–39/인/월, MemoryPlugin $10–25/월, 개인·업무 계정 이중 구독.
- **"소비자"의 실체 = 컨텍스트가 돈인 프로슈머**(빌러블 아워·마감·서류·멀티 법인). 순수 일반 소비자의 지불 천장은 낮음(의료 $13–16/월).
- **반증 패턴 (2026-07-11 검증 리포트 P7)**: 플랫폼들이 사일로 안에서 부분 해결(project-only memory·자동 기억·임포트)하는 동안 **독립 크로스툴 메모리 런칭은 시장 무관심에 죽는다**(HN 1~3pt·0댓글 반복). 함의: ① 서사는 플랫폼 인센티브와 충돌하는 두 페인에 집중 — **오염**(engagement와 충돌)·**감사/삭제 보증**(광고와 충돌) ② 출력 순간 없는 "레이어" 마케팅 금지 ③ 웜스타트 수요는 벤더 임포트(Anthropic·Gemini 2026-03)로 증명됐으나 일방향 무기화로 창 축소 중 ④ 부패도 검진 = 감사 보고서가 아니라 공유형 재미(roast) 톤.

---

## 5. 빅테크 충돌 (2026-07-10 기준)

| 벤더 | 지금 | 최근접 위협 |
|---|---|---|
| OpenAI | 소비자 메모리 고도화(Dreaming). **Memory API 없음**(요청 누적). Codex 기억 로컬·기본 꺼짐·유럽 제외. 커스텀 MCP는 dev mode 전용("developers only") + [Lockdown Mode](https://www.helpnetsecurity.com/2026/06/08/openai-lockdown-mode-available/) | Memory API 출시 / Codex↔ChatGPT 기억 통합 |
| Anthropic | [Managed Agents 기억(조직/유저 스코프·감사) — Claude 전용](https://claude.com/blog/claude-managed-agents-memory). Claude Code 기억 머신 로컬. 경쟁사 기억 임포트 도구(자사 유입용) | Claude Code 기억 클라우드 동기화·팀 공유 |
| Google | Personal Intelligence(소비자)·Vertex Memory Bank(개발자). 서드파티 주입 경로 없음 | 무료 확대 |
| Microsoft | Copilot Memory 전 테넌트. [업무 데이터 개인화 GA 2026-11 예정](https://windowsnews.ai/article/microsoft-365-copilot-to-gain-memory-based-personalization-in-november-2026.429482). Engram 파일럿(사서 해결) | — |
| AWS | AgentCore Memory 월간 릴리즈(스코프·TTL 최선진). mem0 독점 슬롯 | 스타트업 차별점 흡수 지속 |
| 표준 | [AAIF에 메모리 프로젝트 없음, MCP 2026 로드맵에 메모리 없음](https://blog.modelcontextprotocol.io/posts/2026-mcp-roadmap/), [W3C CG 제안 단계](https://www.w3.org/community/blog/2026/05/18/proposed-group-ai-agent-memory-interoperability-community-group-community-group/) | 표준 선점 창구 열려 있음 |

**구조 판정**: 전원이 담장 안 기억을 강화 중 — **벤더 횡단 중립은 여전히 아무도 안 한다**(차터 v0.2 빈틈 판정 유지). 단 소비자 표면 접근은 빅테크의 **문(門) 정책** 리스크에 노출(ChatGPT dev-mode 게이트·Lockdown) → 하네스/오픈 생태계 병행 유지 근거.

---

## 6. 시장·지불 증거

- **개인 기억 구독 잔혹사**: MemoryPlugin ~6개월 "3,800+ users" 정체 / Limitless **ARR $2.0M**에서 [Meta 흡수(2025-12) 후 구독 무료화·한국 철수](https://mlq.ai/news/meta-acquires-ai-wearables-startup-limitless-ending-sales-of-pendant-device/) / Supermemory 소비자→엔터프라이즈 피벗. **매출 공개한 메모리 회사 0곳.**
- **돈이 도는 곳**: 엔터프라이즈([Engram $98M @ ~$600M, 직원 13명, MS·Notion·Harvey 파일럿](https://www.citybiz.co/article/864393/engram-emerges-from-stealth-with-98m-to-build-enterprise-ai-memory-layer/)), 하이퍼스케일러 인프라. 메모리 스타트업 누적 ~$152M 중 H1 2026에 ~$110M.
- **한국**: [카카오벤처스 → Aristo 'Membase'(벤더 독립 메모리 레이어) 프리시드 2026-06-25](https://startuprecipe.co.kr/archives/5823032) — 국내도 빈 땅 아님. 네이버 AI탭 첫 달 3M, 카카오 ChatGPT 연동 11M 가입 — 수요 폭발 중.
- **수익화 방향 (pricing 유보 하에 방향만 기록)**: 획득 = 무료 + 출력 순간(풀 스토리) / 전환 = 무료가 구조적으로 못 주는 것(런타임 횡단 동기화·벽·팀·정밀) / **GB 미터 기각** — 매출이 저장량에 비례하면 축적을 최적화하게 되어 품질(노이즈 제거) 논거와 자충. 참고: Hermes Cloud도 과금 미터는 저장이 아니라 호스팅.

---

## 7. 전략 경로 + 킬 트리거

**경로 A — 슬롯 전략(정밀도 승부)**: Claude 커넥터·Claude Code·Hermes 프로바이더에 꽂는 precision-first 기억 계정. 승부처 = 노이즈 없는 선택(슬롯 실측 공백). **경로 B — 아웃컴 앱**: 서류 컴포저(정부과제 — 지불 실측 유일, Ethan = 유저 0번, 수동 큐레이션으로 시작 가능). **경로 C — 팀/엔터프라이즈**: 돈은 있으나 Engram $98M과 조우, 지금 아님.

**실행 합의**: A·B의 첫 실험은 둘 다 ~공짜 → **병렬 실행**, 순서(무엇을 먼저 빌드)는 결과로 결정. A 실험 = 게이트·선택 정밀도(프로토콜: `Product/MCO_아키텍처_v0_1.md` §8). B 실험 = 실제 지원서 1건 원장 방식 수작업 시뮬레이션 + 지불 의사 인터뷰 3인.

**킬 트리거 (분기 감시):**
1. 정밀도 실험이 최강 염가 베이스라인(full-context·BM25) 대비 +10점 실패 → 원장 기반 전략 전면 재검토.
2. OpenAI Memory API 출시·개방 → 논거 재점검.
3. Nous가 메모리를 Tool Gateway식 자체 번들 → 슬롯 전략 재검토.

## 8. 감시 항목 (경쟁지형 v0.1 §6에 추가)

Hermes Cloud의 자체 메모리 번들 여부 · Aristo(Membase) 행보 · W3C AI Agent Memory Interoperability CG 발족 여부 · ChatGPT Apps 심사 정책(데이터 최소화 조항 vs 기억 제품) · Claude 무료 커넥터 정책 변화(현재: 무료 1개 슬롯).

---

*작성: 2026-07-11 세션. 근거 상세·전체 출처는 세션 로그 및 본문 링크. 차터 v0.3 갱신 대기 항목: §7 웨지 재개정(웹 커넥터 우선 병행), 사고 흐름 4단계, 킬 트리거.*
