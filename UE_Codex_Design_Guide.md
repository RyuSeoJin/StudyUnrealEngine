# UE Codex — 디자인 가이드

> 이 문서는 `index.html`에 실제로 사용된 값과 패턴을 기반으로 정리한 디자인 규칙입니다.
> 새 컴포넌트를 추가하거나 기존 UI를 수정할 때 이 가이드를 기준으로 삼으세요.

---

## 1. 색상 시스템 (CSS Variables)

모든 색은 `:root`에서 CSS 변수로 관리합니다. 직접 hex를 쓰지 않고 **반드시 변수를 사용**합니다.

### 배경 계층 (어두울수록 더 깊은 레이어)

| 변수 | 값 | 용도 |
|---|---|---|
| `--bg` | `#0d1117` | 페이지 기본 배경 |
| `--bg-soft` | `#161b22` | Nav, 입력 필드 배경 등 약간 띄운 표면 |
| `--bg-card` | `#1a2029` | 카드·패널 배경 |
| `--bg-deep` | `#0a0e14` | 코드 블록, 에디터 최심층 배경 |

### 테두리

| 변수 | 값 | 용도 |
|---|---|---|
| `--border` | `#2d3441` | 기본 테두리 |
| `--border-bright` | `#3d4756` | 호버·포커스 시 더 밝은 테두리 |

### 텍스트 계층

| 변수 | 값 | 용도 |
|---|---|---|
| `--text` | `#e6edf3` | 본문 기본색 (제목, 강조 텍스트) |
| `--text-dim` | `#8b949e` | 보조 설명, 서브타이틀 |
| `--text-faint` | `#6e7681` | 메타 정보, 레이블, placeholder |

### 액센트 / 시맨틱 색상

| 변수 | 값 | 의미 |
|---|---|---|
| `--accent` | `#58d6a0` | 브랜드 그린 — 주요 강조, 성공, 완료 |
| `--accent-2` | `#4fb3ff` | 블루 — C++ 계열, 코드 블록 왼쪽 라인 |
| `--accent-3` | `#d99cff` | 퍼플 — 키워드 (`kw`), 보조 포인트 |
| `--amber` | `#f0b860` | 앰버 — 블루프린트 계열, 경고, 매크로 |
| `--coral` | `#ff7b72` | 코럴 — 오류, 숫자 리터럴 |
| `--good` | `#3fb950` | 정답·성공 |
| `--bad` | `#f85149` | 오답·에러 |

```css
/* ✅ 올바른 사용 */
color: var(--accent);
border: 1px solid var(--border);

/* ❌ 잘못된 사용 */
color: #58d6a0;
border: 1px solid #2d3441;
```

---

## 2. 타이포그래피

두 가지 폰트 패밀리를 맥락에 따라 구분합니다.

| 변수 | 폰트 | 용도 |
|---|---|---|
| `--sans` | `Space Grotesk` | UI 텍스트, 제목, 본문, 버튼 |
| `--mono` | `JetBrains Mono` | 코드, 태그, 레이블, 메타 정보, 숫자 |

### 텍스트 크기 스케일

| 용도 | 크기 | 폰트 | 비고 |
|---|---|---|---|
| 뷰 제목 (`h2.view-title`) | 28px / 700 | sans | `letter-spacing: -0.5px` |
| 카드 제목 | 17px / 600 | sans | |
| 예제 카드 제목 | 16px / 600 | sans | |
| 퀴즈 문제 | 18px / 600 | sans | `line-height: 1.45` |
| 본문 | 14.5px / 400 | sans | `line-height: 1.6` |
| 작은 본문 | 14px / 400 | sans | |
| 코드 블록 | 13px / 400 | mono | `line-height: 1.75` |
| 메타·레이블 | 11–12px / 700 | mono | `letter-spacing: 1.5–2px`, 대문자 |
| 브랜드명 | 22px / 700 | sans | |

