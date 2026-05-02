# TArray

> 언리얼의 동적 배열. C++ `std::vector`와 상통하며, 연속 메모리 구조로 인덱스 접근과 순회가 빠르다.

---

## 기본 사용

```cpp
TArray<int32> IntArray;

// 초기화: 값 10으로 5개 채우기
IntArray.Init(10, 5);  // [10, 10, 10, 10, 10]

// 개수 확인
int32 Count = IntArray.Num();
```

---

## 추가

```cpp
IntArray.Add(5);        // 임시 객체를 만들고 복사해서 추가
IntArray.Emplace(6);    // 복사 없이 직접 생성하여 추가 (성능 ↑)
IntArray.AddUnique(6);  // 중복이 없을 때만 추가 (내부 순회 필요 → 느림)
IntArray.Insert(500, 3); // 3번 인덱스에 500 삽입
```

### Add vs Emplace

| 구분 | Add | Emplace |
|---|---|---|
| 동작 | 임시 공간에 만들고 복사 | 배열 내부에 직접 생성 |
| 성능 | 약간 느림 | 빠름 |
| 안전성 | 이미 만든 객체를 복사하므로 명확 | 암시적 변환이 끼어들 여지 있음 |
| 사용 시기 | 일반적인 경우 | 성능을 쥐어짜야 할 때 |

> `AddUnique`는 내부적으로 배열을 순회하므로 성능이 안 좋다. 중복 방지가 필요하면 [[TSet]] 사용 권장.

---

## 접근

```cpp
IntArray[5] = 1000000;  // 인덱스로 즉시 접근 — O(1)
```

---

## 탐색

```cpp
int32 Index = IntArray.Find(30);     // 값의 인덱스 반환, 없으면 -1
bool bExists = IntArray.Contains(30); // 존재 여부만 true/false
```

---

## 정렬

```cpp
IntArray.Sort();  // 오름차순

// 내림차순 — 람다 함수 사용
IntArray.Sort([](int32 A, int32 B) { return A > B; });
```

### 람다 함수란?

함수를 따로 만들기 귀찮을 때 그 자리에서 바로 만드는 익명 함수.

```cpp
[캡처](매개변수) { 본문 }

// 캡처 옵션
// [&] — 외부 변수를 참조로 가져옴 (원본 수정 가능)
// [=] — 외부 변수를 복사로 가져옴 (안전)
// []  — 매개변수로만 동작
```

---

## 필터링

```cpp
// 9보다 작은 값만 새 배열로 반환
TArray<int32> Filtered =
    IntArray.FilterByPredicate([](int32 Value) { return Value < 9; });
```

인벤토리 구현 시 자주 사용되는 패턴.

---

## 삭제

```cpp
IntArray.Remove(10);           // 값 10을 전부 삭제
IntArray.RemoveSingle(10);     // 첫 번째로 찾은 10만 삭제
IntArray.RemoveAt(3);          // 인덱스 3번 삭제 (범위 체크 필수!)

// 안전한 인덱스 삭제
if (IntArray.IsValidIndex(Index))
{
    IntArray.RemoveAt(Index);
}

// 조건부 삭제
IntArray.RemoveAll([](int32 Value) { return Value == 5; });

IntArray.Empty();  // 전부 삭제
```

---

## 순회

```cpp
// 범위 기반 for (가장 간단)
for (const int32 Value : IntArray)
{
    // Value 사용
}

// 인덱스가 필요할 때
for (int32 i = 0; i < IntArray.Num(); i++)
{
    // IntArray[i] 사용
}
```

> [[TObjectPtr]]이 담긴 TArray를 순회할 때는 반드시 `auto&`를 사용할 것.

---

## 관련 노트

- [[언리얼 컨테이너 개요]] — TArray, TSet, TMap 비교
- [[TSet]] — 중복 방지 + 빠른 탐색이 필요할 때
- [[TMap]] — Key-Value 구조가 필요할 때
- [[TObjectPtr]] — TArray에 담을 때 auto& 주의
