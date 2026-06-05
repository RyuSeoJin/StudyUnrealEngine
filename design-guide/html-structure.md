# UE Codex — HTML 전체 구조 (HTML Structure)

> index.html 한 파일 안에 HTML·CSS·JS가 모두 포함된 단일 파일 아키텍처입니다.
> 새 기능을 추가하거나 구조를 수정할 때 이 파일을 먼저 읽으세요.

---

## 1. 파일 개요

```
index.html
├── <head>          Google Fonts 로드
├── <style>         모든 CSS (인라인)
├── <body>          HTML 마크업
└── <script>        모든 JS (인라인)
```

단일 파일로 관리하며 빌드 도구 없이 GitHub Pages에서 바로 서비스됩니다.

---

## 2. HTML DOM 구조

```
<body>
├── <header class="header">
│   ├── .brand-wrap          브랜드명 + 학습 진행률 바
│   ├── .nav-tabs            상위 탭 버튼 (대시보드 / 블루프린트 / C++)
│   └── #sub-nav             하위 탭 버튼 (개념 정리 / 실습 / 퀴즈, 대시보드엔 숨김)
│
└── <main class="main">
    └── .wrap                최대 너비 1000px 컨테이너
        ├── #view-dashboard  대시보드 뷰
        │   ├── .welcome-card
        │   ├── #stats-grid  진행률 통계 카드 4개
        │   └── .quick-actions  퀵 액션 카드 2개
        │
        ├── #view-concepts   개념 정리 뷰
        │   ├── #concepts-title
        │   ├── #concept-search    검색 바
        │   └── #cards             개념 카드 목록
        │
        ├── #view-blueprint  블루프린트 실습 뷰
        │   ├── #bp-filter   카테고리 필터 칩
        │   └── #bp-list     BP 예제 카드 목록
        │
        ├── #view-cpp        C++ 실습 뷰
        │   ├── #cpp-filter  카테고리 필터 칩
        │   └── #cpp-list    C++ 예제 카드 목록
        │
        └── #view-quiz       퀴즈 뷰
            ├── #quiz-title
            ├── #quiz-intro  퀴즈 시작 화면
            ├── #quiz-running 문제 풀기 화면
            └── #quiz-result 결과 화면
```

---

## 3. CSS 구조 (위→아래 순서)

```
<style>
├── :root                  CSS 변수 정의 (색상·폰트·기타)
├── *, body                리셋 + 기본 스타일
│
├── /* Header */           헤더·브랜드·진행률 바
├── /* Nav tabs */         상위 탭 (nav-tab)
├── /* 하위 네비 */        하위 탭 (sub-tab)
│
├── /* Dashboard */        대시보드 컴포넌트 (welcome, stats, quick-actions)
│
├── /* Concept cards */    개념 카드 (card, card-head, card-body, card-tag)
├── /* 코드 토글 */        주석포함/코드만 전환 탭 (code-toggle)
├── /* 개념 검색 바 */     검색 입력·버튼·결과수 (concept-search)
├── /* 개념 비교 박스 */   concept-compare, concept-flow
│
├── /* Blueprint SVG */    BP 에디터 캔버스 (bp-stage, bp-world)
├── /* Interactive BP Ed */BP 에디터 노드·핀·헤더 (bp-node, bp-pin)
├── /* Example cards */    실습 카드 (example-card, example-head)
├── /* category filter */  카테고리 필터 칩 (cat-filter, cat-chip)
│
└── /* Quiz */             퀴즈 UI (quiz-intro, opt, explain, score-ring)
</style>
```

---

## 4. JS 구조 (위→아래 순서)

### 4-1. 데이터 배열 (콘텐츠)

| 변수 | 설명 | 위치(줄) |
|---|---|---|
| `concepts` | 개념 카드 데이터 배열 | ~1126 |
| `blueprintExamples` | 블루프린트 실습 카드 배열 | ~1788 |
| `cppExamples` | C++ 실습 카드 배열 | ~1913 |
| `questions` | 퀴즈 문제 배열 | ~2068 |

### 4-2. 진행률 저장 (localStorage)

```javascript
const STORAGE_KEY = 'ue-codex-progress-v2';
let progress = loadProgress();  // { concepts:[], blueprints:[], cpp:[], lastQuiz:null }

function loadProgress()   // localStorage에서 불러오기
function saveProgress()   // localStorage에 저장
function isDone(cat, id)  // 완료 여부 확인
function toggleDone(cat, id) // 완료 상태 토글
```

### 4-3. 2단 네비게이션

```javascript
let _subject = 'dashboard'; // 'dashboard' | 'bp' | 'cpp'
let _mode    = 'concepts';  // 'concepts' | 'practice' | 'quiz'

const subjectCats  = { bp: ['공통','블루프린트'], cpp: ['공통','C++'] }
const subjectLabel = { bp: '블루프린트', cpp: 'C++' }

function applyNav()          // 상태 → 실제 뷰 전환
function setSubject(s, m)    // 상위 탭 전환 (검색 초기화 포함)
function setMode(m)          // 하위 탭 전환
```

