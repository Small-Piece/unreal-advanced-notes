# 콜리전과 Trace

> 보이지 않는 광선을 쏴서 "누가 거기 있는지" 확인하는 기능. [[콜리전 시스템]]의 설정에 따라 감지 여부가 결정된다.

---

## Trace를 쓰는 이유

- 몬스터가 주변에 누가 있는지 탐지
- 칼에 부착해서 누가 맞았는지 확인
- 총알 대신 Trace로 히트 판정 (멀티플레이에서 총알 물리 시뮬레이션은 부하가 크기 때문)
- 발 밑에 쏴서 어떤 재질을 밟고 있는지 확인

---

## 두 가지 Trace 경로

|구분|UKismetSystemLibrary|UWorld|
|---|---|---|
|대상|블루프린트 호환용 래퍼|엔진 네이티브|
|디버깅|매개변수로 디버그 드로우 내장|DrawDebugLine 등 별도 호출 필요|
|자유도|매개변수 정리가 잘 되어 있음|FCollisionResponseParams로 상호작용 타입 변경 가능|
|속도|약간 느림 (래퍼 오버헤드)|약간 빠름 (미미한 차이)|

> **실전 팁**: Kismet으로 디버깅 열심히 하고, 최적화 단계에서 UWorld로 교체.

---

## SingleTrace — 첫 번째 Block만 감지

```cpp
FHitResult HitResult;
TArray<AActor*> ActorsToIgnore;
ActorsToIgnore.Add(this);

UKismetSystemLibrary::LineTraceSingle(
    GetWorld(),
    GetActorLocation(),
    GetActorForwardVector() * 1000.f + GetActorLocation(),
    UEngineTypes::ConvertToTraceType(ECC_Visibility),
    false,                          // bTraceComplex
    ActorsToIgnore,
    EDrawDebugTrace::ForOneFrame,
    HitResult,
    true,                           // bIgnoreSelf
    FLinearColor::Red,              // 트레이스 색
    FLinearColor::Green             // 히트 색
);
```

> **SingleTrace는 Block만 인식한다.** Overlap은 감지하지 않음.

---

## MultiTrace — 관통하며 여러 오브젝트 감지

SingleTrace와 코드 두 줄만 변경:

```cpp
TArray<FHitResult> HitResults;  // 단일 → 배열

UKismetSystemLibrary::LineTraceMulti(  // Single → Multi
    // ... 나머지 동일
    HitResults,
    // ...
);
```

> **MultiTrace는 Overlap까지 인식한다.** 관통하면서 만나는 모든 오브젝트를 감지.

---

## 비동기 Trace — 백그라운드에서 처리

### 왜 필요한가?

Trace를 대량으로 쏘면 메인 스레드(작업대)가 막힌다.

- **동기 방식**: 작업대 하나로 차근차근 → Trace 끝날 때까지 게임 멈춤
- **비동기 방식**: 다른 작업대에 맡기고 내 할 일 하기 → 결과가 오면 받아서 처리

### 구현

```cpp
// 헤더
void StartAsyncTrace();
void OnAsyncTraceCompleted(const FTraceHandle& Handle, FTraceDatum& Data);
FTraceHandle AsyncTraceHandle;
```

```cpp
// CPP
void ATraceTest::StartAsyncTrace()
{
    FTraceDelegate TraceDelegate;
    TraceDelegate.BindUObject(this, &ATraceTest::OnAsyncTraceCompleted);

    // 상호작용 타입 강제 변경 (UWorld Trace만 가능)
    FCollisionResponseParams ResponseParams;
    ResponseParams.CollisionResponse.WorldDynamic = ECR_Overlap;

    FCollisionQueryParams QueryParams;
    QueryParams.AddIgnoredActor(this);
    QueryParams.bTraceComplex = false;

    AsyncTraceHandle = GetWorld()->AsyncLineTraceByChannel(
        EAsyncTraceType::Multi,
        GetActorLocation(),
        GetActorForwardVector() * 1000.f + GetActorLocation(),
        ECC_Visibility,
        QueryParams,
        ResponseParams,
        &TraceDelegate
    );
}

void ATraceTest::OnAsyncTraceCompleted(const FTraceHandle& Handle, FTraceDatum& Data)
{
    for (const FHitResult& Hit : Data.OutHits)
    {
        AActor* HitActor = Hit.GetActor();
        DrawDebugSphere(GetWorld(), Hit.ImpactPoint, 20.f, 12, FColor::Green, false, 2.f);
    }
}
```

### EAsyncTraceType 옵션

|타입|설명|
|---|---|
|`Single`|첫 번째 히트만 반환|
|`Multi`|모든 히트 반환|
|`Test`|히트 여부만 확인 (Data 비어 있음). `GetActor()` 호출 시 크래시!|

### FCollisionResponseParams 주의사항

트레이스 대상의 상호작용 타입을 강제 변경할 수 있지만, [[콜리전 시스템]]의 우선순위 규칙이 적용된다:

|현재 상태|변경 시도|결과|
|---|---|---|
|Block → Overlap|✅ 가능||
|Block → Ignore|✅ 가능||
|Overlap → Ignore|✅ 가능||
|Ignore → Block|❌ 불가||
|Ignore → Overlap|❌ 불가||

> 몬스터 정찰/탐지 시스템에 비동기 Trace가 적합. 언리얼의 AI Perception보다 가볍다.

---

## Trace 매개변수 정리

|인자|타입|설명|
|---|---|---|
|WorldContextObject|`const UObject*`|월드 접근용 (보통 `this` 또는 `GetWorld()`)|
|Start / End|`FVector`|시작점과 끝점|
|TraceChannel|`ETraceTypeQuery`|[[콜리전 시스템]]의 Trace Channel|
|bTraceComplex|`bool`|true면 복합 콜리전(폴리곤 단위) 사용|
|ActorsToIgnore|`TArray<AActor*>`|무시할 액터 목록|
|DrawDebugType|`EDrawDebugTrace`|디버그 선 표시 방식|
|OutHit|`FHitResult&`|충돌 결과|
|bIgnoreSelf|`bool`|자기 자신 무시 여부|

> `ActorsToIgnore.Add(this)`와 `bIgnoreSelf=true`를 **같이 쓰는 이유**: 플레이어에 컴포넌트가 붙어 있으면 bIgnoreSelf만으로 부족할 수 있어서 안전성 확보.

---

## 관련 노트

- [[콜리전 시스템]] — 트레이스의 기반이 되는 충돌 설정
- [[데미지 시스템]] — 트레이스 히트 결과로 데미지 적용하기
- [[TArray]] — ActorsToIgnore, HitResults 등 배열 활용