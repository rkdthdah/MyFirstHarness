---
id: README.ko
level: 1
owner: or
---

# MyFirstHarness

*[English](README.md) · 한국어*

> 소프트웨어 개발을 위한 파일 기반 멀티 에이전트 하니스. 전문화된 여섯 에이전트가
> 정해진 프로토콜을 따라 함께 일한다 — 런타임도, 오케스트레이션 서버도 없다.
> 파일시스템 그 자체가 상태 머신이고, git이 모든 변경의 흔적을 남긴다.

---

## 소개

스토리 하나는 작성·설계·테스트·구현·출시 단계를 거치며, 각 단계마다 서로 다른
에이전트가 맡는다. 모든 에이전트는 자기 영역의 디렉토리 안에서만 움직이고, 정해진
워크플로우를 따른다. 모든 산출물(스토리, 스펙, 프로토타입, 테스트 설계서, 세션
로그)은 frontmatter에 작성자와 독자가 명시된 markdown 파일이다. 에이전트 간 협업은
파일명 변경, 디렉토리 조회, 그리고 문서 안에 적어 두는 핸드오프 노트로 일어난다.

전체 프로토콜은 **[`docs/harness.md`](docs/harness.md)** 에 정리되어 있다(15개 섹션,
약 500줄). 이 README는 거기로 들어가는 입구다.

---

## 특징

- **상태 머신으로서의 파일시스템.** 상태는 파일명에 담긴다
  (`dev-001-02_designed.md` → `_implement.md` → `_complete.md` → `_verified.md`).
  각 에이전트의 작업 큐는 `ls`와 상태 필터만으로 계산된다. 별도의 큐 서버도, 공유되는
  가변 상태도 필요 없다.
- **다섯 종류의 문서, 각각 하나의 질문에만 답한다.** Persona(누가/언제),
  Task(무엇을, 기술과 무관하게), Rules(어디서·어떻게, 프로젝트별), Template(출력의
  모양), Index(어디서 찾는지). 태스크와 템플릿은 다른 프로젝트로도 그대로 옮길 수
  있고, 스택이 바뀌면 룰만 갈아끼우면 된다.
- **소통 방향에 따른 7단계 문서 레벨.** 모든 파일은 frontmatter에서 누가 누구에게
  쓴 글인지 밝힌다 — `or→user`, `or→agent`, `agent→itself`, `agent→other agent`
  등. 파일을 펼치는 것만으로도 그 파일이 어떤 가정을 해도 되는지 알 수 있다.
- **물리적인 역할 분리.** 각 에이전트는 일부 경로에만 `rw` 권한을 갖고, 그 외에는
  읽기만 가능하다. 이 권한 범위는 모든 페르소나의 File & Folder Access 표에 그대로
  적혀 있다. 역할 경계가 글이 아니라 디렉토리 단위로 그어진다.
- **요구사항 단계에 녹아 있는 DDD.** Glossary가 시스템의 Ubiquitous Language(공용
  언어)를 관리한다. PM은 AC에 도메인 용어를 쓰기 전에 먼저 Glossary에 등록해야 하고,
  코드의 식별자명도 Glossary의 `Code ID` 컬럼과 일치해야 한다.
- **순서로 강제되는 TDD.** QA가 AC의 testability(테스트 가능성)를 검토하는 시점이
  AR의 아키텍처 설계 *이전*이다 — 가장 비용이 적게 드는 게이트. 테스트 시나리오는
  TE가 코드를 짜기 전에 QA가 먼저 설계하고, DE는 테스트 게이트를 통과한 다음에야
  구현에 들어간다.
- **두 단계로 나뉘는 스토리 라이프사이클.** Phase 1(요구사항)은 하나의 Story 문서를
  중심으로 Owner만 에이전트 간에 순환시킨다. Phase 2(구현)에서는 Story 문서가 동결
  되고, 상태가 파일명에 담긴 `dev/`, `test/` 단위 파일들이 작업 단위가 된다. 두
  가지 다른 오케스트레이션 방식, 둘 다 파일 기반.
- **Spec → Rules 추출.** 도메인 권위자(아키텍처는 AR, 테스트는 QA)가 단일 진실
  원천(single source of truth)인 spec을 소유한다. 이들이 consumer별 rules 파일을
  뽑아낸다. 각 에이전트는 자기 몫의 rules 파일만 본다 — spec이나 다른 에이전트의
  rules는 절대 직접 읽지 않는다. 태스크는 `Global Constraints` + `## {task id}` →
  `### {step name}` 규칙에 따라 자기에게 해당하는 rules와 자동으로 맞춰진다.
- **스토리별 git worktree.** PM이 스토리를 만들 때 `.worktrees/story/STORY-XXX/`를
  만들고, 스토리가 완료되면 제거한다. 여러 스토리가 브랜치 전환 없이 동시에 진행될
  수 있다. 모든 커밋은 `[STORY-XXX][agent][workflow]` 형식으로 누가 어느 단계에서
  남긴 변경인지 드러낸다.
