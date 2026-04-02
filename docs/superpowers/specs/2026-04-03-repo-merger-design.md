# repo-merger 하네스 설계서

## 개요

brownfield git repository 여러 개(2~3개)를 하나의 통합 프로젝트로 논리적으로 재구성하는 하네스. git 히스토리 보존 없이 코드베이스 통합에 집중하며, 스택에 무관하게 범용으로 동작한다.

## 핵심 원칙

- **분석 → 제안 → 피드백 루프 → 합의 → 실행**: 사용자와 반복 소통하며 모호성을 해소한 후 실행
- **TDD 스타일**: 검증 기준을 먼저 정의하고, 실행 후 검증하는 루프
- **원본 보존**: 기존 repo를 건드리지 않고, 새 디렉토리에 통합 결과물 생성
- **서비스 경계 재설계**: 단순 파일 병합이 아니라, 마이크로서비스 재그룹핑(예: 인증 불필요 FE 앱 통합) 등 전략적 통합

## 사용자 입력

- repo가 들어있는 디렉토리 경로 또는 repo 목록
- 간단한 통합 목적/방향성 설명 (예: "이 3개 repo를 하나의 Next.js monorepo로 합치고 싶어")
- 상세 사항은 하네스가 질문하며 반복 소통으로 확정

## 아키텍처: 생성-검증 루프 (Approach B)

### 에이전트 구성 (4개)

| 에이전트 | Phase | 역할 |
|---------|-------|------|
| **Analyzer** | 1: 분석-설계 | 각 repo의 기술 스택, 의존성, 빌드 시스템, 디렉토리 구조, 코딩 컨벤션을 스캔하여 분석 리포트 생성. repo 간 중복 모듈, 공유 의존성, 서비스 경계도 식별. |
| **Architect** | 1: 분석-설계 | Analyzer의 리포트 기반으로 통합 구조 설계. 사용자와 반복 대화하며 타겟 구조, 서비스 경계, 의존성 전략, CLAUDE.md 초안 확정. |
| **Migrator** | 2: 실행-검증 | 확정된 설계에 따라 실제 코드 이동, 파일 재구성, 빌드 설정 통합, 컨벤션 적용 수행. |
| **Verifier** | 2: 실행-검증 | 설계 문서 기반으로 먼저 검증 기준(테스트/체크리스트) 작성 후, Migrator 실행 결과 검증. 빌드 성공, 테스트 통과, 구조 정합성, 전후 비교 리포트 생성. |

### Phase 1: 분석-설계

```
사용자 입력 (repo 경로 + 간단한 목적)
    ↓
[Analyzer] repo들 스캔
    ↓ _workspace/01_analyzer_report.md
[Architect] 통합 구조 초안 제안
    ↓
사용자 피드백 → Architect 수정 → 사용자 피드백 → ... (합의까지 반복)
    ↓
확정 산출물:
  - _workspace/02_architect_design.md (통합 설계서)
  - _workspace/02_architect_claude_md.md (CLAUDE.md 초안)
  - _workspace/02_architect_verification_criteria.md (검증 기준)
```

#### Analyzer 분석 항목

- 기술 스택 식별 (언어, 프레임워크, 빌드 도구)
- 의존성 목록 + 버전 충돌 감지
- 디렉토리 구조 매핑
- 코딩 컨벤션 추출 (linter 설정, formatter 설정, 네이밍 패턴)
- 서비스 경계 분석 (엔드포인트, 공유 모듈, 인증 구조)
- 중복 코드/설정 식별

#### Architect 설계 항목

- 타겟 디렉토리 구조
- 서비스 재그룹핑 전략 (예: 인증 불필요 FE 앱 통합)
- 의존성 통합 계획 (버전 통일, 중복 제거)
- 빌드/배포 파이프라인 설계
- 컨벤션 통일 규칙 → CLAUDE.md 반영
- 마이그레이션 단계 분할 (Migrator가 실행할 작업 목록)

#### CLAUDE.md 설계

Architect가 통합 프로젝트의 CLAUDE.md를 설계한다:
- 통합된 코딩 스타일/컨벤션 규칙
- 빌드/테스트 명령어
- 프로젝트 구조 설명
- 기존 repo들의 컨벤션 중 채택/폐기 기준을 사용자와 합의

### Phase 2: 실행-검증 (TDD 루프)

```
[Verifier] 검증 기준으로 테스트/체크리스트 작성
    ↓ _workspace/03_verifier_tests.md
[Migrator] 작업 단계별 실행
    ↓ 새 프로젝트 디렉토리에 결과물 생성
[Verifier] 검증 실행
    ├─ PASS → 다음 작업 단계로
    └─ FAIL → Migrator에게 피드백 → 수정 → 재검증 (반복)

모든 단계 완료 후:
[Verifier] 최종 리포트 생성
  - _workspace/04_verifier_final_report.md
  - 통합 전후 비교 (파일 수, 의존성 수, 빌드 설정 등)
  - PASS/FAIL 항목 요약
```

