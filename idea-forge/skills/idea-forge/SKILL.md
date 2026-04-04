---
name: idea-forge
description: "트렌드 기반 프로젝트 아이디어 생성/평가/고도화 오케스트레이터. 실시간 웹 검색으로 식품 트렌드(두쫀쿠, 버터떡, 탕후루 등), 유튜브/틱톡/인스타그램 인기 콘텐츠, SNS 바이럴, 문화 현상, 소비 트렌드를 동적으로 조사하고, 시장 분석을 통해 대중의 갈망 또는 니치 타겟의 갈망을 만족시키면서 수익성 있는 프로젝트 아이디어를 생성한다. 아이디어 확정 후 발산적 재검토로 블라인드스팟을 발굴하고, 세션 메모리를 누적하여 다음 실행에 반영한다. '아이디어 좀 줘', '프로젝트 뭐 하면 좋을까', '사이드 프로젝트 추천', '이 아이디어 어때?', '아이디어 평가해줘', '아이디어 고도화', '요즘 뭐가 돈이 될까', '트렌드 기반 사업 아이디어', 'idea brainstorm', 'project idea' 등 프로젝트 아이디어를 구상하거나 평가/개선하는 모든 요청에 사용."
---

# Idea Forge — 트렌드 기반 프로젝트 아이디어 오케스트레이터

실시간 트렌드 리서치와 시장 분석을 기반으로 수익성 있는 프로젝트 아이디어를 생성하거나, 사용자 아이디어를 평가·고도화하는 통합 스킬. 아이디어 확정 후 발산적 재검토로 블라인드스팟을 찾고, 세션 메모리를 누적하여 다음 실행을 더 날카롭게 만든다.

## 실행 모드: 에이전트 팀

## 에이전트 구성

| 팀원 | 에이전트 타입 | 역할 | 스킬 | 출력 |
|------|-------------|------|------|------|
| trend-scout | general-purpose | 실시간 트렌드 리서치 | trend-research | `_workspace/01_trend-scout_trends.md` |
| market-analyst | general-purpose | 시장/타겟/수익 분석 | market-analysis | `_workspace/01_market-analyst_analysis.md` |
| idea-architect | general-purpose | 아이디어 생성/평가/고도화 | idea-generation | `_workspace/02_idea-architect_ideas.md` |

## 워크플로우

### Phase 1: 준비

1. **사용자 입력 분석:**

   | 입력 | 필수 여부 | 기본값 |
   |------|----------|--------|
   | 아이디어 | 선택 | 없음 (생성 모드) |
   | 관심 카테고리 | 선택 | 전체 (식품, 기술, 라이프스타일 등) |
   | 타겟 | 선택 | 대중 (매스 마켓) |
   | 제약 조건 | 선택 | 없음 |

2. **모드 결정:**
   - 사용자 아이디어가 있으면 → **고도화 모드**
   - 없으면 → **생성 모드**

3. **세션 메모리 로드:**
   - `_workspace/_memory/` 디렉토리가 존재하는지 확인한다.
   - 존재하면 `_workspace/_memory/session-log.md`를 Read로 읽어 이전 세션의 학습을 파악한다.
   - 메모리에서 다음을 추출하여 이번 세션에 반영한다:
     - **관찰된 트렌드 패턴**: 반복 출현하는 트렌드, 상승/하락 추세
     - **스코어링 인사이트**: 이전에 높은/낮은 점수를 받은 아이디어 유형과 이유
     - **발산적 발견**: 이전 세션의 블라인드스팟, 역발상 아이디어
     - **사용자 피드백**: 사용자가 선호/기각한 아이디어 유형
   - 메모리 요약을 팀원 prompt에 포함하여 "처음부터 시작하지 않고" 이전 세션 위에 쌓는다.

