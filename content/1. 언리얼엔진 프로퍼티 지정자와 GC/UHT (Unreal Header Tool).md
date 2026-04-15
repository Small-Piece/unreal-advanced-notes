# UHT (Unreal Header Tool)

> 컴파일러가 일하기 전에 먼저 돌면서, UCLASS·UPROPERTY·UFUNCTION 매크로를 스캔하고 [[리플렉션]] 코드를 자동 생성하는 도구

---

## 비유: 신분증 발급소

모든 학생(클래스)의 서류(헤더 파일)를 읽고, 매크로를 발견하면 학생증(`.generated.h`)을 발급해 준다.

학생증에 적히는 내용:
- "너는 `AMyActor`라는 클래스야"
- "`Health`라는 int32 변수가 있어"
- "`EditAnywhere`로 에디터에서 수정 가능해"
- "부모는 `AActor`야"

---

## 하는 일 (순서대로)

### 1. 매크로 스캔

헤더 파일에서 `UCLASS()`, `UPROPERTY()`, `UFUNCTION()` 매크로와 괄호 안의 메타데이터를 수집한다.

```cpp
// UHT가 이걸 읽고...
UPROPERTY(EditAnywhere, Category = "Stats")
int32 Health = 100;
```

### 2. .generated.h 파일 생성

수집한 정보를 바탕으로 `MyActor.generated.h` 파일을 자동 생성한다. 이 파일 안에는 해당 클래스를 엔진이 관리하기 위한 코드가 들어간다.

### 3. GENERATED_BODY() 매크로 확장

컴파일러가 클래스 안의 `GENERATED_BODY()`를 만나면, `.generated.h`에서 만든 매크로들을 클래스 내부에 심어준다. 이로써 엔진이 "나는 누구고, 누구랑 연결되어 있고"를 알게 된다.

---

## generated.h의 위치 규칙

```cpp
// ⚠️ 반드시 헤더 include 중 마지막에 위치해야 한다!
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MyActor.generated.h"   // ← 항상 마지막!
```

UHT가 수집한 정보를 읽으려면 다른 헤더들이 먼저 로드되어야 하기 때문.

---

## 헤더 수정 vs CPP 수정

| 수정 대상 | 일어나는 일 | 시간 |
|---|---|---|
| **CPP만 수정** | UHT 재실행 불필요 → 라이브 코딩으로 빠르게 반영 | 빠름 |
| **헤더 수정** | 신상 정보가 바뀐 것 → UHT가 다시 돌아야 함 → 전체 리빌드 | 느림 |

> 이것이 "초기화는 생성자(CPP)에서 하라"고 권장하는 이유. 헤더를 안 건드리면 [[UBT (Unreal Build Tool)]]도 UHT를 재실행하지 않는다.

---

## 빌드 흐름에서의 위치

```
UBT → ⭐ UHT (여기) → 컴파일러 → 엔진 초기화
```

[[빌드 타임라인]] 참고.

---

## 관련 노트

- [[리플렉션]] — UHT가 만들어내는 시스템
- [[UPROPERTY]] — UHT가 스캔하는 대표적인 매크로
- [[UFUNCTION과 UCLASS]] — UHT가 스캔하는 다른 매크로들
- [[UBT (Unreal Build Tool)]] — UHT를 실행시키는 상위 도구
- [[빌드 타임라인]] — 전체 흐름에서의 위치