### 섹션 레이블 패턴
```css
/* SECTION LABEL — 구분선 포함한 소제목 */
font-family: var(--mono);
font-size: 11px;
letter-spacing: 2px;
text-transform: uppercase;
color: var(--text-faint);
```

---

## 3. 배경과 분위기

### 페이지 배경
미묘한 **방사형 그라디언트 두 개**가 모서리에서 빛나는 효과와, 고정 위치의 **격자 패턴**을 레이어로 사용합니다.

```css
body {
  background: var(--bg);
  background-image:
    radial-gradient(circle at 15% 10%, rgba(88,214,160,0.06), transparent 40%),   /* 좌상단 그린 */
    radial-gradient(circle at 85% 90%, rgba(79,179,255,0.06), transparent 40%);   /* 우하단 블루 */
}
body::before {
  /* 48px 격자, opacity 0.15로 은은하게 */
  background-image:
    linear-gradient(var(--border) 1px, transparent 1px),
    linear-gradient(90deg, var(--border) 1px, transparent 1px);
  background-size: 48px 48px;
  opacity: 0.15;
  pointer-events: none;
  position: fixed; inset: 0;
}
```

### 블루프린트 에디터 배경
점·소격자·대격자를 겹쳐 Unreal Engine 편집기 느낌을 재현합니다.
```css
background-color: #0a0d12;
background-image:
  radial-gradient(circle, #1a212c 1.2px, transparent 1.2px),  /* 점 */
  linear-gradient(#10151d 1px, transparent 1px),              /* 소격자 */
  linear-gradient(90deg, #10151d 1px, transparent 1px);       /* 소격자 90° */
background-size: 22px 22px, 110px 110px, 110px 110px;
```

---

## 4. 레이아웃

```css
.wrap {
  max-width: 1000px;
  margin: 0 auto;
  padding: 0 20px;
  position: relative;
  z-index: 1;   /* body::before 격자 위에 올라오도록 */
}
```

- 최대 너비: **1000px**
- 좌우 패딩: **20px**
- 하단 패딩(body): **80px** (footer와 콘텐츠 겹침 방지)

---

## 5. 네비게이션

### 레벨 1 — 주제 탭 (대시보드 / 블루프린트 / C++)

```
형태: 둥근 pill 컨테이너 안에 탭 버튼들
배경: --bg-soft
테두리: 1px solid --border / border-radius 12px
내부 패딩: 6px
```

```css
/* 탭 버튼 */
font-size: 14px; font-weight: 600;
padding: 10px 18px; border-radius: 8px;
color: var(--text-dim);   /* 기본 */

/* active */
background: var(--bg-card);
color: var(--accent);
box-shadow: 0 2px 8px rgba(0,0,0,0.3);
```

### 레벨 2 — 모드 탭 (개념 정리 · 실습 · 퀴즈)

```
형태: 하단 라인 방식 (border-bottom underline)
배경: 없음 (투명)
구분선: border-bottom: 1px solid --border
```

```css
/* 서브탭 버튼 */
font-size: 13.5px; font-weight: 600;
padding: 10px 14px 12px;
border-bottom: 2px solid transparent;  /* 기본 */

/* active */
color: var(--accent);
border-bottom-color: var(--accent);
```

### 규칙
- 레벨 1이 `dashboard`이면 레벨 2를 숨긴다.
- 레벨 2는 레벨 1의 바로 아래, 가로 스크롤 가능.

---

## 6. 카드 컴포넌트

### 개념 카드 (접이식)

```
배경: --bg-card
테두리: 1px solid --border → hover: --border-bright
border-radius: 12px
```

**헤더 (항상 표시)**
```
padding: 18px 22px
레이아웃: [태그] [제목 flex:1] [체크박스] [화살표]
```

**바디 (접힘/펼침)**
```
기본: max-height: 0 (숨김)
열림: max-height: 2000px
전환: transition: max-height 0.4s ease
내부 패딩: 0 22px 22px
텍스트: --text-dim, 14.5px
```

