# RecordTales UE — C++ 파일 구조 설계

> 실제 UE C++ 프로젝트에서 작업할 때 파일별 담당 범위와 의존성을 정의합니다.
> 코딩 시작 전 이 문서를 먼저 확인하고, 어느 파일에 무엇을 작성할지 결정하세요.

---

## 목차

1. [디렉터리 구조](#1-디렉터리-구조)
2. [파일별 담당 범위](#2-파일별-담당-범위)
3. [파일 간 의존성](#3-파일-간-의존성)
4. [파일별 스켈레톤](#4-파일별-스켈레톤)
5. [작업 순서 가이드](#5-작업-순서-가이드)

---

## 1. 디렉터리 구조

UE의 Public / Private 관례를 따른다.  
- **Public**: 다른 파일에서 `#include` 가능한 헤더  
- **Private**: 해당 .cpp 파일에서만 사용하는 구현

```
Source/RecordTales/
│
├── Public/
│   ├── Types/
│   │   ├── FocusTypes.h          ← 집중 시스템 열거형·구조체
│   │   ├── ArchiveTypes.h        ← 아카이브·카테고리 열거형·구조체
│   │   └── QuestTypes.h          ← 퀘스트 열거형·구조체
│   │
│   ├── Subsystems/
│   │   ├── FocusSubsystem.h      ← UFocusSubsystem 클래스 선언
│   │   ├── ArchiveSubsystem.h    ← UArchiveSubsystem 클래스 선언
│   │   └── QuestSubsystem.h      ← UQuestSubsystem 클래스 선언
│   │
│   └── Save/
│       └── RecordTalesSaveGame.h ← URecordTalesSaveGame 클래스 선언
│
└── Private/
    ├── Subsystems/
    │   ├── FocusSubsystem.cpp    ← UFocusSubsystem 구현
    │   ├── ArchiveSubsystem.cpp  ← UArchiveSubsystem 구현
    │   └── QuestSubsystem.cpp    ← UQuestSubsystem 구현
    │
    └── Save/
        └── RecordTalesSaveGame.cpp
```

---

## 2. 파일별 담당 범위

### Public/Types/FocusTypes.h

집중 시스템(타이머·스톱워치)에서만 쓰는 타입.

| 종류 | 이름 | 설명 |
|---|---|---|
| Enum | `EFocusMode` | Timer / Stopwatch |
| Enum | `ETimerPhase` | Idle / Focusing / Resting / LastResting / Paused |
| Enum | `ESessionEndReason` | NormalComplete / EarlyStop |
| Struct | `FTimerConfig` | 타이머 설정값 (영속) |
| Struct | `FTimerRuntimeState` | 타이머 진행 상태 (비영속) |
| Struct | `FStopwatchState` | 스톱워치 진행 상태 (비영속) |

---

### Public/Types/ArchiveTypes.h

아카이브·카테고리·기록 관련 타입.

| 종류 | 이름 | 설명 |
|---|---|---|
| Enum | `EArchPeriodFilter` | Daily / Weekly / Monthly |
| Enum | `EArchModeFilter` | All / TimerOnly / SwOnly |
| Enum | `EDeleteCategoryMode` | MigrateToOther / KeepOrphan / DeleteRecords |
| Struct | `FCategoryDef` | 카테고리 이름 + 색상 (영속) |
| Struct | `FSessionRecord` | 집중 세션 기록 1건 (영속) |
| Struct | `FArchiveDateRange` | 날짜 범위 Start / End (비영속) |
| Struct | `FDayActivityDots` | 캘린더 날짜별 도트 데이터 (비영속) |
| Struct | `FArchiveFilterState` | 현재 필터 상태 (비영속) |
| Struct | `FArchiveQueryResult` | 쿼리 결과 전체 (비영속) |

---

### Public/Types/QuestTypes.h

퀘스트 시스템 타입.

| 종류 | 이름 | 설명 |
|---|---|---|
| Enum | `EQuestState` | Active / Done / Overdue / Suspended / Upcoming |
| Struct | `FQuestInstance` | 퀘스트 인스턴스 (영속) |

---

### Public/Subsystems/FocusSubsystem.h

```
UFocusSubsystem (UGameInstanceSubsystem)
├── 멤버 변수
│   ├── EFocusMode         CurrentMode
│   ├── FTimerConfig       TimerConfig        (영속 — SaveGame 연동)
│   ├── FTimerRuntimeState RuntimeState       (비영속)
│   └── FStopwatchState    SwState            (비영속)
└── 주요 함수 선언
    ├── StartTimer()
    ├── StopTimer(ESessionEndReason)
    ├── TogglePause()
    ├── StartStopwatch()
    ├── StopStopwatch()
    ├── ToggleSwPause()
    └── CreateSessionRecord() → FSessionRecord
```

---

### Public/Subsystems/ArchiveSubsystem.h

```
UArchiveSubsystem (UGameInstanceSubsystem)
├── 멤버 변수
│   ├── TArray<FSessionRecord>  AllRecords      (영속 — SaveGame 연동)
│   ├── TArray<FCategoryDef>    Categories      (영속 — SaveGame 연동)
│   └── FArchiveFilterState     CurrentFilter   (비영속)
└── 주요 함수 선언
    ├── AddRecord(FSessionRecord)
    ├── GenerateRecordId() → FString
    ├── QueryArchive(FArchiveFilterState) → FArchiveQueryResult
    ├── ComputeDateRange(FString, EArchPeriodFilter) → FArchiveDateRange
    ├── ComputeActivityDotMap(int32 Year, int32 Month) → TMap<FString, FDayActivityDots>
    ├── AddCategory(FString Name, FString Color) → bool
    ├── RenameCategory(FString OldName, FString NewName) → bool
    ├── UpdateCategoryColor(FString Name, FString Color) → bool
    ├── DeleteCategory(FString Name, EDeleteCategoryMode, FString Target)
    ├── GetCategoryColor(FString Name) → FString
    └── GetRecordCountByCategory(FString Name) → int32
```

---

### Public/Subsystems/QuestSubsystem.h

```
UQuestSubsystem (UGameInstanceSubsystem)
├── 멤버 변수
│   └── TArray<FQuestInstance>  AllInstances    (영속 — SaveGame 연동)
└── 주요 함수 선언 (추후 퀘스트 시스템 설계 시 확장)
    ├── AddInstance(FQuestInstance)
    ├── CompleteInstance(FString InstanceId)
    └── GetInstancesByDateRange(FArchiveDateRange) → TArray<FQuestInstance>
```

---

### Public/Save/RecordTalesSaveGame.h

```
URecordTalesSaveGame (USaveGame)
├── Focus
│   ├── FTimerConfig   LastTimerConfig
│   └── EFocusMode     LastFocusMode
├── Archive
│   ├── TArray<FSessionRecord>  SessionRecords
│   ├── TArray<FCategoryDef>    Categories
│   ├── FString                 LastRecordDate
│   └── int32                   DailyRecordSequence
└── Quest
    └── TArray<FQuestInstance>  QuestInstances
```

---

### Private/Subsystems/FocusSubsystem.cpp

`FocusSubsystem.h`에 선언된 함수들의 구현.  
타이머 틱, 페이즈 전환, 기록 생성 후 `UArchiveSubsystem::AddRecord()` 호출.

### Private/Subsystems/ArchiveSubsystem.cpp

`ArchiveSubsystem.h`에 선언된 함수들의 구현.  
`GenerateRecordId()`, `QueryArchive()`, 카테고리 CRUD 로직 포함.

### Private/Subsystems/QuestSubsystem.cpp

`QuestSubsystem.h`에 선언된 함수들의 구현.

### Private/Save/RecordTalesSaveGame.cpp

생성자에서 기본 카테고리 초기화.

```cpp
URecordTalesSaveGame::URecordTalesSaveGame()
{
    // 기본 카테고리 세팅
    Categories = {
        { TEXT("공부"),     TEXT("") },
        { TEXT("프로젝트"), TEXT("") },
        { TEXT("업무"),     TEXT("") },
        { TEXT("독서"),     TEXT("") },
        { TEXT("운동"),     TEXT("") },
        { TEXT("기타"),     TEXT("") },
    };
    LastRecordDate      = TEXT("");
    DailyRecordSequence = 0;
}
```

---

## 3. 파일 간 의존성

의존성이 단방향으로만 흐르도록 설계한다. 순환 참조 금지.

```
RecordTalesSaveGame.h
  └── #include "Types/FocusTypes.h"
  └── #include "Types/ArchiveTypes.h"
  └── #include "Types/QuestTypes.h"

FocusSubsystem.h
  └── #include "Types/FocusTypes.h"
  └── #include "Types/ArchiveTypes.h"   ← FSessionRecord 생성에 필요

ArchiveSubsystem.h
  └── #include "Types/ArchiveTypes.h"
  └── #include "Types/QuestTypes.h"     ← 퀘스트 버킷 분류에 필요
  └── #include "Save/RecordTalesSaveGame.h"

QuestSubsystem.h
  └── #include "Types/QuestTypes.h"
  └── #include "Types/ArchiveTypes.h"   ← FArchiveDateRange 범위 쿼리에 필요

FocusSubsystem.cpp
  └── #include "Subsystems/FocusSubsystem.h"
  └── #include "Subsystems/ArchiveSubsystem.h"  ← AddRecord() 호출

ArchiveSubsystem.cpp
  └── #include "Subsystems/ArchiveSubsystem.h"
  └── #include "Subsystems/QuestSubsystem.h"    ← QueryArchive() 내 퀘스트 조회
```

> **Types 파일끼리는 서로 include 하지 않는다.**  
> Types → (의존 없음)  
> Subsystem.h → Types  
> Subsystem.cpp → Subsystem.h + 다른 Subsystem.h  
> SaveGame.h → Types

---

## 4. 파일별 스켈레톤

실제 코딩 시 복사해서 시작하는 뼈대 코드.

### FocusTypes.h

```cpp
#pragma once
#include "CoreMinimal.h"
#include "FocusTypes.generated.h"

UENUM(BlueprintType)
enum class EFocusMode : uint8 { ... };

UENUM(BlueprintType)
enum class ETimerPhase : uint8 { ... };

UENUM(BlueprintType)
enum class ESessionEndReason : uint8 { ... };

USTRUCT(BlueprintType)
struct FTimerConfig { GENERATED_BODY() ... };

USTRUCT(BlueprintType)
struct FTimerRuntimeState { GENERATED_BODY() ... };

USTRUCT(BlueprintType)
struct FStopwatchState { GENERATED_BODY() ... };
```

### ArchiveTypes.h

```cpp
#pragma once
#include "CoreMinimal.h"
#include "ArchiveTypes.generated.h"

UENUM(BlueprintType)
enum class EArchPeriodFilter : uint8 { ... };

UENUM(BlueprintType)
enum class EArchModeFilter : uint8 { ... };

UENUM(BlueprintType)
enum class EDeleteCategoryMode : uint8 { ... };

USTRUCT(BlueprintType)
struct FCategoryDef { GENERATED_BODY() ... };

USTRUCT(BlueprintType)
struct FSessionRecord { GENERATED_BODY() ... };

USTRUCT(BlueprintType)
struct FArchiveDateRange { GENERATED_BODY() ... };

USTRUCT(BlueprintType)
struct FDayActivityDots { GENERATED_BODY() ... };

USTRUCT(BlueprintType)
struct FArchiveFilterState { GENERATED_BODY() ... };

USTRUCT(BlueprintType)
struct FArchiveQueryResult { GENERATED_BODY() ... };
```

### QuestTypes.h

```cpp
#pragma once
#include "CoreMinimal.h"
#include "QuestTypes.generated.h"

UENUM(BlueprintType)
enum class EQuestState : uint8 { ... };

USTRUCT(BlueprintType)
struct FQuestInstance { GENERATED_BODY() ... };
```

### FocusSubsystem.h

```cpp
#pragma once
#include "CoreMinimal.h"
#include "Subsystems/GameInstanceSubsystem.h"
#include "Types/FocusTypes.h"
#include "Types/ArchiveTypes.h"
#include "FocusSubsystem.generated.h"

UCLASS()
class RECORDTALES_API UFocusSubsystem : public UGameInstanceSubsystem
{
    GENERATED_BODY()

public:
    virtual void Initialize(FSubsystemCollectionBase& Collection) override;
    virtual void Deinitialize() override;

    UFUNCTION(BlueprintCallable) void StartTimer();
    UFUNCTION(BlueprintCallable) void StopTimer(ESessionEndReason Reason);
    UFUNCTION(BlueprintCallable) void TogglePause();
    UFUNCTION(BlueprintCallable) void StartStopwatch();
    UFUNCTION(BlueprintCallable) void StopStopwatch();
    UFUNCTION(BlueprintCallable) void ToggleSwPause();

    UPROPERTY(BlueprintReadOnly) EFocusMode         CurrentMode;
    UPROPERTY(BlueprintReadOnly) FTimerConfig        TimerConfig;
    UPROPERTY(BlueprintReadOnly) FTimerRuntimeState  RuntimeState;
    UPROPERTY(BlueprintReadOnly) FStopwatchState     SwState;

private:
    FSessionRecord CreateSessionRecord(ESessionEndReason Reason);
    FTimerHandle   TickHandle;
    void OnTick();
};
```

### ArchiveSubsystem.h

```cpp
#pragma once
#include "CoreMinimal.h"
#include "Subsystems/GameInstanceSubsystem.h"
#include "Types/ArchiveTypes.h"
#include "Types/QuestTypes.h"
#include "ArchiveSubsystem.generated.h"

UCLASS()
class RECORDTALES_API UArchiveSubsystem : public UGameInstanceSubsystem
{
    GENERATED_BODY()

public:
    virtual void Initialize(FSubsystemCollectionBase& Collection) override;

    // 기록
    UFUNCTION(BlueprintCallable) void    AddRecord(const FSessionRecord& Record);
    UFUNCTION(BlueprintCallable) FString GenerateRecordId();

    // 쿼리
    UFUNCTION(BlueprintCallable)
    FArchiveQueryResult QueryArchive(const FArchiveFilterState& Filter);

    UFUNCTION(BlueprintCallable)
    FArchiveDateRange ComputeDateRange(const FString& AnchorDateKey,
                                       EArchPeriodFilter Period);

    UFUNCTION(BlueprintCallable)
    TMap<FString, FDayActivityDots> ComputeActivityDotMap(int32 Year, int32 Month);

    // 카테고리
    UFUNCTION(BlueprintCallable) bool    AddCategory(const FString& Name, const FString& Color);
    UFUNCTION(BlueprintCallable) bool    RenameCategory(const FString& OldName, const FString& NewName);
    UFUNCTION(BlueprintCallable) bool    UpdateCategoryColor(const FString& Name, const FString& Color);
    UFUNCTION(BlueprintCallable) void    DeleteCategory(const FString& Name,
                                                         EDeleteCategoryMode Mode,
                                                         const FString& Target = TEXT(""));
    UFUNCTION(BlueprintCallable) FString GetCategoryColor(const FString& Name) const;
    UFUNCTION(BlueprintCallable) int32   GetRecordCountByCategory(const FString& Name) const;

    UPROPERTY(BlueprintReadOnly) TArray<FSessionRecord> AllRecords;
    UPROPERTY(BlueprintReadOnly) TArray<FCategoryDef>   Categories;
    UPROPERTY(BlueprintReadOnly) FArchiveFilterState     CurrentFilter;

private:
    class URecordTalesSaveGame* SaveGame = nullptr;
    void LoadFromSave();
    void SaveToDisk();
};
```

### RecordTalesSaveGame.h

```cpp
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/SaveGame.h"
#include "Types/FocusTypes.h"
#include "Types/ArchiveTypes.h"
#include "Types/QuestTypes.h"
#include "RecordTalesSaveGame.generated.h"

UCLASS()
class RECORDTALES_API URecordTalesSaveGame : public USaveGame
{
    GENERATED_BODY()

public:
    URecordTalesSaveGame();

    // Focus
    UPROPERTY() FTimerConfig LastTimerConfig;
    UPROPERTY() EFocusMode   LastFocusMode = EFocusMode::Timer;

    // Archive
    UPROPERTY() TArray<FSessionRecord> SessionRecords;
    UPROPERTY() TArray<FCategoryDef>   Categories;
    UPROPERTY() FString                LastRecordDate;
    UPROPERTY() int32                  DailyRecordSequence = 0;

    // Quest
    UPROPERTY() TArray<FQuestInstance> QuestInstances;
};
```

---

## 5. 작업 순서 가이드

파일 간 의존성 때문에 아래 순서대로 작성해야 컴파일 오류가 나지 않는다.

```
1단계 — Types (의존성 없음, 먼저 완성)
  FocusTypes.h
  QuestTypes.h
  ArchiveTypes.h

2단계 — SaveGame (Types 완성 후)
  RecordTalesSaveGame.h
  RecordTalesSaveGame.cpp

3단계 — Subsystem 헤더 (SaveGame 완성 후)
  QuestSubsystem.h
  FocusSubsystem.h
  ArchiveSubsystem.h

4단계 — Subsystem 구현 (헤더 완성 후)
  QuestSubsystem.cpp
  FocusSubsystem.cpp    ← ArchiveSubsystem 필요하므로 3번 이후
  ArchiveSubsystem.cpp  ← QuestSubsystem 필요하므로 1번 이후
```

---

## 업데이트 이력

| 날짜 | 내용 |
|---|---|
| 2026-06-09 | 초안 작성. Public/Private 구조, 파일별 담당 범위, 의존성, 스켈레톤, 작업 순서 정의 |
