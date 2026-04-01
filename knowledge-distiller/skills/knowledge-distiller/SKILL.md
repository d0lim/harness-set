---
name: knowledge-distiller
description: "Zettelkasten 방식의 지식 증류 오케스트레이터. 주제 리서치 → 참조 노트 → 핵심 통찰 추출 → 논리 흐름 합성 → 저작물 통합을 자동화한다. 여러 라운드로 지식을 점진적으로 축적하며, 발표 대본/논문/블로그/보고서 등 모든 저작물의 내용을 체계적으로 보강한다. 리서치해줘, 조사해서 정리해줘, 내용 보강해줘, 자료 조사, 팩트 체크, 통찰 뽑아줘, 인사이트 정리, 근거 보강, 지식 정리, 노트 정리, zettelkasten, knowledge distill, research and synthesize 등 주제를 조사하고 통찰을 추출하여 저작물에 통합하는 작업에 사용. 단순 웹 검색이나 1회성 질문 답변, 코드 작성, 번역, 저작물을 처음부터 새로 작성하는 작업은 이 스킬의 범위가 아님."
---

# Knowledge Distiller — Zettelkasten 지식 증류 오케스트레이터

주제 리서치 → 참조 노트 → 핵심 통찰 → 논리 흐름 합성 → 저작물 통합을 자동화하는 파이프라인.
여러 라운드를 반복하여 지식을 점진적으로 축적한다.

## 실행 모드: 서브 에이전트

리서치(fan-out) → 증류(fan-in) → 합성(순차)의 파이프라인 구조.
각 Phase가 이전 Phase의 산출물에 의존하므로 순차 실행하되, Phase 2의 리서치는 주제별로 병렬 실행한다.

## 워크플로우

### Phase 1: 준비

1. **사용자 입력 분석:**

   | 입력 | 필수 여부 | 기본값 |
   |------|----------|--------|
   | 리서치 주제 (1개 이상) | 필수 | — |
   | 대상 저작물 경로 | 선택 | 없음 (통찰 추출까지만 수행) |
   | 논리 흐름 방향 | 선택 | 저작물 구조에서 추론 |
   | 조사 깊이 | 선택 | 보통 (주제당 5~8개 소스) |

2. **라운드 감지:**
   - `_workspace/` 존재 여부로 첫 라운드인지 판별
   - 기존 라운드가 있으면 `_workspace/permanents/`의 마지막 insight ID를 확인하여 이어서 번호 매긴다

3. **미반영 발견사항(pending) 확인:**
   - `_workspace/pending/` 디렉토리가 존재하고 `.md` 파일이 있으면 사용자에게 알린다:
     ```
     ⚠️ 이전 라운드에서 미반영된 발견사항이 {N}건 있습니다:
     - {파일명}: {제목 요약}
     이번 라운드에서 이 발견사항을 참조 노트로 포함하여 증류합니다.
     ```
   - pending 파일들을 이번 라운드의 `_workspace/references/`로 복사한다 (원본 파일명 유지).
   - 복사 완료 후 pending 파일들을 삭제한다 (`_workspace/pending/` 디렉토리는 유지).

4. **작업 디렉토리 준비:**
   ```
   _workspace/
   ├── 00_input/
   │   ├── topics.md          ← 이번 라운드 리서치 주제
   │   └── target-snapshot.md ← 대상 저작물 스냅샷 (있으면)
   ├── references/            ← 라운드별 참조 노트 누적
   ├── permanents/            ← 영구 노트 누적
   ├── synthesis/             ← 논리 흐름 매핑
   ├── pending/               ← 미반영 발견사항 (다음 라운드에서 자동 반영)
   └── output/                ← 보강된 저작물
   ```

5. **주제 목록을 `_workspace/00_input/topics.md`에 저장:**
   ```markdown
   # 라운드 {N} 리서치 주제

   ## 주제 1: {주제명}
   - 조사 방향: {힌트}

   ## 주제 2: {주제명}
   - 조사 방향: {힌트}
   ```

