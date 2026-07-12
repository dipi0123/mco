# MCO 경쟁 지형 리서치 v0.1 (2026-07-10)

> 차터 §9 백로그 '경쟁·시장 리서치' 수행 결과 **정본**. 방법: 병렬 웹 리서치 3트랙 — ① 메모리 레이어 스타트업 ② 빅테크 내장 메모리 ③ 에이전트 생태계 인프라. 핵심 판정은 차터 v0.2 §2에 반영. 모든 주장에 출처 링크 첨부.

---

## 0. 핵심 판정 (Executive Summary)

| 차터 빈틈 요소 | 2026-07 판정 |
|---|---|
| 크로스 플랫폼 이동성 | **혼잡 — 단독 차별점 사망.** Supermemory MCP · mem0 OpenMemory · MemoryPlugin · MemoryLake가 이미 크로스 툴 개인 메모리 제공. |
| 계정 바인딩·사용자 소유 | **빅테크 레벨 공백.** 저장소 통제권이 전부 벤더/테넌트에 있음. 사용자 소유 진영은 소규모·초기. |
| 에이전트별 권한 스코프 | **벤더 횡단으로는 완전 공백.** 벤더 내부 구현은 확산(= 수요의 증거). |

**종합 평결:** "함대 공유 메모리 + 실패 지식 자산화 + 크로스 벤더 중립층"을 모두 하는 곳은 **없다(2026-07 현재)**. 개별 부품은 각기 다른 진영이 보유. MCO의 차별화 여지는 '존재 여부'가 아니라 **조합 완성도** — 사용자 소유 + 벤더 횡단 권한 스코프 + 정밀 선택 + 게이트 환류(write-back) + 브리핑 팩.

**빈틈 프레임 갱신:** (구) "빅테크가 하기엔 짜치고 사용자는 절실한" → (신) **"벤더는 담장 안에서 전부 구현했다(수요 증거). 구조적으로 못 하는 것은 벤더 횡단 중립이다."**

---

## 1. 메모리 레이어 스타트업

