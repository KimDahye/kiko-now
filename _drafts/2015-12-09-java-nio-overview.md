---
layout: post
title: "Java NIO overview"
tags: [java]
comments: true
---

# Java NIO
* Nonblocking IO가 아니라 New IO 였다.
* 하지만 Nonblocking IO 를 지원한다.
* NIO에서 중요한 것은 Channel, Buffer, Selector 개념이다.

## Channel
- 기존 IO의 Stream과 비슷한 개념이다.
- 하지만 Stream과의 차이점이 있다. 
  - 채널은 양방향(read, write 다 가능)
  - 채널은 비동기로 읽고 쓸 수 있다.
  - 채널은 언제나 버퍼에서 읽어와 쓰고 버퍼에서 읽은 것을 버퍼에 쓴다. 버퍼랑 꼭 함께 쓰여야 한다.

## Buffer
- "essentially a block of memery into which you can write data, which you can then later read again."
- 중요 개념은 capacity, position, limit
  - capacity: buffer의 크기
  - position: write 모드 일 때엔 쓸 위치, read 모드 일 때엔 읽을 위치
  - limit: write 모드 일때는 capacity와 같고, read 모드 일 때엔 write모드일 때의 position 위치
- 함수들
  - flip(): read/write 모드 바꾼다.
  - clear(): 버퍼 비우기
  - compact(): 읽은 데이터만 지우고 안 읽은 부분은 남겨둔다.
  - get(): 1byte 씩 읽어들임, hasRemaining() 과 함게 쓰이네.

## Selector
- 채널들을 한 쓰레드에서 관리할 수 있게 한다! 
- non-blocking 모드로 쓸 때 유용해 보임. (아니 필수적으로 보임)

selector 사용 예시 코드

```language-java
Selector selector = Selector.open();

channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);


while(true) {

  int readyChannels = selector.select();

  if(readyChannels == 0) continue;


  Set<SelectionKey> selectedKeys = selector.selectedKeys();

  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

  while(keyIterator.hasNext()) {

    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.

    } else if (key.isConnectable()) {
        // a connection was established with a remote server.

    } else if (key.isReadable()) {
        // a channel is ready for reading

    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
  }
}
```

## 참고사이트
* [Jencov.com](http://tutorials.jenkov.com/java-nio/overview.html) 의 Java NIO 튜토리얼들