4. **작업 디렉토리 준비:**
   ```
   _workspace/
   ├── 00_input/
   │   ├── request.md          ← 사용자 요청 정리
   │   └── user-idea.md        ← 사용자 아이디어 (있으면)
   ├── 01_trend-scout_trends.md
   ├── 01_market-analyst_analysis.md
   ├── 02_idea-architect_ideas.md
   ├── 03_divergent_insights.md
   └── _memory/                ← 세션 간 누적 메모리 (삭제 금지)
       └── session-log.md
   ```

5. 사용자 요청을 `_workspace/00_input/request.md`에 저장한다:
   ```markdown
   # Idea Forge 요청

   모드: {생성 / 고도화}
   관심 카테고리: {카테고리 목록}
   타겟: {타겟 설명}
   제약 조건: {있으면 기술}
   이전 세션 메모리: {있으면 핵심 요약 3줄}
   ```

6. 사용자 아이디어가 있으면 `_workspace/00_input/user-idea.md`에 저장한다.

### Phase 2: 팀 구성

1. 팀 생성:
   ```
   TeamCreate(
     team_name: "idea-forge-team",
     members: [
       {
         name: "trend-scout",
         agent_type: "trend-scout",
         model: "opus",
         prompt: "
           당신은 trend-scout 에이전트다. agents/trend-scout.md의 원칙을 따른다.
           trend-research 스킬(Skill 도구로 /trend-research 호출)을 사용하여 현재 트렌드를 조사하라.

           요청: {_workspace/00_input/request.md 내용}
           관심 카테고리: {카테고리}
           타겟: {타겟}
           이전 세션 메모리: {메모리 요약 — 관찰된 트렌드 패턴, 주의 사항}

           결과를 {절대경로}/_workspace/01_trend-scout_trends.md에 저장하라.
           market-analyst에게 시장 신호를 SendMessage로 공유하라.
         "
       },
       {
         name: "market-analyst",
         agent_type: "market-analyst",
         model: "opus",
         prompt: "
           당신은 market-analyst 에이전트다. agents/market-analyst.md의 원칙을 따른다.
           market-analysis 스킬(Skill 도구로 /market-analysis 호출)을 사용하여 시장을 분석하라.

           요청: {_workspace/00_input/request.md 내용}
           사용자 아이디어: {있으면 user-idea.md 내용}
           관심 카테고리: {카테고리}
           타겟: {타겟}
           이전 세션 메모리: {메모리 요약 — 스코어링 인사이트, 시장 패턴}

           결과를 {절대경로}/_workspace/01_market-analyst_analysis.md에 저장하라.
           trend-scout에게 수요 신호를 SendMessage로 공유하라.
         "
       },
       {
         name: "idea-architect",
         agent_type: "idea-architect",
         model: "opus",
         prompt: "
           당신은 idea-architect 에이전트다. agents/idea-architect.md의 원칙을 따른다.
           idea-generation 스킬(Skill 도구로 /idea-generation 호출)을 사용하라.

           모드: {생성 / 고도화}
           사용자 아이디어: {있으면 user-idea.md 내용}
           이전 세션 메모리: {메모리 요약 — 이전 아이디어 점수, 사용자 피드백, 발산적 발견}

           trend-scout과 market-analyst의 작업이 완료되면 알림을 받을 것이다.
           두 보고서를 Read로 읽고 아이디어를 {생성/고도화}하라:
           - 트렌드 보고서: {절대경로}/_workspace/01_trend-scout_trends.md
           - 시장 분석 보고서: {절대경로}/_workspace/01_market-analyst_analysis.md

           추가 데이터가 필요하면 trend-scout이나 market-analyst에게 SendMessage로 요청하라.
           결과를 {절대경로}/_workspace/02_idea-architect_ideas.md에 저장하라.
         "
       }
     ]
   )
   ```

