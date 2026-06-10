# RecordTales UE — 집중·아카이브 시스템 데이터 설계

> 집중(타이머/스톱워치) 시스템과 아카이브 시스템의 C++ 데이터 구조 설계서.
> 코딩 전 구조체·열거형·관계를 확정하기 위한 문서입니다.
> project-wiki (focus-panel, archive-panel, quests-panel, record-modal) 전체 반영.

---

## 목차

1. [설계 원칙](#1-설계-원칙)
2. [런타임 vs 영속 데이터 구분](#2-런타임-vs-영속-데이터-구분)
3. [열거형 (Enum)](#3-열거형-enum)
4. [구조체 (Struct)](#4-구조체-struct)
5. [카테고리 관리](#5-카테고리-관리)
6. [시스템 관계도](#6-시스템-관계도)
7. [UE 배치 계획](#7-ue-배치-계획)
8. [RecordId 생성](#8-recordid-생성)
9. [아카이브 쿼리 흐름](#9-아카이브-쿼리-흐름)
10. [열린 질문](#10-열린-질문)

---

## 1. 설계 원칙

| 원칙 | 내용 |
|---|---|
| **런타임 / 영속 분리** | 세션이 진행 중일 때만 필요한 데이터(남은 시간, 사이클 번호)는 저장하지 않는다 |
| **기록은 종료 시 1회 생성** | 세션이 끝날 때 `FSessionRecord` 한 개를 만들어 아카이브에 추가한다 |
| **Blueprint 노출 최소화** | UPROPERTY는 UI에 필요한 것만 BlueprintReadWrite, 내부 계산용은 노출하지 않는다 |
| **날짜 키 통일** | 날짜 식별자는 `"YYYYMMDD"` 8자리 문자열로 통일한다 |
| **쿼리 결과는 비영속** | 아카이브 조회 결과(`FArchiveQueryResult`, `FDayActivityDots` 맵)는 매번 계산한다. 저장하지 않는다 |
| **퀘스트-아카이브 단방향 참조** | 아카이브는 퀘스트 인스턴스를 읽기만 한다. 수정하지 않는다 |

---

## 2. 런타임 vs 영속 데이터 구분

```
집중 세션 진행 중 (런타임 — 저장 안 함)
├── FTimerRuntimeState   타이머 현재 단계·남은 시간·사이클
├── FStopwatchState      스톱워치 가동 여부·경과 시간
└── FTimerConfig (복사본) 시작 시 복사된 설정값

세션 종료 시 생성 (영속 — 저장)
└── FSessionRecord       기록 1건 → UArchiveSubsystem.AllRecords에 추가

아카이브 조회 시 즉석 생성 (비영속 — 저장 안 함)
├── FArchiveDateRange    날짜 범위 {Start, End}
├── FDayActivityDots     날짜별 활동 도트 {bHasTimer, bHasStopwatch, bHasQuest}
├── FArchiveFilterState  현재 필터 상태
└── FArchiveQueryResult  필터링된 세션 목록 + 퀘스트 5개 버킷
```

---

## 3. 열거형 (Enum)

### 3.1. EFocusMode — 집중 패널 모드

```cpp
UENUM(BlueprintType)
enum class EFocusMode : uint8
{
    Timer      UMETA(DisplayName = "타이머"),
    Stopwatch  UMETA(DisplayName = "스톱워치")
};
```

### 3.2. ETimerPhase — 타이머 단계

```cpp
UENUM(BlueprintType)
enum class ETimerPhase : uint8
{
    Idle        UMETA(DisplayName = "대기"),
    Focusing    UMETA(DisplayName = "집중"),
    Resting     UMETA(DisplayName = "휴식"),
    LastResting UMETA(DisplayName = "마지막 휴식"),
    Paused      UMETA(DisplayName = "일시정지")
};
```

> **상태 전이 규칙**
> ```
> Idle ──────────────→ Focusing    (타이머 시작)
> Focusing → Resting               (루틴 + 남은 사이클 있음)
> Focusing → LastResting           (루틴 + 마지막 사이클)
> Focusing → Idle                  (비루틴 완료 or 중도 정지)
> Resting  → Focusing              (휴식 완료, 다음 사이클)
> LastResting → Idle               (autoRestart=false)
> LastResting → Focusing           (autoRestart=true, CycleIndex=1 리셋)
> any      ←→ Paused               (일시정지 / 재개, PrevPhase로 복원)
> ```

### 3.3. ESessionEndReason — 세션 종료 이유

```cpp
UENUM(BlueprintType)
enum class ESessionEndReason : uint8
{
    NormalComplete  UMETA(DisplayName = "정상 완료"),  // 사이클 전부 마침
    EarlyStop       UMETA(DisplayName = "중도 정지"),  // 사용자가 직접 종료
};
```

### 3.4. EArchPeriodFilter — 아카이브 기간 필터

```cpp
UENUM(BlueprintType)
enum class EArchPeriodFilter : uint8
{
    Daily    UMETA(DisplayName = "일간"),   // 선택일 당일만
    Weekly   UMETA(DisplayName = "주간"),   // 선택일 포함 주의 월요일~일요일
    Monthly  UMETA(DisplayName = "월간"),   // 선택일 속한 달 전체
};
```

### 3.5. EArchModeFilter — 아카이브 모드 필터

```cpp
UENUM(BlueprintType)
enum class EArchModeFilter : uint8
{
    All        UMETA(DisplayName = "전체"),
    TimerOnly  UMETA(DisplayName = "타이머만"),
    SwOnly     UMETA(DisplayName = "스톱워치만"),
};
```

### 3.6. EQuestState — 퀘스트 인스턴스 상태

```cpp
UENUM(BlueprintType)
enum class EQuestState : uint8
{
    Active     UMETA(DisplayName = "진행 중"),   // 기한 내, 미완료
    Done       UMETA(DisplayName = "완료"),
    Overdue    UMETA(DisplayName = "기한 초과"), // 기한 지남, 미완료, 비중단
    Suspended  UMETA(DisplayName = "중단"),
    Upcoming   UMETA(DisplayName = "진행 예정"), // 기한이 미래
};
```

---

## 4. 구조체 (Struct)

### 4.1. FTimerConfig — 타이머 설정값 (영속)

```cpp
USTRUCT(BlueprintType)
struct FTimerConfig
{
    GENERATED_BODY()

    // 집중 시간 (분). 기본 25. 범위 1~180
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 FocusDuration = 25;

    // 일반 휴식 시간 (분). 기본 5. RepeatCount > 1일 때 사용
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 RestDuration = 5;

    // 마지막 휴식 시간 (분). 기본 15. RepeatCount > 1일 때 마지막 사이클 후
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 LastRestDuration = 15;

    // 단일 휴식 시간 (분). 기본 5. RepeatCount == 1일 때 사용
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 SingleRestDuration = 5;

    // 반복 횟수. 기본 4. 범위 1~10
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 RepeatCount = 4;

    // 루틴 모드 여부 (false면 비루틴 — 집중 한 번만)
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    bool bIsRoutine = false;

    // 루틴 완료 후 자동 재시작 여부
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    bool bAutoRestart = false;
};
```

> **RepeatCount == 1 특수 처리**  
> - 휴식 필드는 `SingleRestDuration` 하나만 사용  
> - `LastResting` 단계이지만 UI 라벨은 "휴식"으로 표시

---

### 4.2. FTimerRuntimeState — 타이머 런타임 상태 (비영속)

```cpp
USTRUCT(BlueprintType)
struct FTimerRuntimeState
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly)
    ETimerPhase CurrentPhase = ETimerPhase::Idle;

    // 일시정지 전 단계 — 재개 시 복원
    UPROPERTY(BlueprintReadOnly)
    ETimerPhase PrevPhase = ETimerPhase::Idle;

    // 현재 단계 남은 시간 (초)
    UPROPERTY(BlueprintReadOnly)
    int32 RemainingSeconds = 0;

    // 현재 단계 총 시간 (초) — 링 진행률: (Total - Remaining) / Total
    UPROPERTY(BlueprintReadOnly)
    int32 TotalSeconds = 0;

    // 현재 사이클 번호 (1-based)
    UPROPERTY(BlueprintReadOnly)
    int32 CycleIndex = 1;

    // Focusing 단계에서만 누적되는 집중 시간 (초). 휴식 포함 안 함
    UPROPERTY(BlueprintReadOnly)
    int32 AccFocusSeconds = 0;

    // 세션 시작 시 Config 복사 — 진행 중 사용자가 설정 바꿔도 영향 없음
    UPROPERTY()
    FTimerConfig SessionConfig;
};
```

---

### 4.3. FStopwatchState — 스톱워치 런타임 상태 (비영속)

```cpp
USTRUCT(BlueprintType)
struct FStopwatchState
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly)
    bool bRunning = false;

    UPROPERTY(BlueprintReadOnly)
    bool bPaused = false;

    // 경과 시간 (초). 시작 시 0으로 초기화
    UPROPERTY(BlueprintReadOnly)
    int32 ElapsedSeconds = 0;
};
```

---

### 4.4. FSessionRecord — 집중 세션 기록 (영속)

세션이 종료되는 순간 딱 1회 생성. 아카이브에 추가된다.

```cpp
USTRUCT(BlueprintType)
struct FSessionRecord
{
    GENERATED_BODY()

    // 고유 ID — "YYYYMMDD" + 5자리 시퀀스 (예: "2026060800001")
    UPROPERTY(BlueprintReadOnly)
    FString RecordId;

    // 타이머 / 스톱워치 구분
    UPROPERTY(BlueprintReadWrite)
    EFocusMode SessionType = EFocusMode::Timer;

    // 카테고리 (공부, 프로젝트, 업무, 독서, 운동, 기타)
    UPROPERTY(BlueprintReadWrite)
    FString Category;

    // 세션 총 경과 시간 (초)
    // 타이머: 시작~종료 경과 / 스톱워치: ElapsedSeconds
    UPROPERTY(BlueprintReadWrite)
    int32 DurationSeconds = 0;

    // 순수 집중 시간 (초)
    // 타이머: Focusing 단계만 누적 / 스톱워치: DurationSeconds와 동일
    UPROPERTY(BlueprintReadWrite)
    int32 FocusSeconds = 0;

    // 정상 완료 여부 (false = 중도 정지)
    UPROPERTY(BlueprintReadWrite)
    bool bNormalEnd = false;

    // 메모 (Record Modal에서 사용자 입력, 빈 문자열 가능)
    UPROPERTY(BlueprintReadWrite)
    FString Notes;

    // 기록 시각 (세션 종료 시점)
    UPROPERTY(BlueprintReadWrite)
    FDateTime RecordedAt;

    // 날짜 키 — "YYYYMMDD" 8자리. 날짜별 조회에 사용
    UPROPERTY(BlueprintReadWrite)
    FString DateKey;
};
```

> **DurationSeconds vs FocusSeconds**
>
> | 항목 | DurationSeconds | FocusSeconds |
> |---|---|---|
> | 표시 용도 | "총 N분" 통계 | XP 계산 (10G / 6초) |
> | 타이머 루틴 (25분×4회+휴식) | 집중100분 + 휴식30분 = 130분 | 집중만 100분 |
> | 스톱워치 | 동일 | 동일 |

---

### 4.5. FQuestInstance — 퀘스트 인스턴스 (영속)

> 퀘스트 시스템의 핵심 단위. 아카이브는 이 구조체를 읽기만 한다.

```cpp
USTRUCT(BlueprintType)
struct FQuestInstance
{
    GENERATED_BODY()

    // 인스턴스 고유 ID ("inst_" + 타임스탬프 + 시퀀스)
    UPROPERTY(BlueprintReadWrite)
    FString InstanceId;

    // 부모 마스터 퀘스트 ID (반복 퀘스트 참조)
    UPROPERTY(BlueprintReadWrite)
    FString MasterId;

    // 퀘스트 제목 (마스터에서 복사)
    UPROPERTY(BlueprintReadWrite)
    FString Title;

    // 카테고리
    UPROPERTY(BlueprintReadWrite)
    FString Category;

    // 기한 날짜 키 — "YYYYMMDD". 날짜 없으면 빈 문자열
    UPROPERTY(BlueprintReadWrite)
    FString DueDateKey;

    // 현재 상태
    UPROPERTY(BlueprintReadWrite)
    EQuestState State = EQuestState::Active;

    // 완료 시각 (State == Done일 때만 유효)
    UPROPERTY(BlueprintReadWrite)
    FDateTime CompletedAt;

    // 완료된 날짜 키 — "YYYYMMDD". 아카이브 done 버킷 분류에 사용
    UPROPERTY(BlueprintReadWrite)
    FString CompletedDateKey;

    // 반복 타입 ("none" | "daily" | "weekly" | "monthly")
    UPROPERTY(BlueprintReadWrite)
    FString RecurrenceType = TEXT("none");

    // 획득 골드 (완료 시 지급)
    UPROPERTY(BlueprintReadWrite)
    int32 GoldReward = 0;

    // XP 보상
    UPROPERTY(BlueprintReadWrite)
    int32 XpReward = 0;
};
```

> **아카이브 분류 판단 기준**
>
> | 버킷 | 조건 |
> |---|---|
> | Done | `State == Done` → `CompletedDateKey`가 날짜 범위에 포함 |
> | Active | `State == Active` && `DueDateKey` ≤ 오늘 |
> | Overdue | `State == Overdue` — 기한 지남, 미완료, 비중단 |
> | Suspended | `State == Suspended` |
> | Upcoming | `DueDateKey` > 오늘 (반복 퀘스트는 마스터ID 기준 중복 제거) |

---

### 4.6. FArchiveDateRange — 날짜 범위 (비영속)

```cpp
USTRUCT(BlueprintType)
struct FArchiveDateRange
{
    GENERATED_BODY()

    // 범위 시작 날짜 키 — "YYYYMMDD"
    UPROPERTY(BlueprintReadOnly)
    FString Start;

    // 범위 종료 날짜 키 — "YYYYMMDD"
    UPROPERTY(BlueprintReadOnly)
    FString End;
};
```

> **기간 필터별 계산 규칙**
>
> | EArchPeriodFilter | Start | End |
> |---|---|---|
> | `Daily` | AnchorDateKey | AnchorDateKey |
> | `Weekly` | AnchorDateKey 포함 주의 월요일 | 해당 주 일요일 |
> | `Monthly` | 해당 달 1일 ("YYYYMM01") | 해당 달 말일 |
>
> 주간 계산 주의: 일요일(dow=0)이면 6일 전 월요일로 조정.

---

### 4.7. FDayActivityDots — 날짜별 활동 도트 데이터 (비영속)

캘린더 그리드의 각 날짜에 표시할 점(dot). `TMap<FString, FDayActivityDots>` 형태로 사용.

```cpp
USTRUCT(BlueprintType)
struct FDayActivityDots
{
    GENERATED_BODY()

    // 타이머(Focus) 기록 있음 — 오렌지 점
    UPROPERTY(BlueprintReadOnly)
    bool bHasTimer = false;

    // 스톱워치 기록 있음 — 그린 점
    UPROPERTY(BlueprintReadOnly)
    bool bHasStopwatch = false;

    // 퀘스트(완료 또는 기한) 있음 — 옐로 점
    UPROPERTY(BlueprintReadOnly)
    bool bHasQuest = false;
};
```

> **dot 표시 규칙**
>
> | 점 색상 | 조건 |
> |---|---|
> | 오렌지 (`.ddot.f`) | 해당 날짜에 `SessionType == Timer`인 FSessionRecord 1건 이상 |
> | 그린 (`.ddot.s`) | 해당 날짜에 `SessionType == Stopwatch`인 FSessionRecord 1건 이상 |
> | 옐로 (`.ddot.t`) | 해당 날짜에 `DueDateKey` 또는 `CompletedDateKey`가 일치하는 FQuestInstance 1건 이상 |
>
> 모드 필터가 켜져 있어도 점은 필터 무관하게 실제 기록 기준으로 표시한다 (원본 RecordTales 동작 기준).

---

### 4.8. FArchiveFilterState — 아카이브 필터 상태 (비영속)

아카이브 패널이 활성화된 동안 `UArchiveSubsystem`이 보관하는 현재 필터 값.

```cpp
USTRUCT(BlueprintType)
struct FArchiveFilterState
{
    GENERATED_BODY()

    // 기간 필터
    UPROPERTY(BlueprintReadWrite)
    EArchPeriodFilter PeriodFilter = EArchPeriodFilter::Daily;

    // 모드 필터
    UPROPERTY(BlueprintReadWrite)
    EArchModeFilter ModeFilter = EArchModeFilter::All;

    // 카테고리 필터. 빈 문자열 = 전체
    UPROPERTY(BlueprintReadWrite)
    FString CategoryFilter;

    // 사용자가 클릭한 기준 날짜 — "YYYYMMDD"
    // 기간 범위 계산의 anchor로 사용
    UPROPERTY(BlueprintReadWrite)
    FString AnchorDateKey;

    // 캘린더 표시 연도 (UI 상태)
    UPROPERTY(BlueprintReadWrite)
    int32 CalYear = 0;

    // 캘린더 표시 월 (1-indexed, UI 상태)
    UPROPERTY(BlueprintReadWrite)
    int32 CalMonth = 0;
};
```

---

### 4.9. FArchiveQueryResult — 아카이브 쿼리 결과 (비영속)

`UArchiveSubsystem::QueryArchive(FilterState)` 호출 시 반환. 위젯이 이 구조체를 받아 렌더링한다.

```cpp
USTRUCT(BlueprintType)
struct FArchiveQueryResult
{
    GENERATED_BODY()

    // 적용된 날짜 범위 (UI 하이라이트, day-strip 표시에 사용)
    UPROPERTY(BlueprintReadOnly)
    FArchiveDateRange DateRange;

    // 필터링된 집중 기록 (RecordedAt 오름차순)
    // 모드 필터 + 카테고리 필터 + 날짜 범위 모두 적용된 결과
    UPROPERTY(BlueprintReadOnly)
    TArray<FSessionRecord> FilteredSessions;

    // 날짜 범위 내 완료된 퀘스트 (CompletedDateKey 기준)
    // arch-content-left 상단에 표시
    UPROPERTY(BlueprintReadOnly)
    TArray<FQuestInstance> DoneQuests;

    // 날짜 범위 내 진행 중 퀘스트 (DueDateKey ≤ 오늘, State==Active)
    UPROPERTY(BlueprintReadOnly)
    TArray<FQuestInstance> ActiveQuests;

    // 날짜 범위 내 기한 초과 퀘스트 (State==Overdue)
    UPROPERTY(BlueprintReadOnly)
    TArray<FQuestInstance> OverdueQuests;

    // 날짜 범위 내 중단된 퀘스트
    UPROPERTY(BlueprintReadOnly)
    TArray<FQuestInstance> SuspendedQuests;

    // 날짜 범위 내 진행 예정 퀘스트 (DueDateKey > 오늘)
    // 반복 퀘스트는 MasterId 기준 중복 제거하여 1건만 포함
    UPROPERTY(BlueprintReadOnly)
    TArray<FQuestInstance> UpcomingQuests;

    // 날짜별 활동 도트 맵 (캘린더 그리드 전체 월 커버)
    // Key: "YYYYMMDD", Value: FDayActivityDots
    UPROPERTY(BlueprintReadOnly)
    TMap<FString, FDayActivityDots> ActivityDotMap;
};
```

---

## 5. 카테고리 관리

카테고리는 유저가 직접 추가·수정·삭제할 수 있는 영속 데이터다.

### 5.1. FCategoryDef — 카테고리 정의 (영속)

기존의 `TArray<FString> Categories` + `TMap<FString, FString> CategoryColors` 를 하나의 구조체로 통합.

```cpp
USTRUCT(BlueprintType)
struct FCategoryDef
{
    GENERATED_BODY()

    // 카테고리명 — FSessionRecord.Category 와 매칭되는 고유 키
    UPROPERTY(BlueprintReadWrite)
    FString Name;

    // HEX 색상 "#RRGGBB". 빈 문자열 = 기본 색상 fallback
    UPROPERTY(BlueprintReadWrite)
    FString Color;
};

// SaveGame 에서
UPROPERTY()
TArray<FCategoryDef> Categories;
// 초기 기본값: 공부 / 프로젝트 / 업무 / 독서 / 운동 / 기타 (Color = "")
```

---

### 5.2. EDeleteCategoryMode — 카테고리 삭제 시 기록 처리 방식

```cpp
UENUM(BlueprintType)
enum class EDeleteCategoryMode : uint8
{
    MigrateToOther  UMETA(DisplayName = "다른 카테고리로 이전"),
    KeepOrphan      UMETA(DisplayName = "기록 유지 (회색 표시)"),
    DeleteRecords   UMETA(DisplayName = "기록도 함께 삭제"),
};
```

---

### 5.3. 카테고리 CRUD 규칙

#### 추가 (Add)
- `Categories`에 `FCategoryDef` 항목 추가.
- 동일한 `Name`이 이미 존재하면 실패 (중복 불허).
- 개수 제한 없음.

#### 색상 수정 (UpdateColor)
- `FCategoryDef.Color` 값만 교체.
- `FSessionRecord.Category`는 Name을 키로 색상을 런타임에 룩업하므로 **기록 수정 불필요**.

#### 이름 변경 (Rename)
```
RenameCategory(OldName, NewName)
  1. NewName이 이미 존재하면 실패
  2. AllRecords 순회 → Category == OldName 이면 NewName 으로 교체
  3. Categories 배열에서 FCategoryDef.Name 교체
  4. SaveGame 저장
```

#### 삭제 (Delete)
삭제 전 UI에서 해당 카테고리 기록 건수(N)를 표시하고 처리 방식을 선택받는다.

```
DeleteCategory(CategoryName, Mode, TargetCategory)

  Mode == MigrateToOther:
    AllRecords 순회 → Category == CategoryName 이면 TargetCategory 로 교체
    Categories 에서 해당 항목 제거
    SaveGame 저장

  Mode == KeepOrphan:
    AllRecords 건드리지 않음
    Categories 에서 해당 항목 제거
    SaveGame 저장
    → 아카이브에서 해당 기록은 카테고리명은 표시되지만 색상 없이 회색 fallback

  Mode == DeleteRecords:
    AllRecords 에서 Category == CategoryName 인 항목 전부 제거
    Categories 에서 해당 항목 제거
    SaveGame 저장
```

**C++ 함수 시그니처**

```cpp
// CategorySubsystem 또는 ArchiveSubsystem 내 구현

bool AddCategory(const FString& Name, const FString& Color);

bool RenameCategory(const FString& OldName, const FString& NewName);

bool UpdateCategoryColor(const FString& Name, const FString& NewColor);

void DeleteCategory(
    const FString& CategoryName,
    EDeleteCategoryMode Mode,
    const FString& TargetCategory = TEXT("")  // MigrateToOther 일 때만 사용
);

// 렌더링 시 색상 룩업 — 카테고리가 삭제된 경우 빈 문자열 반환
FString GetCategoryColor(const FString& CategoryName) const;

// 카테고리에 속한 기록 건수 반환 — 삭제 전 UI 표시에 사용
int32 GetRecordCountByCategory(const FString& CategoryName) const;
```

---

### 5.4. 색상 fallback 규칙

아카이브·기록 위젯에서 카테고리 색상을 사용할 때:

```
color = GetCategoryColor(record.Category)
if color.IsEmpty():
    → var(--border) 계열 회색으로 표시  // KeepOrphan 상태
```

---

## 6. 시스템 관계도

```
[사용자 조작]
      │
      ├── Focus 패널 ──────────────────────────────────────────────────┐
      │         │                                                      │
      │         ▼                                                      │
      │   UFocusSubsystem  (UGameInstanceSubsystem)                   │
      │     ├── EFocusMode              현재 모드                     │
      │     ├── FTimerConfig            설정값 (영속)                 │
      │     ├── FTimerRuntimeState      타이머 진행 상태 (비영속)     │
      │     ├── FStopwatchState         SW 진행 상태 (비영속)         │
      │     │                                                          │
      │     └── 세션 종료 → CreateSessionRecord()                     │
      │                            │                                   │
      │                            ▼                                   │
      │                     FSessionRecord                             │
      │                            │                                   │
      │                            ▼                                   │
      │                   UArchiveSubsystem                            │
      │                     └── TArray<FSessionRecord> AllRecords      │
      │                                                                │
      ├── Quest 패널                                                   │
      │         │                                                      │
      │         ▼                                                      │
      │   UQuestSubsystem  (UGameInstanceSubsystem)                   │
      │     └── TArray<FQuestInstance> AllInstances (영속)            │
      │                                                                │
      └── Archive 패널 ──────────────────────────────────────────────┘
                │
                ▼
          UArchiveSubsystem
            ├── FArchiveFilterState  CurrentFilter  (UI 상태)
            │
            ├── ComputeDateRange(AnchorDateKey, PeriodFilter)
            │     └── returns FArchiveDateRange
            │
            ├── ComputeActivityDotMap(CalYear, CalMonth)
            │     ├── 순회: AllRecords → bHasTimer / bHasStopwatch
            │     ├── 순회: AllInstances → bHasQuest (DueDateKey or CompletedDateKey)
            │     └── returns TMap<FString, FDayActivityDots>
            │
            └── QueryArchive(FArchiveFilterState)
                  ├── ComputeDateRange() → FArchiveDateRange
                  ├── 필터링: AllRecords → FilteredSessions
                  │     조건: DateKey ∈ [Range.Start, Range.End]
                  │           && (ModeFilter == All || SessionType 일치)
                  │           && (CategoryFilter 빈 문자열 || Category 일치)
                  ├── 분류: AllInstances → 5개 퀘스트 버킷
                  │     DueDateKey or CompletedDateKey ∈ Range
                  ├── ComputeActivityDotMap() → ActivityDotMap
                  └── returns FArchiveQueryResult
```

---

## 7. UE 배치 계획

| 데이터 | 위치 | SaveGame 저장 |
|---|---|---|
| `EFocusMode`, `ETimerPhase`, `ESessionEndReason` | `FocusTypes.h` | — |
| `FTimerConfig`, `FTimerRuntimeState`, `FStopwatchState` | `FocusTypes.h` | Config만 ✅ |
| `EArchPeriodFilter`, `EArchModeFilter`, `EQuestState` | `ArchiveTypes.h` | — |
| `EDeleteCategoryMode` | `ArchiveTypes.h` | — |
| `FCategoryDef` | `ArchiveTypes.h` | ✅ |
| `FSessionRecord` | `ArchiveTypes.h` | ✅ |
| `FDayActivityDots`, `FArchiveDateRange` | `ArchiveTypes.h` | ❌ |
| `FArchiveFilterState`, `FArchiveQueryResult` | `ArchiveTypes.h` | ❌ |
| `FQuestInstance` | `QuestTypes.h` | ✅ |
| `UFocusSubsystem` | `FocusSubsystem.h/.cpp` | — |
| `UArchiveSubsystem` | `ArchiveSubsystem.h/.cpp` | — |
| `UQuestSubsystem` | `QuestSubsystem.h/.cpp` | — |
| `URecordTalesSaveGame` | `RecordTalesSaveGame.h/.cpp` | — |

### SaveGame 구조

```cpp
UCLASS()
class URecordTalesSaveGame : public USaveGame
{
    GENERATED_BODY()

public:
    // --- Focus 시스템 ---
    UPROPERTY()
    FTimerConfig LastTimerConfig;

    UPROPERTY()
    EFocusMode LastFocusMode = EFocusMode::Timer;

    // --- Archive 시스템 ---
    // 세션 기록 전체. 최신이 배열 앞 (Index 0)
    UPROPERTY()
    TArray<FSessionRecord> SessionRecords;

    // 카테고리 정의 목록 (FCategoryDef로 통합 — Name + Color)
    // 초기 기본값: 공부 / 프로젝트 / 업무 / 독서 / 운동 / 기타
    UPROPERTY()
    TArray<FCategoryDef> Categories;

    // --- Quest 시스템 ---
    UPROPERTY()
    TArray<FQuestInstance> QuestInstances;

    // 이후 시스템 추가 시 여기에 확장
    // UPROPERTY() int32 Gold;
    // UPROPERTY() int32 PlayerLevel;
    // UPROPERTY() TArray<FGuildMember> GuildMembers;
};
```

---

## 8. RecordId 생성

### 8.1. 형식

```
RecordId = "YYYYMMDD" + "NNNNN"  (총 13자리)

예시:
  2026060900001  ← 2026-06-09 첫 번째 기록
  2026060900002  ← 2026-06-09 두 번째 기록
  2026061000001  ← 2026-06-10 첫 번째 기록 (날짜 바뀌면 시퀀스 리셋)
```

- 문자열 사전순 정렬 = 시간순 정렬이 자동 성립
- 하루 최대 99,999건 보장 (실사용에서 초과 불가)

---

### 8.2. SaveGame 추가 필드

```cpp
// 마지막 기록이 생성된 날짜 키 — "YYYYMMDD"
UPROPERTY()
FString LastRecordDate;

// 해당 날짜의 현재 시퀀스 번호 (1-based)
UPROPERTY()
int32 DailyRecordSequence = 0;
```

---

### 8.3. 생성 함수

`UArchiveSubsystem` 내에 구현.

```cpp
FString UArchiveSubsystem::GenerateRecordId()
{
    FString Today = GetTodayDateKey();  // "YYYYMMDD"

    // 날짜가 바뀌었으면 시퀀스 리셋
    if (SaveGame->LastRecordDate != Today)
    {
        SaveGame->LastRecordDate      = Today;
        SaveGame->DailyRecordSequence = 0;
    }

    SaveGame->DailyRecordSequence++;

    // "20260609" + "00001" → "2026060900001"
    return FString::Printf(TEXT("%s%05d"),
        *Today,
        SaveGame->DailyRecordSequence);
}
```

호출 시점: `FSessionRecord` 생성 직전 1회. 생성 후 즉시 SaveGame 저장.

---

### 8.4. 정렬·필터 활용

| 목적 | 방법 |
|---|---|
| 날짜 필터 | `RecordId.Left(8) == DateKey` |
| 오름차순 (오래된 순) | RecordId 문자열 오름차순 |
| 내림차순 (최신 순) | RecordId 문자열 내림차순 |
| 날짜 범위 필터 | `RecordId >= RangeStart && RecordId <= RangeEnd + "99999"` |

---

## 9. 아카이브 쿼리 흐름

아카이브 패널이 열릴 때부터 콘텐츠가 렌더링되기까지의 전체 흐름.

```
1. 패널 진입
   UArchiveSubsystem::InitFilterState()
     ├── AnchorDateKey = Today ("YYYYMMDD")
     ├── PeriodFilter  = Daily
     ├── ModeFilter    = All
     ├── CategoryFilter = ""
     ├── CalYear  = Today.Year
     └── CalMonth = Today.Month

2. 캘린더 렌더
   UArchiveSubsystem::ComputeActivityDotMap(CalYear, CalMonth)
     ├── 해당 월의 모든 날짜를 키로 맵 초기화
     ├── AllRecords 순회 → DateKey가 해당 월이면 dot 세팅
     └── AllInstances 순회 → DueDateKey / CompletedDateKey가 해당 월이면 bHasQuest = true
   → Widget: 캘린더 그리드 그리기, 도트 표시

3. 날짜 클릭
   Widget → UArchiveSubsystem::SetAnchorDate("YYYYMMDD")
     └── CurrentFilter.AnchorDateKey 갱신

4. 쿼리 실행
   UArchiveSubsystem::QueryArchive(CurrentFilter)
     ├── Step 1: ComputeDateRange(AnchorDateKey, PeriodFilter) → FArchiveDateRange
     ├── Step 2: AllRecords 필터링
     │     for each record in AllRecords:
     │       if record.DateKey >= Range.Start && record.DateKey <= Range.End
     │         && ModeFilter 조건 통과
     │         && CategoryFilter 조건 통과
     │       → FilteredSessions에 추가
     │     RecordedAt 오름차순 정렬
     ├── Step 3: AllInstances 분류
     │     for each inst in AllInstances:
     │       relevantKey = (inst.State == Done) ? inst.CompletedDateKey : inst.DueDateKey
     │       if relevantKey ∈ [Range.Start, Range.End]:
     │         switch(inst.State):
     │           Done      → DoneQuests
     │           Active    → ActiveQuests
     │           Overdue   → OverdueQuests
     │           Suspended → SuspendedQuests
     │           Upcoming  → UpcomingQuests (MasterId 중복 제거)
     ├── Step 4: ComputeActivityDotMap() → ActivityDotMap
     └── return FArchiveQueryResult

5. 위젯 렌더
   Widget::RenderResult(FArchiveQueryResult):
     ├── 캘린더: Range.Start~End에 .sel 클래스, AnchorDateKey에 .anchor 클래스
     ├── day-strip: "YYYY.MM.DD ~ YYYY.MM.DD" 범위 텍스트 표시
     ├── arch-content-left:
     │     DoneQuests 카드 (초록 아이콘)
     │     FilteredSessions 기록 아이템
     └── arch-content-right:
           ActiveQuests (파랑)
           OverdueQuests (빨강)
           SuspendedQuests (회색)
           UpcomingQuests (보라)
```

---

## 10. 열린 질문

| 항목 | 내용 | 상태 |
|---|---|---|
| **카테고리 관리** | `FCategoryDef` 배열로 통합 관리. 삭제 시 3가지 처리 모드 제공 | ✅ 확정 |
| **RecordId 생성** | "YYYYMMDD" + 5자리 일별 시퀀스. SaveGame에 LastRecordDate + DailyRecordSequence 보관 | ✅ 확정 |
| **세션 중 앱 종료** | 진행 중 상태를 SaveGame에 임시 저장할지 | 미결 |
| **최대 기록 수** | 무제한 보관 vs 일정 기간 후 정리 정책 | 미결 |
| **모드 필터와 도트** | 필터 적용 시 도트도 필터를 따를지, 항상 전체 기준으로 표시할지 | 미결 (원본은 전체 기준) |
| **퀘스트 날짜 없음** | DueDateKey가 빈 문자열인 퀘스트의 아카이브 처리 방식 | 미결 (제외 처리 예정) |
| **반복 퀘스트 Upcoming 중복** | MasterId 기준 Set 중복 제거 방식 확정 필요 | 미결 |

---

## 업데이트 이력

| 날짜 | 내용 |
|---|---|
| 2026-06-08 | 초안 작성. 집중·아카이브 시스템 데이터 설계 |
| 2026-06-09 | 전면 확장. 아카이브 쿼리/필터/캘린더/퀘스트 통합 설계 추가 (EArchPeriodFilter, EArchModeFilter, EQuestState, FQuestInstance, FArchiveDateRange, FDayActivityDots, FArchiveFilterState, FArchiveQueryResult, 쿼리 흐름 섹션) |
| 2026-06-09 | 카테고리 관리 섹션 추가. FCategoryDef 구조체, EDeleteCategoryMode, 이전/유지/삭제 3가지 모드 설계 반영 |
| 2026-06-09 | RecordId 생성 섹션 추가. YYYYMMDD+5자리 일별 시퀀스 방식, GenerateRecordId 함수, 정렬·필터 활용법 확정 |