6. 대상 저작물이 있으면 내용을 읽어 `_workspace/00_input/target-snapshot.md`에 저장한다.

### Phase 2: 리서치 (Fan-out)

각 주제에 대해 researcher 서브 에이전트를 병렬로 스폰한다.

```
Agent(
  name: "researcher-{topic-slug}",
  agent_type: "researcher",
  model: "opus",
  run_in_background: true,
  prompt: "
    당신은 researcher 에이전트다. agents/researcher.md의 원칙을 따른다.
    research-sources 스킬(Skill 도구로 /research-sources 호출)을 사용하여 다음 주제를 조사하라:

    주제: {주제명}
    조사 방향: {힌트}
    대상 저작물 맥락: {target-snapshot 요약}
    라운드: {N}

    결과를 {절대경로}/_workspace/references/R{N}-{topic-slug}.md에 저장하라.
  "
)
```

**모든 주제를 하나의 메시지에서 병렬 스폰한다.** 완료 알림을 대기한다.

**부분 완료 진행 규칙:** 일부 researcher가 완료되지 않아도, 완료된 결과만으로 Phase 3에 진입할 수 있다. 판단 기준:
- 전체 주제의 절반 이상이 완료되었고, 미완료 에이전트에 추가 알림이 없으면 완료된 결과로 진행한다.
- 미완료 주제는 다음 라운드 리서치 후보로 자동 등록한다.
- **뒤늦게 완료된 researcher 결과 처리:** Phase 3 이후에 미완료 researcher가 완료되면, 해당 참조 노트를 `_workspace/pending/`에 저장한다 (Phase 5의 pending 형식 대신 원본 참조 노트 형식 그대로 `P{라운드}-{topic-slug}.md`로 저장). 이미 Phase 5 보고가 완료된 이후라도, 세션이 종료되기 전에 완료 알림이 오면 pending으로 저장한다.

### Phase 3: 증류 (Fan-in)

모든 researcher가 완료되면, distiller 서브 에이전트를 스폰한다.

```
Agent(
  name: "distiller",
  agent_type: "distiller",
  model: "opus",
  prompt: "
    당신은 distiller 에이전트다. agents/distiller.md의 원칙을 따른다.
    distill-insights 스킬(Skill 도구로 /distill-insights 호출)을 사용하여 통찰을 추출하라.

    라운드: {N}
    참조 노트 경로: {절대경로}/_workspace/references/
    기존 영구 노트 경로: {절대경로}/_workspace/permanents/

    새 영구 노트를 {절대경로}/_workspace/permanents/에 저장하라.
    관계 맵을 {절대경로}/_workspace/permanents/_relation-map.md에 저장하라.
  "
)
```

### Phase 4: 합성 및 작곡

대상 저작물이 있는 경우에만 실행한다. 없으면 Phase 3까지의 결과를 사용자에게 보고하고 종료.

```
Agent(
  name: "composer",
  agent_type: "composer",
  model: "opus",
  prompt: "
    당신은 composer 에이전트다. agents/composer.md의 원칙을 따른다.
    compose-document 스킬(Skill 도구로 /compose-document 호출)을 사용하여 저작물을 보강하라.

    라운드: {N}
    영구 노트 경로: {절대경로}/_workspace/permanents/
    대상 저작물: {절대경로}/_workspace/00_input/target-snapshot.md
    사용자 지정 논리 흐름: {있으면 기술}

    Flow map을 {절대경로}/_workspace/synthesis/flow-map-R{N}.md에 저장하라.
    보강된 초안을 {절대경로}/_workspace/output/enriched-draft-R{N}.md에 저장하라.
    변경 요약을 {절대경로}/_workspace/output/changelog-R{N}.md에 저장하라.
  "
)
```

### Phase 5: 결과 보고 및 다음 라운드 준비

1. **결과 요약을 사용자에게 보고:**
   - 이번 라운드에서 조사한 주제 수
   - 생성된 참조 노트 수
   - 추출된 새 통찰 수 (총 누적 통찰 수)
   - 보강된 저작물 경로 (있으면)
   - 발견된 공백/추가 리서치 후보

