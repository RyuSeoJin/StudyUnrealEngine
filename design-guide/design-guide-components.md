# UE Codex — 컴포넌트 가이드 (Design Guide: Components)

> UI 컴포넌트 CSS 패턴 — 네비게이션·카드·버튼·필터칩·BP 에디터·설명블록.  
> 새 컴포넌트를 만들거나 기존 것을 수정할 때 이 파일을 읽으세요.  
> **새 컴포넌트를 추가한 뒤에는 이 파일에 패턴을 기록합니다.**

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

