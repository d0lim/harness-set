# idea-forge

트렌드 리서치 기반 프로젝트 아이디어 생성/평가/고도화 하네스.

실시간 트렌드(식품, 유튜브, SNS, 문화 현상)와 시장 분석을 통해 대중의 갈망을 파악하고, 수익성 있는 프로젝트 아이디어를 생성하거나 사용자 아이디어를 평가·개선한다. 아이디어를 승격하여 독립 관리하고, 심화·검증·피벗으로 성장시킨다.

## 에이전트

| 에이전트 | 역할 |
|---------|------|
| `trend-scout` | 실시간 트렌드 리서치 (식품, 유튜브, SNS, 문화, 라이프스타일) |
| `market-analyst` | 시장 분석 (타겟, 시장 규모, 경쟁, 수익 모델) |
| `idea-architect` | 아이디어 생성/평가/고도화 (스코어링 루브릭 기반) |

## 스킬

| 스킬 | 용도 |
|------|------|
| `trend-research` | 카테고리별 트렌드 조사 워크플로우 |
| `market-analysis` | 시장 분석 프레임워크 (TAM/SAM/SOM, 경쟁, 수익 모델) |
| `idea-generation` | 아이디어 생성/고도화 + 스코어링 루브릭 |
| `idea-forge` | 오케스트레이터 — 에이전트 팀 조율 |
| `verify-output` | 최종 산출물 검증 |
| `promote-idea` | 아이디어 승격 + 성장 관리 |

## 사용법

두 가지 모드를 지원한다:

- **생성 모드**: "사이드 프로젝트 아이디어 좀 만들어줘"
- **고도화 모드**: "이 아이디어 어때? [아이디어 설명]"

아이디어 승격: "1번 아이디어 승격해줘" 또는 `/promote-idea`

## 디렉토리 구조

```
_workspace/              ← 임시 작업 (Phase 7에서 삭제)
├── 00_input/
├── 01_*.md, 02_*.md, 03_*.md

idea-forge/              ← 영구 산출물
├── sessions/
│   ├── 2026-04-05-pet-food/
│   │   └── final-report.md
│   └── 2026-04-08-ai-saas/
│       └── final-report.md
├── ideas/
│   ├── vert-agent/
│   │   ├── card.md          ← 승격 시 생성
│   │   ├── deep-dive.md     ← "심화해줘"
│   │   ├── validation.md    ← "검증 계획"
│   │   └── spec.md          ← "구체화해줘"
│   └── _index.md
└── _memory/
    └── session-log.md
```

## 아키텍처

```
실행 모드: 에이전트 팀 (Fan-out/Fan-in)

[리더/오케스트레이터]
    ├── Phase 1: 메모리 로드 (idea-forge/_memory/)
    ├── Phase 2: TeamCreate
    ├── Phase 3~5: _workspace/에서 작업 + Phase별 commit
    ├── Phase 6: idea-forge/sessions/ + _memory/ 저장
    ├── Phase 7: rm -rf _workspace/ + commit
    └── Phase 8: 검증 + 승격 제안 (/promote-idea)
```

## 세션 메모리

`idea-forge/_memory/session-log.md`에 세션별 학습을 누적 저장한다. 다음 실행 시 자동으로 로드하여 이전 트렌드 패턴, 스코어링 인사이트, 발산적 발견, 사용자 피드백을 반영한다. 최근 5개 세션을 유지하고, 오래된 세션은 누적 패턴으로 압축한다.