- **프로토콜에 박힌 self-evaluation.** 모든 세션 로그는 Persona / Task / Process
  compliance 블록과 개선 노트로 마무리된다 — 프로토콜 자체가 피드백 신호를 만들어낸다.
- **에이전트 간 통신 채널은 둘뿐.** Phase 1에서는 Story 문서 안의 Handoff Notes,
  Phase 2에서는 dev/test 파일 안의 `Notes for {target}` 섹션. 그 외 채널은 없다.
- **Co-design은 절대 규칙.** 스토리와 UX 프로토타입은 사용자와 함께 만들고 다듬는다.
  사용자 확인 없이 마무리하는 일은 금지된다.

---

## 디렉토리 구조

```text
MyFirstHarness/
├── README.md                 (· README.ko.md)
├── AGENTS.md                 ← 에이전트 활성화 규칙과 세션 시작 스캔
├── CLAUDE.md                 ← Claude Code 진입점
├── agents/
│   ├── git-convention.md
│   ├── personas/             ← 에이전트당 1개 (누가/언제)
│   ├── tasks/                ← 기술에 묶이지 않은, 재사용 가능한 태스크 절차 (무엇을)
│   ├── templates/            ← 출력 형태 (story, ux-spec, session-log, ...)
│   └── docs/
│       ├── glossary.md       ← Ubiquitous Language (DDD)
│       └── stories/          ← 스토리별 폴더; 완료되면 archived/로 이동
├── docs/
│   ├── harness.md            ← THE 설계 문서
│   ├── context/              ← 사용자가 제공하는 외부 의존성 문서
│   └── frontend-*.md         ← 예시 스택 적용
├── frontend/                 ← 예시 애플리케이션 워크스페이스
└── backend/                  ← 예약됨
```

---

## 에이전트

```text
        Orchestration (or)
   ┌────┬────┬────┬────┬────┬────┐
   PM   UX   AR   QA   DE   TE
```

| 에이전트 | 역할            | 소유 영역                              |
| -------- | --------------- | -------------------------------------- |
| `pm`     | Product Manager | 스토리, 프로젝트 산출물                 |
| `ux`     | UX Designer     | 프로토타입, UX 스펙                     |
| `ar`     | Architect       | 아키텍처 스펙, 개발 설계                |
| `qa`     | QA Lead         | 테스트 스펙, 테스트 설계, 검증          |
| `de`     | Developer       | 구현                                   |
| `te`     | Tester          | 테스트 구현                             |
| `or`     | Orchestration   | 페르소나, 태스크, 룰, 그리고 하니스 자체 |

활성화 규칙과 세션 시작 스캔은 [`AGENTS.md`](AGENTS.md)에서.

---

## 로드맵

> 용어 안내: 아래의 프로젝트 **Stage**는 [`docs/harness.md`](docs/harness.md)에
> 정의된 스토리 라이프사이클의 **Phase**와는 다른 개념이다. Stage 1이 하니스의 두
> Phase를 모두 포함한다.

### Stage 1 — End-to-End 개발 프로세스

여섯 에이전트가 출시까지의 전체 사이클을 함께 돌린다. 모든 산출물은 어느 에이전트가,
어느 워크플로우 단계에서 만든 것인지 추적할 수 있다.

- **Phase 1 (요구사항: PM / UX / QA)** ✅ — 페르소나, 요구사항 단계 태스크, 공유
  템플릿, Glossary, git 컨벤션, 하니스 설계 문서가 모두 자리잡혀 있다.
- **Phase 2 (구현: AR / DE / TE + QA 검증)** 🛠 — 남은 작업:
  - AR / DE / TE 페르소나
  - 구현 단계 태스크 패밀리: `design-test` / `review-test` / `revise-test-design` /
    `verify-implementation` / `manage-qa-artifact` / `manage-qa-spec`
  - PM의 `refine-story` / `complete-story`
  - `agents/templates/frontend-web-react/` 템플릿에서 만들어 낼 frontend 스펙과,
    거기서 다시 뽑아 낼 consumer rules 파일들

### Stage 2 — 오케스트레이션 레이어

상시 동작하는 `or` 런타임이 markdown 프로토콜을 *능동적으로 관리되는 시스템*으로
바꾼다. 현재 프로덕션 LLM 에이전트 실무가 수렴하고 있는 다섯 갈래의 관심사:

1. **피드백 루프 기반 자가 개선.** 세션 로그는 이미 Persona / Task / Process
   compliance 블록과 개선 노트로 마무리된다. 런타임은 이를 오케스트레이션 trace로
   받아들여, 페르소나·태스크·rules에 대한 타입드 델타(typed delta)를 제안한다 —
   [Reflexion][reflexion], [Multi-Agent Reflexion(MAR)][mar],
   [Instruction-Level Weight Shaping][ilws]이 공통적으로 도달한 패턴이다
   (돌아보고 → 저장하고 → 다음 시도의 조건으로 삼는다. 단일 에이전트의
   self-critique보다, 페르소나별 비평자 여럿이 만들어내는 신호가 더 풍부하다).

