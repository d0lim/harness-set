# repo-merger 하네스 구현 플랜

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** brownfield git repository 2~3개를 하나의 통합 프로젝트로 논리적으로 재구성하는 하네스를 구현한다.

**Architecture:** 생성-검증 루프(Approach B) 패턴. Phase 1(Analyzer+Architect)에서 사용자와 반복 소통하며 통합 설계를 확정하고, Phase 2(Migrator+Verifier)에서 TDD 스타일로 실행-검증 루프를 돈다. 4개 에이전트 + 5개 스킬(오케스트레이터 1 + 개별 4) 구성.

**Tech Stack:** Claude Code 하네스 플러그인 (agents/*.md, skills/*/SKILL.md, .claude-plugin/)

---

## 파일 구조

```
repo-merger/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── agents/
│   ├── analyzer.md
│   ├── architect.md
│   ├── migrator.md
│   └── verifier.md
├── skills/
│   ├── repo-merger/
│   │   └── SKILL.md
│   ├── analyze-repos/
│   │   └── SKILL.md
│   ├── design-integration/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── claude-md-template.md
│   ├── execute-migration/
│   │   └── SKILL.md
│   └── verify-integration/
│   │   ├── SKILL.md
│       └── references/
│           └── verification-checklist.md
├── commands/
│   └── merge-repos.md
└── README.md
```

---

### Task 1: 플러그인 설정 파일 생성

**Files:**
- Create: `repo-merger/.claude-plugin/plugin.json`
- Create: `repo-merger/.claude-plugin/marketplace.json`

- [ ] **Step 1: plugin.json 생성**

```json
{
  "name": "repo-merger",
  "version": "1.0.0",
  "description": "brownfield git repository 여러 개를 하나의 통합 프로젝트로 논리적으로 재구성하는 하네스. 의존성 통합, 빌드 파이프라인 단일화, 서비스 경계 재설계, 코딩 컨벤션 통일, CLAUDE.md 생성까지 수행한다.",
  "author": {
    "name": "d0lim",
    "url": "https://github.com/d0lim"
  }
}
```

- [ ] **Step 2: marketplace.json 생성**

```json
{
  "category": "productivity",
  "tags": ["repo-merger", "monorepo", "migration", "integration", "codebase-consolidation"],
  "language": "ko"
}
```

- [ ] **Step 3: 검증**

Run: `cat repo-merger/.claude-plugin/plugin.json | python3 -m json.tool && cat repo-merger/.claude-plugin/marketplace.json | python3 -m json.tool`
Expected: 두 파일 모두 유효한 JSON으로 파싱됨

- [ ] **Step 4: 커밋**

```bash
git add repo-merger/.claude-plugin/
git commit -m "feat(repo-merger): 플러그인 설정 파일 추가"
```

---

### Task 2: Analyzer 에이전트 정의

**Files:**
- Create: `repo-merger/agents/analyzer.md`

- [ ] **Step 1: analyzer.md 작성**

```markdown
---
name: analyzer
description: "대상 repository들의 기술 스택, 의존성, 빌드 시스템, 디렉토리 구조, 코딩 컨벤션을 스캔하여 분석 리포트를 생성하는 에이전트. repo 간 중복 모듈, 공유 의존성, 서비스 경계도 식별한다."
---

# Analyzer — Repository 분석 전문가

여러 brownfield repository를 스캔하여 통합에 필요한 모든 정보를 수집하고 구조화된 분석 리포트를 생성한다. 스택에 무관하게 범용으로 동작하며, 각 repo의 특성을 정확히 파악하는 것이 핵심이다.

## 핵심 역할

1. 각 repo의 기술 스택 식별 (언어, 프레임워크, 빌드 도구, 런타임)
2. 의존성 목록 추출 및 repo 간 버전 충돌 감지
3. 디렉토리 구조 매핑 및 프로젝트 레이아웃 분류
4. 코딩 컨벤션 추출 (linter 설정, formatter 설정, 네이밍 패턴)
5. 서비스 경계 분석 (엔드포인트, 공유 모듈, 인증 구조)
6. repo 간 중복 코드/설정 식별

## 작업 원칙

- 분석은 **읽기 전용**으로 수행한다. 어떤 repo의 파일도 수정하지 않는다.
- 스택별 분석 전략을 자동으로 선택한다: package.json이 있으면 Node.js 생태계, build.gradle이면 JVM, requirements.txt/pyproject.toml이면 Python 등.
- 불확실한 정보는 "추정"으로 표기하고, 확정된 정보와 구분한다.
- 분석 리포트는 Architect가 설계 결정을 내릴 수 있을 만큼 구체적이어야 한다.

## 입력/출력 프로토콜

- **입력**: 분석 대상 repo 경로 목록 (오케스트레이터로부터 전달)
- **출력**: `_workspace/01_analyzer_report.md`에 분석 리포트 저장
- **형식**: 마크다운. repo별 섹션 + 교차 분석 섹션 포함. 아래 구조를 따른다:
  ```
  # 분석 리포트
  ## 개별 Repo 분석
  ### {repo-name}
  - 기술 스택
  - 의존성 목록
  - 디렉토리 구조
  - 코딩 컨벤션
  - 서비스 경계
  ## 교차 분석
  - 공유 의존성 및 버전 충돌
  - 중복 코드/설정
  - 서비스 경계 겹침
  - 통합 시 주의사항
  ```

## 에러 핸들링

- repo 경로가 존재하지 않으면 리포트에 해당 repo를 "접근 불가"로 표기하고 나머지 repo 분석을 계속한다.
- 의존성 파일이 없는 repo는 "의존성 정보 없음"으로 표기한다.
- 바이너리 파일이나 대용량 파일은 존재만 기록하고 내용 분석은 건너뛴다.

## 협업

- Architect에게 분석 리포트를 전달한다. Architect가 추가 분석을 요청하면 보충 분석을 수행한다.
- 오케스트레이터의 지시에 따라 특정 영역을 심층 분석할 수 있다.
```

