# reveal.js 상세 레퍼런스

build-slides 스킬의 보충 레퍼런스. 필요할 때만 Read로 로드한다.

## 목차
1. [초기화 옵션](#초기화-옵션)
2. [수직 슬라이드](#수직-슬라이드)
3. [프래그먼트 유형](#프래그먼트-유형)
4. [전환 효과](#전환-효과)
5. [배경 설정](#배경-설정)
6. [코드 하이라이팅](#코드-하이라이팅)
7. [커스텀 CSS 패턴](#커스텀-css-패턴)
8. [유용한 플러그인](#유용한-플러그인)

---

## 초기화 옵션

```javascript
Reveal.initialize({
  hash: true,              // URL에 슬라이드 위치 반영
  slideNumber: true,       // 슬라이드 번호 표시
  progress: true,          // 진행 바 표시
  center: true,            // 콘텐츠 수직 중앙 정렬
  transition: 'slide',     // 전환 효과
  width: 1920,             // 슬라이드 너비
  height: 1080,            // 슬라이드 높이
  margin: 0.04,            // 여백
  plugins: [RevealNotes, RevealHighlight]
});
```

## 수직 슬라이드

섹션 내 하위 주제를 수직으로 배치할 때 사용한다. 메인 흐름은 가로, 세부 내용은 세로.

```html
<section>
  <section>메인 섹션 제목</section>
  <section>세부 내용 1</section>
  <section>세부 내용 2</section>
</section>
```

## 프래그먼트 유형

```html
<p class="fragment">기본 (fade-in)</p>
<p class="fragment fade-out">사라짐</p>
<p class="fragment highlight-red">빨간색 강조</p>
<p class="fragment highlight-blue">파란색 강조</p>
<p class="fragment fade-in-then-out">나타났다 사라짐</p>
<p class="fragment fade-up">아래에서 위로</p>
<p class="fragment grow">커짐</p>
<p class="fragment shrink">작아짐</p>
<p class="fragment strike">취소선</p>
```

순서 지정:
```html
<p class="fragment" data-fragment-index="2">두 번째</p>
<p class="fragment" data-fragment-index="1">첫 번째</p>
```

## 전환 효과

슬라이드별 또는 전역:
```html
<!-- 슬라이드별 -->
<section data-transition="zoom">줌 인</section>
<section data-transition="fade">페이드</section>
<section data-transition="slide">슬라이드</section>
<section data-transition="convex">볼록</section>
<section data-transition="concave">오목</section>

<!-- 입장/퇴장 다르게 -->
<section data-transition="slide-in fade-out">입장은 슬라이드, 퇴장은 페이드</section>
```

## 배경 설정

```html
<!-- 단색 배경 -->
<section data-background-color="#4d7e65">내용</section>

<!-- 그라데이션 -->
<section data-background-gradient="linear-gradient(to bottom, #283b95, #17b2c3)">내용</section>

<!-- 이미지 배경 -->
<section data-background-image="url" data-background-size="cover">내용</section>
```

## 코드 하이라이팅

```html
<pre><code data-trim data-noescape data-line-numbers="1-3|5-7">
function hello() {
  console.log("Hello");
  return true;

  // 이 부분이 강조됨
  const x = 42;
  return x;
}
</code></pre>
```

`data-line-numbers="1-3|5-7"`: 먼저 1-3줄 강조, 다음에 5-7줄 강조

## 커스텀 CSS 패턴

### 큰 숫자 강조

```css
.big-number {
  font-size: 4em;
  font-weight: bold;
  color: #2196F3;
}
.big-number-context {
  font-size: 0.8em;
  color: #666;
  margin-top: 0.5em;
}
```

```html
<section>
  <div class="big-number">73%</div>
  <div class="big-number-context">의 기업이 AI를 도입했습니다</div>
</section>
```

### 2열 레이아웃

```css
.two-columns {
  display: flex;
  gap: 2em;
}
.two-columns > div {
  flex: 1;
}
```

```html
<section>
  <h2>Before vs After</h2>
  <div class="two-columns">
    <div>
      <h3>Before</h3>
      <ul><li>항목 1</li></ul>
    </div>
    <div>
      <h3>After</h3>
      <ul><li>항목 1</li></ul>
    </div>
  </div>
</section>
```

### 인용문

```css
.quote {
  font-style: italic;
  font-size: 1.4em;
  border-left: 4px solid #2196F3;
  padding-left: 1em;
  text-align: left;
}
.quote-author {
  font-size: 0.7em;
  color: #888;
  margin-top: 0.5em;
}
```

## 유용한 플러그인

| 플러그인 | CDN | 용도 |
|---------|-----|------|
| Notes | `reveal.js@5/plugin/notes/notes.js` | 발표자 노트 |
| Highlight | `reveal.js@5/plugin/highlight/highlight.js` | 코드 하이라이팅 |
| Math | `reveal.js@5/plugin/math/math.js` | 수식 (KaTeX) |
| Markdown | `reveal.js@5/plugin/markdown/markdown.js` | 마크다운으로 슬라이드 작성 |

### Math 플러그인 사용

```html
<script src="https://cdn.jsdelivr.net/npm/reveal.js@5/plugin/math/math.js"></script>
<script>
  Reveal.initialize({
    plugins: [RevealMath.KaTeX]
  });
</script>

<!-- 사용 -->
<section>
  <p>$E = mc^2$</p>
</section>
```
