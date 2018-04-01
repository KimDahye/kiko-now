---
layout: post
title: "알고리즘 스터디 - red black tree에서 black height 확인하는 문제"
tags: [algorithm]
comments: true
---

* Tree의 각 노드의 색깔은 red 혹은 black이다.
* Leaf node가 가리키는 Null은 모두 black으로 간주한다.

  ```java
  int checkBlackHeight(Node node) {
    if(node == null) return 0;
    int left = checkBlackHeight(node.left);
    int right = checkBlackHeight(node.right);
    if( left == -1 || right == -1) return -1;  // 처음에 이 부분 빼먹었었다.
    if(node.left == null || node.left.color == Color.BLACK) left++;
    if(node.right == null || node.right.color == Color.BLACK) right++;
    if(left != right) return -1;
    return left;
  }
  ```
