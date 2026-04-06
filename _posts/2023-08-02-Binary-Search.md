---
title: "[Algorithm] 이진 탐색(Binary Search) 알고리즘 이해하기"
date: 2023-08-02T17:30:00
categories: [algorithm, javascript]
tags: [algorithm, javascript, binary-search, search-algorithm]
---

코딩테스트 문제를 풀고 자신 있게 제출했는데, 결과창에 **'시간 초과(Time Limit Exceeded)'**가 뜨면 참 허탈하죠. 저 역시 선형 탐색($O(n)$)의 한계에 부딪혔을 때, 탐색 효율을 비약적으로 높여주는 **이진 탐색(Binary Search)**을 만나며 알고리즘의 중요성을 실감했습니다.

자바스크립트 개발자를 위한 이진 탐색의 개념과 구현 방법을 정리

---

## 1. 이진 탐색(Binary Search)이란?

**이진 탐색(Binary Search)**은 정렬된 배열에서 특정 값을 효율적으로 찾는 알고리즘입니다. 이 알고리즘의 핵심은 반드시 데이터가 **정렬된 상태**여야 한다는 점입니다.

### 동작 원리

이진 탐색은 배열의 중간 값과 찾고자 하는 값을 비교하여 탐색 범위를 매 단계마다 **절반**으로 줄여나갑니다. 이로 인해 데이터가 아무리 많아도 매우 빠른 속도로 원하는 값을 찾을 수 있습니다.

- **시간 복잡도:** $O(\log n)$
- **전제 조건:** 데이터가 반드시 정렬되어 있어야 함 (JS의 경우 `sort()` 활용)

---

## 2. 왜 이진 탐색을 써야 할까? (Performance)

우리가 흔히 쓰는 `Array.prototype.indexOf`나 `find`는 배열의 처음부터 끝까지 확인하는 **선형 탐색**입니다.

- **데이터 1,000,000개일 때:**
  - 선형 탐색: 최대 1,000,000번 비교
  - 이진 탐색: 최대 **20번**의 비교로 끝

이처럼 $O(\log n)$의 효율성은 대규모 데이터를 다루는 코딩테스트나 프론트엔드 환경에서 필수적인 최적화 기술입니다.

---

## 3. JavaScript를 이용한 구현

반복문(While)을 이용한 가장 효율적인 자바스크립트 구현 방식입니다.

```javascript
/**
 * 이진 탐색 함수
 * @param {Number[]} arr - 정렬된 숫자 배열
 * @param {Number} targetValue - 찾고자 하는 값
 * @returns {Number} - 찾은 값의 인덱스 (없을 경우 -1)
 */
function binarySearch(arr, targetValue) {
  let left = 0;
  let right = arr.length - 1;

  while (left <= right) {
    // (left + right) / 2 시 발생할 수 있는 오버플로우 방지 및 정수화
    const mid = Math.floor(left + (right - left) / 2);

    if (arr[mid] === targetValue) {
      return mid; // 값을 찾으면 인덱스 반환
    } else if (arr[mid] < targetValue) {
      left = mid + 1; // 중간 값보다 크면 오른쪽 반 탐색
    } else {
      right = mid - 1; // 중간 값보다 작으면 왼쪽 반 탐색
    }
  }

  return -1; // 값을 찾지 못한 경우
}

const array = [1, 2, 3, 4, 5, 6, 7, 8, 9].sort((a, b) => a - b);
const targetValue = 7;

const index = binarySearch(array, targetValue);
console.log(`찾은 인덱스: ${index}`); // 출력: 6
```

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<p>단순히 정답을 맞히는 것을 넘어, 효율적인 알고리즘을 선택하는 것은 프론트엔드 성능 최적화의 첫걸음입니다.</p>

<p>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.

지적이나 피드백은 언제나 환영입니다.</p>
