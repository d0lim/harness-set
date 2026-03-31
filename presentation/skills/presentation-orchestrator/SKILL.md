---
name: presentation-orchestrator
description: "새로운 발표 자료(대본+슬라이드)를 처음부터 생성하는 오케스트레이터. 주제 리서치, 대본 작성, reveal.js 슬라이드 제작, 청중 페르소나 피드백, 반복 개선을 하나의 워크플로우로 조율한다. 발표 준비, 프레젠테이션 만들기, 슬라이드 만들어줘, 발표 자료 만들기, PT 만들기, keynote 준비, pitch deck, 강의 자료 만들기, 데크 만들기, 발표 도와줘 등 새 발표 자료를 생성하는 요청에 사용. 기존 발표 자료의 검색·번역·변환·부분 수정, 보고서·문서 작성, 발표 기술 코칭, UI 슬라이더 컴포넌트 구현은 이 스킬의 범위가 아님."
---

# Presentation Orchestrator

발표 대본 작성과 reveal.js 슬라이드 제작을 총괄하는 오케스트레이터. 리서치 → 생성 → 피드백 → 개선 사이클을 에이전트 팀으로 조율한다.

## 실행 모드: 에이전트 팀

## 에이전트 구성

| 팀원 | 에이전트 타입 | 역할 | 스킬 | 출력 |
|------|-------------|------|------|------|
| researcher | general-purpose | 주제 리서치 | research-topic | `_workspace/01_researcher_findings.md` |
| scriptwriter | scriptwriter (커스텀) | 대본 작성 | write-script | `_workspace/02_scriptwriter_script.md` |
| slide-builder | slide-builder (커스텀) | 슬라이드 제작 | build-slides | `_workspace/03_slide-builder_slides.html` |
| audience-reviewer | audience-reviewer (커스텀) | 청중 피드백 | audience-review | `_workspace/04_audience-reviewer_feedback.md` |

## 워크플로우

### Phase 1: 준비

1. 사용자 입력에서 다음 정보를 파악한다:
   - **발표 주제**: 무엇에 대한 발표인가
   - **발표 시간**: 몇 분짜리 발표인가 (기본값: 15분)
   - **청중**: 누구 앞에서 발표하는가 (기본값: 비즈니스 전문가)
   - **발표 목적**: 정보 전달 / 설득 / 교육 / 영감 (기본값: 정보 전달)
   - **특별 요구사항**: 특정 내용 포함, 톤, 언어 등
   - **슬라이드 테마**: 사용자가 지정한 reveal.js 테마 (기본값: 발표 성격에 맞게 자동 선택)

2. 누락된 정보가 있으면 사용자에게 질문한다. 단, 최소한 주제만 있으면 진행 가능.

3. 작업 디렉토리에 `_workspace/` 생성. 입력 정보를 `_workspace/00_input.md`에 저장.

### Phase 2: 리서치 + 초안 작성 (에이전트 팀)

1. 팀 생성:
   ```
   TeamCreate(
     team_name: "presentation-team",
     members: [
       {
         name: "researcher",
         agent_type: "researcher",
         model: "opus",
         prompt: "발표 주제 '[주제]'에 대해 리서치를 수행하라. _workspace/00_input.md를 읽고 research-topic 스킬(Skill 도구로 /research-topic 호출)을 따라 조사한 뒤, 결과를 _workspace/01_researcher_findings.md에 저장하라. 완료 후 scriptwriter와 slide-builder에게 핵심 발견을 SendMessage로 공유하라."
       },
       {
         name: "scriptwriter",
         agent_type: "scriptwriter",
         model: "opus",
         prompt: "발표 대본을 작성하라. researcher의 리서치 결과를 기다린 후 _workspace/01_researcher_findings.md를 읽고, write-script 스킬(Skill 도구로 /write-script 호출)을 따라 대본을 작성하여 _workspace/02_scriptwriter_script.md에 저장하라. 작성 중 slide-builder와 슬라이드 구조를 조율하라."
       },
       {
         name: "slide-builder",
         agent_type: "slide-builder",
         model: "opus",
         prompt: "reveal.js 슬라이드를 제작하라. researcher의 리서치 결과와 scriptwriter의 대본을 기다린 후, build-slides 스킬(Skill 도구로 /build-slides 호출)을 따라 슬라이드를 제작하여 _workspace/03_slide-builder_slides.html에 저장하라. scriptwriter와 슬라이드-대본 매핑을 조율하라."
       },
       {
         name: "audience-reviewer",
         agent_type: "audience-reviewer",
         model: "opus",
         prompt: "청중 관점에서 발표를 평가하라. scriptwriter의 대본과 slide-builder의 슬라이드가 모두 완성된 후, audience-review 스킬(Skill 도구로 /audience-review 호출)을 따라 평가하고 결과를 _workspace/04_audience-reviewer_feedback.md에 저장하라. Critical/Important 피드백은 즉시 scriptwriter와 slide-builder에게 SendMessage로 전달하라."
       }
     ]
   )
   ```

