---
name: knowledge-distiller
description: "Zettelkasten 방식의 지식 증류 오케스트레이터. 주제 리서치 → 참조 노트 → 핵심 통찰 추출 → 논리 흐름 합성 → 저작물 통합을 자동화한다. 여러 라운드로 지식을 점진적으로 축적하며, 발표 대본/논문/블로그/보고서 등 모든 저작물의 내용을 체계적으로 보강한다. 리서치해줘, 조사해서 정리해줘, 내용 보강해줘, 자료 조사, 팩트 체크, 통찰 뽑아줘, 인사이트 정리, 근거 보강, 지식 정리, 노트 정리, zettelkasten, knowledge distill, research and synthesize 등 주제를 조사하고 통찰을 추출하여 저작물에 통합하는 작업에 사용. 단순 웹 검색이나 1회성 질문 답변, 코드 작성, 번역은 이 스킬의 범위가 아님."
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

3. **작업 디렉토리 준비:**
   ```
   _workspace/
   ├── 00_input/
   │   ├── topics.md          ← 이번 라운드 리서치 주제
   │   └── target-snapshot.md ← 대상 저작물 스냅샷 (있으면)
   ├── references/            ← 라운드별 참조 노트 누적
   ├── permanents/            ← 영구 노트 누적
   ├── synthesis/             ← 논리 흐름 매핑
   └── output/                ← 보강된 저작물
   ```

4. **주제 목록을 `_workspace/00_input/topics.md`에 저장:**
   ```markdown
   # 라운드 {N} 리서치 주제

   ## 주제 1: {주제명}
   - 조사 방향: {힌트}

   ## 주제 2: {주제명}
   - 조사 방향: {힌트}
   ```

5. 대상 저작물이 있으면 내용을 읽어 `_workspace/00_input/target-snapshot.md`에 저장한다.

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

2. **다음 라운드 제안:**
   - distiller의 관계 맵에서 고립된 통찰
   - composer의 flow-map에서 발견된 공백
   - 이들을 기반으로 다음 라운드 리서치 주제를 제안한다

3. **사용자 선택지 제시:**
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
