---
layout: post
title: "Dynamic Programming과 Greedy Algorithm"
tags: [algorithm]
comments: true
---

다이나믹 프로그래밍(Dynamic Programming)과 그리디 알고리즘(Greedy Algorithm)은 모두 최적화 문제를 풀 때 사용하는 알고리즘 기법이다. (두 기법 모두 알고리즘 자체는 아니고, 알고리즘을 짜기 위한 기법(or 전략)인데, 왜 하나는 Programming이 붙고, 다른 하나는 Algorithm이 붙을까? 그 이유는 모르겠다) 두 기법은 모두 비슷한 문제를 푸는데 사용되기 때문에 언뜻 보면 두 개의 차이가 애매모호할 수 있다. 오늘은 이 두 기법의 개념을 정리하여 차이점을 확실히 알아두려 한다. 개념 정리 부분은 NHN NEXT의 자료구조 및 알고리즘 과목 lecture note를 참고하였다.

## Dynamic Programming 개념
* 용도 : 최적화 문제 등
* 구조
  1. 주어진 문제를 여러 개의 작은 문제로 분할한다. 특히, 최적화 문제의 경우 분할 가능한 모든 경우를 고려한다.
  1. 작은 문제의 해를 이용해서 주어진 문제의 해를 구한다.
  1. 작은 문제 중 이미 해를 구한 것은 재사용한다. (memoization)
  1. 작은 문제가 더 작은 문제로 분할되어 재귀 호출되는 과정에서 동일한 문제가 중복 발생할 경우 처음에 계산된 결과를 저장한 후 재호출 시 이전에 계산된 결과를 바로 return 한다.
* 특징
  *  Brute-force 방식의 접근이다.
  * 모든 경우를 확인할 경우 exponential time algorithm 이지만, **중간 계산 결과를 재활용하면 (즉, memoization하면) polynomial time으로 개선된다.**
  * **[optimal substructure]** 주어진 문제의 optimal solution은 subproblem의 optimal solution을 포함한다. 이 조건을 만족하지 못하면 Dynamic Programming 기법을 적용할 수 없다!
  * 재귀적으로 구현하거나(top down), iteration을 이용해 구현(bottom up)할 수 있는데, 두 구현방식 모두 subproblem으로 이루어진 tree의 **topological sorting 순서대로** 문제가 풀린다.

## Greedy Algorithm 개념
* 용도 : 최적화 문제
* 구조
  1. 전체 optimal solution을 구할 수 있는 local optimal choice를 발견한다.
  2. 발견한 local optimal choice가 전체 optimal solution을 만들 수 있는지 증명한다.
  3. 증명한 local optimal choice에 따라 알고리즘을 구성한다.
* 특징  
  * **[optimal substructure]** 주어진 문제의 optimal solution은 subproblem의 optimal solution을 포함한다.
  * **[Greedy-choice property]** 전체적인 optimal solution은 locally optimal (greedy) choice에 의해서 구성될 수 있다. 이 때, 매 step 에서 locally optimal choice를 할 경우 globally optimal solution을 구해진다는 것은 증명되어야 한다.

##  Dynamic Programming vs. Greedy Algorithm
* 공통점
  * Optimal substructure를 사용한다.
* 차이점
  * Dynamic programming은 매 step에서 여러 개의 sub-problem 해결한다.
  * Greedy algorithm은 greedy choice property로 인해 매 step에서 한 개의 sub-problem만 처리한다.
  * 따라서 대개 Greedy algorithm의 시간 복잡도가 더 낮다.

## 차이점 비교 예제
* Dynamic: 0/1 knapsack problem
* Greedy: Fractional knapsack problem
* 예제 코드는 곧 올리도록 하자!
