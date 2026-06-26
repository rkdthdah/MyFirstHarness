# harness 001

> 멀티 에이전트 소프트웨어 개발 하니스 — 여섯 개의 역할 에이전트가 문서를 주고받으며 하나의 스토리를 요구사항 정의부터 통합까지 진행하는 문서 기반 운영 체계입니다.

---

# 한국어

## 목차

1. [개요](#1-개요)
2. [핵심 아이디어](#2-핵심-아이디어)
3. [에이전트](#3-에이전트)
4. [작업 흐름](#4-작업-흐름)
5. [문서 시스템](#5-문서-시스템)
6. [원칙](#6-원칙)
7. [저장소 구조](#7-저장소-구조)
8. [시작하기](#8-시작하기)
9. [주요 용어](#9-주요-용어)

---

## 1. 개요

harness 001은 LLM 에이전트가 역할을 나누어 소프트웨어 개발을 수행하도록 설계된 **문서 기반 운영 체계**입니다. 이 하니스는 실행 코드가 아니라 페르소나, 태스크, 템플릿, 스펙, 룰과 같은 문서로 구성됩니다. 각 문서는 에이전트가 누구인지, 언제 작업을 시작해야 하는지, 무엇을 어떤 절차로 만들어야 하는지, 그리고 산출물을 누구에게 넘겨야 하는지를 정의합니다.

핵심 원리는 단순합니다. **산출물을 만든 에이전트가 그 산출물을 직접 검수하지 않습니다.** 설계자와 구현자, 구현자와 검증자를 분리하고, 각 산출물의 소유권을 한 번에 한 에이전트에게만 부여한 뒤 순차적으로 넘깁니다. 이를 통해 결함을 더 빠르고 비용이 낮은 단계에서 발견하고, 자기 검수에서 발생하기 쉬운 사각지대를 줄입니다.

---

## 2. 핵심 아이디어

전체 시스템은 `docs/` 아래의 **두 개의 캐논 문서**에서 출발합니다.

* **`harness.md`** — 문서 시스템과 운영 구조를 정의합니다. 파일 수명주기, 소유권 회전, 워크플로의 골격, 문서 간 참조 방향처럼 구조의 *무엇*과 *어떻게*를 다룹니다.
* **`development-principles.md`** — 방법론과 가치를 정의합니다. TDD, DDD, Make-or-Buy, Shift-Left, Doer/Evaluator 분리처럼 이 하니스가 왜 그런 방식으로 움직이는지를 설명합니다.

페르소나, 태스크, 룰, 스펙, 템플릿 등 나머지 문서들은 이 두 문서로부터 **재생성 가능(regenerable)**해야 합니다. 규칙은 정확히 한 곳에만 존재해야 하며, 다른 문서에서 같은 규칙이 필요할 때는 복사하지 않고 해당 위치를 참조합니다. 이것이 Single Source of Truth 원칙입니다.

문서를 배치할 때의 기준은 다음과 같습니다.

* **기술 스택과 무관한 내용**은 캐논 문서와 원칙 문서에 둡니다.
* **특정 스택에 묶이는 내용**(경로, 도구, 디렉터리 구조 등)은 스펙에 둡니다.
* **실행 절차**는 태스크에 둡니다.

판단 기준은 하나입니다.

**“기술 스택이 바뀌어도 이 문장이 그대로 유효한가?”**

그렇다면 캐논 또는 원칙에 가까운 내용이고, 그렇지 않다면 스펙이나 태스크에 가까운 내용입니다.

---

## 3. 에이전트

| ID     | 역할              | 책임                                                                              |
| ------ | --------------- | ------------------------------------------------------------------------------- |
| **OR** | Orchestration   | 하니스 자체를 관리하고 개선합니다. 페르소나, 태스크, 템플릿, 세션 로그 리뷰, init 라인을 담당하며 스토리 작업에는 참여하지 않습니다. |
| **PM** | Product Manager | 무엇을 만들지와 왜 필요한지를 정의합니다. 요구사항 발굴, 스토리 작성, 핸드오프, 산출물 관리를 담당합니다.                   |
| **UX** | UX Designer     | 사용자가 직접 보게 되는 화면과 상호작용을 설계합니다. 프로토타입과 UX 스펙을 담당합니다.                             |
| **AR** | Architecture    | 아키텍처, 인터페이스, 구조, 룰을 정의합니다. 런타임 스펙의 책임자입니다.                                      |
| **QA** | QA Lead         | 결과가 요구사항을 충족하는지 검증합니다. 테스트 스펙을 소유하고 테스트 설계를 담당합니다.                              |
| **DE** | Developer       | AR이 정의한 아키텍처 범위 안에서 기능을 구현합니다.                                                  |
| **TE** | Tester          | QA가 설계한 테스트를 실행 가능한 테스트 코드로 구현합니다.                                              |

역할 경계는 엄격하게 유지됩니다. PM은 기술적 구현 방식(HOW)을 결정하지 않으며, QA는 구현 내부가 아니라 결과를 검증합니다. OR은 사용자와 소통하는 유일한 채널이며, 작업 에이전트는 사용자와의 직접 소통을 최소화합니다.

---

## 4. 작업 흐름

작업의 기본 단위는 **스토리(`STORY-XXX`)**이며, 전체 흐름은 두 단계로 나뉩니다.

### Phase 1 — 요구사항

이 단계에서는 Story 문서 하나가 작업 단위입니다. 소유권은 **PM → UX → QA → AR** 순서로 이동합니다. 각 에이전트는 자기 단계에서 DoD(Definition of Done) 체크박스를 완료하고, 핸드오프 노트(`[시각] FROM → TO: 메모`)를 남긴 뒤, 문서의 Owner를 다음 에이전트로 변경합니다.

### Phase 2 — 구현

이 단계에서는 작업 단위가 **dev/test 문서 쌍(pair)** 으로 바뀝니다. AR은 스토리를 여러 구현 단위로 분해하고, 각 단위마다 `dev` 문서(계약 + 구현)와 `test` 문서(검증 설계)를 짝지어 생성합니다. 두 문서는 동일한 번호로 연결됩니다.

```text
dev-XXX-NN ↔ test-XXX-NN
```

이 기간 동안 Story 문서는 읽기 전용이며, Owner는 AR로 유지됩니다. 즉, Story 문서는 구현 단계에서 잠시 “수면 상태”가 됩니다.

### Pair Camp 모델

Phase 2의 핵심은 **Pair Camp 모델**입니다. dev/test 쌍은 항상 하나의 캠프에 속합니다.

* **AR 캠프** — 시니어 AR이 계약을 설계하거나 리뷰하고, 주니어 DE가 본문을 구현합니다.
* **QA 캠프** — 시니어 QA가 테스트를 설계하거나 리뷰하고, 주니어 TE가 테스트 코드를 구현합니다.

캠프 전환(AR ↔ QA)은 두 문서가 모두 해당 캠프의 시니어에게 있을 때만 일어납니다. 따라서 dev/test를 함께 봐야 하는 재설계 상황에서도 한 명의 시니어가 두 문서를 동시에 소유한 상태에서 판단합니다. 소유권은 공유되지 않고, 항상 한 명에게만 있습니다.

DoD 흐름은 다음과 같습니다.

```text
dev:   Designed(AR) → Implemented(DE) → Reviewed(AR) → Verified(QA)
test:  Designed(QA) → Implemented(TE) → Reviewed(QA)
```

TDD 원칙에 따라 **테스트 설계는 기능 구현보다 먼저 이루어집니다.** 구현이 통과했다는 기준은 문서 리뷰가 아니라, 해당 테스트가 실제로 초록불(green)로 실행되는 것입니다.

모든 dev 문서가 `Complete` 상태가 되고, 모든 test 문서가 QA 소유로 정리되면 AR이 통합을 수행하고 스토리를 종료합니다.

---

## 5. 문서 시스템

모든 문서는 **통신 방향**에 따라 레벨이 정해집니다.

| 레벨 | 방향          | 예시                                        |
| -- | ----------- | ----------------------------------------- |
| 1  | OR → 사용자    | harness, development-principles, README   |
| 2  | OR → OR     | 오케스트레이션 내부 문서                             |
| 3  | OR → 에이전트   | AGENTS.md, 페르소나, 템플릿, 태스크, git-convention |
| 4  | 에이전트 → 사용자  | 프로젝트 산출물                                  |
| 5  | 에이전트 → 자신   | 세션 로그, latest-state, 스펙(spec)             |
| 6  | 에이전트 → 에이전트 | UX 스펙, 룰(rules)                           |
| 7  | 다중 소유       | 스토리                                       |

문서 타입별 역할은 다음과 같습니다.

| 타입           | 담는 내용                                     | 위치                                   |
| ------------ | ----------------------------------------- | ------------------------------------ |
| **persona**  | 에이전트가 언제, 어떤 순서로 움직이는지                    | `agents/personas/{agent}.md`         |
| **task**     | 일을 어떻게 수행하는지. 기술 스택과 무관한 절차               | `agents/tasks/{name}.task.md`        |
| **template** | 산출 문서의 구조                                 | `agents/templates/{name}.tmp.md`     |
| **spec**     | 경로, 도구, 구조 규칙처럼 특정 스택에 묶인 내용. 도메인 권위자가 소유 | `{area}/{agent}.spec.md`             |
| **rules**    | 스펙에서 에이전트별로 추출한 실행 규칙                     | `{area}/{consumer}.{owner}.rules.md` |
| **index**    | 선택적 로딩을 위한 룩업 테이블                         | `{area}/{name}.idx.md`               |
| **story**    | 하나의 스토리 수명주기 문서                           | `agents/docs/stories/STORY-XXX/`     |

문서 간 참조 방향은 다음과 같습니다.

```text
workflow → task → rules → spec
```

이는 하위 문서가 상위 문서를 참조하지 않는다는 뜻입니다. 두 캐논 문서인 `harness.md`와 `development-principles.md`는 어느 문서도 참조하지 않고, 다른 문서에서도 직접 참조되지 않습니다. 두 문서는 사람이 읽는 기준 문서이며, 에이전트는 이들로부터 생성된 문서를 기반으로 동작합니다.

에이전트는 스펙을 직접 읽지 않습니다. 스펙에서 추출된 자기 전용 룰 파일만 읽습니다. 룰 파일은 두 가지 섹션으로 구성됩니다.

* `## Global Constraints` — 해당 룰 파일을 연결한 모든 태스크에 적용됩니다.
* `## {task id} → ### {step name}` — 특정 태스크의 특정 스텝이 실행될 때만 적용됩니다.

---

## 6. 원칙

`development-principles.md`에는 다음 원칙들이 기준 문서로 정리되어 있습니다.

### 방법론

* **Test-Driven** — 테스트 설계가 구현보다 먼저 이루어집니다. 게이트는 “테스트가 초록불로 실행되는 것”입니다.
* **Domain-Driven** — Glossary는 프로젝트의 유비쿼터스 언어입니다. 용어가 흔들리면 모델도 흔들립니다.
* **Make-or-Buy** — 직접 구현하기보다 검증된 라이브러리 도입을 우선합니다. 단, AC에 묶여 있어야 하며 스펙 권위자가 소유합니다.
* **Shift-Left** — 결함은 가능한 한 가장 이른, 가장 비용이 낮은 단계에서 발견합니다. 애매한 사항은 상위 단계로 되돌립니다.

### 공통 가치

* **Doer/Evaluator 분리** — 만든 사람이 직접 검수하지 않습니다. 계약을 먼저 정의하고 본문은 나중에 작성하며, 내부 구현이 아니라 결과로 검증합니다. 신호와 판단도 분리합니다.
* **Single Source of Truth** — 하나의 사실은 한 곳에만 존재해야 합니다. 다른 곳에서는 복사하지 않고 참조합니다.
* **Ownership Rotates, Never Shares** — 소유권은 한 번에 한 명에게만 있으며, 순차적으로 핸드오프됩니다. 경계는 파일 단위로 관리합니다.
* **Co-Design Boundary** — 가치 판단은 사용자와 함께하고, 기술 판단은 에이전트가 수행합니다. 각 주체에게는 자신이 답할 수 있는 질문만 묻습니다.
* **Document versus Code** — 코드로 표현할 수 있는 것은 코드에 둡니다. 문서는 코드로 표현하기 어려운 의도, 경계, 절차를 설명합니다.

---

## 7. 저장소 구조

```text
harness 001/
├── AGENTS.md                      # L3 진입점: 에이전트 6종 표, 글로벌 룰 15개, 세션 스타트업 스캔
├── CLAUDE.md                      # AGENTS.md로 리다이렉트
│
├── docs/                          # 캐논 문서와 프로젝트 문서
│   ├── harness.md                 # ★ 캐논: 문서 시스템과 운영 구조
│   ├── development-principles.md  # ★ 캐논: 방법론과 가치
│   ├── frontend-agents.md         # 프론트엔드 에이전트 가이드
│   ├── frontend-tech.md           # 프론트엔드 기술 문서
│   └── context/                   # 외부 의존성 문서
│
├── agents/                        # 에이전트 운영 자산
│   ├── personas/                  # pm, ux, ar, qa, te, de
│   ├── tasks/                     # 23개 태스크. HOW를 정의하며 기술 스택에 독립적
│   ├── templates/
│   │   ├── agent/                 # story, dev, test, ux-spec, rules, session-log 등
│   │   └── frontend-web-react/    # 스택별 템플릿
│   ├── docs/
│   │   ├── glossary.md            # 공유 도메인 용어
│   │   └── stories/               # 스토리 수명주기 문서와 archived/
│   └── git-convention.md
│
└── frontend/                      # 스택 종속 spec과 rules
    ├── ar.spec.md  qa.spec.md     # 도메인 권위자가 소유하는 스펙
    └── *.{ar,qa}.rules.md         # 에이전트별 추출 룰
```

---

## 8. 시작하기

이 하니스는 코드를 실행하는 프로그램이 아니라, LLM 에이전트가 **읽고 그에 따라 행동하는 문서 묶음**입니다. 기본 사용 흐름은 다음과 같습니다.

1. **에이전트 활성화**
   에이전트 ID(`pm`, `ux`, `ar`, `qa`, `de`, `te`)를 지정해 호출합니다. 에이전트는 자신의 페르소나 파일을 읽고 역할을 잡은 뒤, `AGENTS.md`에 정의된 세션 스타트업 스캔을 수행합니다.

2. **세션 스타트업 스캔**
   에이전트는 먼저 Data 섹션에 적힌 파일을 읽습니다. 이후 `latest-state.md`에 중단된 작업이 있으면 해당 작업을 우선 재개합니다. 그다음 `.worktrees/story/*/`를 알파벳 순으로 훑어, 자신의 Owner로 표시된 대기 작업을 실행합니다. 처리할 작업이 없으면 입력을 기다립니다. 사용자가 직접 지시한 작업은 이 스캔을 건너뜁니다.

3. **새 작업 시작**
   새 기능이나 변경 요청은 PM에게 전달합니다. PM은 `STORY-XXX`를 만들고, 브랜치와 워크트리를 생성한 뒤 스토리 흐름을 시작합니다.

4. **태스크는 필요할 때 로드**
   페르소나는 에이전트가 *언제* 움직이는지를 정의하고, 태스크는 *어떻게* 수행하는지를 정의합니다. 모든 태스크를 시작 시점에 읽지 않고, 필요한 순간에만 로드합니다.

처음 읽는 경우에는 다음 순서를 권장합니다.

```text
README → docs/harness.md → docs/development-principles.md → AGENTS.md → 관심 있는 persona
```

참고로 `AGENTS.md`와 페르소나에는 `~/{{ProjectRoot}}/...` 같은 플레이스홀더 경로가 들어 있습니다. 실제 프로젝트에 적용할 때는 이 값과 각 페르소나의 `Customization` 섹션을 채워야 합니다. 이것이 OR의 init 라인이 수행하는 작업입니다.

---

## 9. 주요 용어

| 용어                        | 뜻                                                                                |
| ------------------------- | -------------------------------------------------------------------------------- |
| **스토리(Story)**            | 작업의 기본 단위입니다. 요구사항 정의부터 통합까지의 수명주기를 담는 문서이며 `STORY-XXX` 형식을 사용합니다.               |
| **쌍(Pair)**               | 하나의 작업 단위를 이루는 `dev` 문서와 `test` 문서의 조합입니다.                                       |
| **캠프(Camp)**              | dev/test 쌍을 소유한 역할 라인입니다. AR 캠프 또는 QA 캠프로 나뉩니다.                                  |
| **DoD**                   | Definition of Done의 약어입니다. 문서 안의 체크박스로 진행 상태를 표시합니다.                             |
| **핸드오프 노트(Handoff Note)** | `[시각] FROM → TO: 메모` 형식의 기록입니다. 에이전트 간 유일한 공식 통신 채널입니다.                          |
| **소유자(Owner)**            | 현재 해당 문서를 처리할 권한을 가진 에이전트입니다. 파일명과 프론트매터에 표시됩니다.                                 |
| **스펙(Spec)**              | 경로, 도구, 구조처럼 기술 스택에 묶인 바인딩을 담는 문서입니다. AR 또는 QA 같은 도메인 권위자가 소유합니다.                |
| **룰(Rules)**              | 스펙에서 에이전트별로 추출한 실행 규칙입니다. 에이전트가 실제로 읽는 문서입니다.                                    |
| **캐논(Canon)**             | `harness.md`와 `development-principles.md`를 가리킵니다. 다른 문서는 이 두 문서로부터 재생성 가능해야 합니다. |
| **인프라(Infrastructure)**   | 스펙 파일 변경이 필요한 작업입니다. 스펙 권위자의 워크플로로 라우팅됩니다.                                       |

---

# English

> A multi-agent software-development harness — six role agents drive a single story from requirements to integration by handing documents back and forth.

---

## Table of Contents

1. [What is this](#1-what-is-this)
2. [Core idea](#2-core-idea)
3. [Agents](#3-agents)
4. [How work flows](#4-how-work-flows)
5. [Document system](#5-document-system)
6. [Principles](#6-principles)
7. [Repository layout](#7-repository-layout)
8. [Getting started](#8-getting-started)
9. [Glossary of key terms](#9-glossary-of-key-terms)

---

## 1. What is this

harness 001 is an *operating system for LLM agents* that build software by **dividing roles**. It is made of **documents**, not code — personas, tasks, templates, specs, rules. These documents tell each agent who it is, when it acts, what it produces and how, and to whom it hands off.

The core bet is simple: **whoever makes an artifact does not evaluate it.** Designer and implementer, implementer and verifier are kept separate; an artifact has exactly one owner at a time and is passed in sequence. Defects get caught at the cheapest stage, and the blind spot of self-review disappears.

---

## 2. Core idea

The whole system derives from **two canonical documents** (`docs/`):

* **`harness.md`** — the document system and operating structure (the structural *what/how*): file lifecycle, ownership rotation, workflow skeleton, reference direction.
* **`development-principles.md`** — methodologies and values (the *why*): TDD, DDD, Make-or-Buy, Shift-Left, and cross-cutting values such as Doer/Evaluator separation.

Everything else (personas, tasks, rules, specs, templates) is **regenerable** from these two. A rule lives in exactly one place (Single Source of Truth); where it is needed elsewhere, documents **point** to it rather than copy it.

A second axis: **stack-invariant content goes in the canon/principles, stack-bound content (paths, tools, directories) goes in specs, and procedure goes in tasks.** The single placement test — "would it read identically under a different technology stack?"

---

## 3. Agents

| ID     | Role            | Responsibility                                                                                                                      |
| ------ | --------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **OR** | Orchestration   | Manages and improves the harness itself: personas, tasks, templates, session-log review, and the init line. Does not do story work. |
| **PM** | Product Manager | Defines WHAT and WHY. Owns requirements discovery, story writing, handoff, and artifact stewardship.                                |
| **UX** | UX Designer     | Designs WHAT USER SEES. Owns prototypes and UX specs.                                                                               |
| **AR** | Architecture    | Defines HOW: architecture, interfaces, structure, and rules. Owns the runtime spec.                                                 |
| **QA** | QA Lead         | Validates OUTCOMES. Owns the test spec and designs tests.                                                                           |
| **DE** | Developer       | Implements within AR's architecture.                                                                                                |
| **TE** | Tester          | Implements test code within QA's design.                                                                                            |

Role boundaries are strict: PM never decides HOW; QA validates outcomes, not implementation. OR is the only channel that talks to the user; working agents minimize user contact.

---

## 4. How work flows

The unit of work is a **story (`STORY-XXX`)**, run in two phases.

### Phase 1 — Requirements

The Story document is the unit. Ownership rotates **PM → UX → QA → AR**. At each step the agent checks off DoD items, leaves a Handoff Note (`[DateTime] FROM → TO: note`), and renames the Owner to the next agent.

### Phase 2 — Implementation

The unit becomes a **dev/test pair**. AR decomposes the story into work units; each unit pairs a `dev` document (contract + implementation) with a `test` document (its verification design), matched by sequence:

```text
dev-XXX-NN ↔ test-XXX-NN
```

The Story document is read-only and Owner stays `AR` (sleeping) throughout.

### Pair Camp model

A dev/test pair always lives in one camp:

* **AR camp** — senior AR designs or reviews the contract; junior DE implements the body.
* **QA camp** — senior QA designs or reviews the test; junior TE implements the test code.

A camp transition (AR ↔ QA) happens only when both documents are held by that camp's senior. So any cross-document redesign is performed by one senior owning both documents at once. Ownership is never shared.

DoD flow:

```text
dev:   Designed(AR) → Implemented(DE) → Reviewed(AR) → Verified(QA)
test:  Designed(QA) → Implemented(TE) → Reviewed(QA)
```

Per TDD, **test design precedes dev implementation.** The gate on implementation is the **test executing green**, not a document review.

When every dev document is `Complete` and every test document is QA-owned, AR integrates and the story closes.

---

## 5. Document system

Every document gets a level by **communication direction**:

| Level | Direction             | Examples                                              |
| ----- | --------------------- | ----------------------------------------------------- |
| 1     | OR → user             | harness, development-principles, README               |
| 2     | OR → OR               | orchestration internal docs                           |
| 3     | OR → agent            | AGENTS.md, personas, templates, tasks, git-convention |
| 4     | agent → user          | project artifacts                                     |
| 5     | agent → self          | session logs, latest-state, **spec**                  |
| 6     | agent → agent         | UX specs, **rules**                                   |
| 7     | multi-agent ownership | stories                                               |

**Document types and what they hold:**

| Type         | Holds                                                                     | Location                             |
| ------------ | ------------------------------------------------------------------------- | ------------------------------------ |
| **persona**  | *when* and in what order an agent acts                                    | `agents/personas/{agent}.md`         |
| **task**     | *how* a job is done (procedure, stack-agnostic)                           | `agents/tasks/{name}.task.md`        |
| **template** | *what* an output document looks like (structure only)                     | `agents/templates/{name}.tmp.md`     |
| **spec**     | stack bindings (paths, tools, structure rules); owned by domain authority | `{area}/{agent}.spec.md`             |
| **rules**    | per-agent extraction of spec bindings                                     | `{area}/{consumer}.{owner}.rules.md` |
| **index**    | lookup table for selective loading                                        | `{area}/{name}.idx.md`               |
| **story**    | lifecycle document for one story                                          | `agents/docs/stories/STORY-XXX/`     |

**Reference direction (hard rule):**

```text
workflow → task → rules → spec
```

Lower levels never reference higher levels. The two canon documents reference nothing and are referenced by nothing — they are human-facing canon, and agents operate from the documents *generated* from them.

**Agents never read specs directly** — only their own extracted rules file. A rules file has two section kinds:

* `## Global Constraints` — applies to every task linking it.
* `## {task id} → ### {step name}` — applies only while that step runs.

---

## 6. Principles

Stated canonically in `development-principles.md`:

### Methodologies

* **Test-Driven** — test design precedes implementation; the gate is "test runs green."
* **Domain-Driven** — the Glossary is the project's ubiquitous language. Term drift is model drift.
* **Make-or-Buy** — prefer introducing a library over hand-building; AC-bound, owned by the spec authority.
* **Shift-Left** — catch defects at the cheapest stage; when in doubt, bounce upstream.

### Cross-cutting values

* **Separation of Doer and Evaluator** — the maker doesn't review; contract before body; verify outcomes not internals; signal vs. judgment.
* **Single Source of Truth** — a fact lives in one place; elsewhere points to it.
* **Ownership Rotates, Never Shares** — one owner at a time, sequential handoff, file-level boundaries.
* **Co-Design Boundary** — value decisions with the user, technical decisions by agents; each asked only what it can answer.
* **Document versus Code** — if expressible in code, it belongs in code; documents describe only what code cannot.

---

## 7. Repository layout

```text
harness 001/
├── AGENTS.md                      # L3 entry point: agent table, 15 global rules, session startup scan
├── CLAUDE.md                      # redirects to AGENTS.md
│
├── docs/                          # canon + project docs
│   ├── harness.md                 # ★ canon: document system + operating structure
│   ├── development-principles.md  # ★ canon: methodologies + values
│   ├── frontend-agents.md         # frontend agent guide
│   ├── frontend-tech.md           # frontend tech
│   └── context/                   # external dependency docs
│
├── agents/                        # agent operating assets
│   ├── personas/                  # pm, ux, ar, qa, te, de
│   ├── tasks/                     # 23 tasks. HOW, stack-agnostic
│   ├── templates/
│   │   ├── agent/                 # story, dev, test, ux-spec, rules, session-log ...
│   │   └── frontend-web-react/    # stack-specific templates
│   ├── docs/
│   │   ├── glossary.md            # shared domain terms
│   │   └── stories/               # story lifecycle docs (+ archived/)
│   └── git-convention.md
│
└── frontend/                      # stack-bound spec + rules
    ├── ar.spec.md  qa.spec.md     # domain-authority specs
    └── *.{ar,qa}.rules.md         # per-agent extracted rules
```

---

## 8. Getting started

This harness is not "run" — it is a document bundle that LLM agents **read and act on**. To use it:

1. **Activate an agent.** Invoke it by agent ID (`pm`, `ux`, `ar`, `qa`, `de`, `te`). The agent reads its persona file, stays in character, then runs the **Session Startup Scan** (see `AGENTS.md`).

2. **Session Startup Scan.** The agent first reads every file in the Data section. If there is interrupted work in `latest-state.md`, it resumes that work first. It then scans `.worktrees/story/*/` alphabetically for queued work whose Owner matches, and executes it. If there is no queued work, it waits for input. User-triggered work bypasses this scan.

3. **Start new work.** Bring a new feature or change to PM. PM creates `STORY-XXX`, the branch and worktree, and the flow begins.

4. **Tasks load on demand.** Personas define *when*; tasks define *how*. Tasks are not all read at startup — they are loaded only when needed.

If you're new, read in this order:

```text
README → docs/harness.md → docs/development-principles.md → AGENTS.md → a persona you care about
```

Note: paths in `AGENTS.md` and the personas use placeholders like `~/{{ProjectRoot}}/...`. To apply the harness to a real project, fill these and the `Customization` sections. This is what OR's init line does.

---

## 9. Glossary of key terms

| Term               | Meaning                                                                                                     |
| ------------------ | ----------------------------------------------------------------------------------------------------------- |
| **Story**          | The unit of work — a lifecycle document from requirements to integration, named `STORY-XXX`.                |
| **Pair**           | The `dev` + `test` documents of one work unit.                                                              |
| **Camp**           | The role line owning a pair, either AR camp or QA camp.                                                     |
| **DoD**            | Definition of Done — in-document checkboxes marking progress.                                               |
| **Handoff Note**   | `[DateTime] FROM → TO: note` — the only official communication channel between agents.                      |
| **Owner**          | The agent that may move a document now. Recorded in the filename and frontmatter.                           |
| **Spec**           | Stack-bound bindings such as paths, tools, and structure. Owned by a domain authority such as AR or QA.     |
| **Rules**          | Per-agent extraction of the spec — what agents actually read.                                               |
| **Canon**          | `harness.md` + `development-principles.md`. Everything else should be regenerable from these two documents. |
| **Infrastructure** | Any change requiring a spec-file change. Routed through the workflow of the spec authority.                 |