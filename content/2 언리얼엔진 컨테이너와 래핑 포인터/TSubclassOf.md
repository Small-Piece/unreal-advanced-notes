# TSubclassOf

> 특정 클래스 계열만 담을 수 있는 타입 안전한 클래스 포인터. 기존 `UClass*`를 대체한다.

---

## 비유: 라벨이 붙은 바구니

`UClass*`는 아무 클래스나 담을 수 있는 빈 바구니. `TSubclassOf<AActor>`는 "AActor 계열만 넣으세요"라고 라벨이 붙은 바구니.

---

## TObjectPtr과의 차이

| 구분 | [[TObjectPtr]] | TSubclassOf |
|---|---|---|
| 담는 것 | 월드에 존재하는 **인스턴스** (레벨에 배치된 것) | 콘텐츠 브라우저에 있는 **클래스 자체** |
| 용도 | 이미 생성된 객체를 가리킴 | SpawnActor 등에 넣을 클래스를 지정 |

---

## 왜 UClass* 대신 쓰나?

### 1. 에디터 필터링

```cpp
// ❌ 필터링 안 됨 — 모든 클래스가 다 나옴
UPROPERTY(EditAnywhere)
UClass* MyClass;

// ✅ 필터링 됨 — UActorComponent 계열만 나옴
UPROPERTY(EditAnywhere)
TSubclassOf<UActorComponent> MyClass;
```

`UClass*`로 받으면 에디터 드롭다운에 관련 없는 클래스까지 전부 표시되어 실수로 잘못된 클래스를 고를 확률이 높아진다.

### 2. 컴파일 타임 안전성

빌드 시점에 호환되지 않는 클래스를 넣으려고 하면 **빌드 자체가 실패**한다.

### 3. 런타임 안전성

게임 도중 잘못된 클래스가 들어오면 **nullptr로 변환**해서 크래시를 방지한다.

---

## 사용 예시

```cpp
// 헤더
UPROPERTY(EditAnywhere, Category = "Spawning")
TSubclassOf<AActor> EnemyClass;

// CPP — SpawnActor에 클래스를 넘겨서 스폰
void AMySpawner::SpawnEnemy()
{
    if (EnemyClass)
    {
        GetWorld()->SpawnActor<AActor>(EnemyClass, SpawnLocation, SpawnRotation);
    }
}
```

### StaticClass()와의 관계

에디터에서 드롭다운으로 클래스를 선택하는 대신, 코드에서 직접 클래스를 지정할 때 사용:

```cpp
TSubclassOf<AActor> MyClass = AMyEnemy::StaticClass();
```

`StaticClass()`는 해당 클래스의 UClass 정보를 직접 반환하는 함수. 블루프린트에서 드롭다운으로 클래스를 골라 꽂아주는 것과 같은 역할.

---

## 관련 노트

- [[TObjectPtr]] — 인스턴스를 담는 포인터 (비교 대상)
- [[UPROPERTY]] — TSubclassOf도 UPROPERTY와 함께 사용
- [[CDO (Class Default Object)]] — SpawnActor 시 CDO에서 인스턴스가 복사됨
