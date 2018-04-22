---
layout: post
title: "Importing JDK source code into IntelliJ"
tags: [ide, java]
comments: true
---

#### 1. download source code
사용하고 있는 버전에 맞게 JDK 소스코드를 다운 받는다. 나는 JDK 1.6 버전이어서 [여기](http://download.java.net/openjdk/jdk6/)에서 다운 받았다.

#### 2. 압축파일을 풀어주고, src/share/classes를 jar로 묶는다. 이를 JDK Home 폴더 아래에 넣어준다.

```linux
cd ~/Downloads
mkdir jdk6src
cd jdk6src
tar xf ../openjdk-6-src-b27-26_oct_2012.tar.gz
cd jdk/src/share/classes
sudo jar cf [path to jdk directory]/1.6.0.jdk/Contents/Home/src.jar *
```

내 맥북에서는 [path to jdk directory]는 `/System/Library/Java/JavaVirtualMachines`이었다.
  - 마지막 명령에서 `*`는 왜 붙이는 건지 모르겠다.

#### 3. Add src.jar path

* Go to: Project Structure (Project Settings) > Platform Settings > SDKs > Sourcepath

* Add the path to src.jar
    - OSX example: /Library/Java/JavaVirtualMachines/1.6.0_45-b06-451.jdk/Contents/Home/src.jar
    - Windows example: C:\Program Files\Java\jdk1.7.0_03 (check Program (x86) for 32-bit)

#### 참고 사이트
* http://stackoverflow.com/questions/2896727/where-to-find-java-jdk-source-code
* http://stackoverflow.com/questions/1313922/step-through-jdk-source-code-in-intellij-idea