2. 작업 등록:
   ```
   TaskCreate(tasks: [
     { title: "트렌드 리서치", description: "현재 시점의 트렌드를 카테고리별로 웹 검색 조사", assignee: "trend-scout" },
     { title: "시장 분석", description: "타겟 세그먼트, 시장 규모, 경쟁 환경, 수익 모델 분석", assignee: "market-analyst" },
     { title: "아이디어 생성/고도화", description: "트렌드×시장 교차점에서 아이디어 생성 및 스코어링 평가", assignee: "idea-architect", depends_on: ["트렌드 리서치", "시장 분석"] },
     { title: "발산적 재검토", description: "아이디어 확정 후 역발상/유추/블라인드스팟 발굴", depends_on: ["아이디어 생성/고도화"] }
   ])
   ```

### Phase 3: 리서치 (Fan-out)

**실행 방식:** trend-scout과 market-analyst가 병렬로 조사, 발견을 실시간 공유

팀원 간 통신 규칙:
- trend-scout은 market-analyst에게 트렌드 키워드, 시장 규모 힌트를 SendMessage로 전달
- market-analyst는 trend-scout에게 시장 데이터에서 발견된 수요 신호를 SendMessage로 전달
- 두 에이전트 모두 조사 완료 시 파일을 저장하고 리더에게 알림

산출물 저장:

| 팀원 | 출력 경로 |
|------|----------|
| trend-scout | `_workspace/01_trend-scout_trends.md` |
| market-analyst | `_workspace/01_market-analyst_analysis.md` |

리더 모니터링:
- 팀원이 유휴 상태가 되면 자동 알림 수신
- 특정 팀원이 막혔을 때 SendMessage로 지시
- 전체 진행률은 TaskGet으로 확인

**체크포인트 커밋:** 두 리서처 완료 후:
```
git add _workspace/01_*.md
git commit -m "idea-forge: research complete"
```

### Phase 4: 아이디에이션

**실행 방식:** idea-architect가 리서치 결과를 기반으로 아이디어 생성/고도화

1. trend-scout과 market-analyst 완료 확인 (TaskGet)
2. idea-architect에게 두 보고서가 준비되었음을 SendMessage로 알림
3. idea-architect가 두 보고서를 Read로 읽고 아이디어 작업 수행
4. 추가 데이터가 필요하면 trend-scout이나 market-analyst에게 SendMessage로 요청
5. 아이디어 보고서를 `_workspace/02_idea-architect_ideas.md`에 저장

**체크포인트 커밋:**
```
git add _workspace/02_*.md
git commit -m "idea-forge: ideation complete"
```

### Phase 5: 발산적 재검토

**목적:** 수렴적 사고(Phase 4)로 확정된 아이디어를 의도적으로 뒤흔들어 블라인드스팟을 발견한다. "우리가 놓친 것은 무엇인가?"

**실행 방식:** 팀이 아직 활성 상태. 리더가 3명 모두에게 발산적 질문을 던지고, 팀원들이 자유롭게 토론한다.

1. idea-architect의 아이디어 보고서 완료 확인 (TaskGet)

2. 리더가 각 팀원에게 관점 전환 과제를 SendMessage로 할당한다:

   **trend-scout에게:**
   ```
   아이디어 보고서를 읽고 다음 질문에 답하라:
   - 모두가 주목하는 트렌드 말고, 지금 아무도 안 보는데 6개월 후 터질 트렌드는?
   - 우리 아이디어가 기반한 트렌드가 내일 사라지면 어떻게 되는가?
   - 정반대 트렌드(예: 디지털 피로 → 아날로그 회귀)에서 나올 수 있는 아이디어는?
   결과를 다른 팀원에게도 SendMessage로 공유하라.
   ```

   **market-analyst에게:**
   ```
   아이디어 보고서를 읽고 다음 질문에 답하라:
   - 이 아이디어가 가장 실패하기 쉬운 이유 3가지는? (악마의 변호인)
   - 우리가 배제한 타겟 중 오히려 더 열광할 세그먼트가 있는가?
   - 전혀 다른 수익 모델을 적용하면? (광고→구독, 구독→일회성, B2C→B2B)
   결과를 다른 팀원에게도 SendMessage로 공유하라.
   ```

   **idea-architect에게:**
   ```
   다른 팀원들의 발산적 의견을 수신하고, 추가로 다음을 수행하라:
   - 스코어가 가장 낮았던 아이디어를 살릴 수 있는 한 가지 피벗은?
   - 전혀 관련 없어 보이는 두 트렌드를 억지로 결합하면 어떤 아이디어가 나오는가?
   - 이 아이디어들이 5년 후에도 유효한가? 유효하지 않다면 5년 뒤 버전은?
   모든 팀원의 의견과 자신의 분석을 종합하여
   {절대경로}/_workspace/03_divergent_insights.md에 저장하라.
   ```