2. 작업 등록:
   ```
   TaskCreate(tasks: [
     { title: "주제 리서치", description: "발표 주제에 대한 심층 조사 수행", assignee: "researcher" },
     { title: "발표 대본 작성", description: "리서치 결과 기반 발표 대본 작성", assignee: "scriptwriter", depends_on: ["주제 리서치"] },
     { title: "슬라이드 제작", description: "대본 기반 reveal.js 슬라이드 생성", assignee: "slide-builder", depends_on: ["주제 리서치", "발표 대본 작성"] },
     { title: "청중 피드백", description: "대본+슬라이드 완성 후 청중 관점 평가", assignee: "audience-reviewer", depends_on: ["발표 대본 작성", "슬라이드 제작"] },
     { title: "피드백 반영 - 대본", description: "청중 피드백 중 Critical/Important 반영. _workspace/04_audience-reviewer_feedback.md를 읽고 수정본을 _workspace/05_scriptwriter_script_v2.md에 저장하라.", assignee: "scriptwriter", depends_on: ["청중 피드백"] },
     { title: "피드백 반영 - 슬라이드", description: "청중 피드백 중 Critical/Important 반영. _workspace/04_audience-reviewer_feedback.md를 읽고 수정본을 _workspace/05_slide-builder_slides_v2.html에 저장하라.", assignee: "slide-builder", depends_on: ["청중 피드백"] }
   ])
   ```

**팀원 간 통신 규칙:**
- researcher → scriptwriter, slide-builder: 핵심 발견, 시각화 가능 데이터 공유
- scriptwriter ↔ slide-builder: 슬라이드 구조, 핵심 메시지 실시간 조율
- audience-reviewer → scriptwriter, slide-builder: Critical/Important 피드백 즉시 전달
- researcher ← audience-reviewer: 청중이 궁금해할 추가 정보 요청

**산출물 저장:**

| 팀원 | 출력 경로 |
|------|----------|
| researcher | `_workspace/01_researcher_findings.md` |
| scriptwriter | `_workspace/02_scriptwriter_script.md` |
| slide-builder | `_workspace/03_slide-builder_slides.html` |
| audience-reviewer | `_workspace/04_audience-reviewer_feedback.md` |

**리더 모니터링:**
- TaskGet으로 전체 진행률 확인
- 팀원이 막혔을 때 SendMessage로 지시
- researcher가 조기 완료되면 scriptwriter/slide-builder에게 알림
- researcher가 주제 범위가 너무 광범위하다고 보고하면, 사용자에게 3가지 세부 방향을 제시하고 선택을 요청한다

### Phase 3: 피드백 반영 및 최종화

1. 모든 팀원의 작업 완료를 대기 (TaskGet)
2. audience-reviewer의 피드백 파일을 Read
3. scriptwriter와 slide-builder가 피드백을 반영한 수정본 생성:
   - `_workspace/05_scriptwriter_script_v2.md`
   - `_workspace/05_slide-builder_slides_v2.html`
4. 수정본을 Read하여 피드백이 반영되었는지 확인:
   - `04_feedback.md`의 Critical 항목이 모두 v2에 반영되었는지 대조
   - 미반영 항목이 있으면 해당 에이전트에게 SendMessage로 구체적 항목을 재지시 (1회만 재시도)
   - 재시도 후에도 미반영이면 사용자에게 보고하고 현재 상태로 진행