**태그 배지**
```css
font-family: var(--mono);
font-size: 11px; font-weight: 700;
padding: 4px 9px; border-radius: 5px;

.tag-1 { background: rgba(88,214,160,0.15); color: var(--accent); }    /* 공통 — 그린 */
.tag-2 { background: rgba(79,179,255,0.15); color: var(--accent-2); }  /* C++ — 블루 */
.tag-3 { background: rgba(217,156,255,0.15); color: var(--accent-3); } /* 퍼플 */
.tag-4 { background: rgba(240,184,96,0.15); color: var(--amber); }     /* BP — 앰버 */
.tag-5 { background: rgba(255,123,114,0.15); color: var(--coral); }    /* 코럴 */
```

**완료 체크박스**
```
크기: 22×22px, border-radius 6px
기본: border: 1.5px solid --border-bright, 텍스트 투명
완료: background + border: --accent, 텍스트 --bg
```

### 예제 카드 (개념 카드와 동일한 구조)
```
바디 max-height 열림: 4000px (코드가 더 길기 때문)
헤더 구성: [난이도 별] [제목+메타 flex:1] [체크박스] [화살표]
난이도: mono 13px, --amber, ★/★★/★★★
```

---

## 7. 코드 블록

### pre 기본 스타일
```css
font-family: var(--mono);
background: var(--bg-deep);
border: 1px solid var(--border);
border-left: 3px solid var(--accent-2);   /* 블루 왼쪽 강조선 */
border-radius: 8px;
padding: 14px 16px;
font-size: 13px;
line-height: 1.75;
```

**변형:**
- `pre.bp` — `border-left-color: var(--amber)` (블루프린트 코드)
- `pre.cpp` — `border-left-color: var(--accent-2)` (기본과 동일)

### 구문 강조 스팬 클래스

| 클래스 | 색 | 대상 |
|---|---|---|
| `.cm` | `--text-faint` | 주석 |
| `.kw` | `--accent-3` (퍼플) | 키워드 (`class`, `void`, `const`…) |
| `.mac` | `--amber` | UE 매크로 (`UCLASS`, `UPROPERTY`…) |
| `.str` | `--accent` (그린) | 문자열 리터럴 |
| `.num` | `--coral` | 숫자 리터럴 |

### 인라인 코드 (`<code>`)
```css
font-family: var(--mono);
background: var(--bg);
border: 1px solid var(--border);
padding: 1px 6px; border-radius: 4px;
font-size: 12.5px;
color: var(--accent);
```

---

## 8. 버튼 시스템

### Primary `.btn`
```css
font-family: var(--mono); font-size: 14px; font-weight: 700;
padding: 12px 28px; border-radius: 9px;
background: var(--accent); color: var(--bg);
```
```css
/* hover */
transform: translateY(-2px);
box-shadow: 0 6px 20px rgba(88,214,160,0.3);

/* disabled */
opacity: 0.4; cursor: not-allowed;
```

### Ghost `.btn.ghost`
```css
background: var(--bg-soft);
color: var(--text);
border: 1px solid var(--border-bright);
/* hover: border-color → --accent */
```

### 정답 공개 버튼 `.reveal-btn`
```css
font-family: var(--mono); font-size: 13px; font-weight: 700;
padding: 11px 22px; border-radius: 8px;
border: 1px dashed var(--accent);       /* ← 점선 테두리 */
background: rgba(88,214,160,0.05);
color: var(--accent);
/* hover: background → rgba(88,214,160,0.12) */
```

### 학습 완료 버튼 `.mark-btn`
```css
font-family: var(--mono); font-size: 12px; font-weight: 700;
padding: 8px 14px; border-radius: 7px;
background: var(--bg-soft);
border: 1px solid var(--border-bright);
/* done: background + border-color → --accent, color → --bg */
```