3. 팀원들이 SendMessage로 자유 토론한다. 리더는 개입하지 않고 대화가 수렴될 때까지 대기한다.

4. idea-architect가 발산적 인사이트 보고서를 저장하면 Phase 5 완료.

**발산적 인사이트 보고서 형식:**
```markdown
# 발산적 재검토 보고서

## 블라인드스팟 발견
{우리가 놓친 관점, 배제한 세그먼트, 간과한 리스크}

## 역발상 아이디어
{정반대 방향에서 나온 대안 아이디어. 스코어링 점수 포함.}

## 위기 시나리오
{핵심 트렌드 소멸 시 생존 전략, 최악의 경쟁 시나리오}

## 와일드카드
{전혀 다른 조합에서 나온 예상 외 아이디어. 실현 가능성은 낮지만 잠재력 있는 것.}

## 다음 세션 제안
{이번에 깊이 파지 못했지만 다음에 탐색할 가치가 있는 방향}
```

**체크포인트 커밋:**
```
git add _workspace/03_*.md
git commit -m "idea-forge: divergent review complete"
```

### Phase 6: 팀 정리 및 최종 보고서 통합

1. 모든 산출물을 Read로 수집:
   - `_workspace/01_trend-scout_trends.md`
   - `_workspace/01_market-analyst_analysis.md`
   - `_workspace/02_idea-architect_ideas.md`
   - `_workspace/03_divergent_insights.md`

2. 팀원들에게 종료 요청 (SendMessage) → 팀 정리 (TeamDelete)

3. **최종 보고서 통합:** 4개 산출물을 하나의 `_workspace/final-report.md`로 통합한다:

   ```markdown
   # Idea Forge 최종 보고서

   생성 일시: {날짜}
   모드: {생성 / 고도화}

   ## 요청 요약
   {모드, 카테고리, 타겟, 제약 조건}

   ## 트렌드 요약
   {trends.md에서 핵심 트렌드 표 + 교차 트렌드 맵 발췌}

   ## 시장 기회 요약
   {analysis.md에서 기회 영역 표 + 타겟 세그먼트 + 시장 규모 발췌}

   ## 아이디어 스코어보드
   {ideas.md에서 전체 순위표}

   ## Top 아이디어 상세
   {ideas.md에서 Top 3 상세 — 원라이너, 킬러 유즈케이스, 수익 모델, MVP, 리스크}

   ## 발산적 재검토
   {divergent_insights.md 전문 — 블라인드스팟, 역발상, 위기 시나리오, 와일드카드}

   ## 다음 세션 제안
   {발산적 재검토의 다음 세션 후보 + 미탐색 영역}
   ```

4. **세션 메모리 저장:**

   `_workspace/_memory/session-log.md`에 append한다 (기존 내용 보존 + 새 세션 추가):

   ```markdown
   ---
   ## 세션 {N} — {날짜}

   ### 요청 요약
   모드: {생성/고도화} | 카테고리: {목록} | 타겟: {설명}

   ### 트렌드 패턴
   - {핵심 트렌드 3~5개, 생명주기 단계 포함}
   - {이전 세션 대비 변화: 상승/하락/신규 출현}

   ### 스코어링 인사이트
   - Top 아이디어: {이름} ({점수}/50) — 강점: {핵심 이유}
   - 약한 아이디어 공통점: {패턴}

   ### 발산적 발견
   - 블라인드스팟: {핵심 1~2개}
   - 역발상: {가장 흥미로운 1개}
   - 와일드카드: {잠재력 있는 1개}

   ### 사용자 반응
   - 선호/기각/추가 요청

   ### 다음 세션 제안
   - {탐색 후보 목록}
   ---
   ```

   **메모리 관리:** 최근 5개 세션만 유지. 6번째부터 가장 오래된 세션을 `## 누적 패턴` 섹션에 압축 후 삭제. `_memory/`는 절대 삭제하지 않는다.

