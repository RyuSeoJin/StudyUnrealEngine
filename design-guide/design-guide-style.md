# UE Archive — 스타일 가이드 (Design Guide: Style)

> CSS 기반값 가이드 — 색상 변수·폰트·여백·배경·애니메이션.  
> UI 스타일 값을 정하거나 수정할 때 이 파일을 읽으세요.

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
