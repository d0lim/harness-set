# harness-set

Claude Code 하네스 플러그인 marketplace 레포. 각 하네스는 독립 플러그인으로 개별 설치 가능.

## 구조

- 루트에 각 하네스를 디렉토리로 직접 배치 (플랫 구조)
- 네이밍: kebab-case
- 하네스 디렉토리명 = 플러그인 설치명

```
<harness-name>/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── agents/
│   └── <agent>.md
├── skills/
│   └── <skill>/
│       ├── SKILL.md
│       └── references/
└── README.md
```

## 버전 관리

semver 사용:
- patch: 버그 수정
- minor: 기능 추가
- major: 호환성 깨지는 변경

## git push 전 체크리스트

push 전에 반드시 아래 항목을 확인하고, 사용자에게 질문할 것:

1. **하네스 버전 확인**: 변경된 하네스의 `plugin.json` 버전이 업데이트되었는지 확인. 안 되었으면 사용자에게 버전 업데이트 여부와 변경 수준(patch/minor/major)을 질문.
2. **루트 marketplace 동기화**: 하네스가 추가/삭제/버전 변경된 경우, 루트 `.claude-plugin/marketplace.json`의 `plugins` 배열이 최신 상태인지 확인. 각 하네스의 name, source, description, version이 해당 하네스의 `plugin.json`과 일치해야 한다.
3. **루트 plugin.json 버전**: 하네스 추가/삭제 시 루트 `.claude-plugin/plugin.json`의 version을 minor 이상 올리고, description과 keywords도 업데이트.
4. **배포 확인**: 이 push가 marketplace에 배포되는 것임을 사용자에게 알리고, 의도한 배포인지 확인.
5. **README 동기화**: 새 하네스가 추가되었거나 삭제된 경우, 루트 README.md의 하네스 목록이 최신 상태인지 확인.
