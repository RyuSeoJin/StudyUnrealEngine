# UE Archive — 탭 구조 가이드 (Design Guide: Tab)

> 탭 구조 정의, 각 탭의 콘텐츠와 작동 규칙.
> 탭을 추가하거나 탭 내부 구조를 수정할 때 이 파일을 읽으세요.
> 탭 규칙이 바뀌면 이 파일을 업데이트합니다.

---

## 1. 전체 탭 구조

2단 계층으로 구성됩니다. **상위 탭(주제)** → **하위 탭(모드)**.

```
[대시보드]  [블루프린트]  [C++]         ← 상위 탭 (nav-tab)
                ↓
     [📚 개념 정리] [🛠 실습] [✏️ 퀴즈]   ← 하위 탭 (sub-tab, 대시보드엔 없음)
```

- 대시보드는 하위 탭 없음.
- 블루프린트/C++ 탭을 선택하면 하위 탭이 나타남.
- 기본 하위 탭은 `개념 정리`.

---

## 2. JS 상태 변수

```javascript
let _subject = 'dashboard';  // 'dashboard' | 'bp' | 'cpp'
let _mode    = 'concepts';   // 'concepts' | 'practice' | 'quiz'
```

뷰 전환 함수:
- `setSubject(s, m)` — 상위 탭 전환. 검색 상태 초기화 포함.
- `setMode(m)` — 하위 탭 전환.
- `applyNav()` — 상태에 따라 실제 뷰를 보여줌.

---

## 3. 상위 탭별 정의

### 대시보드

| 항목 | 내용 |
|---|---|
| HTML view | `#view-dashboard` |
| 하위 탭 | 없음 |
| 콘텐츠 | 진행률 통계 카드 4개 + 퀵 액션 카드 2개 |
| 퀵 액션 | [블루프린트 →] [C++ →] |

**통계 카드 4개:**
| 항목 | 데이터 출처 |
|---|---|
| BP 개념 | `concepts.filter(cat=공통·BP)` 완료 수 |
| BP 실습 | `blueprintExamples` 완료 수 |
| C++ 개념 | `concepts.filter(cat=공통·C++)` 완료 수 |
| C++ 실습 | `cppExamples` 완료 수 |

### 블루프린트 탭 (`_subject = 'bp'`)

| 하위 탭 | HTML view | 데이터 | 렌더 함수 |
|---|---|---|---|
| 개념 정리 | `#view-concepts` | `concepts` (cat: 공통·블루프린트) | `renderConcepts()` |
| 실습 | `#view-blueprint` | `blueprintExamples` | `renderExamples(...)` |
| 퀴즈 | `#view-quiz` | `questions` (cat: 공통·블루프린트) | `initQuiz()` |

### C++ 탭 (`_subject = 'cpp'`)

| 하위 탭 | HTML view | 데이터 | 렌더 함수 |
|---|---|---|---|
| 개념 정리 | `#view-concepts` | `concepts` (cat: 공통·C++) | `renderConcepts()` |
| 실습 | `#view-cpp` | `cppExamples` | `renderExamples(...)` |
| 퀴즈 | `#view-quiz` | `questions` (cat: 공통·C++) | `initQuiz()` |

---

## 4. 카테고리 매핑 (`subjectCats`)

```javascript
const subjectCats = {
  bp:  ['공통', '블루프린트'],
  cpp: ['공통', 'C++']
};
```

개념 카드와 퀴즈는 이 매핑으로 현재 탭에 맞는 항목만 필터링합니다.

---

## 5. 탭 전환 시 동작 규칙

| 상황 | 동작 |
|---|---|
| 상위 탭 전환 | 개념 검색어 초기화 (`clearConceptSearch()` 호출) |
| 하위 탭 → 개념 정리 | `_conceptCat` 갱신 + 검색 이벤트 바인딩 + 세션 검색어 복원 |
| 하위 탭 → 실습 | 해당 subject의 `view-blueprint` 또는 `view-cpp` 활성 |
| 하위 탭 → 퀴즈 | `initQuiz()` 호출 — 현재 subject의 문제 필터링 |

---

## 6. 새 탭을 추가할 때 체크리스트

새 상위 탭(예: "Shader")을 추가해야 한다면 아래 항목을 모두 수행합니다.

- [ ] HTML에 `<button class="nav-tab" data-subject="shader">` 추가
- [ ] `subjectCats` 객체에 `shader: ['공통', 'Shader']` 추가
- [ ] `subjectLabel` 객체에 `shader: 'Shader'` 추가
- [ ] 해당 subject의 데이터 배열(`shaderExamples` 등) 생성
- [ ] `applyNav()` 분기에 해당 subject 처리 추가
- [ ] 이 파일(design-guide-tap.md)에 새 탭 정의 기록
- [ ] `dictionary.md`에 새 탭 관련 용어 추가

새 하위 탭(예: "프로젝트")을 추가해야 한다면:

- [ ] HTML에 `<button class="sub-tab" data-mode="project">` 추가
- [ ] `applyNav()` 분기에 `_mode === 'project'` 케이스 추가
- [ ] 해당 모드의 view HTML 추가 (`#view-project`)
- [ ] 렌더 함수 작성
- [ ] 이 파일에 새 하위 탭 정의 기록

---

## 7. 진행률 저장 구조

```javascript
const progress = {
  concepts:   [],   // 완료한 개념 카드 id 배열
  blueprints: [],   // 완료한 BP 실습 카드 id 배열
  cpp:        [],   // 완료한 C++ 실습 카드 id 배열
  lastQuiz:   null  // { score, total, date }
};
// localStorage key: 'ue-codex-progress-v2'
```

---

## 8. 작업물 탭 (`_subject = 'works'`) — 2025-06 추가

### 구조 개요
- 상위 탭: 작업물 (`data-subject="works"`)
- 하위 탭: 별도 `#works-sub-nav` 사용 (기존 `#sub-nav`와 독립)
  - `data-works-mode="bp"` → 블루프린트 작업물
  - `data-works-mode="cpp"` → C++ 작업물
- 상태 변수: `let _worksMode = 'bp';`

### 데이터 배열
```javascript
const worksBpItems  = [];  // 블루프린트 작업물
const worksCppItems = [];  // C++ 작업물
```

작업물 항목 구조:
```javascript
{
  id:    'w-bp-01',       // 'w-bp-' 또는 'w-cpp-' + 2자리 번호
  title: '작업물 제목',
  desc:  '...',           // HTML 문자열
}
```

### 뷰 HTML id
| 하위 탭 | HTML id |
|---|---|
| 블루프린트 | `#view-works-bp` |
| C++ | `#view-works-cpp` |

### 관련 함수
```javascript
renderWorks()           // _worksMode에 맞는 목록 렌더 (빈 배열이면 empty state 표시)
toggleWorkCard(id)      // 카드 열기/닫기
setWorksMode(m)         // _worksMode 변경 + applyNav()
```

### applyNav() 처리 방식
- `_subject === 'works'`일 때 기존 `#sub-nav`는 숨기고 `#works-sub-nav`를 표시
- `_worksMode` 기준으로 `view-works-bp` 또는 `view-works-cpp` 활성화

### 카드 CSS 클래스
`.work-card` / `.work-card-head` / `.work-card-title` / `.work-card-arrow` / `.work-card-body`  
빈 상태: `.works-empty` / `.works-empty-icon`
