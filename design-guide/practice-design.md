# UE Archive — 실습 카드 작성 가이드 (Practice Design)

> 블루프린트·C++ 실습 카드의 데이터 구조와 작성 규칙.
> 실습 카드를 새로 추가하거나 수정할 때 이 파일을 읽으세요.
> 기존 예제(`blueprintExamples`, `cppExamples`)를 분석해 도출한 규칙입니다.

---

## 1. 데이터 배열 위치

```
index.html 내 JS 스크립트
├── blueprintExamples = [...] ← 블루프린트 실습 카드
└── cppExamples       = [...] ← C++ 실습 카드
```

---

## 2. 블루프린트 실습 카드 구조

```javascript
{
  id:         'bp-01',           // 'bp-' + 2자리 번호 (순서 유지)
  title:      '게임 시작할 때 화면에 메시지 출력하기',
  category:   '기초',            // 카테고리 필터 칩에 표시됨
  difficulty: 1,                 // 1 | 2 | 3  (★☆☆ | ★★☆ | ★★★)
  desc:       '...',             // HTML 문자열
  hint:       '...',             // 일반 문자열
  graph:      { nodes, connections },
  steps:      ['...', '...'],    // 문자열 배열
  explain:    '...'              // HTML 문자열
}
```

### 카드 렌더링 순서

```
[카드 헤더] 난이도 별 / 제목 / 카테고리 / 완료 체크
  ↓ (카드 열면)
[desc-block]   예제 설명 (desc)
[think-prompt] 💡 먼저 생각해보세요 (hint)
[정답 보기 버튼]
  ↓ (정답 보기 클릭 후)
[인터랙티브 노드 그래프] (graph 데이터로 BP 에디터 렌더링)
[단계별 따라하기]         (steps 배열 → <ol>)
[해설]                    (explain)
[학습 완료 버튼]
```

---

## 3. 블루프린트 graph 데이터 구조

```javascript
graph: {
  nodes: [
    {
      id:      'begin',             // 노드 고유 id (카드 내에서 유일)
      title:   'Event BeginPlay',   // 에디터에 표시될 이름
      type:    'event',             // 노드 유형 (아래 표 참조)
      x:       20,                  // 월드 좌표 X (노드 간 간격 250~300 권장)
      y:       30,                  // 월드 좌표 Y
      w:       175,                 // 노드 너비 (px)
      inputs:  [],                  // 입력 핀 배열
      outputs: [{ id, type, label?, value? }]  // 출력 핀 배열
    }
  ],
  connections: [
    { from: ['nodeId', 'pinId'], to: ['nodeId', 'pinId'] }
  ]
}
```

### 노드 유형 (type) — 헤더 색상 결정

| type | 헤더 색 | 사용 예 |
|---|---|---|
| `event` | 빨강 | Event BeginPlay, Space Bar, On Component Begin Overlap |
| `function` | 파랑 | Print String, Set Timer, Cast To |
| `variable` | 초록 | Health Get/Set 노드 |
| `pure` | 초록 | Pure function 노드 |
| `flow` | 회색 | Branch, Sequence, ForEach |

### 핀 타입 (pin.type) — 색상 결정

| type | 색 | 의미 |
|---|---|---|
| `exec` | 흰색 | 실행 흐름 (삼각형 모양) |
| `float` | 초록 | 실수 값 |
| `string` | 분홍 | 문자열 |
| `int` | 시안 | 정수 |
| `bool` | 어두운 빨강 | 불리언 |
| `vector` | 노랑 | 3D 벡터 |
| `object` | 파랑 | 오브젝트 참조 |
| `name` | 보라 | FName |

### 노드 배치 권장 규칙

- 이벤트 노드: `x: 20, y: 30` (가장 왼쪽)
- 연결된 다음 노드: 이전 노드 `x + 250~350`
- 여러 출력 흐름이 있을 때: `y` 차이 `60~80` 간격

---

## 4. 블루프린트 실습 카드 작성 규칙

### id
```
'bp-01', 'bp-02', ... 형식. 기존 최대 번호 + 1.
현재 마지막: bp-04
다음 추가 시: bp-05
```

### category
기존 카테고리 유지 권장. 새 카테고리 추가 시 일관성 있는 한국어 단어 사용.
```
현재 카테고리: 기초 | 입력 | 변수 | 충돌
```

### difficulty
```
1 = 단일 개념, 2개 이하 노드 연결
2 = 2개 이상 노드, 데이터 핀 활용
3 = 멀티 흐름, 캐스팅, 조건 분기 등 복합 구조
```

### desc (예제 설명)
- HTML 문자열. 2–3 문장.
- 핵심 노드 이름은 `<strong>`으로 감쌈.
- "무엇을 만드는지" + "어떤 노드/패턴을 배우는지" 포함.

```html
<!-- 예시 -->
'레벨이 시작될 때 화면에 메시지를 띄우는 가장 기본적인 패턴입니다.
<strong>BeginPlay</strong> 이벤트와 <strong>Print String</strong> 노드를 사용합니다.'
```