2. **미반영 발견사항을 pending에 저장:**
   다음 중 하나라도 해당하면 `_workspace/pending/`에 마크다운 파일로 저장한다:
   - **뒤늦게 완료된 리서치**: Phase 2에서 부분 완료로 진행했을 때, 이후 완료된 researcher 결과 중 아직 증류에 반영되지 않은 것
   - **증류 시점 이후 발견된 중요 사실**: Phase 3~5 진행 중 발견했으나 이번 라운드 영구 노트에 포함되지 않은 것
   - **사용자가 세션 중 언급한 추가 정보**: 대화 중 사용자가 제공했으나 리서치/증류 파이프라인에 포함되지 않은 데이터

   **pending 파일 형식:** `_workspace/pending/P{라운드}-{slug}.md`
   ```markdown
   # {발견사항 제목}

   - **출처**: {어디서 발견했는지 — researcher 에이전트명, 사용자 발언, 외부 소스 등}
   - **라운드**: R{N} (Phase {X} 이후 발생)
   - **발견 시점**: {Phase 완료 후 / 세션 종료 시점}
   - **영구 노트 미반영 사유**: {시간 부족 / 부분 완료 진행 / 증류 이후 발견 등}

   ## 핵심 내용

   {발견사항의 구체적 내용. 가능한 한 출처와 데이터를 포함.}

   ## 영향 범위

   - **관련 영구 노트**: {기존 insight-XXX 중 관련된 것이 있으면 나열}
   - **저작물 영향**: {대상 저작물의 어느 부분에 영향을 줄 수 있는지}
   - **다음 라운드 권장 조치**: {재증류 / 추가 리서치 / 직접 반영 등}
   ```

   pending 파일이 생성되면 결과 보고에 포함한다:
   ```
   📋 미반영 발견사항 {N}건이 _workspace/pending/에 저장되었습니다.
   다음 라운드(/distill-next) 시작 시 자동으로 참조 노트에 포함됩니다.
   ```

3. **다음 라운드 제안:**
   - distiller의 관계 맵에서 고립된 통찰
   - composer의 flow-map에서 발견된 공백
   - pending 파일의 내용 (미반영 발견사항이 있으면 다음 라운드 주제에 반영)
   - 이들을 기반으로 다음 라운드 리서치 주제를 제안한다

4. **사용자 선택지 제시:**
   - "다음 라운드를 진행하시겠습니까? 제안된 주제로 진행 / 주제를 수정 / 여기서 종료"
   - 사용자가 다음 라운드를 선택하면 Phase 1로 돌아간다

## 데이터 흐름

```
[사용자]
   │
   ├── 주제 + 대상 저작물
   ↓
[Phase 1: 준비]
   │
   ├── topics.md + target-snapshot.md
   ↓
[Phase 2: 리서치 팬아웃]
   │
   ├── Agent(researcher-1, background) ──→ references/R1-topic-1.md
   ├── Agent(researcher-2, background) ──→ references/R1-topic-2.md
   └── Agent(researcher-N, background) ──→ references/R1-topic-N.md
   │
   ↓ (모든 researcher 완료 대기)
   │
[Phase 3: 증류 팬인]
   │
   ├── Agent(distiller) ──→ permanents/insight-001-*.md
   │                    ──→ permanents/insight-002-*.md
   │                    ──→ permanents/_relation-map.md
   ↓
[Phase 4: 합성] (대상 저작물 있을 때만)
   │
   ├── Agent(composer) ──→ synthesis/flow-map-R1.md
   │                   ──→ output/enriched-draft-R1.md
   │                   ──→ output/changelog-R1.md
   ↓
[Phase 5: 보고 + 다음 라운드?]
   │
   ├── 사용자에게 결과 요약
   ├── 미반영 발견사항 ──→ pending/P{N}-*.md (다음 라운드 자동 반영)
   └── 다음 라운드 주제 제안 ──→ (Phase 1로 반복)
```

## 에러 핸들링