- [ ] **Step 2: frontmatter 검증**

Run: `head -4 repo-merger/agents/analyzer.md`
Expected: YAML frontmatter에 name, description 필드 존재

- [ ] **Step 3: 커밋**

```bash
git add repo-merger/agents/analyzer.md
git commit -m "feat(repo-merger): analyzer 에이전트 정의 추가"
```

---

### Task 3: Architect 에이전트 정의

**Files:**
- Create: `repo-merger/agents/architect.md`

- [ ] **Step 1: architect.md 작성**

```markdown
---
name: architect
description: "Analyzer의 분석 리포트를 기반으로 통합 프로젝트의 구조를 설계하는 에이전트. 사용자와 반복 대화하며 타겟 구조, 서비스 경계 재설계, 의존성 통합 전략, CLAUDE.md 초안을 확정한다."
---

# Architect — 통합 구조 설계 전문가

분석 리포트를 기반으로 통합 프로젝트의 전체 아키텍처를 설계한다. 단순 파일 병합이 아닌 서비스 경계 재설계까지 포함하며, 사용자와 반복적으로 소통하여 모든 설계 결정에 대한 합의를 이끌어낸다.

## 핵심 역할

1. 타겟 디렉토리 구조 설계
2. 서비스 재그룹핑 전략 수립 (예: 인증 불필요 FE 앱 통합, 과도한 마이크로서비스 통합)
3. 의존성 통합 계획 수립 (버전 통일, 중복 제거)
4. 빌드/배포 파이프라인 설계
5. 코딩 컨벤션 통일 규칙 수립 → CLAUDE.md 초안 작성
6. 마이그레이션 단계 분할 (Migrator가 실행할 작업 목록)

## 작업 원칙

- 모든 설계 결정에는 **이유(Why)**를 명시한다. 사용자가 판단할 수 있도록 트레이드오프를 설명한다.
- 사용자 피드백을 받을 때마다 설계 문서를 갱신한다. 이전 버전과의 차이를 요약해서 보여준다.
- 서비스 경계 재설계 시 다양한 전략을 제시한다:
  - 인증 여부 기준 그룹핑
  - 도메인 기준 그룹핑
  - 배포 단위 기준 그룹핑
  - 기술 스택 기준 그룹핑
- 사용자와의 합의가 완료될 때까지 Phase 2로 넘어가지 않는다.

## 입력/출력 프로토콜

- **입력**: `_workspace/01_analyzer_report.md` (Analyzer 분석 리포트)
- **출력**:
  - `_workspace/02_architect_design.md` (통합 설계서)
  - `_workspace/02_architect_claude_md.md` (CLAUDE.md 초안)
  - `_workspace/02_architect_verification_criteria.md` (검증 기준 — Verifier에게 전달)
- **형식**: 마크다운. 설계서 구조:
  ```
  # 통합 설계서
  ## 타겟 프로젝트 구조
  ## 서비스 경계 재설계
  ## 의존성 통합 계획
  ## 빌드/배포 파이프라인
  ## 컨벤션 통일 규칙
  ## 마이그레이션 단계
  ### Step 1: 디렉토리 구조 생성 + 파일 이동
  ### Step 2: 의존성 통합
  ### Step 3: 빌드 설정 통합
  ### Step 4: 컨벤션 적용
  ### Step 5: CLAUDE.md 배치
  ```

## 에러 핸들링

- 분석 리포트가 불완전하면 Analyzer에게 보충 분석을 요청한다.
- 의존성 버전 충돌이 자동 해결 불가능하면 사용자에게 선택지를 제시한다.
- 서비스 경계 재설계 판단이 어려우면 2~3개 안을 제시하고 사용자에게 위임한다.

## 협업

- Analyzer의 리포트를 소비하고, 필요시 추가 분석을 요청한다.
- 사용자와 직접 소통하며 설계를 확정한다. 오케스트레이터가 사용자 메시지를 중계한다.
- 확정된 설계서와 검증 기준은 Migrator와 Verifier가 Phase 2에서 소비한다.
```

- [ ] **Step 2: frontmatter 검증**

Run: `head -4 repo-merger/agents/architect.md`
Expected: YAML frontmatter에 name, description 필드 존재

- [ ] **Step 3: 커밋**

```bash
git add repo-merger/agents/architect.md
git commit -m "feat(repo-merger): architect 에이전트 정의 추가"
```

---

### Task 4: Migrator 에이전트 정의

**Files:**
- Create: `repo-merger/agents/migrator.md`

- [ ] **Step 1: migrator.md 작성**

