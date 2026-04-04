---
name: verify-output
description: "idea-forge 최종 산출물 검증 스킬. 최종 보고서의 완성도, _workspace/ 정리 상태, 세션 메모리 저장 여부, git 커밋 이력을 검증한다. idea-forge 오케스트레이터가 마지막 Phase에서 호출하는 내부 스킬이므로 단독 트리거하지 않는다."
---

# Verify Output — 최종 산출물 검증

idea-forge의 모든 Phase가 완료된 후, 최종 상태가 정상인지 검증하는 스킬.

## 검증 항목

### 1. 최종 보고서 검증

`_workspace/final-report.md`가 존재하는지 확인하고, 다음 필수 섹션이 포함되어 있는지 검증한다:

| 필수 섹션 | 검증 기준 |
|----------|----------|
| 요청 요약 | 모드, 카테고리, 타겟이 기술되어 있다 |
| 트렌드 요약 | 최소 5개 트렌드가 표로 정리되어 있다 |
| 시장 기회 요약 | 최소 3개 기회가 매력도 등급과 함께 정리되어 있다 |
| 아이디어 스코어보드 | 스코어링 점수가 포함된 순위표가 있다 |
| Top 아이디어 상세 | Top 3 각각에 원라이너, 킬러 유즈케이스, 수익 모델, MVP, 리스크가 있다 |
| 발산적 재검토 | 블라인드스팟, 역발상 아이디어, 와일드카드가 있다 |
| 다음 세션 제안 | 최소 3개 후속 탐색 방향이 있다 |

### 2. _workspace/ 정리 상태 검증

cleanup 후 `_workspace/`에 남아야 하는 것:

| 경로 | 상태 |
|------|------|
| `_workspace/final-report.md` | 존재해야 함 |
| `_workspace/_memory/session-log.md` | 존재해야 함 |
| `_workspace/00_input/` | 삭제되어야 함 |
| `_workspace/01_*.md` | 삭제되어야 함 |
| `_workspace/02_*.md` | 삭제되어야 함 |
| `_workspace/03_*.md` | 삭제되어야 함 |

### 3. 세션 메모리 검증

`_workspace/_memory/session-log.md`가 다음을 포함하는지 확인한다:

- `## 누적 패턴` 섹션이 존재한다
- 최신 세션 로그가 오늘 날짜로 기록되어 있다
- 트렌드 패턴, 스코어링 인사이트, 발산적 발견, 다음 세션 제안이 모두 있다

### 4. git 커밋 이력 검증

`git log --oneline`에서 이번 세션의 커밋이 올바른 순서로 존재하는지 확인한다:

- `idea-forge: research complete` (Phase 3 후)
- `idea-forge: ideation complete` (Phase 4 후)
- `idea-forge: divergent review complete` (Phase 5 후)
- `idea-forge: final report + cleanup` (Phase 7 후)

## 검증 워크플로우

1. 위 4개 항목을 순차 검증한다.
2. 각 항목에 대해 PASS / FAIL 판정한다.
3. FAIL 항목이 있으면 구체적 원인과 수정 방법을 제시한다.
4. 전체 결과를 사용자에게 보고한다.

## 검증 보고서 형식

```markdown
## Idea Forge 검증 결과

| # | 항목 | 결과 | 비고 |
|---|------|------|------|
| 1 | 최종 보고서 완성도 | PASS/FAIL | {누락 섹션 또는 OK} |
| 2 | _workspace/ 정리 | PASS/FAIL | {잔여 파일 또는 OK} |
| 3 | 세션 메모리 | PASS/FAIL | {누락 항목 또는 OK} |
| 4 | git 커밋 이력 | PASS/FAIL | {누락 커밋 또는 OK} |

**종합**: {N}/4 PASS
```

FAIL이 있으면 "수정 후 재검증하시겠습니까?"를 제안한다.
