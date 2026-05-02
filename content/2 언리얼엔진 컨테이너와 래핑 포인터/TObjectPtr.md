# TObjectPtr

> UE5에서 도입된 템플릿 기반 포인터. 기존 `UObject*` 원시 포인터를 대체하며, 지연 로딩과 액세스 트래킹 기능을 제공한다.

---

## 한 줄 정리

"에디터와 개발자를 위한 똑똑한 포인터. 패키징하면 원시 포인터로 돌아간다."

---

## 기존 방식 vs UE5 권장 방식

```cpp
// UE4 방식
UPROPERTY(EditAnywhere)
USceneComponent* RootComponent;

// UE5 권장 방식
UPROPERTY(EditAnywhere)
TObjectPtr<USceneComponent> RootComponent;
```

---

## 핵심 기능 두 가지

### 1. 지연 로딩 (Lazy Loading)

에셋을 처음부터 전부 메모리에 올리지 않고, **실제로 접근할 때** 로드한다.

로드가 발생하는 시점:

```cpp
MyMesh->GetName();              // 멤버 함수 호출
UStaticMesh* RawMesh = MyMesh;  // 다른 변수에 대입
if (MyMesh != nullptr) { ... }  // 조건문 검사
```

수만 개의 에셋이 있는 프로젝트에서 클릭도 하지 않은 에셋 때문에 메모리가 터지는 걸 방지할 수 있다.

### 2. 액세스 트래킹 (Access Tracking)

[[가비지 컬렉션]]은 [[UPROPERTY]]로 선언된 변수를 추적하면서 어떤 객체가 사용 중인지 판단한다. TObjectPtr은 이 과정에서 **"이 객체가 어디서 참조되고 있는가?"**를 더 정밀하게 추적해 준다.

---

## 언제 쓰고 언제 안 쓰나?

| 상황 | 선택 | 이유 |
|---|---|---|
| 헤더에서 `UPROPERTY()` 멤버 변수 | `TObjectPtr<T>` | 지연 로딩 + 액세스 트래킹 필요 |
| 지역 변수, 매개변수 | `T*` 원시 포인터 | 이미 메모리에 올라온 객체를 잠깐 가리키는 것이므로 부가 기능 불필요 |

지역 변수는 함수가 끝나면 사라지므로 [[가비지 컬렉션]]의 추적 대상이 아니다. 따라서 TObjectPtr의 부가 기능이 필요 없다.

---

## 성능에 대한 오해

> "TObjectPtr을 쓰면 게임 성능이 좋아지나요?" → **아니요.**

최종 패키징을 하면 TObjectPtr은 **원시 포인터로 자동 변환**된다. 지연 로딩, 액세스 트래킹 전부 사라진다. 에디터와 개발 단계에서만 동작하는 **관리 도구**일 뿐이다.

실제 게임 최적화는 레벨 스트리밍이나 소프트 포인터 등으로 진행한다.

---

## 반복문 주의사항: auto* vs auto&

TObjectPtr이 담긴 컨테이너를 순회할 때는 `auto*` 대신 `auto&`를 사용해야 한다.

```cpp
UPROPERTY(EditAnywhere)
TArray<TObjectPtr<USceneComponent>> Components;

// ❌ auto* — 매번 주소 계산 + 로드 확인 로직이 실행됨
for (auto* Component : Components) { ... }

// ✅ auto& — TObjectPtr 자체를 참조하므로 내부 로직이 한 번만 실행됨
for (auto& Component : Components) { ... }
```

**이유:** `auto*`는 배열에서 주소를 꺼낼 때마다 TObjectPtr 내부의 "로드됐나 확인 → 주소 계산 → 반환" 로직이 매번 실행된다. `auto&`는 TObjectPtr 객체 자체를 참조로 가져오므로 이 과정을 건너뛴다.

---

## 관련 노트

- [[UPROPERTY]] — TObjectPtr은 UPROPERTY 멤버 변수에 사용
- [[가비지 컬렉션]] — 액세스 트래킹이 GC 추적을 보조
- [[TSubclassOf]] — 클래스 타입을 담는 또 다른 템플릿 포인터
- [[언리얼 컨테이너 개요]] — TObjectPtr을 담는 TArray 등의 컨테이너
