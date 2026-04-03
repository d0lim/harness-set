# harness-set

Claude Code 하네스 플러그인 marketplace 레포. 각 하네스는 독립 플러그인으로 개별 설치 가능.

## 구조

- 루트에 각 하네스를 디렉토리로 직접 배치 (플랫 구조)
- 네이밍: kebab-case
- 하네스 디렉토리명 = 플러그인 설치명

```
harness-set/
├── .claude-plugin/
│   └── marketplace.json          # 전체 하네스 목록
├── CLAUDE.md
├── README.md
│
└── <harness-name>/               # 각 하네스 = 독립 플러그인
    ├── .claude-plugin/
    │   └── plugin.json
    ├── agents/
    │   └── <agent>.md
    ├── skills/
    │   └── <skill>/
    │       ├── SKILL.md
    │       └── references/
    ├── hooks/
    │   ├── hooks.json
    │   └── <hook-script>
    ├── bin/
    │   └── <script>.sh
    └── README.md
```

> **Note:** commands/ 디렉토리는 사용하지 않는다. 스킬 description의 자동 트리거로 충분하다.

## 버전 관리

semver 사용:
- patch: 버그 수정
- minor: 기능 추가
- major: 호환성 깨지는 변경

## git push 규칙

**push = 배포다.** 이 repo에 push하면 marketplace를 통해 팀원에게 즉시 배포된다.
따라서 push 전에 반드시 아래를 수행한다. 하나라도 빠지면 push하지 않는다.

### 필수: 버전 업데이트

하네스 파일이 하나라도 변경되었으면 **반드시** 버전을 올린다. 버전을 올리지 않으면 기존 사용자에게 변경이 전달되지 않는다.

1. 변경된 하네스의 `plugin.json` 버전을 semver에 따라 올린다.
   - 사용자에게 변경 수준(patch/minor/major)을 질문한다.
2. 루트 `.claude-plugin/marketplace.json`의 해당 하네스 version을 동일하게 올린다.

### 체크리스트

push 전에 반드시 아래 항목을 확인하고, 사용자에게 질문할 것:

1. **버전 업데이트 완료**: 위 "필수: 버전 업데이트" 항목이 수행되었는지 확인. 안 되었으면 push를 진행하지 않고 먼저 버전을 올린다.
2. **루트 marketplace 동기화**: 루트 `.claude-plugin/marketplace.json`의 `plugins` 배열이 최신 상태인지 확인. 각 하네스의 name, source, description, version이 해당 하네스의 `plugin.json`과 일치해야 한다.
3. **배포 확인**: 이 push가 marketplace에 배포되는 것임을 사용자에게 알리고, 의도한 배포인지 확인.
4. **README 동기화**: 새 하네스가 추가되었거나 삭제된 경우, 루트 README.md의 하네스 목록이 최신 상태인지 확인.
5. **개별 하네스 README 동기화**: 하네스의 에이전트, 스킬, 워크플로우가 변경된 경우, 해당 하네스의 README.md가 실제 구조와 일치하는지 확인.