#### TDD 순서

1. Verifier가 **먼저** 검증 기준 정의 (기대 빌드 성공, 기대 구조, 기대 테스트 통과)
2. Migrator가 작업 수행
3. Verifier가 검증 → 실패 시 Migrator 수정 루프

#### Migrator 작업 단위

설계서의 마이그레이션 계획을 단계별로 나누어 실행:
- Step 1: 디렉토리 구조 생성 + 파일 이동
- Step 2: 의존성 통합
- Step 3: 빌드 설정 통합
- Step 4: 컨벤션 적용
- Step 5: CLAUDE.md 배치

각 Step마다 Verifier 검증 루프가 돌아간다.

## 데이터 전달 프로토콜

### Phase 간 전달: 파일 기반

모든 중간 산출물은 새 프로젝트 디렉토리 내부의 `_workspace/` 디렉토리에 저장 (예: `~/projects/merged-app/_workspace/`). 최종 산출물 확정 후에도 `_workspace/`는 삭제하지 않고 보존한다 (사후 검증·감사 추적용):

| 파일 | 생성자 | 소비자 |
|------|--------|--------|
| `01_analyzer_report.md` | Analyzer | Architect |
| `02_architect_design.md` | Architect | Migrator, Verifier |
| `02_architect_claude_md.md` | Architect | Migrator |
| `02_architect_verification_criteria.md` | Architect | Verifier |
| `03_verifier_tests.md` | Verifier | Migrator (참조), Verifier (실행) |
| `04_verifier_final_report.md` | Verifier | 사용자 |

### Phase 내 통신

- Phase 1: Analyzer → (파일) → Architect → (대화) → 사용자
- Phase 2: Verifier ⇄ Migrator (SendMessage로 실시간 피드백 교환)

## 에러 핸들링

| 에러 유형 | 전략 |
|----------|------|
| Analyzer 스캔 실패 (접근 불가 repo) | 해당 repo 건너뛰고 진행, 리포트에 누락 명시 |
| 의존성 버전 충돌 해결 불가 | Architect가 사용자에게 선택지 제시, 합의 후 진행 |
| 빌드 실패 | Verifier가 에러 로그 분석 → Migrator에게 수정 지시 → 재검증 (최대 3회) |
| 테스트 실패 | 실패 테스트 목록과 원인 분석을 사용자에게 보고, 사용자 판단에 따라 진행/중단 |
| 서비스 경계 재설계 판단 불가 | Architect가 선택지를 제시하고 사용자에게 위임 |

## 스킬 구성

| 스킬 | 사용 에이전트 | 역할 |
|------|-------------|------|
| `repo-merger` | 오케스트레이터 | Phase 1 → Phase 2 전체 워크플로우 조율 |
| `analyze-repos` | Analyzer | repo 스캔 및 분석 리포트 생성 |
| `design-integration` | Architect | 통합 구조 설계 + 사용자 피드백 루프 |
| `execute-migration` | Migrator | 설계에 따른 코드 이동/재구성 실행 |
| `verify-integration` | Verifier | 검증 기준 작성 + 검증 실행 + 리포트 생성 |

## 산출물 요약

### 중간 산출물 (_workspace/)
- 분석 리포트, 설계서, 검증 기준, 테스트 결과, 최종 리포트

### 최종 산출물
- 새 프로젝트 디렉토리 (통합된 코드베이스)
- CLAUDE.md (통합 프로젝트 지침)
- 통합 전후 비교 리포트

## 테스트 시나리오

### 정상 흐름
1. 사용자가 React FE 2개 + Node.js BE 1개 repo 경로 제공
2. Analyzer가 3개 repo 스캔 → 공통 의존성, 인증 구조 분석
3. Architect가 "인증 불필요 FE 2개를 하나로 합치고, BE는 별도 패키지" 제안
4. 사용자 피드백 2회 → 구조 확정
5. Verifier가 검증 기준 작성 (빌드 성공, 라우팅 통합, 의존성 단일화)
6. Migrator가 5단계 실행, 각 단계 Verifier 검증 통과
7. 최종 리포트 생성

### 에러 흐름
1. Step 2 (의존성 통합)에서 React 버전 충돌 발생
2. Verifier FAIL → Migrator에게 피드백
3. Migrator 수정 시도 → 재검증 FAIL
4. 사용자에게 "React 17 vs 18 선택 필요" 보고 → 사용자 결정 → 재실행 → PASS