### Phase 7: Cleanup

중간 산출물을 제거하고 최종 상태만 남긴다.

1. 중간 파일 삭제:
   ```
   rm _workspace/00_input/request.md
   rm _workspace/00_input/user-idea.md  (있으면)
   rmdir _workspace/00_input/
   rm _workspace/01_trend-scout_trends.md
   rm _workspace/01_market-analyst_analysis.md
   rm _workspace/02_idea-architect_ideas.md
   rm _workspace/03_divergent_insights.md
   ```

2. 최종 `_workspace/` 상태:
   ```
   _workspace/
   ├── final-report.md          ← 통합 최종 보고서
   └── _memory/
       └── session-log.md       ← 누적 세션 메모리
   ```

3. 최종 커밋:
   ```
   git add _workspace/
   git commit -m "idea-forge: final report + cleanup"
   ```

### Phase 8: 검증

verify-output 스킬(Skill 도구로 `/verify-output` 호출)을 실행하여 최종 상태를 검증한다.

검증 항목:
1. `_workspace/final-report.md`에 필수 섹션이 모두 있는가
2. `_workspace/`에 중간 파일이 남아있지 않은가
3. `_workspace/_memory/session-log.md`가 올바른 형식인가
4. git log에 체크포인트 커밋이 올바른 순서로 존재하는가

검증 통과 시 사용자에게 결과 보고. 실패 시 문제 항목과 수정 방법을 제시한 후 재검증을 제안한다.

**사용자에게 최종 보고:**

```markdown
## Idea Forge 결과

**모드**: {생성 / 고도화}
**조사된 트렌드**: {N}개
**분석된 시장 기회**: {N}개
**생성된 아이디어**: {N}개
**발산적 재검토**: 블라인드스팟 {N}개, 역발상 아이디어 {N}개
**검증**: {N}/4 PASS

### Top 3 아이디어

| 순위 | 아이디어 | 총점 | 핵심 강점 |
|------|---------|------|----------|
| 1 | {이름} | {점수}/50 | {한줄 요약} |
| 2 | {이름} | {점수}/50 | {한줄 요약} |
| 3 | {이름} | {점수}/50 | {한줄 요약} |

### 발산적 재검토 하이라이트
- **블라인드스팟**: {가장 중요한 발견}
- **와일드카드**: {가장 흥미로운 예상 외 아이디어}

최종 보고서: _workspace/final-report.md
세션 메모리 저장 완료. 다음 실행 시 자동 반영.
```

**후속 액션:**
- "특정 아이디어를 더 심화하시겠습니까?"
- "발산적 재검토의 역발상 아이디어를 정식 평가하시겠습니까?"
- "다른 카테고리로 재조사하시겠습니까?"

## 데이터 흐름