### mem0 — 자금·배포·비전 선점 (위협 1티어)
- 2025-10-28 Basis Set 리드 **$24M**(시드 $3.9M + 시리즈A $20M). GitHub 41K+ 스타, 분기 API 호출 1.86억 건(Q3'25), **AWS Agent SDK 독점 메모리 공급자**. 비전을 **"메모리 여권", "Plaid for memory"**로 명시 — MCO 포지셔닝과 직접 충돌. [TechCrunch](https://techcrunch.com/2025/10/28/mem0-raises-24m-from-yc-peak-xv-and-basis-set-to-build-the-memory-layer-for-ai-apps/)
- 주 고객은 앱 개발자(B2B API). 단 **OpenMemory MCP**(2025-05 출시: 로컬 메모리 서버 + 대시보드 + 앱별 접근제어)와 크롬 확장(ChatGPT/Claude/Perplexity/Grok 동기화)으로 사용자 소유형도 실험 중. [OpenMemory MCP](https://mem0.ai/blog/introducing-openmemory-mcp) · [확장 GitHub](https://github.com/mem0ai/mem0-chrome-extension)
- 검색: 시맨틱+BM25+엔티티 멀티시그널, 자체 발표 LongMemEval 94.4 (2026-04). [State of AI Agent Memory 2026](https://mem0.ai/blog/state-of-ai-agent-memory-2026)
- 가격: 무료 → $19/$79/$249 → 엔터프라이즈. [Pricing](https://mem0.ai/pricing)

### Zep — 엔터프라이즈 전환 완료
- 시간형 지식그래프(**Graphiti** 오픈소스, [2025-01 논문](https://arxiv.org/abs/2501.13956))로 "엔터프라이즈 에이전트 메모리" 포지셔닝: Context Lake, **속성 기반 접근제어(ABAC)**, 감사로그, SOC2/HIPAA, <200ms 검색, LongMemEval 90.2% 주장. 고객: Samsung·HoneyBook·Twin Health. **사용자 소유·이식성 없음(테넌트=기업).** [getzep.com](https://www.getzep.com/)

### Letta (구 MemGPT) — 2026-03 런타임 피벗
- 2024-09 $10M 시드(Felicis). [BigDATAwire](https://www.hpcwire.com/bigdatawire/this-just-in/letta-emerges-from-stealth-with-10m-to-build-ai-agents-with-advanced-memory/)
- 2026-03-16 "Letta Code" 피벗: 서버 중심 메모리 API를 버리고 **git 기반 메모리 파일(컨텍스트 리포) + sleep-time compute(에이전트 학습 자동 write-back)**로 전환. 순수 개발자 대상. [Letta's Next Phase](https://www.letta.com/blog/our-next-phase/)
- 멀티에이전트 공유 memory block 지원하나 **에이전트별 차등 권한 불가**(read_only 블록만). [Letta 문서](https://docs.letta.com/guides/agents/multi-agent-shared-memory)

### Supermemory — 기능 세트 최근접 (위협 2티어)
- 2025-10 시드 ~$3M(엔젤: Jeff Dean 등). 본업은 개발자 API(저지연 차별화). [TechCrunch](https://techcrunch.com/2025/10/06/a-19-year-old-nabs-backing-from-google-execs-for-his-ai-memory-startup-supermemory/)
- **Supermemory MCP**: OAuth 로그인 하나로 "Claude에서 기록, Cursor에서 회상" — 개인 컨텍스트 레이어를 명시 표방, 개인용 무료. **단 에이전트별 권한 스코프 없음.** [supermemory.ai/mcp](https://supermemory.ai/mcp/)
- 가격: Free → $19 → $100 → $399 + API 종량제. [Pricing](https://supermemory.ai/pricing)

### 기타
- **LangMem** — LangGraph 장기메모리 부속 라이브러리(write-back 있음), 독립 제품 아님. [LangChain 블로그](https://www.langchain.com/blog/langmem-sdk-launch)
- **Cognee** — 2026-02 €7.5M 시드, 지식그래프+Remember-Recall-Improve-Forget 사이클, 규제산업 지향. 이식성·사용자 소유 없음. [EU-Startups](https://www.eu-startups.com/2026/02/german-ai-infrastructure-startup-cognee-lands-e7-5-million-to-scale-enterprise-grade-memory-technology/)
- **Engram** — 2026-06-23 **$98M 시리즈A**(General Catalyst·KP·Sequoia, Chris Ré 공동창업, Karpathy 엔젤). 조직 컨텍스트를 모델에 '학습'시키는 learned memory(토큰 100배 절감 주장), 파트너 Microsoft·Notion·Harvey. **기업 소유 방향이라 결이 다르나 전장 중첩 가능.** [PRNewswire](https://www.prnewswire.com/news-releases/engram-launches-with-98m-to-build-ai-that-actually-knows-your-organization-302807126.html)
- **XTrace** — MCP로 Claude·Codex에 꽂는 팀 메모리 클라우드, "한 에이전트가 배우면 모두가 안다"(write-back 공유 직접 경쟁). [xtrace.ai](https://xtrace.ai/)
- **Honcho(Plastic Labs)** — $5.35M 프리시드, 사용자 심리모델링 기반 메모리. [PANews](https://www.panewslab.com/en/articles/0w9rpdwo)
- **소비자형**: [MemoryPlugin](https://www.memoryplugin.com/) (21+ 플랫폼, MCP·확장, 연 $79~, 내보내기 가능), [MemoryLake "Memory Passport"](https://www.memorylake.ai/en/products/memory-passport) (ChatGPT/Claude/모든 LLM 동반 개인 메모리, git식 버저닝) — **명칭까지 '메모리 여권'인 직접 경쟁 개념 존재** ⚠️.
- 오픈소스 다수: MemMachine, Memori, Cipher 등. [dev.to 2026 top10](https://dev.to/jonathanfarrow/the-10-best-ai-memory-layers-for-agents-in-2026-448e)

---

## 2. 빅테크 내장 메모리

### OpenAI / ChatGPT
- **Dreaming V3** (2026-06-04): 채팅 이력을 백그라운드에서 자동 종합·갱신(시간 인지 업데이트). [OpenAI](https://openai.com/index/chatgpt-memory-dreaming/) · 프로젝트 한정 메모리(2025-08). [Simon Willison](https://simonwillison.net/2025/Aug/22/project-memory/)
- **메모리 API 없음** — 커뮤니티 요청만 누적. [OpenAI Dev Community](https://community.openai.com/t/will-memory-capabilities-come-to-the-api/934907)
- **Codex Memories**: 유휴 스레드 자동 요약 → 로컬 `~/.codex/memories/` 저장("known pitfalls" 포함). **기기 로컬·ChatGPT 메모리와 미연동·API 없음·기본 꺼짐·EEA/UK 미제공.** [Codex Memories](https://developers.openai.com/codex/memories)
- **Workspace agents** (2026-04-22): 팀 사용으로 학습하는 에이전트 메모리 + 관리자 범위 제한. [OpenAI](https://openai.com/index/introducing-workspace-agents-in-chatgpt/)

### Anthropic / Claude (빅테크 중 잠식 경로 최근접 — 위협 1티어)
- claude.ai 메모리: 2025-09 Team/Enterprise(프로젝트별 분리·열람·편집·인코그니토), **프로젝트 메모리 다운로드 지원**. [VentureBeat](https://venturebeat.com/ai/anthropic-adds-memory-to-claude-team-and-enterprise-incognito-for-all) · 2026-03-02 무료 확대 + **ChatGPT·Gemini·Copilot 메모리 임포트 도구**(일회성). [MacRumors](https://www.macrumors.com/2026/03/02/anthropic-memory-import-tool/)
- **API memory tool GA** (`memory_20250818`): 클라이언트 사이드 파일 기반, 저장 백엔드를 개발자가 통제. [Platform Docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool)
- **Managed Agents 영속 메모리** (2026-04 공개 베타): 조직/사용자 스코프 메모리 스토어, **다중 에이전트 동시 공유, 에이전트·세션별 감사 로그**, API 내보내기·편집. [EdTech Hub](https://www.edtechinnovationhub.com/news/anthropic-brings-persistent-memory-to-claude-managed-agents-in-public-beta)
- Claude Code: CLAUDE.md 4계층 + 오토 메모리(리포 단위, **머신 로컬·클라우드 미공유**, 서브에이전트별 메모리). [Claude Code Docs](https://code.claude.com/docs/en/memory)

### Google Gemini
- **Personal Intelligence**: 2026-01 유료 → 2026-03-27 미국 무료 확대. Gmail·캘린더·드라이브 등 + 과거 대화 기억, 앱별 접근 선택. EU권 제한. [9to5Google](https://9to5google.com/2026/03/27/gemini-free-personal-intelligence-rollout/)
- Gemini Enterprise: 사용자별 메모리, 관리자 조직 단위 제어. **메모리 API·내보내기 없음.** [Google Cloud Docs](https://docs.cloud.google.com/gemini/enterprise/docs/configure-personalization)

### Microsoft Copilot
- Copilot Memory: 사용자 Exchange 사서함 숨김 폴더 저장, 테넌트 정책 적용, 프리뷰. **개인 단위 — 조직 공유 메모리 아님.** [MS Learn](https://learn.microsoft.com/en-us/microsoft-365/copilot/copilot-personalization-memory) · [Office365ITPros](https://office365itpros.com/2026/06/29/copilot-memory-updates/)

### 이식성·표준 동향
- 빅테크 중 이식성 행보는 Anthropic뿐(수출 2025-09 + 수입 2026-03)이나 **일회성 이주 도구, 자사 유입용** — 상시 양방향 동기화 계층 아님. [Fast Company](https://www.fastcompany.com/91501002/anthropic-claude-app-import-chats-from-open-ai-chatgpt-gemini-copilot-memory-tool)
- 정책 진영은 "맥락이 곧 독점력, 메모리가 새 해자"라며 MCP 호환 이식성 의무화 주장 — 규제발 순풍. [New America (2025-11)](https://www.newamerica.org/insights/ai-agents-and-memory/)

---

## 3. 에이전트 생태계 메모리 인프라

### MCP 메모리 서버 생태계
- Anthropic 공식 레퍼런스 = 로컬 파일 기반 [Knowledge Graph Memory Server](https://github.com/modelcontextprotocol/servers/tree/main/src/memory). 생태계 전체 ~2만 서버 색인. [State of MCP servers 2026](https://tooldirectory.ai/blog/state-of-mcp-servers-2026)
- MCP는 2025-12-09 **Linux Foundation 산하 AAIF(Agentic AI Foundation)로 이관** — OpenAI·Anthropic·Block 공동 설립, AGENTS.md·MCP·goose 기증. [Linux Foundation](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation) · [OpenAI 발표](https://openai.com/index/agentic-ai-foundation/)

### 메모리 오픈 표준의 부재 ★
- [AGENTS.md](https://github.com/agentsmd/agents.md)는 6만+ 프로젝트 채택(Codex·Cursor·Copilot·Amp·Windsurf 네이티브 지원). **그러나 이것은 '지시문' 표준일 뿐 — 축적형 에이전트 '메모리'의 오픈 표준은 존재하지 않고 AAIF 차원 제안도 미확인.** [AGENTS.md Field Guide 2026](https://www.iuriio.com/blog/posts/2026/05/agents-md-field-guide-2026)
- → 시사점: 메모리 오픈 포맷을 선제 제안하는 표준 플레이 창구가 열려 있음 (차터 §9).

### 코딩 에이전트 크로스 세션 메모리 — 전부 벤더 사일로
- Claude Code(리포 단위·머신 로컬), Codex(로컬·기본 꺼짐), Cursor Memories(프로젝트 단위 [0.51 베타](https://forum.cursor.com/t/0-51-memories-feature/98509)), **Devin Knowledge(제안→승인 게이트, 조직/리포 스코프 핀)** [Devin 문서](https://docs.devin.ai/product-guides/knowledge).
- 공통 한계: **프로젝트 간·에이전트 간·벤더 간 공유 계층 부재.** Devin의 승인 게이트는 MCO '가치 판단 게이트' 개념의 벤더 내부 검증 사례.

### 멀티에이전트 함대 공유 메모리
- Letta 공유 블록(차등 권한 불가), AWS Bedrock **AgentCore Memory**(네임스페이스 스코핑+만료). [AWS 블로그](https://aws.amazon.com/blogs/machine-learning/organizing-agents-memory-at-scale-namespace-design-patterns-in-agentcore-memory/) CrewAI·AutoGen 계열은 [mem0 연동](https://mem0.ai/integrations)이 사실상 표준. OpenAI Agents SDK [Sessions](https://openai.github.io/openai-agents-python/sessions/)는 대화 이력 지속용.
- **'매니저→워커 브리핑 팩' 프리미티브는 어느 제품에도 없음.**

### 실패 지식(negative knowledge) 자산화
- 연구 최전선: Google **[ReasoningBank](https://research.google/blog/reasoningbank-enabling-agents-to-learn-from-experience/)** — 성공+실패 궤적에서 전략 증류·재사용, WebArena/SWE-Bench 검증, [오픈소스](https://github.com/google-research/reasoning-bank). **제품화 전 단계 = 타이밍 창구.**
- 출하된 근접물: Claude Code 오토 메모리·Codex pitfalls·Devin 게이트 — 전부 벤더 내부용.
- 스타트업: ByteRover(팀 공유 에이전틱 메모리), [Cipher](https://mcpmarket.com/server/cipher)(오픈소스 MCP), [Hindsight](https://hindsight.vectorize.io/blog/2026/04/08/adding-memory-to-codex-with-hindsight)(Codex에 학습 주입).
- 출처·TTL 부품: [Zep Graphiti](https://github.com/getzep/graphiti) bi-temporal 무효화(valid_at/invalid_at), AgentCore 네임스페이스+만료 — **부품은 존재, 통합 제품 없음.**

### 관측(Observability) → 메모리 이동 여부
- LangSmith [Insights Agent](https://www.langchain.com/blog/insights-agent-multiturn-evals-langsmith)(2025-10 GA)는 실패 모드 군집화까지 — **메모리 자동 환류는 없음**(사람 수동). Braintrust는 [프롬프트 최적화 루프](https://www.braintrust.dev/articles/prompt-optimization-loop)까지. Langfuse·W&B Weave는 트레이싱/평가에 머묾. [2026 비교](https://latitude.so/blog/ai-agent-observability-tools-compared-latitude-vs-langfuse-langsmith-braintrust)

---

## 4. 위협 지도

| 티어 | 플레이어 | 이유 |
|---|---|---|
| **1티어 (정면 충돌 경로)** | **mem0** | 자금($24M)·배포(AWS 채널)·비전 문구("Plaid for memory") 선점. 단 본업은 개발자 API, 사용자 소유형은 사이드. |
| | **Anthropic** | API memory tool GA + Managed Agents 조직/사용자 스코프 메모리 — 빅테크 중 기능적 잠식 최근접. 단 자사 담장 안. |
| **2티어 (기능 중첩)** | Supermemory | OAuth MCP 개인 레이어 출하(스코프 없음). |
| | XTrace, ByteRover, Cipher, Hindsight | 에이전트 학습 공유/주입 — write-back 전장 선점 시도. |
| **3티어 (방향 상이 거인)** | Engram ($98M) | 조직 학습형(learned memory) — 결이 다르나 전장 중첩 가능성. |
| | Zep, Cognee | 엔터프라이즈 그래프 메모리 — 테넌트 모델, 사용자 소유 아님. |

## 5. 시사점 — MCO 경쟁력

1. **"세계 최초 크로스 플랫폼 메모리" 류 주장 불가.** 차별점은 존재가 아니라 조합: **사용자 소유 + 벤더 횡단 권한 스코프 + 정밀 선택 + 게이트 환류 + 브리핑 팩** — 이 조합 전부를 하는 곳 없음(2026-07).
2. **정밀 선택은 여전히 고유 의견.** 시장은 '전부 저장 + top-k 유사도'가 지배적, 정밀도는 벤치마크 마케팅 수준. 노이즈 필터를 제품 철학으로 세운 곳 없음.
3. **write-back 방향성은 검증됨, 통합은 공백.** Devin 게이트(벤더 내부)·ReasoningBank(연구)·Letta sleep-time(자사 런타임) — 크로스 벤더 + 게이트 + 실패 지식의 결합이 빈 자리.
4. **웨지 함의:** 이동성 웨지(브라우저 확장)는 혼잡, 하네스/함대 웨지는 공백 → MVP 웨지 스왑 타당(차터 §7).
5. **리스크:** ① 속도 — 벤더 내장 메모리가 2026년 기본값, AAIF가 메모리로 표준 확장 시 공백 급속 축소 ② OpenAI 메모리 API 출시 시 논거 재점검 ③ mem0의 채널 장악 ④ 용어 충돌(Passport) ⑤ 자원 비대칭($24M~$98M 상대) → 웨지 최소화 필수.

## 6. 감시 항목 (분기별 재확인)

- OpenAI 메모리 API 출시 여부 · Codex Memories의 클라우드 동기화 여부
- AAIF의 메모리 표준화 논의 개시 여부
- mem0 OpenMemory의 에이전트별 권한 스코프 추가 여부
- Anthropic Managed Agents 메모리의 외부(비 Anthropic 에이전트) 개방 여부
- Engram의 개인/크로스 툴 확장 여부
