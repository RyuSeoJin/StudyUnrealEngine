# RecordTales UE — 집중·아카이브 시스템 데이터 설계

> 집중(타이머/스톱워치) 시스템과 아카이브 시스템의 C++ 데이터 구조 설계서.
> 코딩 전 구조체·열거형·관계를 확정하기 위한 문서입니다.

---

## 목차

1. [설계 원칙](#1-설계-원칙)
2. [런타임 vs 영속 데이터 구분](#2-런타임-vs-영속-데이터-구분)
3. [열거형 (Enum)](#3-열거형-enum)
4. [구조체 (Struct)](#4-구조체-struct)
5. [관계도](#5-관계도)
6. [UE 배치 계획](#6-ue-배치-계획)
7. [열린 질문](#7-열린-질문)

---

## 1. 설계 원칙

| 원칙 | 내용 |
|---|---|
| **런타임 / 영속 분리** | 세션이 진행 중일 때만 필요한 데이터(남은 시간, 사이클 번호)는 저장하지 않는다 |
| **기록은 종료 시 1회 생성** | 세션이 끝날 때 `FSessionRecord` 한 개를 만들어 아카이브에 추가한다 |
| **Blueprint 노출 최소화** | UPROPERTY는 UI에 필요한 것만 BlueprintReadWrite, 내부 계산용은 노출하지 않는다 |
| **날짜 키 통일** | 날짜 식별자는 `"YYYYMMDD"` 8자리 문자열로 통일한다 |

---

## 2. 런타임 vs 영속 데이터 구분

```
집중 세션 진행 중 (런타임 - 저장 안 함)
├── FTimerRuntimeState   타이머 현재 단계·남은 시간·사이클
├── FStopwatchState      스톱워치 가동 여부·경과 시간
└── FTimerConfig         시작 시 복사된 설정값 (설정 변경이 진행 중에 영향 주지 않도록)

세션 종료 시 생성 (영속 - 저장)
└── FSessionRecord       기록 1건 → 아카이브에 추가
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
> LastResting → Focusing           (autoRestart=true, cycleIndex=1 리셋)
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

> 세션 종료 함수 시그니처에서 사용. FSessionRecord에는 bool bNormalEnd 로 저장.

---

## 4. 구조체 (Struct)

### 4.1. FTimerConfig — 타이머 설정값 (영속)

사용자가 설정한 값. 앱 종료 후에도 유지되어야 하므로 SaveGame 저장 대상.

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
> - 휴식 필드는 SingleRestDuration 하나만 사용
> - LastResting 단계이지만 페이즈 라벨은 "휴식"으로 표시

---

### 4.2. FTimerRuntimeState — 타이머 런타임 상태 (비영속)

세션이 진행 중일 때만 존재. 저장하지 않음.

```cpp
USTRUCT(BlueprintType)
struct FTimerRuntimeState
{
    GENERATED_BODY()

    // 현재 단계
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

    // 메모 (Record Modal에서 사용자 입력)
    UPROPERTY(BlueprintReadWrite)
    FString Notes;

    // 기록 시각
    UPROPERTY(BlueprintReadWrite)
    FDateTime RecordedAt;

    // 날짜 키 — "YYYYMMDD" 8자리. 날짜별 조회에 사용
    UPROPERTY(BlueprintReadWrite)
    FString DateKey;
};
```

> **DurationSeconds vs FocusSeconds**
> | 항목 | DurationSeconds | FocusSeconds |
> |---|---|---|
> | 표시 용도 | 링 UI, "총 N분 집중" | XP 계산, 통계 |
> | 타이머 루틴 예시 | 집중25 + 휴식5 × 4 = 120분 | 집중만 25 × 4 = 100분 |
> | 스톱워치 | 동일 | 동일 |

---

### 4.5. FDailyArchive — 날짜별 아카이브 묶음 (비영속)

아카이브 UI 조회 시 즉석 생성. 저장하지 않음.

```cpp
USTRUCT(BlueprintType)
struct FDailyArchive
{
    GENERATED_BODY()

    // 날짜 키 — "YYYYMMDD"
    UPROPERTY(BlueprintReadOnly)
    FString DateKey;

    // 해당 날짜의 세션 기록 (RecordedAt 기준 오름차순)
    UPROPERTY(BlueprintReadOnly)
    TArray<FSessionRecord> Sessions;

    // 편의 함수 (헤더 선언, cpp 구현)
    // int32 GetTotalFocusSeconds() const;
    // int32 GetSessionCount()      const;
};
```

---

## 5. 관계도

```
[사용자 조작]
      │
      ▼
UFocusSubsystem  (UGameInstanceSubsystem)
  ├── EFocusMode              현재 모드
  ├── FTimerConfig            설정값 (영속)
  ├── FTimerRuntimeState      타이머 진행 상태 (비영속)
  ├── FStopwatchState         스톱워치 진행 상태 (비영속)
  │
  └── 세션 종료 시 → FSessionRecord 생성
                          │
                          ▼
                  UArchiveSubsystem  (UGameInstanceSubsystem)
                    └── TArray<FSessionRecord> AllRecords  (영속)
                              │
                              ▼
                    GetDailyArchive(DateKey)
                      └── FDailyArchive  (비영속, 즉석 생성)
                                │
                                ▼
                        [아카이브 UI 위젯]
```

---

## 6. UE 배치 계획

| 데이터 | 위치 | SaveGame 저장 |
|---|---|---|
| `EFocusMode`, `ETimerPhase`, `ESessionEndReason` | `FocusTypes.h` | — |
| `FTimerConfig` | `FocusTypes.h` | ✅ |
| `FTimerRuntimeState` | `FocusTypes.h` | ❌ |
| `FStopwatchState` | `FocusTypes.h` | ❌ |
| `FSessionRecord` | `ArchiveTypes.h` | ✅ |
| `FDailyArchive` | `ArchiveTypes.h` | ❌ |
| `UFocusSubsystem` | `FocusSubsystem.h/.cpp` | — |
| `UArchiveSubsystem` | `ArchiveSubsystem.h/.cpp` | — |
| `URecordTalesSaveGame` | `RecordTalesSaveGame.h/.cpp` | — |

### SaveGame 구조 (현재 범위)

```cpp
UCLASS()
class URecordTalesSaveGame : public USaveGame
{
    GENERATED_BODY()

public:
    // 마지막 타이머 설정 유지
    UPROPERTY()
    FTimerConfig LastTimerConfig;

    UPROPERTY()
    EFocusMode LastFocusMode = EFocusMode::Timer;

    // 세션 기록 전체
    UPROPERTY()
    TArray<FSessionRecord> SessionRecords;

    // 이후 시스템 추가 시 여기에 확장
    // UPROPERTY() TArray<FQuestData> Quests;
    // UPROPERTY() int32 Gold;
    // UPROPERTY() int32 GuildLevel;
};
```

---

## 7. 열린 질문

| 항목 | 내용 | 상태 |
|---|---|---|
| **카테고리 관리** | 목록을 DataTable로 관리할지, SaveGame 배열로 관리할지 | 미결 |
| **RecordId 생성** | 날짜+시퀀스 방식 vs FGuid 방식 | 미결 |
| **세션 중 앱 종료** | 진행 중 상태를 SaveGame에 임시 저장할지 | 미결 |
| **최대 기록 수** | 무제한 보관 vs 일정 기간 후 정리 정책 | 미결 |

---

## 업데이트 이력

| 날짜 | 내용 |
|---|---|
| 2026-06-08 | 초안 작성. 집중·아카이브 시스템 데이터 설계 |
