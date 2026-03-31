---
name: build-slides
description: "reveal.js를 활용해 발표 슬라이드 HTML을 생성하는 스킬. 대본의 핵심 메시지를 시각적으로 효과적인 슬라이드로 변환하고, 애니메이션·전환·발표자 노트를 포함한 단일 HTML 파일을 생성한다. 슬라이드 제작, 프레젠테이션 제작, reveal.js, 발표 자료 만들기 시 사용."
---

# Build Slides — reveal.js 슬라이드 제작

대본을 기반으로 reveal.js HTML 슬라이드를 제작한다. 상세 reveal.js API 레퍼런스는 `references/revealjs-reference.md`를 Read로 참조한다.

## 슬라이드 설계 원칙

### 시각적 원칙
- **텍스트 최소화**: 슬라이드에 문장을 쓰지 않는다. 키워드, 수치, 짧은 구문만 사용
- **1슬라이드 1비주얼**: 하나의 시각 요소에 집중 (차트 하나, 이미지 하나, 키워드 하나)
- **여백 활용**: 빽빽한 슬라이드는 읽히지 않는다. 넉넉한 여백을 확보
- **일관된 디자인**: 색상 팔레트, 폰트, 여백을 전체 슬라이드에서 통일

### 슬라이드 유형별 레이아웃

| 유형 | 용도 | 핵심 요소 |
|------|------|----------|
| **타이틀** | 발표 제목, 섹션 구분 | 큰 텍스트, 서브타이틀 |
| **키워드** | 핵심 개념 강조 | 1-3개 단어, 큰 폰트 |
| **통계** | 수치 강조 | 큰 숫자 + 한줄 맥락 |
| **비교** | 전후, A vs B | 2-3열 레이아웃 |
| **리스트** | 순차적 포인트 | fragment로 하나씩 노출 |
| **인용** | 권위 있는 발언 | blockquote 스타일 |
| **코드** | 기술 설명 | 코드 하이라이팅 |

### 프래그먼트 활용

리스트나 단계별 설명에는 fragment를 사용하여 하나씩 노출한다. 청중이 발표자보다 앞서 읽는 것을 방지하고, 발표자의 설명 흐름을 따라가게 한다.

```html
<ul>
  <li class="fragment">첫 번째 포인트</li>
  <li class="fragment">두 번째 포인트</li>
  <li class="fragment">세 번째 포인트</li>
</ul>
```

### 발표자 노트

모든 슬라이드에 발표자 노트를 포함한다. 대본의 핵심 키워드를 발표자 노트에 넣어 발표자가 `S` 키로 확인할 수 있게 한다.

```html
<section>
  <h2>슬라이드 제목</h2>
  <p class="fragment">핵심 포인트</p>
  <aside class="notes">
    여기에 발표자 노트. 핵심 키워드와 전환 멘트 포함.
  </aside>
</section>
```

## HTML 기본 구조

CDN 기반 단일 HTML 파일로 생성한다. 별도 빌드 없이 브라우저에서 바로 열 수 있어야 한다.

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>발표 제목</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5/dist/reveal.css">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5/dist/theme/white.css">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5/plugin/highlight/monokai.css">
  <style>
    /* 커스텀 스타일 */
  </style>
</head>
<body>
  <div class="reveal">
    <div class="slides">
      <!-- 슬라이드들 -->
    </div>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/reveal.js@5/dist/reveal.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/reveal.js@5/plugin/notes/notes.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/reveal.js@5/plugin/highlight/highlight.js"></script>
  <script>
    Reveal.initialize({
      hash: true,
      slideNumber: true,
      plugins: [RevealNotes, RevealHighlight]
    });
  </script>
</body>
</html>
```

## 테마 선택 가이드

| 테마 | 분위기 | 적합한 발표 |
|------|--------|-----------|
| white | 깔끔, 밝음 | 비즈니스, 교육 |
| black | 모던, 세련 | 기술, 컨퍼런스 |
| league | 다크, 프로페셔널 | 기조연설 |
| beige | 따뜻, 클래식 | 학술 |
| sky | 경쾌, 가벼움 | 워크숍 |
| moon | 어둡고 부드러움 | 크리에이티브 |
| solarized | 가독성 우수 | 코드 중심 |

기본값은 `white`. 사용자가 지정하지 않으면 발표 성격에 맞는 테마를 선택하고 선택 이유를 설명한다.

## 피드백 반영

audience-reviewer의 시각적 피드백을 반영할 때:
- 슬라이드 구조 변경 시 scriptwriter에게 SendMessage로 알린다
- 수정 전/후를 비교할 수 있도록 수정 내역을 남긴다