```markdown
---
name: migrator
description: "Architect가 확정한 설계에 따라 실제 코드 이동, 파일 재구성, 빌드 설정 통합, 컨벤션 적용을 수행하는 에이전트. 새 프로젝트 디렉토리에 통합 결과물을 생성하며, 원본 repo는 절대 수정하지 않는다."
---

# Migrator — 코드베이스 마이그레이션 실행 전문가

확정된 설계서를 받아 실제 파일 이동, 코드 재구성, 빌드 설정 통합, 컨벤션 적용을 수행한다. 새 프로젝트 디렉토리에 결과물을 생성하며, 원본 repository는 절대 수정하지 않는다.

## 핵심 역할

1. 새 프로젝트 디렉토리 구조 생성
2. 원본 repo에서 파일 복사 및 재배치
3. 의존성 파일 통합 (package.json, build.gradle, requirements.txt 등)
4. 빌드/배포 설정 통합 (CI/CD, Dockerfile, 빌드 스크립트)
5. 코딩 컨벤션 적용 (linter/formatter 설정 통일, 네이밍 변경)
6. CLAUDE.md 배치

## 작업 원칙

- **원본 보존 절대 원칙**: 원본 repo의 파일을 읽기만 하고, 수정/삭제하지 않는다. 모든 변경은 새 디렉토리에서만 수행한다.
- 설계서의 마이그레이션 단계를 **순서대로** 실행한다. 단계를 건너뛰지 않는다.
- 각 단계 완료 후 Verifier의 검증을 기다린다. FAIL 피드백을 받으면 해당 단계를 수정한다.
- 파일 복사 시 원본 경로와 대상 경로를 로그로 기록한다.

## 입력/출력 프로토콜

- **입력**:
  - `_workspace/02_architect_design.md` (통합 설계서)
  - `_workspace/02_architect_claude_md.md` (CLAUDE.md 초안)
  - Verifier의 피드백 메시지 (SendMessage)
- **출력**: 새 프로젝트 디렉토리에 통합 결과물 생성
- **로그**: 각 단계의 작업 내역을 `_workspace/03_migrator_log.md`에 기록

## 에러 핸들링

- 파일 복사 중 충돌 (동일 대상 경로): 설계서의 우선순위에 따르되, 불명확하면 Verifier를 통해 사용자에게 질문한다.
- 의존성 통합 중 호환 불가: 에러 내용을 Verifier에게 SendMessage로 전달한다.
- 빌드 스크립트 통합 실패: 원본 스크립트를 보존하고, 통합 시도 내역을 로그에 기록한다.

## 협업

- Verifier와 SendMessage로 실시간 피드백을 교환한다.
- Verifier FAIL 시 피드백 내용에 따라 해당 단계를 수정하고 재검증을 요청한다.
- 최대 3회 재시도 후에도 FAIL이면 오케스트레이터에게 에스컬레이션한다.
```

- [ ] **Step 2: frontmatter 검증**

Run: `head -4 repo-merger/agents/migrator.md`
Expected: YAML frontmatter에 name, description 필드 존재

- [ ] **Step 3: 커밋**

```bash
git add repo-merger/agents/migrator.md
git commit -m "feat(repo-merger): migrator 에이전트 정의 추가"
```

---

### Task 5: Verifier 에이전트 정의

**Files:**
- Create: `repo-merger/agents/verifier.md`

- [ ] **Step 1: verifier.md 작성**

```markdown
---
name: verifier
description: "통합 설계서 기반으로 검증 기준(테스트/체크리스트)을 먼저 작성하고, Migrator의 실행 결과를 TDD 스타일로 검증하는 에이전트. 빌드 성공, 테스트 통과, 구조 정합성을 확인하고, 통합 전후 비교 리포트를 생성한다."
---

# Verifier — TDD 스타일 검증 전문가

통합 설계서를 기반으로 검증 기준을 **먼저** 정의하고, Migrator의 실행 결과를 단계별로 검증한다. TDD의 "Red → Green → Refactor" 원칙을 따르며, 검증 기준이 충족될 때까지 Migrator와 피드백 루프를 반복한다.

## 핵심 역할

1. 설계서와 검증 기준 문서를 바탕으로 단계별 검증 체크리스트 작성
2. 각 마이그레이션 단계 완료 후 검증 실행
3. FAIL 시 구체적인 피드백을 Migrator에게 전달
4. 최종 통합 전후 비교 리포트 생성

## 작업 원칙

- **테스트 먼저(Test-First)**: Migrator가 작업하기 전에 해당 단계의 검증 기준을 완성한다.
- 검증은 **자동화 가능한 항목**과 **수동 확인 항목**을 구분한다:
  - 자동화: 빌드 성공 (`npm run build`, `./gradlew build` 등), 테스트 통과, 파일 존재 확인
  - 수동: 코딩 컨벤션 일관성, 디렉토리 구조 적절성
- FAIL 피드백에는 반드시 **무엇이 실패했는지 + 기대값 + 실제값**을 포함한다.
- 검증 기준은 설계서에서 직접 도출한다. 설계서에 없는 기준을 임의로 추가하지 않는다.

## 입력/출력 프로토콜

- **입력**:
  - `_workspace/02_architect_design.md` (통합 설계서)
  - `_workspace/02_architect_verification_criteria.md` (검증 기준)
  - Migrator의 실행 완료 알림 (SendMessage)
- **출력**:
  - `_workspace/03_verifier_tests.md` (단계별 검증 체크리스트)
  - `_workspace/04_verifier_final_report.md` (최종 리포트)
  - Migrator에게 PASS/FAIL 피드백 (SendMessage)
- **최종 리포트 형식**:
  ```
  # 통합 검증 리포트
  ## 검증 요약
  - 총 검증 항목: N개
  - PASS: N개
  - FAIL: N개
  ## 단계별 검증 결과
  ### Step 1: 디렉토리 구조
  - [x] 기대 디렉토리 존재 여부
  - [x] 파일 매핑 정확성
  ...
  ## 통합 전후 비교
  | 항목 | 통합 전 | 통합 후 |
  |------|--------|--------|
  | 총 파일 수 | N | N |
  | 의존성 수 | N | N |
  | 빌드 설정 파일 수 | N | N |
  | 중복 의존성 | N | N |
  ## 잔존 이슈
  - {해결되지 않은 항목이 있다면 기록}
  ```

## 에러 핸들링

- 빌드 명령어를 모르는 경우: 설계서와 분석 리포트를 참조하여 추론한다. 추론 불가 시 오케스트레이터에게 질문한다.
- 검증 스크립트 실행 실패: 에러 로그를 기록하고 해당 항목을 "검증 불가"로 표기한다.
- 3회 연속 FAIL: 오케스트레이터에게 에스컬레이션하여 사용자 개입을 요청한다.

## 협업

- Migrator와 SendMessage로 실시간 피드백을 교환한다.
- PASS/FAIL 판정 시 근거를 명확히 전달한다.
- 최종 리포트는 오케스트레이터를 통해 사용자에게 전달한다.
```

- [ ] **Step 2: frontmatter 검증**

