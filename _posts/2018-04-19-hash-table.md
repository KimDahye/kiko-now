---
layout: post
title: "Dynamic Programming과 Greedy Algorithm"
tags: [algorithm]
comments: true
---

함께하고 있는 [알고리즘 스터디](https://github.com/golden-girls/algorithm) 멤버들을 위해 Hash table 자료구조를 정리합니다.

## Background
- 목표: insert, delete, search 를 모두 평균 O(1)의 시간복잡도로 하고 싶다!
- If we have only `linked list`, `binary search tree`, and `array`, then...

  |   | insert   | search      |  delete | 
  |:----------|:-------------:|:------:| :----: |
  | Linked list | O(1) | O(N) | O(N) |
  | BST         | O(log(N)) | O(log(N)) | O(log(N)) |
  | Array       | O(1) | O(1) | O(1) |

### Array: direct addressing
Array can be a candidate. But it has the following shortcuts:
- Keys must be integer.
- Range of Keys must be small // 그렇지 않으면 무지막지한 공간낭비를 마주하게 되겠죠?!

## Hash table
내부적으로 Array를 데이터 저장공간으로 사용하지만, key가 꼭 integer일 필요 없다. Hash function을 이용하여 key를 integer 값으로 변환하여 그 값을 array 의 index로 사용한다!
  - `h(key) = index`  // "hashing 한다"

### 충돌을 피하는 2가지 방법: open addressing & separate chaining
#### open addressing
- 특징
  - 충돌이 나면 빈 slot을 찾을 때까지 인덱스를 찾는다. 즉, 기존 hash function `h(k)` 에 대해 충돌이 나면, `h(k, i) (i는 충돌 횟수)` function으로 다음 index 를 구한다. 
  - [단점] slot size 만큼만 key, value pairs를 저장할 수 있다.
  - [장점] 충돌이 났을 때 인근 slot에서 검색하던 key를 찾을 수도 있으므로(locality), cache hit ratio가 높다. 
- 종류
  - **linear probing**
    -  open addressing시에 `h(k,i)` 가 `h(k) + [i 에 대한 1차식]`으로 이루어진 식일 때 linear probing이라고 한다. 
    - 예시) `h(k) = k % 10` and  `h(k, i) = h(k) + c*i` where c = 1일 때
     
     <img src="https://user-images.githubusercontent.com/6873655/38993509-cbcc04d0-441e-11e8-9bec-4b50e884ad58.png" width=600/>
  - **quadratic probing**
    - open addressing시에 `h(k,i)` 가 `h(k) + [i 에 대한 2차식]`으로 이루어진 식일 때 quadratic probing이라고 한다. ex) `h(k,i) = h(k) + i^2`
  - 이외에도 probing 방법에 따라 다양한 open addressing이 있다. (double probing 등..) 

#### separate chaining
- 특징
  - 충돌이 나면, 해당 slot이 가리키는 linked list (or tree)에 해당 key, value 를 저장한다. 
  - [장점] open addressing과 달리 slot size 보다 큰 key, value pairs를 저장할 수 있다. 
  - [단점] 다만, cache hit ratio는 낮기 때문에, `저장할 key size / 전체 slot size`값이 어느정도 높아지면, 검색시 linked list or tree를 순회해야 하므로 성능이 떨어질 수 있다. 
    
    <img src="https://user-images.githubusercontent.com/6873655/38995985-24730718-4425-11e8-880e-9b1a6686abaf.png" width=200/>

### 시간 복잡도
따라서, insert, search, delete에 대해 평균 constant time으로 할 수 있다.

  |   | insert   | search      |  delete | findMin | findMax
  |:----------|:-------------:|:------:| :----: | :---: | :---: |
  | Hash table       | O(1) avg | O(1) avg | O(1) avg | O(N) | O(N) |

### 각 언어에서의 Hash Table
- Java: `HashMap`
  ```java
  HashMap<String, Integer> map = new HashMap<>();
  map.put("a", 1);
  map.put("b", 2);
  int a = map.get("a"); // a = 1;
  map.remove("b");
  bool contains = map.containsKey("b"); // contains = false;
  ```

## References
- https://www.comp.nus.edu.sg/~ooiwt/tp/cs1102-0203-s1/lecture/10-hash.pdf
- https://www.youtube.com/watch?v=h2d9b_nEzoA
- http://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/
