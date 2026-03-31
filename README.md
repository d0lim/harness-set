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
| *(아직 하네스가 없습니다)* | |

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