```
[사용자]
   │
   ├── (선택) 아이디어 + 카테고리 + 타겟
   ↓
[Phase 1: 준비 + 메모리 로드]
   │
   ├── request.md + user-idea.md
   ├── _memory/session-log.md → 이전 학습 추출
   ↓
[Phase 2: 팀 구성]
   │
   ├── TeamCreate(trend-scout, market-analyst, idea-architect)
   ↓
[Phase 3: 리서치 Fan-out] → git commit "research complete"
   │
   ├── trend-scout ←SendMessage→ market-analyst
   │       │                           │
   │       ↓                           ↓
   │  trends.md                   analysis.md
   ↓
[Phase 4: 아이디에이션] → git commit "ideation complete"
   │
   ├── idea-architect ← Read(trends.md + analysis.md)
   │       ↓
   │  ideas.md
   ↓
[Phase 5: 발산적 재검토] → git commit "divergent review complete"
   │
   ├── 팀원 간 자유 토론 (SendMessage)
   │       ↓
   │  divergent_insights.md
   ↓
[Phase 6: 통합 + 메모리]
   │
   ├── 4개 산출물 → final-report.md 통합
   ├── _memory/session-log.md append
   ↓
[Phase 7: Cleanup]
   │
   ├── 중간 파일 삭제 (01_*, 02_*, 03_*, 00_input/)
   ├── git commit "final report + cleanup"
   ↓
[Phase 8: 검증]
   │
   ├── /verify-output 스킬 실행
   ├── 4개 항목 PASS/FAIL 판정
   └── 사용자 최종 보고
```

## 에러 핸들링

| 상황 | 전략 |
|------|------|
| trend-scout 실패 | market-analyst 결과만으로 진행. 트렌드 데이터 부재를 보고서에 명시. |
| market-analyst 실패 | trend-scout 결과만으로 진행. 시장 분석 부재를 보고서에 명시. |
| 두 리서처 모두 실패 | 오케스트레이터가 직접 WebSearch 5회로 최소 트렌드 3개 + 시장 개요를 작성 후 idea-architect에 전달 |
| idea-architect 실패 | 오케스트레이터가 두 보고서를 읽고 Top 3 기회 영역을 간략히 정리하여 사용자에게 제시 |
| Phase 5 토론 교착 | 리더가 3분 대기 후 현재까지의 발산적 의견만으로 보고서 작성 지시 |
| 메모리 파일 손상/부재 | 메모리 없이 진행. 이번 세션 종료 시 새 메모리 파일 생성. |
| 팀원 간 데이터 충돌 | 출처 명시 후 병기, 삭제하지 않음 |
| 웹 검색 결과 부족 | 영어 키워드로 글로벌 데이터 확보 후 한국 시장 적용 가능성 분석 |

## 테스트 시나리오

### 정상 흐름 — 생성 모드 (첫 세션)
1. 사용자: "요즘 뭐가 핫한지 조사해서 사이드 프로젝트 아이디어 좀 만들어줘"
2. Phase 1: 모드=생성, 메모리 없음 (첫 실행)
3. Phase 2: 3명 팀 구성 + 작업 등록
4. Phase 3: 리서치 → commit "research complete"
5. Phase 4: 아이디에이션 → commit "ideation complete"
6. Phase 5: 발산적 재검토 → commit "divergent review complete"
7. Phase 6: final-report.md 통합 + 메모리 저장
8. Phase 7: 중간 파일 삭제 → commit "final report + cleanup"
9. Phase 8: /verify-output → 4/4 PASS → 사용자 보고

### 정상 흐름 — 두 번째 세션 (메모리 활용)
1. 사용자: "지난번이랑 다른 방향으로 아이디어 더 만들어줘"
2. Phase 1: 메모리 로드 → 이전 누적 패턴을 팀원 prompt에 반영
3. Phase 3~5: 이전 세션과 다른 방향의 아이디어 생성 + 발산적 재검토
4. Phase 6: final-report.md 통합 + 메모리 append + 누적 패턴 업데이트
5. Phase 7: cleanup → Phase 8: 검증

### 에러 흐름
1. Phase 3에서 trend-scout이 WebSearch 반복 실패로 중지
2. market-analyst 결과만으로 Phase 4~5 진행
3. Phase 6: final-report에 "트렌드 리서치 미완료" 명시
4. Phase 7: cleanup 정상 진행
5. Phase 8: 검증에서 최종 보고서 "트렌드 요약" 섹션이 불완전 → FAIL 1개 보고, 원인과 함께 사용자에게 안내