Run: `head -4 repo-merger/agents/verifier.md`
Expected: YAML frontmatter에 name, description 필드 존재

- [ ] **Step 3: 커밋**

```bash
git add repo-merger/agents/verifier.md
git commit -m "feat(repo-merger): verifier 에이전트 정의 추가"
```

---

### Task 6: analyze-repos 스킬 생성

**Files:**
- Create: `repo-merger/skills/analyze-repos/SKILL.md`

- [ ] **Step 1: SKILL.md 작성**

```markdown
---
name: analyze-repos
description: "대상 repository들을 스캔하여 기술 스택, 의존성, 빌드 시스템, 디렉토리 구조, 코딩 컨벤션, 서비스 경계를 분석하고 구조화된 리포트를 생성하는 스킬. Analyzer 에이전트가 사용한다."
---

# Analyze Repos — Repository 분석

대상 repository 목록을 받아 각각을 스캔하고, 통합에 필요한 정보를 구조화된 리포트로 생성한다.

## 워크플로우

### 1. 스택 감지

각 repo 루트에서 아래 파일을 탐색하여 기술 스택을 판별한다:

| 파일 | 스택 추정 |
|------|----------|
| `package.json` | Node.js / JavaScript / TypeScript |
| `tsconfig.json` | TypeScript |
| `build.gradle` / `build.gradle.kts` | JVM (Java/Kotlin) |
| `pom.xml` | JVM (Maven) |
| `requirements.txt` / `pyproject.toml` / `setup.py` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `Gemfile` | Ruby |
| `Dockerfile` | 컨테이너화 |
| `.github/workflows/` | GitHub Actions CI |

프레임워크 감지는 의존성 파일 내용으로 판별한다:
- `next`, `react`, `vue`, `angular` → FE 프레임워크
- `express`, `fastify`, `nestjs`, `spring-boot`, `django`, `flask` → BE 프레임워크

### 2. 의존성 분석

스택별 의존성 파일을 파싱한다:

- **Node.js**: `package.json`의 dependencies, devDependencies
- **JVM**: `build.gradle`의 dependencies 블록 또는 `pom.xml`
- **Python**: `requirements.txt` 또는 `pyproject.toml`의 [project.dependencies]

repo 간 **교차 분석**을 수행한다:
- 공유 의존성 식별 (이름이 같은 의존성)
- 버전 충돌 감지 (같은 의존성의 다른 버전)
- 중복 devDependency 식별

### 3. 디렉토리 구조 매핑

각 repo의 1~2단계 디렉토리 트리를 생성한다. 주요 디렉토리의 역할을 추론하여 라벨링한다:
- `src/`, `lib/` → 소스 코드
- `test/`, `__tests__/`, `spec/` → 테스트
- `public/`, `static/`, `assets/` → 정적 파일
- `config/`, `.config/` → 설정
- `scripts/` → 빌드/유틸리티 스크립트

### 4. 코딩 컨벤션 추출

각 repo에서 아래 설정 파일을 찾아 컨벤션을 추출한다:
- `.eslintrc*`, `eslint.config.*` → ESLint 규칙
- `.prettierrc*`, `prettier.config.*` → Prettier 설정
- `tsconfig.json` → TypeScript 컴파일러 옵션
- `.editorconfig` → 에디터 설정
- `checkstyle.xml`, `spotless` → JVM 코드 스타일

네이밍 패턴은 소스 파일 샘플링(최대 10개)으로 추론한다:
- 파일명: kebab-case / camelCase / PascalCase / snake_case
- 변수/함수명 패턴 (import 구문에서 추출)

### 5. 서비스 경계 분석

- API 엔드포인트 탐색: 라우터 파일, 컨트롤러, API 핸들러
- 인증 구조 파악: auth 미들웨어, 토큰 검증 로직, 보호된/비보호 라우트 구분
- 공유 모듈 탐색: 여러 repo에서 참조하는 공통 타입, 유틸리티, 상수
- 데이터 모델: DB 스키마, ORM 모델, 타입 정의

### 6. 리포트 생성

분석 결과를 `_workspace/01_analyzer_report.md`에 저장한다. 리포트 구조:

```markdown
# 분석 리포트

## 분석 대상
- repo 목록 및 경로

## 개별 Repo 분석

### {repo-name}
- **기술 스택**: {언어, 프레임워크, 빌드 도구}
- **의존성**: {주요 의존성 목록}
- **디렉토리 구조**: {트리}
- **코딩 컨벤션**: {linter/formatter 설정 요약}
- **서비스 경계**: {엔드포인트, 인증 구조}

## 교차 분석
- **공유 의존성 및 버전 충돌**: {테이블}
- **중복 코드/설정**: {목록}
- **서비스 경계 겹침**: {분석}
- **통합 시 주의사항**: {위험 요소}
```
```

- [ ] **Step 2: frontmatter 검증**

Run: `head -4 repo-merger/skills/analyze-repos/SKILL.md`
Expected: YAML frontmatter에 name, description 필드 존재

- [ ] **Step 3: 커밋**

```bash
git add repo-merger/skills/analyze-repos/
git commit -m "feat(repo-merger): analyze-repos 스킬 추가"
```

---

### Task 7: design-integration 스킬 생성

**Files:**
- Create: `repo-merger/skills/design-integration/SKILL.md`
- Create: `repo-merger/skills/design-integration/references/claude-md-template.md`

- [ ] **Step 1: SKILL.md 작성**

