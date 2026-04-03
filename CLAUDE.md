# harness-set

Claude Code 하네스 플러그인 marketplace 레포. 각 하네스는 독립 플러그인으로 개별 설치 가능.

## 구조

- 루트에 각 하네스를 디렉토리로 직접 배치 (플랫 구조)
- 네이밍: kebab-case
- 하네스 디렉토리명 = 플러그인 설치명

```
<harness-name>/
├── .claude-plugin/
│   └── plugin.json          # 메타데이터 + marketplace 정보 (category, tags, language 포함)
├── agents/
│   └── <agent>.md
├── skills/
│   └── <skill>/
│       ├── SKILL.md
│       └── references/
└── README.md
```

> **Note:** commands/ 디렉토리는 사용하지 않는다. 스킬 description의 자동 트리거로 충분하다.

### JSON 파일 역할

| 파일 | 위치 | 역할 |
|------|------|------|
| `plugin.json` | 각 하네스 `.claude-plugin/` | 하네스 메타데이터 (name, version, description, author, category, tags, language) |
| `plugin.json` | 루트 `.claude-plugin/` | 전체 marketplace 메타데이터 (name, version, author, keywords) |
| `marketplace.json` | 루트 `.claude-plugin/` | 하네스 레지스트리 (name, source만 기록 — 상세 정보는 각 하네스 plugin.json 참조) |

## 버전 관리

semver 사용:
- patch: 버그 수정
- minor: 기능 추가
- major: 호환성 깨지는 변경

## git push 전 체크리스트

push 전에 반드시 아래 항목을 확인하고, 사용자에게 질문할 것:

1. **하네스 버전 확인**: 변경된 하네스의 `plugin.json` 버전이 업데이트되었는지 확인. 안 되었으면 사용자에게 버전 업데이트 여부와 변경 수준(patch/minor/major)을 질문.
2. **루트 marketplace 동기화**: 하네스가 추가/삭제된 경우, 루트 `marketplace.json`의 `plugins` 배열에 항목을 추가/삭제.
3. **루트 plugin.json 버전**: 하네스 추가/삭제 시 루트 `plugin.json`의 version을 minor 이상 올리고, description과 keywords도 업데이트.
4. **배포 확인**: 이 push가 marketplace에 배포되는 것임을 사용자에게 알리고, 의도한 배포인지 확인.
5. **README 동기화**: 하네스 추가/삭제 시 루트 README.md, 구조 변경 시 해당 하네스 README.md가 최신 상태인지 확인.
6. **CLAUDE.md 동기화**: 구조 변경(디렉토리 컨벤션, 파일 패턴 등)이 있으면 이 파일의 구조 섹션이 최신 상태인지 확인.
