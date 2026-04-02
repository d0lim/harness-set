# harness-set

Claude Code agent-harness 플러그인 모음.

## 설치

```bash
# 1. marketplace 등록
/plugin marketplace add d0lim/harness-set

# 2. 개별 하네스 설치
/plugin install <harness-name>@d0lim/harness-set
```

## 하네스 목록

| 이름 | 설명 |
|------|------|
| [presentation](./presentation) | 발표 대본 작성 + reveal.js 슬라이드 제작 + 청중 피드백 |
| [knowledge-distiller](./knowledge-distiller) | Zettelkasten 방식 지식 증류 — 리서치 → 통찰 추출 → 저작물 통합 |
| [repo-merger](./repo-merger) | brownfield repository 통합 — 분석 → 설계 → TDD 마이그레이션 → 검증 리포트 |

## 하네스 구조

각 하네스는 독립 플러그인으로 아래 구조를 따릅니다:

```
<harness-name>/
├── .claude-plugin/
│   ├── plugin.json          # 플러그인 메타데이터
│   └── marketplace.json     # marketplace 배포 정보
├── agents/                  # 에이전트 정의
├── skills/                  # 스킬 정의
└── README.md                # 사용법
```

## 라이선스

Apache License 2.0
