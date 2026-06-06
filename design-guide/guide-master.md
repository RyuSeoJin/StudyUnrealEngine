# UE Archive — 가이드 마스터 (Guide Master)

> Claude Code로 작업하기 전에 이 파일을 먼저 읽어 어떤 가이드 파일을 열지 결정하세요.

---

## 가이드 파일 목록

| 파일 | 역할 |
|---|---|
| `guide-master.md` | 이 파일. 전체 색인 및 파일 선택 기준 |
| `dictionary.md` | 프로젝트 고유 명사 사전. 작업 영역이 불분명하면 먼저 여기서 확인 |
| `html-structure.md` | index.html의 DOM·CSS·JS 전체 구조. 새 기능 추가 전 참조 |
| `design-guide-style.md` | 색상 변수·폰트·여백·배경·애니메이션 등 CSS 기반값 |
| `design-guide-components.md` | 버튼·카드·네비·BP 에디터 등 UI 컴포넌트 CSS 패턴 |
| `design-guide-tap.md` | 탭 구조 정의, 각 탭의 콘텐츠와 작동 규칙 |
| `card-design.md` | 개념 카드 본문 작성 규칙 |
| `practice-design.md` | 블루프린트·C++ 실습 카드 데이터 구조 및 작성 규칙 |
| `quiz-design.md` | 퀴즈 UI 및 데이터 구조 |
| `search-design.md` | 개념 검색 기능 동작·UI·keywords 배열 규칙 |

---

## 작업 유형 → 읽을 파일

### 작업 영역이 불분명할 때
1. `dictionary.md`에서 용어 확인
2. 없는 단어라면 유추 후 유저에게 재확인
3. 확인 완료 후 아래 기준으로 해당 가이드 파일 읽기

### 전체 구조 파악이 필요할 때
- 새 기능 추가 전 전체 흐름 확인 → `html-structure.md`
- 어느 탭·뷰에 어떤 데이터가 연결되는지 → `design-guide-tap.md`

### CSS / 스타일 수정
- 색상 변수, 폰트, 여백 변경 → `design-guide-style.md`
- 특정 컴포넌트 스타일 변경 → `design-guide-components.md`

### UI 컴포넌트 추가·수정
- 버튼·카드·탭·필터칩·코드블록 → `design-guide-components.md`
- 블루프린트 에디터 노드·핀·와이어 → `design-guide-components.md`
- **새 컴포넌트 추가 후 → `design-guide-components.md`에 패턴 기록**

### 탭 구조 수정
- 새 탭 추가, 탭 내 콘텐츠 변경 → `design-guide-tap.md`

### 개념 카드 작업
- 개념 카드 본문·코드 추가·수정 → `card-design.md`
- keywords 배열 추가 → `card-design.md` + `search-design.md`

### 실습 카드 작업
- 블루프린트 예제 추가·수정 → `practice-design.md`
- C++ 예제 추가·수정 → `practice-design.md`

### 퀴즈 작업
- 퀴즈 UI·데이터 구조 확인 → `quiz-design.md`

### 검색 기능 수정
- 검색 동작·버튼 상태·sessionStorage → `search-design.md`
- keywords 배열 규칙 → `search-design.md`
- 검색 바 CSS → `design-guide-components.md`

---

## 핵심 규칙 12가지 (빠른 참조)

1. **색은 변수만** — 직접 hex 사용 금지. 반드시 `var(--변수명)` 사용
2. **폰트 두 가지만** — UI는 `--sans`, 코드·메타는 `--mono`
3. **카드 반경** — 대형 카드 `14px`, 일반 카드 `12px`, 소형 요소 `8–10px`
4. **테두리 패턴** — 기본 `--border`, hover `--border-bright`, 강조 `--accent`
5. **SVG filter 금지** — 와이어 글로우는 이중 선(wire-glow + wire)으로만
6. **전환은 0.2s ease** — 카드 접힘·진행 바 등 특수한 경우만 별도 지정
7. **여백 리듬** — 컴포넌트 간 `14px`, 섹션 간 `28–32px`, 카드 내부 `18–22px`
8. **텍스트 계층** — 제목 `--text`, 본문 `--text-dim`, 메타 `--text-faint`
9. **액센트 배분** — 그린=완료·브랜드, 블루=C++·코드·검색, 앰버=BP·경고, 코럴=초기화·삭제
10. **grid / flex** — 통계 카드 `auto-fit minmax(180px,1fr)`, 퀵액션 `minmax(220px,1fr)`
11. **코드 예문 주석** — 키워드·연산자·이름 하나하나에 "무엇인지 + 왜 있는지" 명시
12. **검색은 버튼·Enter로만** — 실시간 필터링 금지. keywords는 본문에 핵심 개념어가 있는 카드에만