### Phase 4: 최종 산출물 생성 및 정리

1. 최종 산출물을 사용자 지정 경로 (또는 현재 디렉토리)에 복사:
   - `presentation-script.md` — 최종 발표 대본
   - `presentation-slides.html` — 최종 reveal.js 슬라이드
2. 팀원들에게 종료 요청 (SendMessage)
3. 팀 정리 (TeamDelete)
4. `_workspace/` 보존 (중간 산출물 감사 추적용)
5. 사용자에게 결과 요약 보고:
   - 생성된 파일 경로
   - 슬라이드 수, 예상 발표 시간
   - 청중 피드백 종합 점수
   - "브라우저에서 slides.html을 열고 S 키로 발표자 노트를 확인하세요" 안내

## 데이터 흐름

```
[리더] → TeamCreate → [researcher] ──SendMessage──→ [scriptwriter]
                          │                              ↕ SendMessage
                          │                         [slide-builder]
                          │                              │
                          └─→ [audience-reviewer] ←── 대본+슬라이드 완성 대기
                                     │
                                     ├──SendMessage──→ [scriptwriter] (피드백)
                                     └──SendMessage──→ [slide-builder] (피드백)
                                                           │
                                                    [리더: 최종 산출물 수집]
```

## 에러 핸들링

| 상황 | 전략 |
|------|------|
| researcher 실패 | 리더가 WebSearch로 주제 관련 2-3회 검색 후 research-topic 출력 형식에 맞춰 `01_researcher_findings.md`를 직접 작성하고 scriptwriter에게 전달 |
| scriptwriter 실패 | 리더가 리서치 결과를 바탕으로 write-script 출력 형식(슬라이드별 핵심 메시지 + 시간 배분)에 맞춰 최소 구조의 대본을 `02_scriptwriter_script.md`에 직접 작성 |
| slide-builder 실패 | 대본만으로 최종 산출물 제공, 슬라이드 미생성 알림 |
| audience-reviewer 실패 | 피드백 없이 초안을 최종본으로 사용, 사용자에게 직접 리뷰 요청 |
| 팀원 간 통신 지연 | 리더가 TaskGet으로 확인 후 SendMessage로 재촉 |
| 데이터 충돌 | 출처 병기, 삭제하지 않음 |

## 추가 반복 (선택)

사용자가 결과에 만족하지 않으면:
1. 사용자 피드백을 `_workspace/06_user_feedback.md`에 저장
2. 해당 피드백을 scriptwriter와 slide-builder에게 전달하여 수정
3. 필요시 audience-reviewer에게 2차 리뷰 요청
4. 사용자가 만족할 때까지 반복

## 테스트 시나리오

### 정상 흐름
1. 사용자: "AI 도입 전략에 대해 20분짜리 발표를 만들어줘. 청중은 C레벨 경영진이야"
2. Phase 1: 주제=AI 도입 전략, 시간=20분, 청중=C레벨 경영진 파악
3. Phase 2: 팀 구성 (4명), 작업 등록 (6개)
   - researcher가 AI 도입 트렌드, ROI 데이터, 성공 사례 조사
   - scriptwriter가 경영진 대상 설득형 대본 작성
   - slide-builder가 세련된 reveal.js 슬라이드 생성
   - audience-reviewer가 C레벨 페르소나로 평가
4. Phase 3: 피드백 반영, 수정본 생성
5. Phase 4: `presentation-script.md` + `presentation-slides.html` 생성
6. 예상 결과: 20분 분량의 대본 + 15-20장의 reveal.js 슬라이드

### 에러 흐름
1. Phase 2에서 researcher가 검색 실패로 중지
2. 리더가 유휴 알림 수신
3. SendMessage로 상태 확인 → 재시작 시도
4. 재시작 실패 시 리더가 기본 리서치 직접 수행
5. 기본 리서치 결과로 나머지 팀원 작업 진행
6. 최종 보고서에 "리서치 제한적 — 추가 조사 권장" 명시