### 에디터 소형 버튼 `.bpe-btn`
```css
font-family: var(--mono); font-size: 11px; font-weight: 700;
padding: 5px 12px; border-radius: 6px;
background: var(--bg-soft);
border: 1px solid var(--border-bright);
/* hover: border-color + color → --accent */
```

---

## 9. 카테고리 필터 칩

```css
.cat-chip {
  font-family: var(--mono); font-size: 12px;
  padding: 6px 12px; border-radius: 7px;
  border: 1px solid var(--border);
  background: var(--bg-soft);
  color: var(--text-dim);
}
.cat-chip.active {
  background: var(--accent);
  border-color: var(--accent);
  color: var(--bg);
  font-weight: 700;
}
```

---

## 10. 퀴즈 컴포넌트

### 질문 카드
```css
background: var(--bg-card);
border: 1px solid var(--border);
border-radius: 14px; padding: 28px;
```

### 선택지 `.opt`
```css
padding: 14px 18px; border-radius: 10px;
background: var(--bg-soft);
border: 1px solid var(--border);
font-size: 14.5px;
/* hover: border-color → --accent + translateX(3px) */
```
```css
/* 결과 상태 */
.opt.correct { border: --good; background: rgba(63,185,80,0.12); }
.opt.wrong   { border: --bad;  background: rgba(248,81,73,0.12); }
.opt.dim     { opacity: 0.5; }
```

### 진행 바 (Progress)
```css
height: 6px; border-radius: 3px;
.progress-fill {
  background: linear-gradient(90deg, var(--accent), var(--accent-2));
}
```

### 해설 블록 `.explain`
```css
border-left: 3px solid var(--accent-2);
background: var(--bg-soft);
padding: 16px 18px; border-radius: 10px;
```

---

## 11. 대시보드 컴포넌트

### 웰컴 카드
```css
background: linear-gradient(135deg, rgba(88,214,160,0.08), rgba(79,179,255,0.06));
border: 1px solid var(--border-bright);
border-radius: 14px; padding: 28px;
```

### 통계 카드 `.stat-card`
```css
background: var(--bg-card);
border-radius: 12px; padding: 20px;
/* 우상단에 색상 원형 장식 (opacity 0.08) */
```
```
.s1 → --accent   (그린)
.s2 → --accent-2 (블루)
.s3 → --accent-3 (퍼플)
.s4 → --amber    (앰버)
```

### 퀵 액션 카드
```css
border-radius: 12px; padding: 18px;
/* hover: border-color → --accent + translateY(-2px) */
```
아이콘 박스: `40×40px, border-radius: 10px, background: --bg-soft`

---

## 12. 블루프린트 에디터

### 노드 헤더 색상 (유형별)

| 유형 | 헤더 그라디언트 | 의미 |
|---|---|---|
| `event` | `#d24a3b → #7c1812` | 이벤트 — 빨강 |
| `function` | `#3b82d6 → #163a66` | 함수 호출 — 파랑 |
| `pure` / `variable` | `#27a055 → #114f2a` | 순수 함수·변수 — 초록 |
| `flow` | `#8a8a8a → #3a3a3a` | 흐름 제어 — 회색 |

### 핀 색상 (`PIN_COLORS` JS 변수)

| 타입 | 색 |
|---|---|
| `exec` | `#ffffff` |
| `float` | `#9ccc3c` |
| `string` | `#e84bc9` |
| `int` | `#3dcbdb` |
| `bool` | `#a83232` |
| `vector` | `#ffd700` |
| `object` | `#3a9bdc` |
| `name` | `#c8a4e0` |

