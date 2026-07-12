# MCO 레퍼런스 아키텍처 v0.1 (2026-07-11)

> **정본.** 전제: 호스티드 백엔드 + 전용 스몰 모델(5역할) + MCP/OAuth로 모든 표면 서비스. **UX 최우선 — 사용자가 원하는 바를 백엔드가 최대한 처리.** 모든 선택에 현업 실측 근거. **구현 전에 싸게 검증되는 미검증 코어는 게이트·선택 정밀도 하나**(§8 실험이 관문, 제품·유저 없는 API 수준 eval). 단 이건 미검증의 전부가 아니다 — 수요·행동 변화(무영향)·코어 지불 의사(WTP)는 인터뷰가 아니라 출시(실사용·실결제)로만 검증되는 별도 다발이다(§8·§9). 사고 흐름: needs→UX→아키텍처→구현 (pricing 유보). **2026-07-11 적대적 검증(`docs/리서치/MCO_검증리포트_v0_1.md`) 패치 P1~P6 + 한국어 검색 스택 심층 검증(`docs/리서치/MCO_한국어검색스택_v0_1.md`) 패치 P8a~P8f 반영 완료** — 주요 개정: 검색 채널 확정(Kiwi 형태소 레그·5채널)·기록-후-되돌리기 역전·리랭커 1순위 Qwen3-Reranker·임베딩 기본값 bge-m3·호출률 계측·인프라/모델 선택 2중 게이트.
>
> **2026-07-12 정합 감사 반영(일관성·정직 라벨링만, 설계·수치 불변):** §0 '미검증 코어 단 하나'→'pre-build 1개 + post-ship 다발' 이원화 / §1 무영향 need 행 = §8 반영 채점으로 측정 표기 / §4 gen_queries 봉합 메모(회수는 넓게·정밀은 리랭커+k캡) / §5 적시주입 하네스 스코프 / §8-1 한국어 라벨셋 / §8-5 **반영 채점 슬라이스(측정 항목) 추가** / §9 'Semantic Tool Selection=역할⑤'→유비. 신규는 §8 반영 채점 **측정 항목 1건**뿐 — 반영도 기반 능동 개입 런타임 상시 컴포넌트로의 승격은 실험 후 별도 결정.

**제품 한 줄**: *Hermes = 기억을 가진 에이전트 하나. MCO = 당신의 모든 에이전트가 공유하는 기억 하나.* — 어떤 모델·표면으로 바꿔도 같은 진실이 조립되어 주입된다("원 브레인, 멀티 서피스").

---

## 1. UX 약속 → 기술 예산

