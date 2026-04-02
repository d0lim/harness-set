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
