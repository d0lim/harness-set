---
name: repo-merger
description: "brownfield git repository 여러 개를 하나의 통합 프로젝트로 논리적으로 재구성하는 오케스트레이터 스킬. 'repo 합쳐줘', 'monorepo로 만들어줘', 'repository 통합', '프로젝트 합치기', '여러 repo를 하나로', 'codebase consolidation' 등의 요청 시 반드시 이 스킬을 사용할 것. repo 분석, 통합 구조 설계, 사용자 피드백 루프, TDD 스타일 마이그레이션 실행, 검증 및 리포트 생성까지 전체 워크플로우를 조율한다. 단순히 git merge나 subtree를 수행하는 것이 아니라, 의존성 통합, 빌드 파이프라인 단일화, 서비스 경계 재설계, 코딩 컨벤션 통일, CLAUDE.md 생성까지 수행한다."
---

# Repo Merger — Repository 통합 오케스트레이터

brownfield git repository 여러 개를 하나의 통합 프로젝트로 논리적으로 재구성한다.

## 실행 모드: 서브 에이전트

Phase별로 다른 에이전트 조합이 필요하므로, 오케스트레이터가 Agent 도구로 서브 에이전트를 직접 호출한다.

- Phase 1: Analyzer → Architect (순차)
- Phase 2: Verifier + Migrator (TDD 루프)

모든 Agent 호출에 `model: "opus"` 파라미터를 명시한다.

## 사전 조건

- 대상 repo들이 로컬에 클론되어 있어야 한다.
- 사용자가 repo 경로(또는 repo들이 들어있는 디렉토리)와 간단한 통합 목적을 제공해야 한다.

## 워크플로우

### Phase 0: 입력 확인 및 준비

1. 사용자 입력에서 repo 경로 목록과 통합 목적을 파악한다.
2. 디렉토리 경로만 주어진 경우, 하위 디렉토리 중 `.git`이 있는 것을 repo로 식별한다.
3. 각 repo 경로가 실제 존재하는지 확인한다.
4. 새 프로젝트 디렉토리 경로를 사용자에게 제안하고 확인받는다.
5. 새 프로젝트 디렉토리와 `_workspace/` 하위 디렉토리를 생성한다.

### Phase 1: 분석-설계

**1-1. Analyzer 실행:**

```
Agent(
  prompt: "analyze-repos 스킬을 읽고 실행하라. 대상 repo 경로: {repo_paths}. 분석 리포트를 {target}/_workspace/01_analyzer_report.md에 저장하라.",
  agents: ["repo-merger/agents/analyzer.md"],
  model: "opus"
)
```

Analyzer 완료 후 `_workspace/01_analyzer_report.md`가 생성되었는지 확인한다.

**1-2. Architect 실행:**

```
Agent(
  prompt: "design-integration 스킬을 읽고 실행하라. 분석 리포트: {target}/_workspace/01_analyzer_report.md. 통합 목적: {purpose}. 설계 결과를 _workspace/02_architect_*.md 파일들에 저장하라.",
  agents: ["repo-merger/agents/architect.md"],
  model: "opus"
)
```

Architect는 사용자와 피드백 루프를 돈다. 오케스트레이터는 사용자 메시지를 Architect에게 중계한다. Architect가 설계 확정을 알리면 Phase 2로 진행한다.

확정 산출물 확인:
- `_workspace/02_architect_design.md` 존재
- `_workspace/02_architect_claude_md.md` 존재
- `_workspace/02_architect_verification_criteria.md` 존재

### Phase 2: 실행-검증 (TDD 루프)

**2-1. Verifier 검증 기준 작성:**

```
Agent(
  prompt: "verify-integration 스킬을 읽고, 검증 체크리스트를 작성하라. 설계서: {target}/_workspace/02_architect_design.md, 검증 기준: {target}/_workspace/02_architect_verification_criteria.md. 체크리스트를 {target}/_workspace/03_verifier_tests.md에 저장하라.",
  agents: ["repo-merger/agents/verifier.md"],
  model: "opus"
)
```

**2-2. Step별 실행-검증 루프:**

설계서의 마이그레이션 단계(Step 1~5) 각각에 대해:

```
# Migrator 실행
Agent(
  prompt: "execute-migration 스킬을 읽고, Step {N}을 실행하라. 설계서: {target}/_workspace/02_architect_design.md. 원본 repo: {repo_paths}. 타겟: {target}.",
  agents: ["repo-merger/agents/migrator.md"],
  model: "opus"
)

# Verifier 검증
Agent(
  prompt: "verify-integration 스킬을 읽고, Step {N}의 검증을 실행하라. 체크리스트: {target}/_workspace/03_verifier_tests.md. 타겟: {target}.",
  agents: ["repo-merger/agents/verifier.md"],
  model: "opus"
)
```

FAIL 시:
1. Verifier의 피드백을 Migrator에게 전달하여 수정 실행
2. 재검증
3. 최대 3회 재시도 후에도 FAIL이면 사용자에게 보고하고 판단을 요청

**2-3. 최종 리포트:**

모든 Step 통과 후:

```
Agent(
  prompt: "verify-integration 스킬을 읽고, 최종 리포트를 생성하라. 체크리스트: {target}/_workspace/03_verifier_tests.md. 원본 repo: {repo_paths}. 타겟: {target}. 리포트를 {target}/_workspace/04_verifier_final_report.md에 저장하라.",
  agents: ["repo-merger/agents/verifier.md"],
  model: "opus"
)
```

### Phase 3: 완료

1. 최종 리포트를 사용자에게 제시한다.
2. 새 프로젝트 디렉토리에서 `git init`을 실행할지 사용자에게 묻는다.
3. `_workspace/`는 보존한다 (감사 추적용).

## 에러 핸들링

| 에러 유형 | 전략 |
|----------|------|
| repo 경로 불존재 | 사용자에게 알리고 올바른 경로를 다시 요청 |
| Analyzer 실패 | 접근 불가 repo 건너뛰기, 리포트에 누락 명시 |
| 의존성 충돌 미해결 | 사용자에게 선택지 제시 |
| 빌드 실패 (3회 재시도 후) | 사용자에게 보고, 수동 개입 요청 |
| 테스트 실패 | 실패 목록 보고, 사용자 판단에 따라 진행/중단 |

## 테스트 시나리오

### 정상 흐름
1. 사용자: "~/projects/repos/ 안의 3개 repo를 하나의 Next.js monorepo로 합쳐줘"
2. Phase 0: repo 3개 식별, 새 디렉토리 ~/projects/merged-app 생성
3. Phase 1-1: Analyzer가 React FE 2개 + Node.js BE 1개로 분석
4. Phase 1-2: Architect가 "인증 불필요 FE 2개 통합 + BE 별도 패키지" 제안 → 사용자 합의
5. Phase 2: 5단계 실행-검증 모두 PASS
6. Phase 3: 최종 리포트 제시, git init 여부 확인

### 에러 흐름
1. Step 2에서 React 버전 충돌 → Verifier FAIL
2. Migrator 수정 시도 → 재검증 FAIL
3. 사용자에게 "React 17 vs 18 선택 필요" 보고
4. 사용자 결정 → Migrator 재실행 → PASS