### 와이어 렌더링 (이중 선 글로우)
필터(SVG filter) 없이 두 개의 path로 표현합니다.
```html
<!-- 1. 글로우 후광: 7px, opacity 0.22 -->
<path class="wire-glow" stroke-width="7" opacity="0.22" />
<!-- 2. 선명한 선: 2.4px, 불투명 -->
<path class="wire" stroke-width="2.4" stroke-linecap="round" />
```
> ⚠️ SVG `<filter>` 를 사용하면 여러 에디터 인스턴스가 열렸을 때 filter id 충돌로 와이어가 보이지 않는 버그가 발생합니다. 반드시 이중 선 방식을 유지하세요.

---

## 13. 애니메이션

| 이름 | 내용 | 사용처 |
|---|---|---|
| `fade` | `opacity 0→1, translateY 8px→0` | 뷰 진입, 해설 등장, 정답 공개 |
| `hover lift` | `translateY(-2px)` | Primary 버튼, 퀵 액션 카드 |
| `hover slide` | `translateX(3px)` | 퀵 액션 화살표, 퀴즈 선택지 |
| 카드 접힘 | `max-height 전환` | 개념 카드·예제 카드 |
| 진행 바 | `width 전환 0.4s` | 퀴즈 진행 바 |

```css
/* 기본 전환 */
transition: all 0.2s ease;   /* 버튼, 테두리, 색 */
transition: all 0.18s ease;  /* 서브탭, 선택지 */
transition: max-height 0.4–0.5s ease;  /* 카드 접힘 */
```

---

## 14. 설명 블록 패턴

### `desc-block` — 파란 왼쪽 라인 설명
```css
background: var(--bg-soft);
border-left: 3px solid var(--accent-2);
border-radius: 0 8px 8px 0;
padding: 14px 18px;
```

### `think-prompt` — 앰버 점선 고민 유도
```css
background: rgba(240,184,96,0.08);
border: 1px dashed var(--amber);
border-radius: 10px;
padding: 16px 18px;
```

### `explain` — 채점 후 해설
```css
border-left: 3px solid var(--accent-2);
background: var(--bg-soft);
```

---

## 15. 핵심 규칙 요약

1. **색은 변수만** — 직접 hex 사용 금지.
2. **폰트 두 가지만** — UI는 `--sans`, 코드·메타는 `--mono`.
3. **카드 반경** — 대형 카드 `14px`, 일반 카드 `12px`, 소형 요소 `8–10px`.
4. **테두리는 기본 → 호버 패턴** — 기본 `--border`, hover `--border-bright`, 강조 `--accent`.
5. **SVG filter 금지** — 와이어 글로우는 이중 선으로만.
6. **전환은 0.2s ease** — 특수한 경우(카드 접힘, 진행 바)만 별도 지정.
7. **여백 리듬** — 컴포넌트 간 `14px`, 섹션 간 `28–32px`, 카드 내부 패딩 `18–22px`.
8. **텍스트 계층** — 제목 `--text`, 본문 `--text-dim`, 메타 `--text-faint`.
9. **액센트 배분** — 그린(`--accent`)은 완료·브랜드, 블루(`--accent-2`)는 C++·코드, 앰버(`--amber`)는 블루프린트·경고.
10. **grid / flex** — 통계 카드는 `auto-fit minmax(180px, 1fr)`, 퀵 액션은 `minmax(220px, 1fr)`.
11. **코드 예문 주석은 요소 단위로** — 개념 카드에 코드가 있으면 키워드·연산자·이름 하나하나에 "이게 무엇인지 + 왜 있는지"를 주석으로 작성한다.

---

## 16. 콘텐츠 작성 규칙 — 개념 카드

개념 카드를 새로 추가하거나 수정할 때 따르는 글쓰기 규칙입니다.

### 16-1. 코드 예문 주석 원칙

**"처음 보는 사람이 코드 블록 하나만 읽고도 전체 흐름을 이해할 수 있어야 한다."**

코드 예문이 있는 카드는 아래 세 가지를 모두 지킵니다.