### hint (생각 유도)
- 일반 문자열 (HTML 없이).
- "어디서 찾는지"(에디터 위치) + "무엇을 주목해야 하는지" 포함.
- 유저가 직접 해보기 전에 읽는 힌트.

### steps (단계별 따라하기)
- 문자열 배열. 각 항목은 에디터에서 수행하는 한 단계.
- "어디서(위치) → 무엇을(동작)" 형태로 작성.
- 6–8개 이하 권장.

```javascript
steps: [
  '레벨 블루프린트 열기 (툴바 > Blueprints > Open Level Blueprint)',
  'Event Graph 빈 곳에 우클릭 → "Begin Play" 검색 후 추가',
  '...컴파일 후 플레이 — 화면 좌상단에 메시지 표시됨'
]
```

### explain (해설)
- HTML 문자열.
- 핵심 개념어: `<strong>`, 인라인 코드: `<code>`.
- "왜 이렇게 연결됐는지" + "응용 방향 또는 주의사항" 포함.

---

## 5. C++ 실습 카드 구조

```javascript
{
  id:         'cpp-01',          // 'cpp-' + 2자리 번호
  title:      '가장 기본적인 Actor 클래스 만들기',
  category:   '기초',
  difficulty: 1,
  desc:       '...',             // HTML
  hint:       '...',             // 일반 문자열
  files: [
    {
      name: 'MyActor.h',         // 파일 이름 (탭 레이블로 표시됨)
      type: 'cpp',               // 'cpp' | 'bp' (테두리 색 결정)
      code: '...'                // HTML (구문 강조 span 포함)
    }
  ],
  explain: '...'                 // HTML
}
```

### 카드 렌더링 순서

```
[카드 헤더] 난이도 / 제목 / 카테고리 / 완료 체크
  ↓ (카드 열면)
[desc-block]   예제 설명
[think-prompt] 💡 먼저 생각해보세요
[정답 보기 버튼]
  ↓ (정답 보기 후)
[파일 레이블]  MyActor.h
[pre 코드블록] .h 코드
[파일 레이블]  MyActor.cpp
[pre 코드블록] .cpp 코드
[해설]
[학습 완료 버튼]
```

---

## 6. C++ 실습 카드 작성 규칙

### id
```
'cpp-01', 'cpp-02', ... 형식.
현재 마지막: cpp-04
다음 추가 시: cpp-05
```

### category
```
현재 카테고리: 기초 | 변수 | 함수
```

### files 배열 순서
항상 `.h` 파일을 먼저, `.cpp` 파일을 다음에 배치합니다.
`.h`만 있어도 됩니다 (변경 부분만 보여줄 때).

### code (구문 강조)
HTML 문자열. 아래 span 클래스로 구문 강조를 직접 작성합니다.

| 클래스 | 색 | 대상 |
|---|---|---|
| `.cm` | `--text-faint` | 주석 (`// ...`) |
| `.kw` | `--accent-3` (퍼플) | 키워드 (`class`, `void`, `const`, `if`…) |
| `.mac` | `--amber` | UE 매크로 (`UCLASS`, `UPROPERTY`, `UFUNCTION`…) |
| `.str` | `--accent` (그린) | 문자열 리터럴 (`"Hello"`) |
| `.num` | `--coral` | 숫자 리터럴 |

```javascript
// 예시 코드 필드
code: `<span class="mac">UCLASS()</span>
<span class="kw">class</span> MYPROJECT_API AMyActor : <span class="kw">public</span> AActor
{
    <span class="mac">GENERATED_BODY()</span>
    ...
};`
```

### explain
- HTML 문자열.
- 핵심 포인트를 ①②③ 또는 산문으로 설명.
- `<code>`로 인라인 코드 강조.
- 블루프린트 대응 개념이 있으면 언급 (예: "블루프린트 예제 03의 C++ 버전").

---

## 7. 카드 추가 체크리스트

새 실습 카드를 추가할 때 확인합니다.

**블루프린트 예제 추가 시**
- [ ] id가 `bp-XX` 형식이고 기존과 중복되지 않음
- [ ] `category`가 기존 카테고리와 일치하거나 새 카테고리가 필요한 이유가 있음
- [ ] `graph.nodes`의 모든 핀 타입이 정의된 타입 중 하나
- [ ] `connections`의 from/to가 실제 존재하는 nodeId·pinId와 일치
- [ ] `steps`가 실제 언리얼 에디터 UI 위치와 동작 순서로 작성됨
- [ ] `explain`에 "왜 이렇게 연결됐는지"와 응용 방향 포함

**C++ 예제 추가 시**
- [ ] id가 `cpp-XX` 형식
- [ ] `files` 배열이 `.h` → `.cpp` 순서
- [ ] 코드 내 구문 강조 span 클래스 정확히 적용
- [ ] `explain`에 핵심 포인트 ①②③ 등 구조적으로 설명
- [ ] 블루프린트 대응 예제가 있으면 언급