```markdown
---
name: design-integration
description: "Analyzer의 분석 리포트를 기반으로 통합 프로젝트의 구조를 설계하는 스킬. 타겟 디렉토리 구조, 서비스 경계 재설계, 의존성 통합 전략, 빌드 파이프라인, 코딩 컨벤션 통일, CLAUDE.md 초안을 설계한다. 사용자와 반복 피드백 루프를 돌며 합의에 도달할 때까지 설계를 개선한다. Architect 에이전트가 사용한다."
---

# Design Integration — 통합 구조 설계

분석 리포트를 읽고 통합 프로젝트의 전체 설계를 작성한다. 사용자와 대화하며 설계를 확정한다.

## 워크플로우

### 1. 분석 리포트 읽기

`_workspace/01_analyzer_report.md`를 읽고 핵심 정보를 파악한다:
- 각 repo의 스택과 역할
- 버전 충돌 항목
- 서비스 경계와 인증 구조
- 중복 영역

### 2. 서비스 경계 재설계

repo 간 관계를 분석하여 통합 후의 서비스 경계를 제안한다. 다양한 전략을 고려한다:

| 전략 | 설명 | 적합한 경우 |
|------|------|-----------|
| **인증 기준** | 인증 필요/불필요로 그룹핑 | FE 앱이 여러 개일 때, 인증 없는 앱끼리 하나로 |
| **도메인 기준** | 비즈니스 도메인으로 그룹핑 | 도메인이 명확히 구분될 때 |
| **배포 기준** | 배포 주기/대상으로 그룹핑 | 배포 파이프라인 단순화가 목표일 때 |
| **기술 기준** | 같은 스택끼리 그룹핑 | 이종 스택을 기술별로 정리할 때 |

2~3개 안을 제시하고 각각의 트레이드오프를 설명한다. 사용자 피드백을 받아 확정한다.

### 3. 타겟 구조 설계

확정된 서비스 경계에 따라 디렉토리 구조를 설계한다:

```
{project-root}/
├── apps/          (또는 packages/, services/ 등 스택에 맞는 이름)
│   ├── {app-1}/
│   └── {app-2}/
├── libs/          (공유 라이브러리)
│   └── {shared}/
├── infra/         (CI/CD, Docker, IaC)
├── CLAUDE.md
├── {build-config} (package.json, build.gradle 등)
└── _workspace/    (하네스 중간 산출물)
```

구조는 스택에 따라 유연하게 조정한다. monorepo 도구(turborepo, nx, lerna 등) 사용 여부도 제안한다.

### 4. 의존성 통합 계획

- 공유 의존성을 루트 레벨로 호이스팅
- 버전 충돌 해결 방안 (최신 버전 채택, 호환성 확인)
- 불필요 의존성 제거
- devDependency 통합

### 5. 빌드/배포 파이프라인 설계

- 통합 빌드 스크립트 설계
- CI/CD 워크플로우 설계 (기존 파이프라인에서 필요한 것만 합치기)
- Dockerfile 통합 (해당 시)

### 6. CLAUDE.md 초안 작성

`references/claude-md-template.md`를 참조하여 통합 프로젝트의 CLAUDE.md 초안을 작성한다. 기존 repo들의 컨벤션 중 채택/폐기 기준을 사용자와 합의한다.

### 7. 검증 기준 작성

Verifier가 사용할 검증 기준을 `_workspace/02_architect_verification_criteria.md`에 작성한다:
- 각 마이그레이션 Step별 성공 기준
- 빌드 명령어와 기대 결과
- 테스트 명령어와 기대 결과
- 구조 검증 항목 (파일 존재, 디렉토리 구조)

### 8. 사용자 피드백 루프

설계 초안을 사용자에게 제시하고 피드백을 받는다. 피드백에 따라 2~7단계를 반복한다. 합의가 이루어지면 최종 설계서를 `_workspace/02_architect_design.md`에 저장한다.
```

- [ ] **Step 2: claude-md-template.md 작성**

```markdown
# CLAUDE.md 템플릿

통합 프로젝트의 CLAUDE.md에 포함할 섹션 가이드.

## 필수 섹션

### 프로젝트 개요
- 프로젝트 이름과 한 줄 설명
- 통합된 repo 출처 (원본 repo 이름 기록)

### 프로젝트 구조
- 주요 디렉토리와 역할 설명
- 모듈/패키지 간 의존 관계

### 개발 환경
- 필수 런타임 및 버전
- 설치 명령어
- 환경 변수 설정

### 빌드 & 실행
- 빌드 명령어
- 개발 서버 실행
- 프로덕션 빌드

### 테스트
- 테스트 실행 명령어
- 테스트 커버리지 확인

### 코딩 컨벤션
- 채택된 스타일 가이드
- linter/formatter 설정 위치
- 네이밍 규칙 (파일, 변수, 함수, 컴포넌트)
- import 정렬 규칙

### 커밋 컨벤션
- 커밋 메시지 형식
- 브랜치 네이밍

## 선택 섹션

### CI/CD
- 파이프라인 설명
- 배포 프로세스

### 아키텍처 결정 기록
- 주요 설계 결정과 이유
```

- [ ] **Step 3: frontmatter 검증**

Run: `head -4 repo-merger/skills/design-integration/SKILL.md`
Expected: YAML frontmatter에 name, description 필드 존재

- [ ] **Step 4: 커밋**

```bash
git add repo-merger/skills/design-integration/
git commit -m "feat(repo-merger): design-integration 스킬 추가 (CLAUDE.md 템플릿 포함)"
```

---

### Task 8: execute-migration 스킬 생성

**Files:**
- Create: `repo-merger/skills/execute-migration/SKILL.md`

- [ ] **Step 1: SKILL.md 작성**

