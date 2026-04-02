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
