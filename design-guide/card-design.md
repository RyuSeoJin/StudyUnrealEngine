# UE Archive — 개념 카드 작성 가이드 (Card Design)

> 개념 카드 본문 작성 규칙.  
> 개념 카드를 새로 추가하거나 수정할 때 이 파일을 읽으세요.

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


---

## 검색 keywords 배열

개념 카드에 선택적으로 추가합니다. 제목만으로는 찾기 어려운 개념어가 본문에 있을 때만 추가합니다.

```javascript
{
  id: 'c-15',
  title: "GENERATED_BODY() — 리플렉션과 직렬화",
  keywords: ['GENERATED_BODY()', '리플렉션', '직렬화', 'Reflection', 'Serialization'],
  ...
}
```

- keywords가 없는 카드는 필드 자체를 생략합니다.
- 한국어·영어 모두 포함해 어느 쪽으로 검색해도 찾을 수 있게 합니다.
- 추가 기준: 제목에 없는 핵심 개념어가 본문에서 설명될 때.