2. **결정론 / 비결정론 분리.** 프로토콜은 이미 선을 그어 두었다 — 워크플로우는
   정해져 있고, 태스크는 판단의 영역이다. 다만 그 분리를 지키는 책임이 지금은 모델에
   있다. Stage 2는 결정론에 속하는 부분을 코드로 옮긴다: 세션 시작 스캔, 파일명 상태
   전이, Handoff Note 추가와 Owner 리네임, 커밋 메시지 템플릿화, DoD 집계, 브랜치와
   worktree 라이프사이클, rules 파일 매칭. [상태 전이는 결정적이어야 하며 LLM은
   분산 시스템 안의 planner/executor라는 프로덕션 컨센서스][agent-arch]와 같은
   방향이다.

3. **토큰 사용 최소화.** 양보할 수 없는 가드레일 세 가지 — 세션당 토큰 예산, 루프당
   최대 반복 횟수, 종료 판정자(termination evaluator) — 와 컨텍스트 압축
   ([anchored iterative summarization, 공급자가 제공하는 compaction API][compaction]).
   토큰 사용량은 선형으로 늘지 않는다. 도구와 메모리 소스가 하나 추가될 때마다
   비용이 곱으로 늘어난다.

4. **시스템 무결성.** 엄격한 도구 계약(tool contract), 모든 LLM 호출·도구
   호출·핸드오프에 붙는 OpenTelemetry trace span, 과거 trace를 회귀(regression)
   데이터셋으로 변환해 CI에서 돌리는 eval, 모든 루프에 명시된 종료 조건. 신뢰성은
   프롬프트 엔지니어링이 아니라 아키텍처에서 나온다.

5. **성능.** 하이브리드 모델 라우팅 — 가벼운/로컬 모델을 주력으로 쓰고, 어려운
   추론은 frontier 모델로 fallback. Phase 2의 `dev/`, `test/` 큐가 허용하는 지점에서
   서브 에이전트 병렬 실행. 페르소나 경계를 넘나드는 prompt caching. self-evaluation
   블록을 실패 코퍼스로 삼는 failure-driven prompt 최적화.

### Stage 3 — 비개발자를 위한 소프트웨어 개발 킷

사용자를 마주하는 인터페이스 — 채팅과 UI — 를 통해, AI도 도메인도 프로그래밍도
모르는 사람이 이 시스템을 software development kit처럼 쓸 수 있게 한다. 사용자가
의도를 말하면, 하니스가 소프트웨어를 만들어 출시하고 유지보수한다.

이미 실체가 있고 경쟁이 치열한 영역이다. Gartner는 AI-native 개발 플랫폼을 2026년
최상위 전략 기술 트렌드로 꼽았고, 시장 규모는 약 47억 달러, 그중 63%가 비개발자
사용자다. [Replit Agent][replit]와 [Lovable][lovable]은 출시 1년이 채 되지 않아 각자
ARR $100M를 돌파했다.

이 접근이 가지는 차별점은 분명하다. 위 도구들은 비전문가가 *맹목적으로 신뢰해야
하는* 코드를 내놓는다. 이 하니스는 *프로토콜의 부산물로* 비전문가도 직접 검증할 수
있는 산출물을 함께 만들어낸다 — acceptance criteria가 붙어 있는 읽을 수 있는 스토리,
코드가 작성되기 전에 검토되는 동작하는 프로토타입, 시스템의 언어를 고정해 주는
Glossary, 무엇이 반드시 성립해야 하는지 명시한 테스트 설계서, 커밋 단위로 되돌릴 수
있는 git 히스토리. UI는 한쪽에서 사용자의 의도를 스토리로 옮기고, 다른 쪽에서 이
산출물들을 보여 준다. 엔지니어링 자체는 하니스가 맡는다.

Stage 3가 현실성을 가질 수 있는 이유는, Stage 1과 2가 그 검증 가능한 기반을 먼저
만들어 두기 때문이다.

[reflexion]: https://arxiv.org/pdf/2505.24726
[mar]: https://arxiv.org/pdf/2512.20845
[ilws]: https://arxiv.org/pdf/2509.00251
[agent-arch]: https://andriifurmanets.com/blogs/ai-agents-2026-practical-architecture-tools-memory-evals-guardrails
[compaction]: https://currentstack.io/stories/agent-context-compression-gateway-pattern-2026/
[replit]: https://replit.com/discover/replit-vs-bolt
[lovable]: https://lovable.dev/guides/best-ai-app-builders

---

## 어디부터 읽으면 되는가

1. **[`docs/harness.md`](docs/harness.md)** — 15개 섹션 설계 문서. 딱 하나만 읽는다면 이것.
2. **[`AGENTS.md`](AGENTS.md)** — 에이전트 표, 활성화 규칙, 세션 시작 스캔.
3. **[`agents/personas/pm.md`](agents/personas/pm.md)** — Workflow / Tasks / Data / File Access가 어떻게 맞물리는지 보여 주는 구체적인 페르소나.
4. **[`agents/git-convention.md`](agents/git-convention.md)** — 브랜치 모델, 커밋 포맷, 스토리별 worktree 라이프사이클.