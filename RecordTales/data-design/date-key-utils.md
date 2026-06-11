# RecordTales UE — 날짜 키(DateKey) 유틸리티 설계

> `focus-archive-data.md`, `quest-data.md`에서 이름만으로 호출하던 날짜 함수들
> (`GetTodayDateKey`, `NextDateKey`, `AddDays`, `DayOfWeek` 등)을 정의하는 공용 유틸리티.
> 의존성 트리의 가장 아래에 위치 — 다른 모든 시스템이 이 모듈에 의존한다.

---

## 목차

1. [개요](#1-개요)
2. [설계 원칙](#2-설계-원칙)
3. [함수 목록](#3-함수-목록)
4. [구현](#4-구현)
5. [사용처 매핑](#5-사용처-매핑)
6. [엣지 케이스 & 열린 질문](#6-엣지-케이스--열린-질문)
7. [UE 배치 계획](#7-ue-배치-계획)

---

## 1. 개요

`FQuestMaster.StartDateKey`, `FSessionRecord.DateKey`, `RecordId`의 앞 8자리 등 — RecordTales 전반에서 날짜는 `"YYYYMMDD"` 형식의 8자리 문자열(DateKey)로 다룬다. 이 문자열에 대해 "다음 날", "N일 후", "요일", "두 날짜 사이 일수" 같은 연산이 `quest-data.md`(자정 롤오버, 발생일 판정)와 `focus-archive-data.md`(기간 필터, 캘린더 도트)에서 반복적으로 필요하다.

이 문서는 그 연산들을 **`UDateKeyLibrary`** 라는 단일 정적 함수 라이브러리로 정의한다.

---

## 2. 설계 원칙

| 원칙 | 내용 |
|---|---|
| **DateKey는 항상 "YYYYMMDD" 8자리 문자열** | 다른 형식(예: "YYYY.MM.DD")은 UI 표시 시점에만 변환 |
| **문자열 비교 = 날짜 비교** | "YYYYMMDD"는 사전식 정렬이 곧 시간순 정렬이므로 `Dk1 < Dk2` 같은 단순 문자열 비교를 그대로 날짜 비교에 사용 (이미 `RecordId` 정렬, `QuestOccursOn` 범위 체크 등에서 사용 중) |
| **내부 계산은 FDateTime 경유** | 윤년·월말 등 달력 규칙을 직접 구현하지 않고 UE의 `FDateTime`에 위임. DateKey ↔ FDateTime 변환 함수 2개가 핵심 |
| **요일은 일요일=0 기준** | HTML 프로토타입의 `Date.getDay()`(JS, 일요일=0)와 동일하게 맞춰서 기존 로직(`focus-archive-data.md`의 "일요일(dow=0)이면 6일 전 월요일로 조정" 등)을 그대로 포팅 가능하게 함. UE의 `EDayOfWeek`(월요일=0)과는 다르므로 변환 필요 |
| **무상태 정적 함수** | `UBlueprintFunctionLibrary` 기반, 멤버 변수 없음. SaveGame/Subsystem 어디서든 의존 가능 |

---

## 3. 함수 목록

| 함수 | 시그니처 | 설명 |
|---|---|---|
| `GetTodayDateKey` | `FString GetTodayDateKey()` | 현재 로컬 날짜를 `"YYYYMMDD"`로 반환 |
| `IsValidDateKey` | `bool IsValidDateKey(const FString& Dk)` | 8자리 숫자 + 유효한 월/일 범위인지 검증 |
| `AddDays` | `FString AddDays(const FString& Dk, int32 Days)` | `Dk` + N일. 음수 가능 (과거 방향) |
| `NextDateKey` | `FString NextDateKey(const FString& Dk)` | `AddDays(Dk, 1)` |
| `PrevDateKey` | `FString PrevDateKey(const FString& Dk)` | `AddDays(Dk, -1)` |
| `AddYears` | `FString AddYears(const FString& Dk, int32 Years)` | `Dk` + N년 (윤년 2/29 → 2/28 보정) |
| `DaysBetween` | `int32 DaysBetween(const FString& From, const FString& To)` | `To - From` (일수, 음수 가능) |
| `DayOfWeek` | `int32 DayOfWeek(const FString& Dk)` | 0=일 ~ 6=토 (JS `Date.getDay()` 호환) |
| `DayOfMonth` | `int32 DayOfMonth(const FString& Dk)` | 1~31 |
| `StartOfWeek` | `FString StartOfWeek(const FString& Dk)` | `Dk`가 속한 주의 월요일 |
| `EndOfWeek` | `FString EndOfWeek(const FString& Dk)` | `Dk`가 속한 주의 일요일 |
| `StartOfMonth` | `FString StartOfMonth(const FString& Dk)` | `"YYYYMM01"` |
| `EndOfMonth` | `FString EndOfMonth(const FString& Dk)` | 해당 월의 마지막 날 |
| `FormatDisplayDate` | `FString FormatDisplayDate(const FString& Dk)` | UI 표시용 `"YYYY.MM.DD"` 변환 |

---

## 4. 구현

### 4.1. 핵심 변환 — DateKey ↔ FDateTime

다른 모든 함수는 이 두 변환 위에서 동작한다.

```cpp
FDateTime UDateKeyLibrary::DateKeyToDateTime(const FString& Dk)
{
    int32 Year  = FCString::Atoi(*Dk.Mid(0, 4));
    int32 Month = FCString::Atoi(*Dk.Mid(4, 2));
    int32 Day   = FCString::Atoi(*Dk.Mid(6, 2));
    return FDateTime(Year, Month, Day);
}

FString UDateKeyLibrary::DateTimeToDateKey(const FDateTime& Dt)
{
    return FString::Printf(TEXT("%04d%02d%02d"), Dt.GetYear(), Dt.GetMonth(), Dt.GetDay());
}
```

### 4.2. 기본 함수

```cpp
FString UDateKeyLibrary::GetTodayDateKey()
{
    return DateTimeToDateKey(FDateTime::Now());
}

bool UDateKeyLibrary::IsValidDateKey(const FString& Dk)
{
    if (Dk.Len() != 8) return false;
    for (TCHAR C : Dk) { if (!FChar::IsDigit(C)) return false; }

    int32 Month = FCString::Atoi(*Dk.Mid(4, 2));
    int32 Day   = FCString::Atoi(*Dk.Mid(6, 2));
    return (Month >= 1 && Month <= 12 && Day >= 1 && Day <= 31);
}
```

### 4.3. 일 단위 연산

```cpp
FString UDateKeyLibrary::AddDays(const FString& Dk, int32 Days)
{
    FDateTime Dt = DateKeyToDateTime(Dk) + FTimespan::FromDays(Days);
    return DateTimeToDateKey(Dt);
}

FString UDateKeyLibrary::NextDateKey(const FString& Dk) { return AddDays(Dk, 1); }
FString UDateKeyLibrary::PrevDateKey(const FString& Dk) { return AddDays(Dk, -1); }

int32 UDateKeyLibrary::DaysBetween(const FString& From, const FString& To)
{
    FTimespan Diff = DateKeyToDateTime(To) - DateKeyToDateTime(From);
    return static_cast<int32>(Diff.GetDays());
}
```

### 4.4. 연 단위 연산

```cpp
FString UDateKeyLibrary::AddYears(const FString& Dk, int32 Years)
{
    int32 Year  = FCString::Atoi(*Dk.Mid(0, 4)) + Years;
    int32 Month = FCString::Atoi(*Dk.Mid(4, 2));
    int32 Day   = FCString::Atoi(*Dk.Mid(6, 2));

    // 2/29 + N년이 윤년이 아닌 해로 가면 2/28로 보정
    if (Month == 2 && Day == 29 && !FDateTime::IsLeapYear(Year))
    {
        Day = 28;
    }
    return FString::Printf(TEXT("%04d%02d%02d"), Year, Month, Day);
}
```

### 4.5. 요일 / 일자

UE `FDateTime::GetDayOfWeek()`는 `EDayOfWeek` (Monday=0 ... Sunday=6)을 반환한다. JS 스타일(Sunday=0 ... Saturday=6)로 변환한다.

```cpp
int32 UDateKeyLibrary::DayOfWeek(const FString& Dk)
{
    EDayOfWeek Dow = DateKeyToDateTime(Dk).GetDayOfWeek();
    // EDayOfWeek: Monday=0..Sunday=6  →  JS: Sunday=0..Saturday=6
    return (static_cast<int32>(Dow) + 1) % 7;
}

int32 UDateKeyLibrary::DayOfMonth(const FString& Dk)
{
    return DateKeyToDateTime(Dk).GetDay();
}
```

### 4.6. 주/월 범위 (focus-archive-data.md `ComputeDateRange` 용)

```cpp
FString UDateKeyLibrary::StartOfWeek(const FString& Dk)
{
    int32 Dow = DayOfWeek(Dk); // 0=일 ~ 6=토
    // 월요일까지 며칠 거슬러 올라가야 하는지: 일요일(0)이면 6일 전, 그 외엔 (Dow-1)일 전
    int32 BackOffset = (Dow == 0) ? 6 : (Dow - 1);
    return AddDays(Dk, -BackOffset);
}

FString UDateKeyLibrary::EndOfWeek(const FString& Dk)
{
    return AddDays(StartOfWeek(Dk), 6); // 월요일 + 6일 = 일요일
}

FString UDateKeyLibrary::StartOfMonth(const FString& Dk)
{
    return Dk.Left(6) + TEXT("01"); // "YYYYMM01"
}

FString UDateKeyLibrary::EndOfMonth(const FString& Dk)
{
    // 다음 달 1일에서 하루 빼면 이번 달 마지막 날
    int32 Year  = FCString::Atoi(*Dk.Mid(0, 4));
    int32 Month = FCString::Atoi(*Dk.Mid(4, 2));

    int32 NextMonth = Month + 1;
    int32 NextYear  = Year;
    if (NextMonth > 12) { NextMonth = 1; NextYear++; }

    FString FirstOfNextMonth = FString::Printf(TEXT("%04d%02d01"), NextYear, NextMonth);
    return PrevDateKey(FirstOfNextMonth);
}
```

### 4.7. 표시용 변환

```cpp
FString UDateKeyLibrary::FormatDisplayDate(const FString& Dk)
{
    // "20260611" → "2026.06.11"
    return FString::Printf(TEXT("%s.%s.%s"), *Dk.Mid(0, 4), *Dk.Mid(4, 2), *Dk.Mid(6, 2));
}
```

---

## 5. 사용처 매핑

| 함수 | 사용처 (문서 / 함수) |
|---|---|
| `GetTodayDateKey` | `quest-data.md` 전반 (`ComputeInstanceState`, `ProcessDailyRollover` 등), `focus-archive-data.md` `GenerateRecordId`, `InitFilterState` |
| `NextDateKey` / `PrevDateKey` | `quest-data.md` `ProcessDailyRollover`, `GenerateUpcomingInstance` (Cursor 전진) |
| `AddDays` | `quest-data.md` `GenerateInstanceIfNeeded`(마감일 = 발생일+오프셋) |
| `AddYears` | `quest-data.md` `QuestOccursOn`, `GenerateUpcomingInstance` (`RecEndDateKey` 기본값 = 시작일+1년) |
| `DaysBetween` | `quest-data.md` `ComputeDday` |
| `DayOfWeek` | `quest-data.md` `QuestOccursOn` (Weekly 반복 판정), `focus-archive-data.md` `ComputeDateRange` (Weekly) |
| `DayOfMonth` | `quest-data.md` `QuestOccursOn` (Monthly 반복 판정) |
| `StartOfWeek` / `EndOfWeek` | `focus-archive-data.md` `ComputeDateRange` (`EArchPeriodFilter::Weekly`) |
| `StartOfMonth` / `EndOfMonth` | `focus-archive-data.md` `ComputeDateRange` (`EArchPeriodFilter::Monthly`), `ComputeActivityDotMap`(해당 월 전체 날짜 초기화) |
| `FormatDisplayDate` | UI 위젯 — `FQuestMaster.StartDateKey` 등을 화면에 `"2026.06.11"` 형태로 표시할 때 |

---

## 6. 엣지 케이스 & 열린 질문

| 항목 | 내용 | 상태 |
|---|---|---|
| **자정 기준 시간대** | `GetTodayDateKey`가 `FDateTime::Now()`(로컬 시스템 시간) 기준. 시스템 시간 변경/시간대 이슈에 대한 방어는 범위 밖으로 둔다 | 미결 |
| **2/29 + AddYears 보정** | 2/28로 내림 처리. (예: 2024.02.29 + 1년 → 2025.02.28). HTML 프로토타입의 실제 동작과 일치하는지 확인 필요 | 미결 |
| **월요일 기준 주(週)** | `StartOfWeek`/`EndOfWeek`는 월~일 기준으로 고정. 이는 `focus-archive-data.md`의 "주간 계산 주의: 일요일(dow=0)이면 6일 전 월요일로 조정" 규칙을 그대로 반영한 것 — 변경 시 `quest-data.md`의 Weekly 반복 판정과는 무관(요일만 비교하므로 영향 없음) | ✅ 확정 |
| **DateKey 형식 위반 입력** | `IsValidDateKey`로 검증만 제공. 실제 호출부에서 어디까지 방어적으로 체크할지는 각 Subsystem 구현 시 결정 | 미결 |

---

## 7. UE 배치 계획

| 데이터/함수 | 위치 |
|---|---|
| `UDateKeyLibrary` (UBlueprintFunctionLibrary) | `DateKeyUtils.h` / `DateKeyUtils.cpp` (신규) |

### 의존성

`DateKeyUtils.h`는 **어떤 RecordTales 타입에도 의존하지 않는다** (엔진 기본 타입 `FString`, `FDateTime`, `FTimespan`만 사용). 따라서 `cpp-file-structure.md`의 작업 순서에서 **가장 먼저** 작성되어야 하며, `QuestTypes.h`, `FocusTypes.h`, `ArchiveTypes.h`, 각 Subsystem 모두 이 헤더를 `#include`한다.

```
작업 순서 갱신:
  0) DateKeyUtils.h / .cpp        ← 신규, 최우선
  1) FocusTypes.h, QuestTypes.h, ArchiveTypes.h  (DateKeyUtils.h 포함)
  2) RecordTalesSaveGame.h/.cpp
  3) Subsystem 헤더
  4) Subsystem .cpp
```

> `cpp-file-structure.md`에 0번 단계와 의존성 규칙("모든 헤더 → DateKeyUtils.h")을 추가 반영할 것.

---

## 업데이트 이력

| 날짜 | 내용 |
|---|---|
| 2026-06-11 | 초안 작성. `quest-data.md`/`focus-archive-data.md`에서 이름만 참조되던 날짜 유틸 함수 14개를 `UDateKeyLibrary`로 정의 |