| 상황 | 전략 |
|------|------|
| researcher 1개 실패 | 해당 주제 제외하고 나머지 결과로 진행. 보고서에 누락 명시. |
| 모든 researcher 실패 | 오케스트레이터가 직접 WebSearch 3회로 최소 참조 노트 1개 작성 후 진행 |
| distiller 실패 | 오케스트레이터가 참조 노트를 읽고 주요 발견 3개를 간략한 영구 노트로 작성 |
| composer 실패 | 영구 노트 목록과 관계 맵을 사용자에게 제시하고 수동 통합 안내 |
| 대상 저작물 경로 무효 | 사용자에게 경로 재확인 요청. 확인 불가 시 Phase 3까지만 진행 |
| 라운드 간 영구 노트 ID 충돌 | 기존 최대 ID + 1부터 시작하여 충돌 방지 |
| pending 파일이 참조 노트 형식과 다름 | pending 파일을 참조 노트로 복사 시 형식을 검증. 형식이 다르면 내용을 참조 노트 형식으로 변환하여 복사 |
| 다음 라운드 없이 세션 종료 | pending 파일은 _workspace/에 남아있으므로 언제든 /distill-next로 재개 가능 |

## 테스트 시나리오

### 정상 흐름 — 대상 저작물 있음
1. 사용자: "Context Engineering과 Memory Engineering에 대해 조사해서 이 발표 대본(/path/to/script.md)을 보강해줘"
2. Phase 1: 주제 2개 파악, 대본 스냅샷 저장
3. Phase 2: researcher-context-eng + researcher-memory-eng 병렬 스폰
4. Phase 3: distiller가 두 참조 노트 교차 분석 → 5~8개 영구 노트 생성
5. Phase 4: composer가 대본에 통찰 매핑 → 보강 초안 생성
6. Phase 5: 결과 보고 + 다음 라운드 주제 제안

### 정상 흐름 — 대상 저작물 없음
1. 사용자: "Harness Engineering 최신 동향을 조사하고 핵심 통찰을 정리해줘"
2. Phase 1: 주제 1개, 대상 저작물 없음
3. Phase 2: researcher-harness-eng 스폰
4. Phase 3: distiller가 참조 노트에서 통찰 추출
5. Phase 4: 건너뜀
6. Phase 5: 영구 노트 목록 + 관계 맵을 사용자에게 제시

### 에러 흐름
1. Phase 2에서 researcher-1이 WebSearch 실패로 중단
2. researcher-2는 정상 완료
3. 오케스트레이터가 researcher-1 실패를 감지, 보고서에 "주제 1 리서치 미완료" 명시
4. distiller는 researcher-2의 참조 노트만으로 통찰 추출 진행
5. 최종 보고에서 주제 1 재조사를 다음 라운드 주제로 제안

### 멀티 라운드 흐름
1. 라운드 1: 주제 A, B 조사 → insight-001~005 생성
2. 라운드 1 보고: 공백 발견 — "주제 C에 대한 데이터 필요"
3. 라운드 2: 주제 C 조사 → references/R2-topic-c.md 추가
4. distiller가 R1 + R2 참조 노트 + 기존 영구 노트 전체를 교차 분석
5. insight-006~008 추가, insight-002의 관계 업데이트
6. composer가 라운드 2 통찰로 저작물 추가 보강

### Pending 흐름 (뒤늦은 리서치 완료)
1. 라운드 1: 주제 A, B, C, D, E 조사 시작
2. A, B, C, D 완료 (80%), E 미완료 → Phase 3 진행
3. Phase 3 증류 완료 후 E가 뒤늦게 완료
4. Phase 5에서 E의 참조 노트를 `_workspace/pending/P1-topic-e.md`로 저장
5. 결과 보고에 "미반영 발견사항 1건" 포함
6. 새 세션에서 `/distill-next` 실행 → pending 자동 감지
7. Phase 1에서 P1-topic-e.md를 references/로 복사 → 이번 라운드 증류에 포함
8. distiller가 기존 영구 노트 + 새 참조 노트(pending 포함) 교차 분석
