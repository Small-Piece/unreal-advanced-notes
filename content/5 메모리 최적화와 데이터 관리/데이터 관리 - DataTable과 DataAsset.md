# 데이터 관리 - DataTable과 DataAsset

> 게임 데이터를 저장하고 관리하는 두 가지 방식. 표 형태의 DataTable과 개별 파일인 DataAsset을 상황에 맞게, 또는 섞어서 사용한다.

---

## DataTable vs DataAsset 비교

|구분|DataTable|DataAsset|
|---|---|---|
|비유|표 (엑셀)|개별 포장 상품|
|구조|여러 행을 한 파일에|하나하나 독립 파일|
|강점|수치 비교/일괄 편집|개별 로드, 비동기 친화적|
|약점|행 하나만 써도 전체 로드 / 비동기 관리 까다로움|1000개면 파일 1000개 / 일괄 비교 불편|

### DataTable이 좋은 경우

- 아이템 1000개의 공격력·가격·드랍률 같은 **수치 데이터**를 한눈에 관리
- 기획자가 엑셀로 편집 후 임포트
- "레벨 1~100 경험치 테이블" 같은 **순차 데이터**

### DataAsset이 좋은 경우

- 실행 중 **특정 아이템 하나만** 콕 집어 로드
- 무거운 메시·텍스처·사운드를 **비동기**로 불러올 때

> **결론: 둘을 섞어 쓴다.** 수치는 DataTable로, 무거운 에셋 연결은 DataAsset으로.

---

## DataTable 사용법

### 행 구조체 정의 (FTableRowBase 상속)

```cpp
#include "Engine/DataTable.h"

USTRUCT(BlueprintType)
struct FWeaponData : public FTableRowBase
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FString WeaponName;

    // 소프트 참조로 클래스 보관 → 당장 로드 안 함
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TSoftClassPtr<AActor> WeaponClass;
};
```

> [[에셋 참조와 소프트 포인터|TSoftClassPtr]]을 쓰면 테이블에 무거운 클래스를 넣어도 경로만 들고 있다가 필요할 때 로드한다.

### 랜덤 스폰 (동기 버전)

```cpp
void AWeaponBox::OpenBox()
{
    if (!WeaponTable) return;

    TArray<FWeaponData*> AllWeapons;
    WeaponTable->GetAllRows<FWeaponData>(TEXT(""), AllWeapons);  // 모든 행 가져오기
    if (AllWeapons.Num() == 0) return;

    // 랜덤 행 선택
    FWeaponData* Selected = AllWeapons[FMath::RandRange(0, AllWeapons.Num() - 1)];

    if (Selected)
    {
        UClass* LoadedClass = Selected->WeaponClass.LoadSynchronous();  // 동기 로드
        if (LoadedClass)
        {
            GetWorld()->SpawnActor<AActor>(LoadedClass, Location, FRotator::ZeroRotator);
        }
    }
}
```

### 비동기 버전

```cpp
#include "Engine/AssetManager.h"
#include "Engine/StreamableManager.h"

void AWeaponBox::OpenBox()
{
    // ... 랜덤 선택까지 동일 ...

    UAssetManager::GetStreamableManager().RequestAsyncLoad(
        Selected->WeaponClass.ToSoftObjectPath(),
        FStreamableDelegate::CreateUObject(
            this, &AWeaponBox::OnWeaponSpawnDeferred,
            Selected->WeaponClass  // 완료 후 함수로 배달될 인자
        )
    );
}

void AWeaponBox::OnWeaponSpawnDeferred(TSoftClassPtr<AActor> WeaponClassPtr)
{
    UClass* LoadedClass = WeaponClassPtr.Get();
    if (LoadedClass)
    {
        GetWorld()->SpawnActor<AActor>(LoadedClass, SpawnLocation, FRotator::ZeroRotator);
    }
}
```

---

## DataAsset + AssetManager

### 동작 원리

```
DataAsset 등록 시 ID도 함께 등록
        ↓
AssetManager가 ID를 수집해서 보관
        ↓
로드하고 싶을 때 ID만 넘김
        ↓
AssetManager: "이 ID 있네?" → Load 시작
        ↓
필요한 에셋을 받아옴
```

ID는 `FName` 기반이라, 보유 여부 확인이 매우 빠르다. 아이템 실물을 들고 다니며 검사할 필요가 없다.

### PrimaryDataAsset 정의

```cpp
UCLASS()
class UMyItemData : public UPrimaryDataAsset
{
    GENERATED_BODY()

public:
    // 에셋 매니저가 식별할 ID 규칙
    virtual FPrimaryAssetId GetPrimaryAssetId() const override
    {
        return FPrimaryAssetId(ItemType, GetFName());
    }

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item")
    FPrimaryAssetType ItemType;

    // 무거운 데이터는 TSoftObjectPtr + 번들 설정
    UPROPERTY(EditAnywhere, Category = "Visual", meta = (AssetBundles = "Mesh"))
    TSoftObjectPtr<USkeletalMesh> ItemMesh;

    UPROPERTY(EditAnywhere, Category = "Stats")
    float AttackPower;
};
```

### AssetID와 번들

|개념|역할|
|---|---|
|**AssetID**|에셋 매니저가 수집해서 보관하는 식별자. 대조 시 매니저에서 꺼내 비교 (실물 불필요)|
|**번들 (Bundle)**|부분 로드 단위. `AssetBundles = "Mesh"`로 묶어두면 Mesh만 골라서 로드 가능|

> 번들이 없으면 이름·공격력·메시를 전부 로드해야 한다. 번들로 "Mesh"만 콕 집어 로드할 수 있다.

### 로드 실행

```cpp
#include "Engine/AssetManager.h"

void AMyActor::StartLoading()
{
    if (!ItemToLoad.IsValid()) return;

    UAssetManager& AssetManager = UAssetManager::Get();
    TArray<FName> Bundles;  // 비워두면 전체 로드

    // 핸들 보관 필수! 안 하면 로딩 중 메모리에서 사라짐
    LoadingHandle = AssetManager.LoadPrimaryAsset(
        ItemToLoad,
        Bundles,
        FStreamableDelegate::CreateUObject(this, &AMyActor::OnLoadFinished, ItemToLoad)
    );
}

void AMyActor::OnLoadFinished(FPrimaryAssetId LoadedId)
{
    // ID로 데이터에셋 꺼내기
    LoadedItem = Cast<UMyItemData>(
        UAssetManager::Get().GetPrimaryAssetObject(LoadedId));
}
```

> `LoadingHandle`(`TSharedPtr<FStreamableHandle>`)을 멤버로 보관해야 한다. 안 그러면 로딩 중에 핸들이 GC되어 로드가 취소된다. → [[댕글링 포인터와 안전한 참조]]와 같은 맥락.

---

## 관련 노트

- [[에셋 참조와 소프트 포인터]] — TSoftObjectPtr / TSoftClassPtr 기반
- [[델리게이트]] — 비동기 로드 완료 콜백 (FStreamableDelegate)
- [[TSubclassOf]] — 클래스 참조 방식 비교
- [[댕글링 포인터와 안전한 참조]] — 로딩 핸들 보관의 중요성
- [[데미지 시스템]] — DataAsset의 AttackPower 등 수치 활용