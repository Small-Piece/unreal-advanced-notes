# TMap

> Key와 Value를 쌍으로 저장하는 해시 기반 컨테이너. Key만 있으면 Value를 O(1)로 즉시 찾을 수 있다.

---

## 비유: 사물함

Key = 사물함 번호, Value = 안에 든 물건. 번호만 알면 바로 열어서 꺼낼 수 있다.

---

## 추가

```cpp
TMap<int32, FString> ItemMap;

ItemMap.Add(101, TEXT("Sword"));
ItemMap.Add(102, TEXT("Shield"));
ItemMap.Emplace(103, TEXT("Potion"));
```

> 중복 Key를 넣으면? → Value가 **덮어쓰기**된다.

---

## 검색 및 조회

### 안전한 방법 — Contains + Find

```cpp
if (ItemMap.Contains(101))
{
    FString* FoundItem = ItemMap.Find(101);
    if (FoundItem)
    {
        // *FoundItem 사용
    }
}
```

### 편리한 방법 — FindOrAdd

```cpp
// 키가 없으면 만들어서라도 반환
FString& ItemRef = ItemMap.FindOrAdd(104);
ItemRef = TEXT("Bow");
```

### 위험한 방법 — 대괄호 직접 접근

```cpp
// ❌ 키가 없으면 바로 크래시!
FString Name = ItemMap[105];

// ✅ 반드시 존재 확인 후 접근
if (ItemMap.Contains(105))
{
    FString Name = ItemMap[105];
}
```

---

## 순회

```cpp
// 범위 기반 for — TPair로 Key, Value 접근
for (const TPair<int32, FString>& Pair : ItemMap)
{
    int32 Key = Pair.Key;
    FString Value = Pair.Value;
}

// Iterator — 순회 중 삭제가 필요할 때
for (auto It = ItemMap.CreateIterator(); It; ++It)
{
    if (It.Key() == 103)
    {
        It.RemoveCurrent();  // 안전한 삭제
    }
}
```

> **Iterator를 쓰는 이유**: 순회 중 삭제할 때 일반 for문은 길을 잃을 수 있다. Iterator는 안전하게 다음 요소를 찾아준다. 삭제가 필요 없다면 범위 기반 for이 더 간결.

---

## 삭제

```cpp
ItemMap.Remove(102);  // Key 기준으로 삭제
```

---

## 메모리 정리

[[TSet]]과 동일하게 구멍이 쌓이면 정리 필요:

```cpp
ItemMap.Compact();  // 빈 공간을 뒤로 모음
ItemMap.Shrink();   // 모은 빈 공간을 반납
```

---

## 관련 노트

- [[언리얼 컨테이너 개요]] — TArray, TSet, TMap 비교
- [[TArray]] — 순서와 인덱스 접근이 필요할 때
- [[TSet]] — Key만 있고 Value가 필요 없을 때
