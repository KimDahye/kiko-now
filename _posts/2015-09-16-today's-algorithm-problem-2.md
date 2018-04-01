---
layout: post
title: "오늘의 손코딩 - 150916"
tags: [algorithm]
comments: true
---

## level order 로 트리 출력하기 (순회하기)

  ```java
  int printLevelOrderTraverse(Node root) {
  	if(root == null) return 0;
  	Queue que = new LinkedList<Node>();
  	que.offer(root);
  	while(!que.isEmpty()) {
  		Node curNode = que.poll();
  		if(curNode.left != null) que.offer(curNode.left);
  		if(curNode.right != null) que.offer(curNode.right);
  		System.out.println(curNode.data);
  	}
  	return 1;
  }
  ```
