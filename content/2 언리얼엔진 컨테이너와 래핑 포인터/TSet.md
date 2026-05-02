# TSet

> 해시 기반 집합 컨테이너. 중복을 허용하지 않으며, 탐색·삽입·삭제가 O(1)로 빠르다. 순서는 보장하지 않는다.

---

## 핵심 특징

- **중복 방지** — 같은 값이 들어오면 무시
- **인덱스 접근 불가** — 비연속 메모리 구조
- **탐색·삽입·삭제 O(1)** — 해시 테이블 기반

---

## 추가

```cpp
TSet<int32> IntSet;

IntSet.Add(100);
IntSet.Add(200);
IntSet.Add(100);  // 중복 → 무시됨
```

---

## 탐색

```cpp
// 존재 여부 확인
if (IntSet.Contains(200))
{
    // 있다!
}

// 값 찾기 — 인덱스가 없으므로 포인터로 반환
int32* FoundPtr = IntSet.Find(200);
if (FoundPtr)
{
    // *FoundPtr 사용
}
```

---

## 순회 — Iterator 사용

TSet은 비연속 메모리라서 중간에 구멍이 있을 수 있다. 다음 값이 있는 곳으로 찾아가야 하므로 **Iterator**를 사용한다.

```cpp
// 읽기 전용 순회
for (auto It = IntSet.CreateConstIterator(); It; ++It)
{
    int32 Value = *It;
}

// 수정 가능 순회 (삭제 가능)
for (auto It = IntSet.CreateIterator(); It; ++It)
{
    if (*It < 60)
    {
        It.RemoveCurrent();  // 순회 중 안전하게 삭제
    }
}
```

> **삭제하면서 순회할 때는 반드시 Iterator를 사용할 것.** 일반 범위 기반 for에서 삭제하면 길을 잃을 수 있다.

---

## 집합 연산

```cpp
TSet<int32> SetA = { 1, 2, 3 };
TSet<int32> SetB = { 3, 4, 5 };

// 교집합 — 공통된 부분만
TSet<int32> Intersection = SetA.Intersect(SetB);  // { 3 }

// 합집합 — 두 집합 합치기
TSet<int32> Union = SetA.Union(SetB);  // { 1, 2, 3, 4, 5 }
```

---

## 메모리 정리

삭제를 반복하면 구멍이 많이 생겨서 쓸모없는 메모리 공간이 낭비된다.

```cpp
IntSet.Compact();  // 구멍들을 뒤로 밀어서 모아놓음
IntSet.Shrink();   // 모아놓은 빈 공간을 반납
```

---

## 관련 노트

- [[언리얼 컨테이너 개요]] — TArray, TSet, TMap 비교
- [[TArray]] — 순서가 필요할 때
- [[TMap]] — Key-Value 구조가 필요할 때