| UX 약속 | 기술 예산 (실측 근거) |
|---|---|
| 딸깍 연결 — OAuth 한 번, 모든 표면 | OAuth 2.1 + DCR/CIMD 동시 지원, 단일 캐노니컬 URL ([Supermemory 패턴](https://supermemory.ai/mcp/)) |
| 첫 5분 — 연결하자마자 보여줌 | 웜스타트 임포트(비동기 잡) + 첫 리포트. claude.ai엔 elicitation 없음 → OAuth 리다이렉트 페이지 = 온보딩 화면 |
| 회상 무감각 | 업계 수렴: [Zep P95<200ms](https://blog.getzep.com/scaling-agent-memory-zep-30x/)·AgentCore ~200ms·[Supermemory 500ms 예산](https://supermemory.ai/blog/latency-budgets-memory-retrieval) → **쿼리 시점 LLM 금지** |
| 작동 투명·신뢰 가시 | 추출·통합 백그라운드(현업 정상: 4초~40초 지연) + **자동 기록·즉시 undo 기본, 주간 changelog, 승인은 충돌·교차 스코프만**(승인 피로·Cursor 승인형 제거 실증 — 검증 P2) + 웹 대시보드 = 예외 처리 인박스·감사·권한·원클릭 수출 |
| 무영향 해소 (기억→행동 반영) — 페인 2위 | **측정 = §8 '반영 채점 슬라이스'**(주입 브리핑↔런 출력 반영 채점, 관측 아님 경계 내 기억 유용성 채점) + §8-2 최종 주입 k 스윕(k=0 슬라이스 = 주입 유/무 비교). 입력 성형(브리핑 팩·적시주입·승격순환)이 반영을 만들고 채점이 그 성패를 잰다 — 인터뷰 아닌 출시(경로 B 라이브 라벨)로 검증. **반영도 기반 능동 개입 런타임 컴포넌트 승격은 실험 후 결정.** |

## 2. 시스템 개요

```
[Claude 웹/모바일(커넥터)]──┐
[Claude Code / Cursor(MCP)]─┼→ 원격 MCP 서버(무상태, Workers) → 코어 API → Postgres(+pgvector+FTS/trgm)
[Hermes(프로바이더 플러그인)]┘        ↑ OAuth 2.1(DCR+CIMD)          ↓↑
[웹 대시보드 = 결정 인박스·감사·권한] ←──────────────── 백그라운드 파이프라인(큐=PG, 스몰 모델 5역할)
```

데이터 흐름: 수집(챗·런 배기가스) → ①게이트 ②스코프 ③충돌검사 → **DB에 텍스트 레코드 커밋** → 인덱싱(벡터·FTS·관계) → 질의 시 ⑤선별·조립 → **MCP로 텍스트 컨텍스트 전달** → 어떤 프론티어 모델이든 같은 입력으로 추론.

**대원칙**: 정본 = 구조화 텍스트 레코드(DB 행). 벡터·FTS·그래프 = 정본에서 재생성 가능한 파생 인덱스. 이유: 우리 임베딩 모델 교체·업그레이드(자체 파인튜닝 포함) 시 전량 재인덱싱이 무손실 배치 작업이 되도록 — 지능 투자를 보호하는 배치 규칙이다. 프론티어 모델에 최종 전달되는 것은 어차피 텍스트(MCP). *정합 감사(발견 14): 단 사전 조립 브리핑 팩은 완전한 파생 인덱스가 아니다 — 팩 멤버십(승격/강등)이 정본에 없는 런타임 사용 상태(인출 빈도)에 의존한다. 무손실 재생성을 지키려면 티어 멤버십·인출 빈도를 정본 필드로 승격하거나, 팩을 '파생 인덱스'가 아니라 '정본+누적 라벨 입력의 규칙 캐시'로 명시 재분류한다(이주 시 순수 파생만 재생성한다는 따름정리 범위에서 팩 제외).*

## 3. 연결층

- **원격 MCP 서버**: Streamable HTTP, 루트 경로, RFC 9728, **무상태 설계 필수** — [2026-07-28 MCP 개정이 세션 핸드셰이크를 제거하는 파괴적 변경](https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/). 세션 의존 설계 금지.
- **OAuth 2.1**: DCR + CIMD 둘 다(Anthropic이 [고트래픽 커넥터에 CIMD 권장](https://claude.com/docs/connectors/building/authentication)), PKCE S256. 인증은 빌드하지 말 것 — workers-oauth-provider(무료) 시작, 필요 시 WorkOS/Auth0(MCP 전용 지원 GA).
- **호스팅**: Cloudflare Workers + McpAgent — [Asana·Atlassian·Linear·Stripe 등 검증한 최다 프로덕션 경로](https://blog.cloudflare.com/mcp-demo-day/). 상태는 우리 백엔드에(DO 아님). Workers→Postgres는 **Hyperdrive/커넥션 풀링 필수** — 비풀링 크로스리전 커넥션 셋업 ~625ms+로 예산 초과, 풀링 시 수~수십 ms(검증 P5).
- **도구 설계 원칙**: 적고, 겹치지 않고, when-to-use가 설명에 명시(호출 신뢰성 = [설명 엔지니어링](https://sunpeak.ai/blogs/debugging-claude-connectors/)). 전 도구 `title`+`readOnlyHint`([Anthropic 디렉토리 요건](https://claude.com/docs/connectors/building/submission) + ChatGPT 읽기 마찰 제거). 초안: `memory_brief`(티어1, read) / `memory_search`(티어2, read) / `memory_remember`(자동 write — 즉시 undo 토큰 반환, 미드태스크 승인 프롬프트 금지 — 검증 P2) / `memory_audit`(출처·접근 로그, read). 2025-11-25 스펙의 실험적 Tasks API는 사용 금지(07-28 개정에서 파괴적 이행 — 검증 T1).
- **표면 어댑터**: ① Claude 커넥터(웹·데스크톱·모바일 동일 — **무료 플랜 커스텀 커넥터 1개 제한** = 그 한 슬롯을 이기는 게임; **Anthropic 디렉토리 등재 = 출시 요건**, Claude가 작업 중 커넥터를 제안·원클릭 설치하는 일반인 유입 경로, 심사 실측 2주~수개월 예산 — 검증 P3) ② Claude Code·Cursor(MCP) ③ [Hermes 네이티브 프로바이더 플러그인](https://hermes-agent.nousresearch.com/docs/developer-guide/memory-provider-plugin/)(MCP 아님, 같은 코어 API의 얇은 어댑터) ④ ChatGPT(후순위: dev mode는 개발자 전용 게이트, 일반인 경로는 Apps 디렉토리 심사 — 데이터 최소화 정책과의 긴장 기록) ⑤ 웹 대시보드(결정 인박스).
- **알려진 한계·대응**: claude.ai 연결 무음 실패 악명 → 전 레이어 로깅 1일차부터. 300s 타임아웃/150k자 결과 캡(여유). 웹 표면은 모델이 도구를 불러야 작동(주입 제어권 없음) → 첫 경험은 '연결하자마자 보여주는'(웜스타트·검진 = 우리가 제어하는 호출), 자동 주입 완전판은 하네스에서. **호출 의존 3중 완화(검증 P3·P6)**: ⓐ 온보딩 마지막 단계 = "작업 시작 전 memory_brief 호출" 스니펫을 사용자 지침/프로젝트 지침에 **원클릭 설치**(효과가 문서화된 유일한 워크어라운드를 기본값화) + 연결 직후 도구 발화 self-test ⓑ 대시보드에 앱별 **회상 호출률 텔레메트리** — 침묵 열화를 가시 열화로(ChatGPT 2025-10 커넥터 미노출 회귀 전례, 플랫폼 회귀 조기 경보) ⓒ 첫 5분(웜스타트·검진)은 우리가 제어하는 호출 = 첫인상을 모델 변덕에서 분리.

## 4. 데이터층

**Postgres 한 채(+pgvector) — 현업의 수렴**: Supermemory(PG+pgvector)·Honcho(PG, 큐까지 PG)·Letta Cloud(Aurora PG)·Hindsight(PG); 그래프DB-우선이던 Zep은 [스케일 위해 벡터·BM25를 그래프 밖으로 분리](https://blog.getzep.com/scaling-agent-memory-zep-30x/). 유저당 수백~수만 건은 pgvector 2~6ms 영역, 0.8.x iterative index scan이 스코프 필터드 벡터 검색을 해결. **검색 채널 확정(P8 — 상세 근거: `docs/리서치/MCO_한국어검색스택_v0_1.md`)**: ① **한국어 랭킹 FTS = 앱사이드 Kiwi 형태소 정규화 → morpheme 섀도 컬럼 → `to_tsvector('simple')`+GIN** — 한국어 IR 실측에서 형태소 토크나이저 ≫ 공백 분리, Kiwi가 유일한 활성 유지 분석기(2026-06 릴리즈; mecab 사전은 2018 동결), 쿼리도 동일 토크나이즈. 확장 의존 0 = Supabase/Neon/RDS 전부 이식, WAL/PITR 안전 ② **pg_trgm 정확·퍼지 채널**(이름·ID·전문용어 — ⚠️ 한글 trigram은 로케일 의존: 배포 DB에서 `show_trgm('공원')` 비어있지 않은지 **1일차 검증 필수**) ③ **정확 키 조회 채널**(Cloudflare Agent Memory 최고 가중치 채널 선례) ④ pgvector dense ⑤ gen_queries·엔티티 임베딩(쓰기 시점 사전 생성). 융합 = RRF(k=60, 레그당 ~20 후보). ParadeDB pg_search **기각 유지**(검증 P1: 관리형 경로 부재). **PGroonga = Supabase 한정 보조 백스톱만**(한국어엔 bigram 엔진·TF-only 스코어링·인덱스 WAL 미기록 기본 — 1번 채널 대체 불가). **외부 엔진 이주 임계값 3개**(충족 전 금지): 행 ~50만+ / 사용자 대면 검색 UX 기능화 / 리랭커 후 한국어 리콜 실패 계측. 참고: mem0 BM25는 영어 하드코딩(CJK 무음 무력화, [#4884](https://github.com/mem0ai/mem0/issues/4884)) → **한국어 하이브리드 검색 자체가 경쟁 공백**. 테넌트 격리 = RLS + 문서화된 완화책(initPlan 래핑·정책 컬럼 인덱스). 관리형 1순위 Supabase(auth 번들, 대시보드와 공유), 대안 Neon. 전용 벡터DB(Turbopuffer 등)는 시작점이 아니라 **이주 경로**(Cursor·Notion 전례).

**기억 원자 스키마** (한 건 = 한 행):

```
id           : SHA-256 콘텐츠 주소 (멱등 재수집 — Cloudflare 패턴)
content      : 1~3문장 자연어 (모델 불가지론)
type         : 사실 | 선호 | 결정(+근거+기각된 대안) | 실패/안티패턴 | 약속 | 에피소드요약
scope        : 계정 / 법인 / 프로젝트 / 에이전트        ← 오염 페인(벽). 태그가 아니라 권한 강제
provenance   : 출처 세션·날짜 + 단언 주체(유저 발화 vs 에이전트 추론)  ← "왜 알아?" 감사
trust_level  : 사람 단언 > 검증된 런 산출 > 비신뢰 콘텐츠 유래    ← 저신뢰는 고권한 문맥 자동 주입 금지(보안 통제 ①)
ttl_class    : 불변 | 저부패(선호·스택) | 고부패(현재 상태) | 만기고정(계약·마감)
bi-temporal  : created_at / valid_at / invalid_at / expired_at   ← Graphiti 패턴, 삭제 대신 무효화
status       : active | deprecated | conflicted
relations    : supersedes / conflicts-with / derived-from        ← 모순 동시-인출 방지
confidence   : 게이트 판정 + 사용 피드백으로 갱신
gen_queries  : 쓰기 시점 생성 검색 쿼리 3~5개(별도 임베딩)      ← 회수율을 쓰기 시점에 구매
```

> *정합 감사 메모(모순 아님, 봉합): gen_queries·5채널은 후보 **회수(recall)**만 넓히고, **선택 정밀도(precision)**는 리랭커 + 최종 주입 k-캡이 강제한다. '저장<선택'은 저장·검색을 줄이라는 뜻이 아니라 **최종 주입 단계의 규율**이다 — 표준 2단계 IR로 정합한다(후보는 넓게, 선택은 좁게).*

**2계층 기억 — 물리 법칙이 다르다** (2026-07-11 합의):

| | 설정성 기억 (Hermes 차용) | 사건성 기억 (벡터가 일하는 곳) |
|---|---|---|
| 내용 | 정체성·톤·선호·스택·규칙·환경 = "에이전트 세팅" | 결정·약속·사건·실패·논의 이력 = 원장 |
| 필요 패턴 | 해당 스코프에서 **항상** — 놓치면 사고 | 가끔, 질문 따라 — 검색으로 |
| 방식 | **검색 안 함. 상시 주입** — T0 코어(~500tok) + T1 스코프 세트(스코프당 상한 강제) | **하이브리드 검색** — T2 원장(벡터+BM25+관계→리랭커) + T3 에피소드 아카이브(재추출 광산) |

원칙: **확실히 필요한 건 주입, 아마 필요한 건 검색.** Hermes에서 차용: 상한 강제 큐레이션·메모리 도구 인터페이스(꽉 차면 에러)·주입 전 보안 스캔(인젝션·시크릿). 개선: 전역→스코프 벽, 동결 스냅샷→라이브 갱신. **승격/강등 순환**(역할④): T2 반복 인출→T1 승격, T1 미사용→T2 강등 — 상한이 순환을 강제, "발전"의 메커니즘.

## 5. 파이프라인

**쓰기(환류)**: 후보 추출 → ①가치 게이트(남길/버릴 이유 명시) → ②스코프 배정 → ③충돌 검사 → **자동 커밋(승인 없음, 즉시 undo)** → 인덱싱. **기록-후-되돌리기 원칙(검증 P2)**: 승인 요구는 ⓐ충돌(사람만 판정 가능) ⓑ교차 스코프 접근 둘뿐. 미드태스크 인터럽트 금지. 가시성 = 주간 changelog("이번 주 배운 것 N개" — 항목별 원클릭 제외·수정·출처). 결정 인박스 = 모든 쓰기의 관문이 아니라 **예외 처리함**(차단형 사람 승인은 **충돌·교차 스코프 둘뿐**; 승격/강등은 수면 사이클이 자동 집행하는 비차단 항목, 만기 임박은 정보성 알림 — 방치해도 시스템은 default로 굴러감). append-only+invalid_at 스키마 덕에 undo는 무비용. 설계 규칙(현업 교훈 반영): **LLM은 판단만, 기계적인 건 비-LLM**(중복=LSH/임베딩 유사도, 시간 질의=날짜 연산 — Zep·Cloudflare 교훈), **ADD-only + invalid_at 마킹, 파괴적 재작성 금지**([mem0 v3가 LLM UPDATE/DELETE를 폐기한 이유](https://mem0.ai/blog/mem0-the-token-efficient-memory-algorithm): 덮어쓰기가 핵심 정보를 지움). 큐 = Postgres 자체(Honcho deriver 패턴 — 2~4인 팀 인프라 최소). 지연 예산: 초~수십 초(현업 정상).

**읽기 — 2티어** (Supermemory `/v4/profile`·Honcho representation·Zep context block 3사 수렴 패턴):
- **티어1 — 사전 조립 브리핑 팩**: 스코프별 백그라운드 사전 조립(materialized) → 도구 호출 시 DB 읽기 즉시 반환(~50–200ms). 쿼리 시점 LLM 없음. 팩 구성: 정체성 / 관련 결정(+기각 대안) / 제약 / 안티패턴 / 미결 약속 — 줄마다 출처 태그. **같은 팩이 어떤 모델에든 = 입력 일관성.**
- **티어2 — 딥 서치**: 하이브리드 5채널(Kiwi-FTS/벡터/gen_queries·HyDE/엔티티/정확키) → RRF → 크로스인코더 리랭커 — **1순위 Qwen3-Reranker-0.6B/4B(DeepInfra $0.01/1M, Apache-2.0)**: 한국어 18,945쿼리 실측에서 4B MRR@10 0.8324 **1위**, 0.6B 0.8095 (P8). 폴백 = bge-reranker-v2-m3(0.8113, 한국어 실전 최다; 소형 GPU ~80ms·CPU 불가). Voyage rerank-2.5-lite는 한국어 공개 실측 0 → 자체 벤치 승격제; zerank-2는 비상업 라이선스로 협의 항목(검증 P4). LLM 리스트와이즈 리랭킹은 실시간 금지(1–3s+).
- 안티패턴은 상시 주입이 아니라 **관련 행동 감지 시 적시 주입**(상시 주입은 무시됨 — #37314 교훈). *정합 감사 메모: '행동 감지→push'는 티어1(정적 사전조립)·티어2(모델 pull) 어느 쪽에도 담기지 않으므로 **자동 주입 제어가 가능한 하네스 표면 전용**으로 스코프하고, 웹에서는 티어2 pull(+호출 의존)로 열화됨을 명기한다.*

**유지(수면 사이클)**: 중복 병합·미사용 감쇠·TTL 만기 강등·충돌 스윕·스키마 개선 시 T3 재추출. 배치 API(50% 할인) 활용.

## 6. 지능층 (스몰 모델 5역할)

- **쓰기 경로(①–④)**: **Gemini 2.5 Flash-Lite에 모델 핀**($0.10/$0.40 per 1M — 3.1 Flash-Lite는 $0.25/$1.50로 비용 2.5~3.75×, §7 비용 모델은 2.5 기준. 검증 P4) — 구조화 출력 테스트 100% 유효 JSON, [Honcho가 파인튜닝해 프로덕션 운용하는 베이스](https://plasticlabs.ai/blog/research/Benchmarking-Honcho). 대안 GPT-5-nano($0.05/$0.40). **한국어+코드스위칭 자체 벤치 게이트 조건부**(Flash-Lite 한국어 공개 벤치 부재 — 증거 기반 스왑 후보 = GPT-5-mini, KMMLU 76.47 실측; 벤치에 한·영 혼용 케이스 + 출력 언어·스키마 안정성 assert 포함. P8).
- **읽기 경로(⑤)**: 티어1 = 사전 조립(LLM은 백그라운드에서만), 티어2 = 리랭커.
- **임베딩(P8 확정)**: **기본값 = bge-m3** — 한국어 실증(Ko-MTEB IR 79.30, recall@10 0.792) + 관리형 실재([DeepInfra $0.01/1M](https://deepinfra.com/BAAI/bge-m3/api)·Cloudflare Workers AI $0.012/1M) = **Gemini($0.15) 대비 15분의 1 비용**. 참고: OpenAI 3-large는 한국어 [리콜 −17.1%](https://github.com/nlpai-lab/KURE). Gemini embedding·voyage-4-lite(둘 다 한국어 공개 실측 0)·KURE-v1(+1.5 NDCG이나 관리형 호스트 없음, 2024-12 이후 갱신 정체)·Qwen3-Embedding-0.6B/4B(Apache-2.0·MRL, 다국어 MTEB 상위이나 한국어 공개 실측 얇음 — 2026-07-12 후보 추가)는 **자체 한국어 벤치에서 이기면 승격**. 주의: Gemini Embedding 2는 001과 공간 비호환(교체=전량 재임베딩 — 텍스트 정본 규칙이 배치 작업화), Gemini 배치 캡 500k tok. MRL 1024.
- **파인튜닝 경로(라벨 축적 후)**: Fireworks LoRA 훈련 $0.50/1M tok·파인튜닝 모델 서빙 추가비 0, [증류로 5–30× 비용 절감 실증](https://www.tensorzero.com/blog/distillation-programmatic-data-curation-smarter-llms-5-30x-cheaper-inference/). 자산의 본체 = 스키마 + 정밀도 평가셋 + 사용자 라벨(차터 §9 가설 유지).
- **보안 — 통제 6항 (2026-07-12 합의, 정본: `.claude/memory/mco-security-model.md`)**: 전제 = MCO는 [lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)(비공개 데이터 접근 × 비신뢰 콘텐츠 노출 × 외부 통신)의 세 변을 구조로 갖는 제품 — write-back이 비신뢰 유래 콘텐츠를 흡수하고 그 기억이 외부 통신 가능한 에이전트에 주입된다. **방어 본체 = 아키텍처 제약, 스캔 = 보조**(가드레일 95% 차단 = 앱보안 낙제 — Willison). ① 기억 신뢰 티어링: provenance에 trust_level(사람 단언 > 검증된 런 산출 > 비신뢰 유래), 저신뢰는 고권한 문맥 자동 주입 금지 — 강제 지점 = 게이트·선택(역할 ①⑤), 스키마 v0.1 필드 ② write-back = 중대 행동: 런은 자기 스코프에만 write, 교차 스코프 write는 예외 없이 사람 승인(RLS read 격리와 대칭 — confused-deputy 봉쇄) ③ 주입 전 스캔(인젝션·크리덴셜 — Hermes 차용): 유지하되 보조로 명기 ④ 대시보드: 인증 자체 빌드 금지 — [identity-aware proxy](https://www.pomerium.com/blog/secure-access-for-mcp) 앞단 게이트(§9 미결 ⓑ의 입력) ⑤ 신뢰 헤더 규율: forwarded 계열은 엣지 세팅분만 신뢰, 클라이언트 신원 헤더는 경계에서 스트립([OpenClaw trusted-proxy 사례](https://github.com/openclaw/openclaw)의 표준 함정 — 헤더 스푸핑 = 인증 무력화) ⑥ 제로 상태 인증 테스트: 온보딩·OAuth·첫 5분 경로는 신규 미연결 계정으로 검증. 스코프 = 정책 강제(태그 필터 아님)는 §4 데이터층(RLS)과 함께 유지.

## 7. 비용 모델 (인프라 성립성 — pricing 아님)

헤비 유저(일 5세션×10k tok): 5역할 파이프라인 ≈ 세션당 $0.005–0.01 → **월 $0.75–1.5**. 티어1 회상 ≈ DB 읽기(무료급), 딥서치 리랭킹 일 20회 = 몇 센트. 웜스타트(3년치 ~5M tok 발굴+임베딩) ≈ **1회성 $1–2**(배치 시 절반; 임베딩 기본값 bge-m3 $0.01/1M 적용 시 임베딩분 ~$0.05 — 사실상 추출 비용만 남음). 캐주얼 유저 월 몇 센트. → **무료 티어 인프라적으로 성립**(웜스타트 깊이 캡). 참고 과금 구조: mem0(월 add/retrieval 건수), Supermemory(중복 제거 토큰).

## 8. 정밀도 실험 프로토콜 (관문 — 사고 흐름 ④의 판정 수단)

1. **게이트(①)**: 실제 트랜스크립트 층화 샘플 300–500청크 → "기억할 가치" 이진 루브릭(Claude Code 휴리스틱+Devin Knowledge 카테고리 시드) → 2–3인 라벨(파일럿 κ 확인, 5–15% 이중 라벨) → precision/recall. (라벨셋에 **한국어 트랜스크립트 슬라이스 명시 포함** — 관문 계측 모델=추출 LLM이 한국어 미검증이므로 §8-4 부품 벤치와 언어 기준을 맞춘다.)
2. **선택(⑤)**: LongMemEval 2층 — 회수 P/R@k({10,20,50}) + 최종 QA 정확도(고정 LLM 저지, ~100건 인간 검증). **베이스라인 = full-context와 BM25/grep 둘 다** ([LoCoMo에서 full-context 73%가 mem0 68%를 이김](https://blog.getzep.com/lies-damn-lies-statistics-is-mem0-really-sota-in-agent-memory/)). 킬 기준: 최강 염가 베이스라인 +10점. **모든 점수는 토큰/쿼리·p95와 쌍으로 보고.** **+ 최종 주입 k 스윕(5/10/20, 리랭커 후 주입 개수별 QA 정확도 — 2026-07-12 추가)**: 문서 RAG의 top-20 권고([Anthropic Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval))와 반대 방향 실증([Chroma context rot](https://www.trychroma.com/research/context-rot): 방해 청크 1개도 유해, 집중 ~300tok ≫ 전체 113k)이 충돌 — 기억 주입 상한은 실측으로 확정. **+ 집계·전수(aggregation/whole-set) 질의 슬라이스(2026-07-12 추가 — 에이전트 실패 기법 스캔)**: 정답이 관련 기억 전체의 집계·카운트·"모든 X"·교차 스코프 합산을 요구하는 질의에서 벡터 top-k 샘플링의 조작률 측정 — 유사도 top-k가 전수를 추측하는 실패 모드(AWS Graph-RAG 데모의 우리식 번역: 유사도 검색 vs 구조적 질의). 구조적 질의 경로(pg 집계·관계 조인·엔티티 엣지, Neo4j 도입 아님)가 조작을 줄이는지 A/B, 전수 불가 시 §8-3 기권(정직한 제로)과 연결. 충돌③(모순 페어 F1, 아래 3)과 별 실패 모드 — 중복 아님.
3. **기권(abstain) 클래스 포함** — 꺼낼 게 없으면 침묵(노이즈 철학의 시험). 충돌③·신선도④ = 모순 페어 합성 F1.
4. **한국어 슬라이스 별도 채점 + 부품 벤치 3건 통합(P8)** — 리랭커(Qwen3 vs bge-reranker vs Voyage) / 임베딩(bge-m3 vs Gemini vs voyage-lite vs KURE **vs Qwen3-Embedding-0.6B/4B** — [2025-06 공개](https://qwenlm.github.io/blog/qwen3-embedding/), Apache-2.0·MRL·instruction-aware, 다국어 MTEB 상위이나 한국어 공개 실측 얇음 → 2중 게이트 동일 적용, 2026-07-12 후보 추가) / 추출(Flash-Lite vs GPT-5-nano·mini, 한·영 코드스위칭 + 출력 언어·스키마 안정성 assert). 공개 하네스 재사용(instructkr reranker bench·AutoRAGRetrieval·Ko-StrategyQA·KURE). **+ 원자 임베딩 입력 A/B(2026-07-12 추가)**: 원자 본문 단독 임베딩 vs 스코프·엔티티 메타 프리픽스 결합 임베딩 — 짧은 원자의 dense 유사도 신호 부족 가설(컨텍스추얼 리트리벌의 우리식 번역) 검증, **gen_queries 채널과의 중복 여부 판정 목적**(무효과면 "원자 스키마+gen_queries로 충분" 증거로 기록). 한국어 슬라이스 포함. **전부 구현 전 API 수준 평가 — 제품 빌드 불필요.**
5. 출시 후 라이브 라벨 루프(P2 반영): 자동 기록에 대한 사용자의 **undo·제외·수정 액션 + 충돌 판정**이 라벨 — 승인 클릭 대신 예외 행동에서 정밀도를 측정하고 평가셋을 축적(복리 자산 루프). **+ 반영 채점 슬라이스(정합 감사 — 페인 2위 무영향 측정)**: 주입한 브리핑 팩이 런 출력에 실제로 반영됐는지를 채점한다 — write-back이 이미 흡수하는 런 배기가스를 입력으로 '주입 기억 ↔ 산출 반영' 일치를 스코어링(관측 제품이 아니라 *기억 유용성* 채점이므로 '관측 아님' 경계 내). §8-2 최종 주입 k 스윕(k=0 슬라이스 포함)을 이 지표의 오프라인 짝으로 쓰고 출시 후 라이브 지표로 승격한다. **주의: 이는 측정 항목이다 — 반영도에 따라 능동 개입하는 런타임 상시 컴포넌트로의 승격은 실험 결과에 따른 별도 설계 결정.**
6. **호출률 계측(검증 P6)**: 웹 표면에서 memory_brief 선제 호출률 측정 — 사용자 지침 스니펫 유/무 A/B. 대시보드 recall-rate 지표의 기준선이자 플랫폼 회귀 감시 기준.

## 9. 리스크·미결

- **스펙 처닝**: 2026-07-28 MCP 파괴적 개정 — 무상태 설계로 흡수, 출시 전 RC 검증.
- **claude.ai 무음 실패** → 전 레이어 로깅 1일차.
- **ChatGPT Apps 정책**: 데이터 최소화 조항(건강·결제 수집 금지)과 "뭐든 기억" 제품의 긴장 — 진출 시 스코프 필터 설계 필요(기록만, 후순위).
- **미결**: ⓐ 한국어 부품 벤치 3건(§8-4 — 기본값은 확정: bge-m3·Qwen3-Reranker·Flash-Lite, 벤치는 승격/스왑 판정용) ⓑ 대시보드 스택(Supabase auth 공유 시 저비용 + **IAP 앞단 게이트 전제 — §6 보안 통제 ④ 입력**) ⓒ 페르소나 상세·첫 5분 플로우 화면 수준 설계 ⓓ 차터 v0.3 반영 ⓔ zerank-2 상업 라이선스 협의(선택).
- **프로세스 규칙(P8f)**: 모든 인프라·모델 선택에 **2중 게이트** — ⓐ 관리형 호환성 매트릭스 확인 ⓑ 한국어 실측 근거 확인(없으면 자체 벤치 게이트 뒤로). 이 규칙 부재가 P1(ParadeDB)·P8(Voyage·Gemini 임베딩)의 공통 원인이었다.
- **개정 이력**: 2026-07-11 검증 리포트 P1~P6 반영(검색 스택 교체·기록-후-되돌리기·디렉토리 출시 요건·리랭커/모델 핀·풀링·호출률 계측) · 같은 날 한국어 검색 스택 P8a~P8f 반영(Kiwi 형태소 레그·5채널 확정·Qwen3-Reranker 1순위·bge-m3 기본값·추출 한국어 게이트·2중 게이트) · **2026-07-12 보안 통제 6항 반영**(§6 보안 확장 — trifecta 전제·신뢰 티어링·write-back 중대행동·IAP 게이트·신뢰 헤더·제로 상태 테스트, 정본 `.claude/memory/mco-security-model.md`) · **2026-07-12 컨텍스추얼 검색 동향 점검 반영(기능 항목만)** — §8-2 최종 주입 k 스윕·§8-4 원자 임베딩 입력 A/B·임베딩 벤치 후보 +Qwen3-Embedding(§6·§8-4). 캐시 친화 팩 직렬화는 스키마 v0.1 설계 입력으로 보류, 롱컨텍스트+캐싱 반론 방어는 차터 v0.3 후보로 이관(근거: `docs/리서치/MCO_컨텍스추얼검색동향_v0_1.md`). · **2026-07-12 에이전트 실패 기법 스캔 반영(기능 항목만)** — §8-2 집계·전수 질의 슬라이스 추가(벡터 top-k 샘플링 조작률 vs 구조적 질의, 기권 연결). Semantic Tool Selection = 역할⑤의 **유비(기억 선택의 툴 도메인 대응 — 서사 한정, 구현 채택 아님)**·Neurosymbolic Guardrails=§6 보안 통제①②·Multi-agent=write-back 게이트로 이미 흡수, 툴 라우터 확장은 오케스트레이션 가드레일로 기각(근거: `.claude/memory/mco-agent-failure-scan.md`). 상세: `docs/리서치/MCO_검증리포트_v0_1.md` · `docs/리서치/MCO_한국어검색스택_v0_1.md`.

---

*작성: 2026-07-11 세션 정렬 정본. 관련: `docs/리서치/MCO_사업성분석_v0_1.md`(판정·경로·킬 트리거), `docs/charter.md` v0.2(§7 웨지 재개정 대기), `.claude/memory/mco-architecture-v01.md`(요약 메모리).*