```markdown
---
name: execute-migration
description: "확정된 통합 설계서에 따라 실제 코드 이동, 파일 재구성, 빌드 설정 통합, 컨벤션 적용을 단계별로 수행하는 스킬. 원본 repo는 읽기 전용으로만 접근하고, 새 프로젝트 디렉토리에 결과물을 생성한다. Migrator 에이전트가 사용한다."
---

# Execute Migration — 마이그레이션 실행

설계서(`_workspace/02_architect_design.md`)의 마이그레이션 계획에 따라 단계별로 코드를 이동하고 재구성한다.

## 핵심 원칙

- **원본 절대 보존**: 원본 repo에 대해 읽기(Read, Glob, Grep)만 수행한다. 쓰기(Write, Edit) 도구는 새 프로젝트 디렉토리에서만 사용한다.
- **단계별 실행**: 설계서의 Step 순서를 따른다. 한 Step이 Verifier 검증을 통과해야 다음 Step으로 넘어간다.
- **작업 로그**: 모든 파일 이동/생성/수정을 `_workspace/03_migrator_log.md`에 기록한다.

## 워크플로우

### Step 1: 디렉토리 구조 생성 + 파일 이동

1. 설계서의 타겟 구조에 따라 새 프로젝트 디렉토리를 생성한다.
2. `_workspace/` 디렉토리를 생성한다.
3. 원본 repo에서 파일을 **복사**하여 타겟 위치에 배치한다.
4. 파일 매핑을 로그에 기록한다:
   ```
   [COPY] {source-repo}/src/utils.ts → {target}/libs/shared/utils.ts
   [COPY] {source-repo}/src/App.tsx → {target}/apps/web/App.tsx
   ```

### Step 2: 의존성 통합

1. 설계서의 의존성 통합 계획을 읽는다.
2. 루트 레벨 의존성 파일을 생성한다 (package.json, build.gradle 등).
3. 공유 의존성을 호이스팅하고, 버전 충돌은 설계서의 결정에 따라 해결한다.
4. 서브 패키지의 의존성 파일을 조정한다 (workspace 참조 등).

### Step 3: 빌드 설정 통합

1. 빌드 스크립트를 설계서에 따라 통합한다.
2. CI/CD 워크플로우를 통합한다.
3. Dockerfile이 있으면 통합한다.
4. 기존 빌드 설정 중 불필요한 것은 포함하지 않는다.

### Step 4: 컨벤션 적용

1. 설계서의 컨벤션 규칙에 따라 linter/formatter 설정을 배치한다.
2. 필요하면 파일/변수 네이밍을 통일한다.
3. import 경로를 새 구조에 맞게 수정한다.
4. 소스 파일 내 상대 경로 참조를 업데이트한다.

### Step 5: CLAUDE.md 배치

1. `_workspace/02_architect_claude_md.md`의 내용을 프로젝트 루트 `CLAUDE.md`로 복사한다.
2. 실제 빌드/테스트 명령어가 동작하는지 기준으로 내용을 미세 조정한다.

## Verifier 피드백 처리

Verifier로부터 FAIL 메시지를 받으면:
1. 실패 항목과 기대값/실제값을 확인한다.
2. 해당 Step의 작업을 수정한다.
3. 수정 내역을 로그에 기록한다:
   ```
   [FIX] Step 2: react 버전 18.2.0 → 18.3.1로 통일 (Verifier 피드백)
   ```
4. Verifier에게 재검증을 요청한다.
```

- [ ] **Step 2: frontmatter 검증**

Run: `head -4 repo-merger/skills/execute-migration/SKILL.md`
Expected: YAML frontmatter에 name, description 필드 존재

- [ ] **Step 3: 커밋**

```bash
git add repo-merger/skills/execute-migration/
git commit -m "feat(repo-merger): execute-migration 스킬 추가"
```

---

### Task 9: verify-integration 스킬 생성

**Files:**
- Create: `repo-merger/skills/verify-integration/SKILL.md`
- Create: `repo-merger/skills/verify-integration/references/verification-checklist.md`

- [ ] **Step 1: SKILL.md 작성**

```markdown
---
name: verify-integration
description: "통합 설계서 기반으로 검증 기준(체크리스트)을 먼저 작성하고, Migrator의 실행 결과를 TDD 스타일로 단계별 검증하는 스킬. 빌드 성공, 테스트 통과, 구조 정합성, 코딩 컨벤션 일관성을 확인하고, 통합 전후 비교 리포트를 생성한다. Verifier 에이전트가 사용한다."
---

# Verify Integration — TDD 스타일 검증

통합 설계서의 검증 기준을 구체적인 체크리스트로 변환하고, 마이그레이션 각 단계를 검증한다.

## 핵심 원칙

- **테스트 먼저**: Migrator가 작업을 시작하기 전에 해당 Step의 검증 항목을 완성한다.
- **FAIL 시 구체적 피드백**: "빌드 실패"가 아니라 "package.json에 react-dom 누락으로 빌드 실패. 기대: react-dom@18.3.1 존재"처럼 구체적으로.
- **자동 vs 수동 구분**: 자동화 가능한 검증(빌드, 테스트, 파일 존재)은 실제 실행으로, 수동 검증(컨벤션 일관성)은 파일 검토로 수행한다.

## 워크플로우

### 1. 검증 기준 수신

`_workspace/02_architect_verification_criteria.md`를 읽고 각 마이그레이션 Step별 성공 기준을 파악한다.

### 2. 체크리스트 작성

`references/verification-checklist.md`를 참조하여 구체적인 검증 체크리스트를 `_workspace/03_verifier_tests.md`에 작성한다. Step별로 구분한다:

```markdown
# 검증 체크리스트

## Step 1: 디렉토리 구조 + 파일 이동
- [ ] 타겟 디렉토리 구조가 설계서와 일치
- [ ] 모든 소스 파일이 올바른 위치에 존재
- [ ] 원본 repo 파일이 수정되지 않음

## Step 2: 의존성 통합
- [ ] 루트 의존성 파일 존재
- [ ] 공유 의존성 호이스팅 완료
- [ ] 버전 충돌 해결됨
- [ ] `{install-command}` 성공

## Step 3: 빌드 설정 통합
- [ ] `{build-command}` 성공
- [ ] CI/CD 설정 파일 존재
- [ ] 빌드 산출물 정상 생성

