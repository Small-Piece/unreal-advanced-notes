# UFUNCTION과 UCLASS

> [[리플렉션]] 시스템에 함수와 클래스를 등록하는 매크로들

---

## UCLASS()

클래스를 언리얼 엔진에 등록한다. 이 매크로가 있어야 엔진이 해당 클래스를 인식하고, [[CDO (Class Default Object)]]를 생성하고, [[가비지 컬렉션]]으로 관리할 수 있다.

```cpp
UCLASS()
class MYPROJECT_API AMyActor : public AActor
{
    GENERATED_BODY()
    // ...
};
```

- `GENERATED_BODY()` — [[UHT (Unreal Header Tool)]]가 생성한 리플렉션 코드를 클래스 안에 삽입하는 매크로. 반드시 클래스 본문 최상단에 위치해야 함.

---

## UFUNCTION()

함수를 언리얼 엔진에 등록한다. 블루프린트에서 호출하거나, 에디터에서 보이게 하거나, 네트워크 리플리케이션에 사용할 수 있다.

```cpp
// 블루프린트에서 호출 가능한 함수
UFUNCTION(BlueprintCallable, Category = "Combat")
void TakeDamage(float Amount);

// 블루프린트에서 구현할 수 있는 이벤트
UFUNCTION(BlueprintImplementableEvent, Category = "Combat")
void OnDeath();

// 서버에서만 실행되는 RPC
UFUNCTION(Server, Reliable)
void ServerFire();
```

### 자주 쓰는 옵션

| 옵션 | 의미 |
|---|---|
| `BlueprintCallable` | 블루프린트에서 호출 가능 |
| `BlueprintPure` | 출력만 있고 실행 핀 없음 (Getter 용도) |
| `BlueprintImplementableEvent` | C++에서 선언, 블루프린트에서 구현 |
| `BlueprintNativeEvent` | C++에서 기본 구현, 블루프린트에서 오버라이드 가능 |
| `Server` / `Client` / `NetMulticast` | 네트워크 RPC 방향 |

---

## 세 매크로 한눈에 비교

| 매크로 | 대상 | 역할 |
|---|---|---|
| `UCLASS()` | 클래스 | 엔진에 클래스 등록, CDO 생성, GC 관리 |
| [[UPROPERTY|UPROPERTY()]] | 변수 | 에디터 노출, GC 추적, 네트워크 리플리케이션 |
| `UFUNCTION()` | 함수 | 블루프린트 호출, 네트워크 RPC |

세 매크로 모두 [[UHT (Unreal Header Tool)]]가 스캔해서 `.generated.h`에 리플렉션 정보를 기록한다.

---

## 관련 노트

- [[리플렉션]] — 이 매크로들이 속하는 시스템
- [[UPROPERTY]] — 변수 등록 매크로 (상세)
- [[UHT (Unreal Header Tool)]] — 매크로를 스캔하는 도구
- [[빌드 타임라인]] — 매크로가 처리되는 과정