| 기준 | 내용 |
|---|---|
| **요소 단위 주석** | 키워드·연산자·이름 하나하나에 주석을 달 것 |
| **이름 + 이유** | "이게 뭔지"와 "왜 쓰는지" 둘 다 설명 |
| **결과 명시** | 이 코드가 없거나 틀리면 어떻게 되는지 언급 |

```html
<!-- ✅ 올바른 예 — class, 상속, GENERATED_BODY 모두 개별 설명 -->
<pre>
<span class="kw">class</span> AMyActor : <span class="kw">public</span> AActor {
<span class="cm">// class   : C++에서 클래스를 만드는 키워드
// AMyActor: 클래스 이름. 'A' = Actor 계열 (월드에 배치 가능)
// : public AActor : 콜론 = 상속 선언. AActor를 부모로 삼겠다
//   public = 부모의 public·protected 멤버를 그대로 유지</span>

  <span class="mac">GENERATED_BODY()</span>  <span class="cm">// 리플렉션 코드 자동 삽입. 없으면 UPROPERTY 동작 안 함
                    // 항상 클래스 안 첫 번째 줄</span>
...
</pre>

<!-- ❌ 잘못된 예 — 주석이 너무 짧거나 "왜"가 없음 -->
<pre>
<span class="kw">class</span> AMyActor : <span class="kw">public</span> AActor {
  <span class="mac">GENERATED_BODY()</span>  <span class="cm">// UE가 코드를 자동 생성하는 자리</span>
</pre>
```

### 16-2. 주석 작성 체크리스트

새 코드 예문을 작성할 때 아래 항목을 하나씩 확인합니다.

- [ ] **키워드** (`class`, `void`, `virtual`, `override`, `const`, `nullptr`…) — 역할 설명
- [ ] **연산자** (`::`, `->`, `.`, `:`, `&`, `*`, `= 0`…) — 의미와 용도 설명
- [ ] **UE 매크로** (`UCLASS`, `UPROPERTY`, `UFUNCTION`, `GENERATED_BODY`…) — 없으면 어떻게 되는지 포함
- [ ] **이름** (클래스·함수·변수명) — 접두사(A·U·F·I) 또는 UE 관례가 있으면 언급
- [ ] **위치 제약** (첫 줄, 마지막 include 등) — 순서가 중요하면 명시
- [ ] **없으면 어떻게 되나** — 해당 요소를 빠뜨렸을 때의 결과(컴파일 오류, 크래시, 버그) 언급

### 16-3. 구조 패턴 — .h / .cpp 구분 카드

.h와 .cpp를 나란히 보여줄 때는 파일 구분 헤더 주석을 반드시 달아 어느 파일의 코드인지 명확히 합니다.

```html
<pre>
<span class="cm">// ═══════════════════════════════════
// 📄 MyActor.h — 선언 파일 ("무엇이 있는지"만)
// ═══════════════════════════════════</span>
... .h 코드 ...
</pre>

<pre>
<span class="cm">// ═══════════════════════════════════
// 📄 MyActor.cpp — 구현 파일 ("어떻게 동작하는지")
// ═══════════════════════════════════</span>
... .cpp 코드 ...
</pre>
```

### 16-4. 구분 섹션 패턴 — 개념이 여러 갈래일 때

같은 카드 안에 포인터/참조처럼 비교 대상이 둘 이상이면 섹션 구분 주석으로 코드 블록 안을 나눕니다.

```html
<pre>
<span class="cm">// ══ 1) 변수 앞 const ════════════════</span>
<span class="kw">const float</span> MaxHealth = 100.f;
<span class="cm">// 값을 바꿀 수 없음. 바꾸면 컴파일 오류</span>

<span class="cm">// ══ 2) 함수 인자의 const & ══════════</span>
<span class="kw">void</span> Print(<span class="kw">const</span> FString&amp; Name);
<span class="cm">// 복사 없이 읽기만. 성능 + 안전성</span>
</pre>
```

### 16-5. 텍스트 본문 작성 원칙

