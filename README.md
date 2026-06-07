# UE Archive

언리얼 엔진(Blueprint · C++)을 체계적으로 학습하기 위한 단일 파일 웹 앱입니다.

---

## 특징

- **단일 파일**: `index.html` 하나에 HTML · CSS · JS가 모두 포함됩니다. 빌드 도구 없이 브라우저에서 바로 열거나 GitHub Pages로 서비스할 수 있습니다.
- **2단 탭 구조**: 블루프린트 / C++ 주제 탭 아래에 개념 정리 · 실습 · 퀴즈 모드가 있습니다.
- **인터랙티브 BP 에디터**: 블루프린트 노드를 드래그해 핀을 직접 연결하며 예제를 따라할 수 있습니다.
- **학습 진행률 저장**: 개념 카드·실습 카드 완료 여부를 `localStorage`에 저장합니다.
- **개념 검색**: 카드 제목 및 키워드 배열을 대상으로 부분 일치 검색을 지원합니다.

---

## 구조

```
index.html          단일 파일 (HTML + CSS + JS)
design-guide/       작업 규칙 가이드 문서
  guide-master.md       전체 색인 — 작업 유형별 가이드 선택 기준
  html-structure.md     DOM · CSS · JS 전체 구조
  design-guide-style.md 색상 변수 · 폰트 · 여백 · 배경 규칙
  design-guide-components.md  UI 컴포넌트 CSS 패턴
  design-guide-tap.md   탭 구조 정의 및 작동 규칙
  card-design.md        개념 카드 본문 · 코드 주석 작성 규칙
  practice-design.md    BP · C++ 실습 카드 데이터 구조
  quiz-design.md        퀴즈 UI · 데이터 구조
  search-design.md      검색 기능 동작 · keywords 배열 규칙
  dictionary.md         프로젝트 고유 용어 사전
```

---

## 학습 콘텐츠

| 탭 | 모드 | 내용 |
|---|---|---|
| 블루프린트 | 개념 정리 | 블루프린트 핵심 개념 카드 (접이식) |
| 블루프린트 | 실습 | 인터랙티브 노드 그래프 예제 |
| 블루프린트 | 퀴즈 | 객관식 / 코드 보고 맞히기 |
| C++ | 개념 정리 | C++ · UE 매크로 개념 카드 |
| C++ | 실습 | .h / .cpp 코드 예제 |
| C++ | 퀴즈 | 객관식 / 코드 보고 맞히기 |

---

## 디자인 규칙 (핵심)

- 모든 색은 CSS 변수(`var(--변수명)`)만 사용합니다. hex 직접 기입 금지.
- 폰트는 UI에 `Space Grotesk`, 코드·메타에 `JetBrains Mono` 두 가지만 사용합니다.
- 전환 애니메이션 기본값은 `0.2s ease`입니다.
- BP 에디터 와이어 글로우는 SVG filter 없이 이중 선(`wire-glow + wire`)으로만 표현합니다.
- 검색은 버튼 클릭 또는 Enter로만 실행합니다. 실시간 필터링 없음.