## Step 4: 컨벤션 적용
- [ ] linter/formatter 설정 존재
- [ ] `{lint-command}` 성공 (에러 0)
- [ ] 네이밍 규칙 일관성

## Step 5: CLAUDE.md
- [ ] CLAUDE.md 존재
- [ ] 빌드/테스트 명령어가 실제 동작
- [ ] 프로젝트 구조 설명이 실제와 일치
```

### 3. 단계별 검증 실행

각 Step 완료 알림을 받으면:
1. 해당 Step의 체크리스트를 순서대로 실행한다.
2. 자동 검증은 Bash 도구로 실행한다.
3. 수동 검증은 Read/Glob/Grep 도구로 파일을 검토한다.
4. 각 항목에 PASS/FAIL을 기록하고, FAIL에는 상세 설명을 추가한다.

### 4. PASS/FAIL 판정

- 모든 항목 PASS → 해당 Step PASS, Migrator에게 다음 Step 진행 요청
- 1개 이상 FAIL → 해당 Step FAIL, Migrator에게 피드백 전달:
  ```
  FAIL: Step 2 의존성 통합
  - [FAIL] 버전 충돌 미해결: lodash 4.17.21 vs 4.17.15 (apps/web/package.json)
  - 기대: 단일 버전으로 통일
  - 실제: 두 버전 혼재
  ```

### 5. 최종 리포트 생성

모든 Step 검증 완료 후 `_workspace/04_verifier_final_report.md`를 생성한다.

통합 전후 비교 데이터를 수집한다:
- 총 파일 수 (원본 repo들 합계 vs 통합 후)
- 의존성 수 (원본 합계 vs 통합 후)
- 중복 의존성 수 (원본 vs 통합 후)
- 빌드 설정 파일 수
- 빌드 성공 여부
- 테스트 통과 여부
```

- [ ] **Step 2: verification-checklist.md 작성**

```markdown
# 검증 체크리스트 참조

Step별 검증 시 사용할 구체적 검증 방법 가이드.

## 자동 검증 방법

### 파일 존재 확인
```bash
# 특정 파일 존재 확인
test -f {path} && echo "PASS" || echo "FAIL: {path} not found"

# 디렉토리 존재 확인
test -d {path} && echo "PASS" || echo "FAIL: {path} not found"
```

### 의존성 설치 확인
```bash
# Node.js
cd {project-root} && npm install 2>&1
# 종료 코드 0 = PASS

# Python
cd {project-root} && pip install -r requirements.txt 2>&1

# JVM
cd {project-root} && ./gradlew dependencies 2>&1
```

### 빌드 확인
```bash
# Node.js
cd {project-root} && npm run build 2>&1

# JVM
cd {project-root} && ./gradlew build 2>&1

# Python
cd {project-root} && python -m py_compile {main-file} 2>&1
```

### 테스트 확인
```bash
# Node.js
cd {project-root} && npm test 2>&1

# JVM
cd {project-root} && ./gradlew test 2>&1

# Python
cd {project-root} && pytest 2>&1
```

### 린트 확인
```bash
# ESLint
cd {project-root} && npx eslint . 2>&1

# Prettier
cd {project-root} && npx prettier --check . 2>&1
```

## 수동 검증 방법

### 원본 보존 확인
원본 repo 디렉토리에서 git status를 실행한다:
```bash
cd {original-repo} && git status
```
"nothing to commit, working tree clean"이 출력되면 PASS.

### 컨벤션 일관성 확인
소스 파일을 샘플링하여 네이밍 규칙, import 스타일, 포맷팅이 설계서의 컨벤션 규칙과 일치하는지 확인한다.

### CLAUDE.md 정합성 확인
CLAUDE.md에 기술된 빌드/테스트 명령어를 실제로 실행하여 동작하는지 확인한다.

## 통합 전후 비교 데이터 수집

```bash
# 파일 수
find {original-repos} -type f -not -path '*/.git/*' -not -path '*/node_modules/*' | wc -l
find {target} -type f -not -path '*/.git/*' -not -path '*/node_modules/*' -not -path '*/_workspace/*' | wc -l

# 의존성 수 (Node.js 예시)
cat {repo}/package.json | python3 -c "import sys,json; d=json.load(sys.stdin); print(len(d.get('dependencies',{}))+len(d.get('devDependencies',{})))"
```
```

- [ ] **Step 3: frontmatter 검증**

Run: `head -4 repo-merger/skills/verify-integration/SKILL.md`
Expected: YAML frontmatter에 name, description 필드 존재

- [ ] **Step 4: 커밋**

```bash
git add repo-merger/skills/verify-integration/
git commit -m "feat(repo-merger): verify-integration 스킬 추가 (체크리스트 참조 포함)"
```

---

### Task 10: repo-merger 오케스트레이터 스킬 생성

**Files:**
- Create: `repo-merger/skills/repo-merger/SKILL.md`

- [ ] **Step 1: SKILL.md 작성**

```markdown
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
```

- [ ] **Step 2: frontmatter 검증**

Run: `head -4 repo-merger/skills/repo-merger/SKILL.md`
Expected: YAML frontmatter에 name, description 필드 존재

- [ ] **Step 3: 커밋**

```bash
git add repo-merger/skills/repo-merger/
git commit -m "feat(repo-merger): repo-merger 오케스트레이터 스킬 추가"
```

---

### Task 11: 커맨드 엔트리포인트 생성

**Files:**
- Create: `repo-merger/commands/merge-repos.md`

- [ ] **Step 1: merge-repos.md 작성**

```markdown
---
name: merge-repos
description: "여러 brownfield git repository를 하나의 통합 프로젝트로 합칩니다. 사용법: /merge-repos [repo 경로 또는 디렉토리] [통합 목적]"
---

repo-merger 스킬을 읽고 실행하라.

사용자 입력: $ARGUMENTS
```

- [ ] **Step 2: 커밋**