코드 블록 앞뒤의 `<p>` 텍스트는 다음 역할을 담당합니다.

| 위치 | 역할 | 예시 |
|---|---|---|
| 코드 **앞** `<p>` | 이 개념이 "왜 필요한지" 한 문장 요약 | "생성자는 객체가 만들어질 때 딱 한 번 자동 호출되는 함수입니다." |
| 코드 **뒤** `<p>` | 핵심 규칙 요약 또는 자주 하는 실수 경고 | "**Super::**를 빠뜨리면 초기화 누락 버그가 생길 수 있어요." |

- `<strong>` — 기억해야 할 핵심 문구 강조 (남용 금지, 문단당 1–2개)
- `<code>` — 인라인 코드 참조 (키워드, 함수명, 파일명 등)
- 글머리 기호 대신 ①②③ 또는 자연스러운 문장으로 서술

---

## 17. 개념 검색 기능 설계 규칙

개념 정리(개념 카드) 화면에서만 동작하는 검색 기능의 동작 방식과 구현 규칙입니다.

### 17-1. 검색 범위

검색은 **현재 보이는 카드 목록 안에서만** 동작합니다. 별도로 범위를 선택하거나 설정할 필요가 없습니다.

| 현재 탭 | 검색 대상 |
|---|---|
| C++ > 개념 정리 | C++ 카드 + 공통 카드 |
| 블루프린트 > 개념 정리 | 블루프린트 카드 + 공통 카드 |

탭을 전환하면 검색 범위도 자동으로 새 탭의 카드 목록으로 바뀝니다.

### 17-2. 검색 트리거

실시간 필터링(글자 입력마다 실행)을 사용하지 않습니다. 카드가 많아질수록 입력 한 글자마다 처리량이 커지기 때문입니다.

검색은 다음 두 가지 방법으로만 실행됩니다.

- `[검색]` 버튼 클릭
- 텍스트 필드에서 `Enter` 키 입력

### 17-3. 매칭 방식

검색어가 대상 텍스트에 **포함**되어 있으면 노출합니다. 전체 일치가 아닌 부분 일치입니다. 대소문자를 무시합니다.

```
"Generated"  검색 → "GENERATED_BODY()" 포함 카드 노출  ✅
"직렬"        검색 → "직렬화"           포함 카드 노출  ✅
"reflection" 검색 → keywords에 "Reflection" 있는 카드 노출  ✅
```

검색 대상 우선순위는 다음과 같습니다.

1. 카드 **제목** (title) — 포함 여부 확인
2. 카드 **keywords 배열** — 포함 여부 확인

둘 중 하나라도 매칭되면 카드가 노출됩니다. 두 가지 모두 미매칭이면 숨깁니다.

### 17-4. 버튼 상태와 색상

```
입력 전   [🔍 ________________] [검색 - 회색/비활성]
입력 후   [🔍 리플렉션_________] [검색 - 파란색/활성]
검색 후   [🔍 리플렉션_________] [검색 - 파란색] [검색 초기화 - 붉은색]  결과: 2개
```

| 버튼 | 활성 색 | 비활성 색 | 비고 |
|---|---|---|---|
| `[검색]` | `--accent-2` (파랑) | `--border` (회색, disabled) | 텍스트 필드가 비어 있으면 비활성 |
| `[검색 초기화]` | `--coral` (붉은색) | 노출 안 함 | 검색 실행 후에만 노출 |
| 결과 개수 | `--text-dim` (기본 텍스트 색) | 노출 안 함 | 검색 실행 후에만 노출. "결과: N개" |

### 17-5. 매칭 하이라이트

검색어와 일치하는 텍스트를 **카드 제목에만** 시각적으로 표시합니다. 본문에는 하이라이트를 적용하지 않습니다.

```
검색어: "리플렉션"
카드 제목: "GENERATED_BODY() — [리플렉션]과 직렬화"
                                ↑ 하이라이트 표시
```

