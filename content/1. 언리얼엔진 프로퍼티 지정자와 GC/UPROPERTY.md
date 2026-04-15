# UPROPERTY

> 변수를 언리얼 엔진에 등록하는 매크로. [[리플렉션]]과 [[가비지 컬렉션]]의 핵심 연결 고리.

---

## 한 줄 정리

"이 변수, 엔진이 알아야 해!" 라고 선언하는 이름표.

---

## UPROPERTY가 없으면 벌어지는 일

```cpp
// UPROPERTY 있음 → 엔진이 인식
UPROPERTY()
UMyTestObject* SafeObject;

// UPROPERTY 없음 → 엔진이 모름
UMyTestObject* DangerObject;
```

| 항목 | UPROPERTY 있음 | UPROPERTY 없음 |
|---|---|---|
| 에디터 디테일 패널 | 표시됨 | 표시 안 됨 |
| [[가비지 컬렉션]] 추적 | 추적됨 (삭제 방지) | 추적 안 됨 (언제든 삭제될 수 있음) |
| GC 후 포인터 상태 | `nullptr`로 정리됨 | 주소값 그대로 남음 → [[댕글링 포인터와 안전한 참조\|댕글링 포인터]] 위험 |
| 블루프린트 접근 | 설정에 따라 가능 | 불가 |
| 네트워크 리플리케이션 | 설정에 따라 가능 | 불가 |

---

## 자주 쓰는 메타데이터 옵션

UPROPERTY 괄호 안에 넣는 옵션들. [[UHT (Unreal Header Tool)]]가 이 메타데이터를 수집해서 `.generated.h`에 기록한다.

```cpp
// 에디터에서 읽기/쓰기 가능, 카테고리 지정
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Stats")
int32 Health = 100;

// 에디터에서 읽기만 가능
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Info")
FString Name;
```

### 에디터 노출 관련

| 옵션 | 의미 |
|---|---|
| `EditAnywhere` | 에디터 디테일 창에서 수정 가능 |
| `VisibleAnywhere` | 에디터에서 볼 수만 있고 수정 불가 |
| `EditDefaultsOnly` | 블루프린트 기본값에서만 수정 가능 |
| `EditInstanceOnly` | 월드에 배치된 인스턴스에서만 수정 가능 |

### 블루프린트 접근 관련

| 옵션 | 의미 |
|---|---|
| `BlueprintReadWrite` | 블루프린트에서 읽기/쓰기 가능 |
| `BlueprintReadOnly` | 블루프린트에서 읽기만 가능 |

---

## GC와의 관계: 이름표 줄 비유

[[가비지 컬렉션]]에서 UPROPERTY는 **이름표 줄** 역할을 한다.

- 뿌리(Root Set)에서 UPROPERTY 줄을 따라가면서 "이건 아직 쓰이는 물건"이라고 체크(마킹)
- 줄이 없는 객체는 고립된 것으로 판정 → 삭제
- UPROPERTY가 달린 포인터는 GC가 객체를 삭제할 때 자동으로 `nullptr`로 밀어줌

---

## 관련 노트

- [[리플렉션]] — UPROPERTY가 속하는 시스템
- [[가비지 컬렉션]] — UPROPERTY가 GC 추적의 핵심인 이유
- [[댕글링 포인터와 안전한 참조]] — UPROPERTY 없이 포인터를 쓸 때의 위험
- [[CDO (Class Default Object)]] — UPROPERTY 변수의 기본값이 CDO에 저장됨