```bash
git add repo-merger/commands/
git commit -m "feat(repo-merger): merge-repos 커맨드 엔트리포인트 추가"
```

---

### Task 12: README.md 작성

**Files:**
- Create: `repo-merger/README.md`

- [ ] **Step 1: README.md 작성**

```markdown
# repo-merger

brownfield git repository 여러 개를 하나의 통합 프로젝트로 논리적으로 재구성하는 하네스.

## 특징

- **범용**: 스택에 무관하게 동작 (Node.js, JVM, Python, Go 등)
- **서비스 경계 재설계**: 단순 파일 병합이 아닌 전략적 통합 (인증 기준 그룹핑, 마이크로서비스 통합 등)
- **TDD 스타일 검증**: 검증 기준을 먼저 정의하고 실행 후 검증
- **원본 보존**: 기존 repo를 건드리지 않고 새 디렉토리에 결과물 생성
- **CLAUDE.md 자동 생성**: 통합 프로젝트의 코딩 컨벤션, 빌드 명령어 등을 포함

## 사용법

```
/merge-repos ~/projects/repos "이 3개 repo를 하나의 monorepo로 합치고 싶어"
```

또는 대화로:

```
이 디렉토리 안의 repo들을 하나의 프로젝트로 통합해줘
```

## 워크플로우

### Phase 1: 분석-설계
1. **Analyzer**가 각 repo를 스캔하여 기술 스택, 의존성, 서비스 경계를 분석
2. **Architect**가 분석 결과를 바탕으로 통합 구조를 설계
3. 사용자와 반복 피드백 루프로 설계 확정

### Phase 2: 실행-검증 (TDD 루프)
4. **Verifier**가 검증 기준을 먼저 작성
5. **Migrator**가 단계별로 코드 이동, 의존성/빌드 통합, 컨벤션 적용
6. 각 단계마다 Verifier가 검증 → 실패 시 수정 루프
7. 최종 통합 전후 비교 리포트 생성

## 에이전트

| 에이전트 | 역할 |
|---------|------|
| Analyzer | repo 스캔 및 분석 리포트 생성 |
| Architect | 통합 구조 설계 + 사용자 피드백 루프 |
| Migrator | 설계에 따른 코드 이동/재구성 실행 |
| Verifier | TDD 스타일 검증 + 리포트 생성 |

## 산출물

- 새 프로젝트 디렉토리 (통합된 코드베이스)
- CLAUDE.md (통합 프로젝트 지침)
- `_workspace/` (분석 리포트, 설계서, 검증 결과, 최종 리포트)
```

- [ ] **Step 2: 커밋**

```bash
git add repo-merger/README.md
git commit -m "feat(repo-merger): README 추가"
```

---

### Task 13: 루트 설정 동기화

**Files:**
- Modify: `.claude-plugin/plugin.json`
- Modify: `.claude-plugin/marketplace.json`
- Modify: `README.md`

- [ ] **Step 1: 루트 plugin.json 업데이트**

version을 minor 올리고 (1.2.0 → 1.3.0), description과 keywords에 repo-merger 추가.

Run: `cat .claude-plugin/plugin.json`으로 현재 내용 확인 후 수정.

keywords 배열에 `"repo-merger"`, `"monorepo"`, `"migration"` 추가.
description에 repo-merger 관련 내용 추가.

- [ ] **Step 2: 루트 marketplace.json 업데이트**

plugins 배열에 repo-merger 항목 추가:
```json
{
  "name": "repo-merger",
  "source": "./repo-merger",
  "description": "brownfield git repository 여러 개를 하나의 통합 프로젝트로 논리적으로 재구성하는 하네스",
  "version": "1.0.0"
}
```

- [ ] **Step 3: 루트 README.md 업데이트**

하네스 목록에 repo-merger 추가.

- [ ] **Step 4: 검증**

Run: `cat .claude-plugin/plugin.json | python3 -m json.tool && cat .claude-plugin/marketplace.json | python3 -m json.tool`
Expected: 두 파일 모두 유효한 JSON

- [ ] **Step 5: 커밋**

```bash
git add .claude-plugin/ README.md
git commit -m "chore: 루트 marketplace에 repo-merger 등록 (v1.3.0)"
```

---

### Task 14: 전체 구조 검증

- [ ] **Step 1: 파일 구조 확인**

Run: `find repo-merger -type f | sort`
Expected:
```
repo-merger/.claude-plugin/marketplace.json
repo-merger/.claude-plugin/plugin.json
repo-merger/README.md
repo-merger/agents/analyzer.md
repo-merger/agents/architect.md
repo-merger/agents/migrator.md
repo-merger/agents/verifier.md
repo-merger/commands/merge-repos.md
repo-merger/skills/analyze-repos/SKILL.md
repo-merger/skills/design-integration/SKILL.md
repo-merger/skills/design-integration/references/claude-md-template.md
repo-merger/skills/execute-migration/SKILL.md
repo-merger/skills/repo-merger/SKILL.md
repo-merger/skills/verify-integration/SKILL.md
repo-merger/skills/verify-integration/references/verification-checklist.md
```

- [ ] **Step 2: 모든 에이전트 frontmatter 검증**

Run: `for f in repo-merger/agents/*.md; do echo "=== $f ==="; head -4 "$f"; done`
Expected: 4개 파일 모두 name, description 필드가 있는 YAML frontmatter

- [ ] **Step 3: 모든 스킬 frontmatter 검증**

Run: `for f in repo-merger/skills/*/SKILL.md; do echo "=== $f ==="; head -4 "$f"; done`
Expected: 5개 파일 모두 name, description 필드가 있는 YAML frontmatter

- [ ] **Step 4: JSON 유효성 검증**

Run: `python3 -m json.tool repo-merger/.claude-plugin/plugin.json > /dev/null && python3 -m json.tool repo-merger/.claude-plugin/marketplace.json > /dev/null && echo "PASS"`
Expected: PASS