하이라이트 스타일은 `--amber` 색상, 배경 또는 언더라인 방식을 사용합니다.

### 17-6. 검색어 유지 정책

| 상황 | 검색어 처리 |
|---|---|
| 탭 전환 (C++ → 블루프린트 등) | **초기화** — 새 탭의 카드 목록으로 새로 시작 |
| 페이지 새로고침 (F5) | **복원** — `sessionStorage`에 저장된 검색어 복원 |
| 브라우저 뒤로가기 후 같은 탭 | **복원** — `sessionStorage`에 저장된 검색어 복원 |

`sessionStorage` 저장 키: `ue-codex-search-query`

저장 시점: 검색 실행 시. 복원 시점: 페이지 로드 + 개념 정리 뷰 진입 시.

### 17-7. keywords 배열 작성 규칙

**모든 카드에 추가하지 않습니다.** 제목만으로는 찾기 어려운 개념을 본문에서 설명하는 카드에만 선택적으로 추가합니다.

추가 기준:
- 카드 본문에 제목에 없는 핵심 개념어가 등장하는 경우
- 한국어와 영어 두 가지 검색어가 필요한 경우

추가 형식:

```javascript
{
  id: 'c-15',
  cat: 'C++',
  title: "GENERATED_BODY() — 리플렉션과 직렬화",
  keywords: ['GENERATED_BODY()', '리플렉션', '직렬화', 'Reflection', 'Serialization'],
  body: `...`
}
```

keywords가 없는 카드는 필드 자체를 생략합니다 (검색 로직에서 없으면 제목만 검색합니다).

추가 예시:

| 카드 | keywords 추가 여부 | 이유 |
|---|---|---|
| `c-15` GENERATED_BODY() | ✅ 추가 | 본문에 리플렉션·직렬화 개념 설명 있음 |
| `c-07` 클래스 구조와 접근 지정자 | ❌ 생략 | 제목에 핵심 단어 포함, 추가 불필요 |
| `c-09` 포인터 * 와 참조 & | ❌ 생략 | 제목에 핵심 단어 포함 |

---

## 18. 핵심 규칙 요약 (업데이트)

1. **색은 변수만** — 직접 hex 사용 금지.
2. **폰트 두 가지만** — UI는 `--sans`, 코드·메타는 `--mono`.
3. **카드 반경** — 대형 카드 `14px`, 일반 카드 `12px`, 소형 요소 `8–10px`.
4. **테두리는 기본 → 호버 패턴** — 기본 `--border`, hover `--border-bright`, 강조 `--accent`.
5. **SVG filter 금지** — 와이어 글로우는 이중 선으로만.
6. **전환은 0.2s ease** — 특수한 경우(카드 접힘, 진행 바)만 별도 지정.
7. **여백 리듬** — 컴포넌트 간 `14px`, 섹션 간 `28–32px`, 카드 내부 패딩 `18–22px`.
8. **텍스트 계층** — 제목 `--text`, 본문 `--text-dim`, 메타 `--text-faint`.
9. **액센트 배분** — 그린(`--accent`)은 완료·브랜드, 블루(`--accent-2`)는 C++·코드·검색, 앰버(`--amber`)는 블루프린트·경고, 코럴(`--coral`)은 초기화·삭제.
10. **grid / flex** — 통계 카드는 `auto-fit minmax(180px, 1fr)`, 퀵 액션은 `minmax(220px, 1fr)`.
11. **코드 예문 주석은 요소 단위로** — 개념 카드에 코드가 있으면 키워드·연산자·이름 하나하나에 "이게 무엇인지 + 왜 있는지"를 주석으로 작성한다.
12. **검색은 버튼·Enter로만** — 실시간 필터링 금지. 검색 버튼은 파랑(`--accent-2`), 초기화 버튼은 코럴(`--coral`). keywords는 본문에 핵심 개념어가 있는 카드에만 선택적으로 추가한다.