### 4-4. 대시보드

```javascript
function renderStats()  // 진행률 통계 카드 렌더링
```

### 4-5. 개념 정리 (Concepts)

```javascript
// 상태 변수
let _conceptCat   = 'bp';   // 현재 subject
let _searchQuery  = '';      // 마지막 실행 검색어
let _searchActive = false;   // 검색 결과 표시 중 여부
const SEARCH_KEY  = 'ue-codex-search-query';  // sessionStorage 키

// 코드 토글 헬퍼
function stripComments(html)  // .cm 스팬 제거 → 코드만 버전
function wrapWithToggle(body) // pre 블록을 토글 UI로 감쌈

// 렌더링
function renderConcepts()     // 카드 목록 렌더 (필터 + 검색 + 하이라이트)

// 검색
function _escapeRegex(s)
function _highlightTitle(title, query)
function _syncSearchUI(count)
function executeSearch()      // 검색 실행 (버튼·Enter)
function resetSearch()        // 검색 초기화 버튼
function clearConceptSearch() // 탭 전환 시 초기화
function initConceptSearchEvents() // 검색 바 이벤트 바인딩 (중복 방지)

// 이벤트 위임
document.addEventListener('click', ...)  // .code-tab 클릭 처리
```

### 4-6. 블루프린트 에디터 (BlueprintEditor 클래스)

```javascript
const PIN_COLORS = { exec, float, string, ... }
const WORLD_W = 4000, WORLD_H = 3000
const ZOOM_MIN = 0.4, ZOOM_MAX = 2.6
let _bpEditorSeq = 0  // 인스턴스 고유 uid용 카운터

class BlueprintEditor {
  constructor(host, graph)   // 에디터 초기화 + 빌드
  build()                    // DOM 구조 생성 + 이벤트 바인딩
  makePin(node, pin, side)   // 핀 요소 생성
  pinAtPoint(x, y)           // 좌표 아래의 핀 요소 탐색
  findSnapPin(cx, cy)        // 가장 가까운 호환 핀 (52px 반경)
  renderNodes()              // 노드 DOM 생성
  pinWorld(node, pinEl)      // 핀의 월드 좌표 계산
  startLink(e, ...)          // 핀 드래그 시작
  updateLiveConnection(snap) // 드래그 중 라이브 연결 커밋
  onMove(e)                  // pointermove 처리
  onUp(e)                    // pointerup 처리 (연결 확정)
  onStageDown(e)             // 스테이지 클릭 (패닝·마퀴)
  onWheel(e)                 // 줌
  redraw()                   // 와이어 SVG 재렌더
  tryConnect(src, dst)       // 연결 유효성 검사 + graph 업데이트
  reset()                    // 그래프 초기화
  resetView()                // 뷰포트 초기화
}
```

### 4-7. 실습 예제 렌더링

```javascript
function renderExamples(listName, listEl, filterEl, exampleData, progressKey)
// 블루프린트·C++ 예제 카드 목록 렌더. 카테고리 필터·완료 상태·정답 보기 포함.
```

### 4-8. 퀴즈

```javascript
let quizSet = []   // 현재 subject로 필터링된 문제 배열
let current = 0    // 현재 문제 인덱스
let answers = []   // 사용자 답변 배열

function initQuiz()         // quizSet 설정 + 인트로 화면
function renderQuestion()   // 문제 렌더링
function showResult()       // 결과 화면 렌더링
```

### 4-9. 초기화 (INIT)

```javascript
// 페이지 로드 시 순서대로 실행
renderStats();
renderExamples('blueprint', ...);
renderExamples('cpp', ...);
applyNav();   // 초기 뷰 표시 (대시보드)
```

---

## 5. 데이터 흐름

```
사용자 탭 클릭
  → setSubject('cpp') / setMode('concepts')
  → applyNav()
      → _conceptCat = 'cpp'
      → initConceptSearchEvents()  (검색 바인딩)
      → sessionStorage 복원 (검색어)
      → renderConcepts()
          → concepts.filter(cat ∈ subjectCats['cpp'])
          → _searchActive ? 검색 필터 추가 : 전체
          → wrapWithToggle(c.body)  (코드 토글 적용)
          → innerHTML 업데이트
          → 카드 이벤트 바인딩
          → _syncSearchUI()
```

---

## 6. 파일 수정 시 주의사항

| 작업 | 주의점 |
|---|---|
| 새 CSS 추가 | `:root` 변수를 사용. hex 직접 기입 금지 |
| 새 JS 섹션 추가 | 기존 블록 구조 유지. 데이터 → 상태 → 함수 → 이벤트 순서 |
| concepts 배열에 카드 추가 | `cat`, `tag`, `tagClass` 필드 필수. 순서는 의존성 기준으로 유지 |
| BlueprintEditor 수정 | SVG filter 사용 금지. 와이어 글로우는 `wire-glow + wire` 이중 선으로만 |
| 탭 추가 | `subjectCats`, `subjectLabel`, `applyNav()` 분기 모두 수정 필요 |
| 렌더 함수 추가 | INIT 블록 맨 끝에 호출 추가 |
