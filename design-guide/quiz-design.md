# UE Codex — 퀴즈 가이드 (Quiz Design)

> 퀴즈 UI 컴포넌트 CSS 패턴 및 데이터 구조.
> 퀴즈를 추가하거나 UI를 수정할 때 이 파일을 읽으세요.
> 문제 작성 규칙은 추후 추가 예정입니다.

---

## 1. 데이터 구조

```javascript
// questions 배열 (index.html 내)
{
  cat:     '공통',             // '공통' | '블루프린트' | 'C++' — 탭별 필터링
  type:    '객관식',           // '객관식' | '코드 보고 맞히기'
  q:       '질문 텍스트',
  code:    '...',              // (선택) 코드 블록 HTML — type이 '코드 보고 맞히기'일 때
  options: ['A', 'B', 'C', 'D'],
  answer:  1,                  // 정답 인덱스 (0~3)
  explain: '해설 텍스트'
}
```

### cat별 노출 탭

| cat | 노출 탭 |
|---|---|
| `'공통'` | 블루프린트 퀴즈 + C++ 퀴즈 모두 |
| `'블루프린트'` | 블루프린트 퀴즈만 |
| `'C++'` | C++ 퀴즈만 |

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

---

## 3. 문제 작성 규칙

> ⚠️ 추후 작성 예정 — 유저가 방향을 정한 뒤 추가합니다.
