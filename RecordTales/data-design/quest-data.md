# RecordTales UE — 퀘스트(할 일) 시스템 데이터 설계

> 퀘스트(할 일) 시스템의 C++ 데이터 구조 설계서.
> project-wiki (quests-panel, recurrence-removal, today-todo-overlay) 반영.
> `focus-archive-data.md`의 `FQuestInstance`(4.5)를 대체·확장한다.

---

## 목차

1. [설계 원칙](#1-설계-원칙)
2. [마스터-인스턴스 모델](#2-마스터-인스턴스-모델)
3. [열거형 (Enum)](#3-열거형-enum)
4. [구조체 (Struct)](#4-구조체-struct)
5. [ID 생성](#5-id-생성)
6. [인스턴스 생성 & 자정 롤오버](#6-인스턴스-생성--자정-롤오버)
7. [상태 판정 & 정렬](#7-상태-판정--정렬)
8. [D-day 계산](#8-d-day-계산)
9. [반복 제거 (Recurrence Removal)](#9-반복-제거-recurrence-removal)
10. [즐겨찾기 & 오늘 복제](#10-즐겨찾기--오늘-복제)
11. [UE 배치 계획](#11-ue-배치-계획)
12. [열린 질문](#12-열린-질문)

---

## 1. 설계 원칙

| 원칙 | 내용 |
|---|---|
| **물리 인스턴스 채택** | HTML 프로토타입은 인스턴스를 매번 가상 계산(`getInstance`)하지만, UE는 인스턴스를 실제 배열(`TArray<FQuestInstance>`)에 저장한다 |
| **마스터는 불변 템플릿** | `FQuestMaster`는 "이 퀘스트가 언제·어떻게 반복되는가"만 정의. 진행 상태는 일절 갖지 않는다 |
| **마스터는 삭제되지 않음** | 반복 제거 동작에서 인스턴스는 삭제될 수 있어도 마스터는 항상 남는다 |
| **단발 퀘스트도 마스터+인스턴스 1쌍** | `Recurrence == None`이어도 마스터 1개 + 인스턴스 1개로 동일하게 관리 (모델 단순화) |
| **날짜 키 통일** | `"YYYYMMDD"` 8자리 문자열. `focus-archive-data.md`와 동일 규칙 |
| **상태는 캐시, 매번 재계산 가능** | `FQuestInstance.State`는 캐시 필드. 날짜가 바뀌거나 done/suspend 토글 시 재계산해서 덮어쓴다 |

---

## 2. 마스터-인스턴스 모델

```
FQuestMaster (영속, 불변 템플릿)
  ├── 단발(Recurrence=None)  → 인스턴스 정확히 1개, 생성 시 즉시 만들어짐
  └── 반복(Daily/Weekly/Monthly)
        → 자정 롤오버마다 해당 날짜에 발생하면 인스턴스 1개씩 생성
        → "다음 발생일" 인스턴스 1개를 항상 미리 만들어둠 (Upcoming 표시용)

FQuestInstance (영속, 실제 리스트에 표시되는 단위)
  ├── MasterId 로 FQuestMaster 참조 (Title/Category/Recurrence 등은 Master에서 조회)
  └── 자기 자신은 진행 상태(Done/Suspended/날짜)만 가짐
```

> **왜 물리 인스턴스인가**
> HTML의 `getAllInstances()`는 매 렌더마다 시작일~오늘 범위를 최대 500회 순회하며 인스턴스를 즉석 생성한다. UMG 위젯은 `TArray`를 그대로 ListView에 바인딩하는 구조가 자연스럽고, SaveGame에 진행 상태(완료/중단)를 저장하려면 어차피 인스턴스가 실체로 존재해야 한다. `handoff-spec.md` 8.1의 리팩터 목표("반복 인스턴스는 실제 물리 객체")와 동일한 방향.

---

## 3. 열거형 (Enum)

### 3.1. EQuestRecurrence — 반복 주기

```cpp
UENUM(BlueprintType)
enum class EQuestRecurrence : uint8
{
    None     UMETA(DisplayName = "단발"),
    Daily    UMETA(DisplayName = "매일"),
    Weekly   UMETA(DisplayName = "매주"),
    Monthly  UMETA(DisplayName = "매월"),
};
```

### 3.2. EQuestState — 인스턴스 상태 (기존 유지)

`focus-archive-data.md` 3.6에 정의된 것을 그대로 사용한다.

```cpp
UENUM(BlueprintType)
enum class EQuestState : uint8
{
    Active     UMETA(DisplayName = "진행 중"),
    Done       UMETA(DisplayName = "완료"),
    Overdue    UMETA(DisplayName = "기한 초과"),
    Suspended  UMETA(DisplayName = "중단"),
    Upcoming   UMETA(DisplayName = "진행 예정"),
};
```

> **상태 우선순위** (높을수록 먼저 판정): `Suspended > Done > Upcoming > Overdue > Active`

---

## 4. 구조체 (Struct)

### 4.1. FQuestMaster — 퀘스트 원본 (영속)

```cpp
USTRUCT(BlueprintType)
struct FQuestMaster
{
    GENERATED_BODY()

    // 고유 ID — "YYYYMMDD" + 5자리 시퀀스 (예: "2026061100001")
    UPROPERTY(BlueprintReadOnly)
    FString MasterId;

    // 제목 (필수, 최대 80자)
    UPROPERTY(BlueprintReadWrite)
    FString Title;

    // 설명 (선택)
    UPROPERTY(BlueprintReadWrite)
    FString Description;

    // 카테고리명 — FCategoryDef.Name 참조
    UPROPERTY(BlueprintReadWrite)
    FString Category;

    // 반복 주기
    UPROPERTY(BlueprintReadWrite)
    EQuestRecurrence Recurrence = EQuestRecurrence::None;

    // 시작일 — "YYYYMMDD". 생성 후 절대 변경 불가
    UPROPERTY(BlueprintReadOnly)
    FString StartDateKey;

    // 마감일 — "YYYYMMDD". 단발 전용. 없으면 빈 문자열 (= StartDateKey와 동일 취급)
    UPROPERTY(BlueprintReadWrite)
    FString DeadlineDateKey;

    // 마감일 오프셋(일) — 반복 전용. -1 = 오프셋 없음(당일 마감)
    // 생성 후 절대 변경 불가
    UPROPERTY(BlueprintReadOnly)
    int32 DeadlineOffsetDays = -1;

    // 반복 종료일 — "YYYYMMDD". 빈 문자열 = StartDateKey + 1년 자동 적용
    UPROPERTY(BlueprintReadWrite)
    FString RecEndDateKey;

    // 즐겨찾기 여부
    UPROPERTY(BlueprintReadWrite)
    bool bIsFavorite = false;
};
```

> **절대 불변 필드** (`focus-archive-data.md` 설계 원칙과 동일한 맥락)
> - `StartDateKey`: ID·인스턴스 발생 판정의 기준이므로 변경 불가
> - `DeadlineOffsetDays`: 반복 인스턴스의 마감일 계산 기준이므로 변경 불가
> - 수정하고 싶다면 → 기존 마스터 삭제 없이, 새 마스터를 만들고 기존 것은 `RecEndDateKey`로 종료 처리 (9. 참조)

---

### 4.2. FQuestInstance — 퀘스트 인스턴스 (영속, 개정판)

`focus-archive-data.md` 4.5의 `FQuestInstance`를 대체한다. `Title`/`Category`/`RecurrenceType`은 `MasterId`로 `FQuestMaster`를 조회해서 얻으므로 인스턴스에 중복 저장하지 않는다.

```cpp
USTRUCT(BlueprintType)
struct FQuestInstance
{
    GENERATED_BODY()

    // 인스턴스 고유 ID — MasterId + "_" + OccurrenceDateKey
    // 예: "2026061100001_20260612"
    UPROPERTY(BlueprintReadOnly)
    FString InstanceId;

    // 부모 마스터 ID
    UPROPERTY(BlueprintReadOnly)
    FString MasterId;

    // 발생일 — "YYYYMMDD"
    // 단발: Master.StartDateKey 와 동일 / 반복: 해당 회차가 발생한 날짜
    UPROPERTY(BlueprintReadOnly)
    FString OccurrenceDateKey;

    // 마감일 — "YYYYMMDD"
    // 단발: Master.DeadlineDateKey 또는 OccurrenceDateKey
    // 반복: Master.DeadlineOffsetDays >= 0 ? OccurrenceDateKey + offset : OccurrenceDateKey
    UPROPERTY(BlueprintReadOnly)
    FString DueDateKey;

    // 완료 여부
    UPROPERTY(BlueprintReadWrite)
    bool bDone = false;

    UPROPERTY(BlueprintReadWrite)
    FDateTime CompletedAt;

    // 완료된 날짜 키 — 아카이브 done 버킷 분류에 사용
    UPROPERTY(BlueprintReadWrite)
    FString CompletedDateKey;

    // 중단 여부
    UPROPERTY(BlueprintReadWrite)
    bool bSuspended = false;

    UPROPERTY(BlueprintReadWrite)
    FDateTime SuspendedAt;

    UPROPERTY(BlueprintReadWrite)
    FString SuspendReason;

    // 완료 시 보상
    UPROPERTY(BlueprintReadWrite)
    int32 GoldReward = 0;

    UPROPERTY(BlueprintReadWrite)
    int32 XpReward = 0;

    // 캐시 — ComputeInstanceState() 결과를 저장. 매번 재계산하지 않도록
    // 날짜 변경(롤오버) 또는 bDone/bSuspended 토글 시 갱신
    UPROPERTY(BlueprintReadOnly)
    EQuestState State = EQuestState::Active;
};
```

---

## 5. ID 생성

### 5.1. MasterId

`FSessionRecord.RecordId`(`focus-archive-data.md` 8.)와 동일한 형식이지만 **카운터는 별도**로 관리한다 — 같은 날 Focus 기록과 퀘스트 생성이 섞여도 시퀀스가 충돌하지 않도록.

```
MasterId = "YYYYMMDD" + "NNNNN"  (총 13자리)

SaveGame 추가 필드:
  FString LastQuestDate;          // 마지막 마스터 생성 날짜 키
  int32   DailyQuestSequence = 0; // 해당 날짜의 시퀀스
```

```cpp
FString UQuestSubsystem::GenerateMasterId()
{
    FString Today = GetTodayDateKey();
    if (SaveGame->LastQuestDate != Today)
    {
        SaveGame->LastQuestDate      = Today;
        SaveGame->DailyQuestSequence = 0;
    }
    SaveGame->DailyQuestSequence++;
    return FString::Printf(TEXT("%s%05d"), *Today, SaveGame->DailyQuestSequence);
}
```

### 5.2. InstanceId

별도 카운터 없이 **결정적(deterministic)으로 조합**한다 — "마스터 X의 Y일 발생분"은 항상 동일한 ID를 가지므로, 롤오버 시 중복 생성을 `InstanceId` 존재 여부만으로 막을 수 있다.

```
InstanceId = MasterId + "_" + OccurrenceDateKey
예: "2026061100001_20260612"
```

---

## 6. 인스턴스 생성 & 자정 롤오버

### 6.1. QuestOccursOn — 발생일 판정

```cpp
bool UQuestSubsystem::QuestOccursOn(const FQuestMaster& Master, const FString& Dk) const
{
    FString RangeEnd = Master.RecEndDateKey.IsEmpty()
        ? AddYears(Master.StartDateKey, 1)
        : Master.RecEndDateKey;

    if (Dk < Master.StartDateKey || Dk > RangeEnd) return false;

    switch (Master.Recurrence)
    {
        case EQuestRecurrence::None:    return Dk == Master.StartDateKey;
        case EQuestRecurrence::Daily:   return true;
        case EQuestRecurrence::Weekly:  return DayOfWeek(Dk)  == DayOfWeek(Master.StartDateKey);
        case EQuestRecurrence::Monthly: return DayOfMonth(Dk) == DayOfMonth(Master.StartDateKey);
        default: return false;
    }
}
```

### 6.2. CreateQuest — 마스터 생성 + 첫 인스턴스

```cpp
FString UQuestSubsystem::CreateQuest(const FQuestMaster& InMaster)
{
    FQuestMaster NewMaster = InMaster;
    NewMaster.MasterId     = GenerateMasterId();
    NewMaster.StartDateKey = GetTodayDateKey(); // 또는 사용자가 선택한 시작일
    SaveGame->QuestMasters.Add(NewMaster);

    // 단발/반복 공통: 시작일 회차 인스턴스는 즉시 1개 생성
    GenerateInstanceIfNeeded(NewMaster, NewMaster.StartDateKey);

    // 반복이면 "다음 발생일" 1건도 미리 생성 (Upcoming 표시용)
    if (NewMaster.Recurrence != EQuestRecurrence::None)
    {
        GenerateUpcomingInstance(NewMaster);
    }
    return NewMaster.MasterId;
}
```

### 6.3. GenerateInstanceIfNeeded — 인스턴스 1건 생성 (중복 방지)

```cpp
void UQuestSubsystem::GenerateInstanceIfNeeded(const FQuestMaster& Master, const FString& Dk)
{
    FString InstId = Master.MasterId + TEXT("_") + Dk;
    if (FindInstance(InstId) != nullptr) return; // 이미 존재 → 생략

    FQuestInstance NewInst;
    NewInst.InstanceId        = InstId;
    NewInst.MasterId          = Master.MasterId;
    NewInst.OccurrenceDateKey = Dk;
    NewInst.DueDateKey        = (Master.DeadlineOffsetDays >= 0)
        ? AddDays(Dk, Master.DeadlineOffsetDays)
        : (Master.Recurrence == EQuestRecurrence::None && !Master.DeadlineDateKey.IsEmpty()
            ? Master.DeadlineDateKey
            : Dk);
    NewInst.State = ComputeInstanceState(NewInst);

    SaveGame->QuestInstances.Add(NewInst);
}
```

### 6.4. 자정 롤오버

```cpp
void UQuestSubsystem::ProcessDailyRollover()
{
    FString Today = GetTodayDateKey();

    // 최초 실행: 오늘부터 시작 (과거로 거슬러 올라가며 생성하지 않음)
    if (SaveGame->LastQuestRolloverDate.IsEmpty())
    {
        SaveGame->LastQuestRolloverDate = PrevDateKey(Today);
    }

    // 마지막 처리일 다음날 ~ 오늘까지 순차 처리 (며칠 동안 앱을 안 켰어도 대응)
    FString Cursor = NextDateKey(SaveGame->LastQuestRolloverDate);
    int32 Safety = 0;
    while (Cursor <= Today && Safety < 400)
    {
        for (const FQuestMaster& Master : SaveGame->QuestMasters)
        {
            if (Master.Recurrence == EQuestRecurrence::None) continue;
            if (QuestOccursOn(Master, Cursor)) GenerateInstanceIfNeeded(Master, Cursor);
        }
        Cursor = NextDateKey(Cursor);
        Safety++;
    }
    SaveGame->LastQuestRolloverDate = Today;

    // 모든 인스턴스의 캐시 상태 재계산 (날짜가 바뀌었으므로 active↔overdue 등 변동)
    RefreshAllInstanceStates();

    // 각 반복 마스터의 "다음 발생일" 1건 보장
    for (const FQuestMaster& Master : SaveGame->QuestMasters)
    {
        if (Master.Recurrence != EQuestRecurrence::None) GenerateUpcomingInstance(Master);
    }
}
```

```cpp
void UQuestSubsystem::GenerateUpcomingInstance(const FQuestMaster& Master)
{
    FString RangeEnd = Master.RecEndDateKey.IsEmpty()
        ? AddYears(Master.StartDateKey, 1)
        : Master.RecEndDateKey;

    FString Cursor = NextDateKey(GetTodayDateKey());
    int32 Safety = 0;
    while (Cursor <= RangeEnd && Safety < 400)
    {
        if (QuestOccursOn(Master, Cursor))
        {
            GenerateInstanceIfNeeded(Master, Cursor);
            return; // 첫 번째 미래 발생일 1건만
        }
        Cursor = NextDateKey(Cursor);
        Safety++;
    }
}
```

> **호출 시점**: `UQuestSubsystem::Initialize()`에서 1회 호출. 앱을 며칠간 안 켜도 `Safety` 캡(400회) 안에서 누락된 날짜를 모두 따라잡는다 (HTML의 `processRollover` 400회 캡과 동일 사상).

---

## 7. 상태 판정 & 정렬

### 7.1. ComputeInstanceState

```cpp
EQuestState UQuestSubsystem::ComputeInstanceState(const FQuestInstance& Inst) const
{
    if (Inst.bSuspended) return EQuestState::Suspended;   // 최우선
    if (Inst.bDone)      return EQuestState::Done;

    FString Today = GetTodayDateKey();
    if (Inst.OccurrenceDateKey > Today) return EQuestState::Upcoming;
    if (Inst.DueDateKey < Today)        return EQuestState::Overdue;
    return EQuestState::Active;
}
```

`State` 캐시는 다음 시점에 갱신한다: `ToggleDone()`, `ToggleSuspend()`, `ProcessDailyRollover()`.

### 7.2. 정렬 규칙 (quests-panel 4.2 동일)

| 상태 | 정렬 기준 |
|---|---|
| Overdue | `DueDateKey` 오래된 순(asc) → `InstanceId` asc |
| Active | `DueDateKey` 가까운 순(asc) → `InstanceId` asc |
| Upcoming | `OccurrenceDateKey` 가까운 순(asc) → `InstanceId` asc |
| Done | `CompletedAt` 빠른 순(asc) → `InstanceId` asc |
| Suspended | `SuspendedAt` 최근 순(desc) → `InstanceId` desc |

### 7.3. 지연 카운트

```cpp
int32 UQuestSubsystem::CountOverdue() const
{
    return SaveGame->QuestInstances.FilterByPredicate(
        [](const FQuestInstance& I){ return I.State == EQuestState::Overdue; }
    ).Num();
}
```

---

## 8. D-day 계산

```cpp
struct FDdayInfo
{
    FString Label;     // "D-3" / "D-DAY" / "D+1"
    bool    bNear;     // D-1~D-3 (노랑/주황 강조)
    bool    bToday;    // D-DAY
    bool    bOverdue;  // D+1 이상
};

FDdayInfo UQuestSubsystem::ComputeDday(const FString& DueDateKey) const
{
    int32 Diff = DaysBetween(GetTodayDateKey(), DueDateKey); // Due - Today

    FDdayInfo Info;
    if (Diff > 0)
    {
        Info.Label = FString::Printf(TEXT("D-%d"), Diff);
        Info.bNear = (Diff <= 3);
    }
    else if (Diff == 0)
    {
        Info.Label  = TEXT("D-DAY");
        Info.bToday = true;
    }
    else
    {
        Info.Label    = FString::Printf(TEXT("D+%d"), -Diff);
        Info.bOverdue = true;
    }
    return Info;
}
```

| 조건 | 색상 |
|---|---|
| D-7 이상 | 초록 |
| D-1 ~ D-6 (`bNear`) | 노랑/주황 |
| D-DAY (`bToday`) | 주황 |
| D+1 이상 (`bOverdue`) | 빨강 |

---

## 9. 반복 제거 (Recurrence Removal)

`handoff-spec.md` 8.4 기준 — `RecEndDateKey` 변경을 인스턴스 물리 삭제로 처리한다. 마스터는 어떤 경우에도 삭제되지 않는다.

```cpp
void UQuestSubsystem::ApplyRecurrenceRemoval(
    const FString& MasterId,
    const FString& NewRecEndDateKey,
    const TArray<EQuestState>& StatusesToDelete)
{
    FQuestMaster* Master = FindMaster(MasterId);
    if (!Master) return;

    FString Today = GetTodayDateKey();
    Master->RecEndDateKey = NewRecEndDateKey;

    SaveGame->QuestInstances.RemoveAll([&](const FQuestInstance& Inst)
    {
        if (Inst.MasterId != MasterId) return false;
        if (Inst.OccurrenceDateKey <= NewRecEndDateKey) return false; // 새 범위 안 → 유지

        // 새 종료일보다 미래인 인스턴스만 삭제 후보
        if (NewRecEndDateKey >= Today)
        {
            return true; // 전부 미래 발생분 → 무조건 삭제
        }
        // 새 종료일 < 오늘 → 과거/오늘/미래 인스턴스가 섞여있음
        // 사용자가 선택한 상태(StatusesToDelete)에 해당하는 것만 삭제
        return StatusesToDelete.Contains(Inst.State);
    });

    GenerateUpcomingInstance(*Master); // 범위가 넓어진 경우 대비 재생성 시도
}
```

| 시나리오 | `NewRecEndDateKey` | UI 동작 |
|---|---|---|
| 미래 날짜 선택 | `>= Today` | 그 이후 인스턴스 전부 자동 삭제 (확인만 받음) |
| 오늘/과거 날짜 선택 | `< Today` | 영향받는 인스턴스를 상태별로 보여주고, 삭제할 상태 태그(지연/진행 예정/진행 중/완료/중단) 복수 선택 |
| 시작일 이전 날짜 선택 | `< StartDateKey` | 사실상 모든 인스턴스가 후보 → 동일하게 상태 태그 선택 |

---

## 10. 즐겨찾기 & 오늘 복제

즐겨찾기는 `FQuestMaster.bIsFavorite`에 저장. "즐겨찾기 칩 클릭" 시 해당 퀘스트를 **오늘 단발 퀘스트로 복제**한다.

```cpp
FString UQuestSubsystem::DuplicateAsTodayQuest(const FString& SourceMasterId)
{
    const FQuestMaster* Source = FindMaster(SourceMasterId);
    if (!Source) return TEXT("");

    FQuestMaster NewMaster;
    NewMaster.Title       = Source->Title;
    NewMaster.Description = Source->Description;
    NewMaster.Category    = Source->Category;
    NewMaster.Recurrence  = EQuestRecurrence::None;
    NewMaster.bIsFavorite = false;
    // DeadlineDateKey, DeadlineOffsetDays는 복제하지 않음 — 오늘 마감 단발로 생성

    return CreateQuest(NewMaster); // 내부에서 MasterId 발급 + StartDateKey=오늘 + 인스턴스 1개 생성
}
```

---

## 11. UE 배치 계획

| 데이터 | 위치 | SaveGame 저장 |
|---|---|---|
| `EQuestRecurrence` | `QuestTypes.h` | — |
| `EQuestState` (기존 유지) | `QuestTypes.h` | — |
| `FQuestMaster` (신규) | `QuestTypes.h` | ✅ |
| `FQuestInstance` (개정) | `QuestTypes.h` | ✅ |
| `FDdayInfo` (신규) | `QuestTypes.h` | ❌ (비영속) |

### SaveGame 추가 필드

```cpp
// --- Quest 시스템 ---
UPROPERTY() TArray<FQuestMaster>   QuestMasters;
UPROPERTY() TArray<FQuestInstance> QuestInstances;

UPROPERTY() FString LastQuestDate;          // MasterId 시퀀스용
UPROPERTY() int32   DailyQuestSequence = 0;

UPROPERTY() FString LastQuestRolloverDate;  // 자정 롤오버 마지막 처리일
```

### QuestSubsystem.h 함수 시그니처 추가

```cpp
// 생성/조회
FString GenerateMasterId();
FString CreateQuest(const FQuestMaster& InMaster);
FQuestMaster*   FindMaster(const FString& MasterId);
FQuestInstance* FindInstance(const FString& InstanceId);

// 발생/롤오버
bool QuestOccursOn(const FQuestMaster& Master, const FString& Dk) const;
void GenerateInstanceIfNeeded(const FQuestMaster& Master, const FString& Dk);
void GenerateUpcomingInstance(const FQuestMaster& Master);
void ProcessDailyRollover();
void RefreshAllInstanceStates();

// 상태/표시
EQuestState ComputeInstanceState(const FQuestInstance& Inst) const;
FDdayInfo   ComputeDday(const FString& DueDateKey) const;
int32       CountOverdue() const;

// 사용자 액션
void   ToggleDone(const FString& InstanceId);
void   ToggleSuspend(const FString& InstanceId, const FString& Reason);
void   DeleteQuest(const FString& MasterId); // 마스터+모든 인스턴스 영구 삭제 (사용자가 명시 요청 시만)
void   ApplyRecurrenceRemoval(const FString& MasterId, const FString& NewRecEndDateKey, const TArray<EQuestState>& StatusesToDelete);
FString DuplicateAsTodayQuest(const FString& SourceMasterId);
```

> `cpp-file-structure.md`의 `QuestTypes.h` / `QuestSubsystem.h` 절을 위 내용으로 갱신할 것.

---

## 12. 열린 질문

| 항목 | 내용 | 상태 |
|---|---|---|
| **마스터/인스턴스 모델** | 물리 인스턴스 + MasterId 참조 방식으로 확정 | ✅ 확정 |
| **인스턴스 ID** | `MasterId + "_" + OccurrenceDateKey` (결정적 조합, 별도 카운터 불필요) | ✅ 확정 |
| **자정 롤오버** | `Initialize()` 1회 호출, 최대 400일 따라잡기, 다음 발생일 1건 항상 보장 | ✅ 확정 |
| **반복 제거** | `RecEndDateKey` 변경 + 상태 태그 선택 삭제로 통일 처리 | ✅ 확정 |
| **마스터 직접 삭제** | "퀘스트 삭제" 시 마스터+인스턴스 전부 삭제할지, 반복은 항상 RecEndDate 조정만 허용할지 | 미결 |
| **완료된 반복 인스턴스 보관 기간** | Done 인스턴스를 무제한 보관할지, 일정 기간 후 정리할지 (`focus-archive-data.md` 10.의 "최대 기록 수"와 동일 맥락) | 미결 |
| **XP/Gold 보상값 산정** | `GoldReward`/`XpReward`를 카테고리·반복 주기별로 어떻게 계산할지 | 미결 (경험치 시스템 설계 시 결정) |

---

## 업데이트 이력

| 날짜 | 내용 |
|---|---|
| 2026-06-11 | 초안 작성. 마스터-인스턴스 모델, ID 생성, 자정 롤오버, 상태 판정, D-day, 반복 제거, 즐겨찾기 복제 설계. `focus-archive-data.md` 4.5 `FQuestInstance`를 대체 |
