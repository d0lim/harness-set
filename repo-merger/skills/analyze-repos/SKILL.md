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
