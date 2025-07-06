---
layout: post
title: "[JVM] 힙 메모리(Heap Memory)에 대한 이해"
date: 2025-07-05 14:50:00 +0900
categories: 
  - java
  - JVM
tags:
  - java
  - JVM
  - Heap
---

## 🎯 배경
저번에 JVM에 대해 학습을 해봤다. 다른 메모리에는 스택,메소드 에리어, 네이티브 스택, pc 레지스터가 존재하는데, 힙 메모리는 조금 특별하다고 생각한다. 뭐랄까 다른 메모리는 그 자체로 설명이 가능한거 같다고 생각하지만 힙 메모리는 그렇지 않는거 같다. 이렇게 생각한 이유는 스택 구조, 메소드 에리어 구조 이런걸 공부하는걸 생각을 못했기때문이다. (찾으면 있을 수 도...)
힙 메모리는 그안에 어떻게 구조가 되어있는지 생각할 필요가 있는거 같다.

## 힙 메모리 구조
힙 메모리는 생각했을때 힙 구조라는 생각할 수 있을 거 같다.
힙 구조는 
<img src="/assets/img/jvm/heap.png" alt="힙 구조 다이어그램" style="width: 55%; height: auto; display: block; margin: 0 auto;">
다음처럼 만들어진다. 근데 저 구조는 힙 메모리 구조와는 전혀 상관이 없다고 한다.<br>
jvm에서의 힙메모리는
 -  자바 프로그램이 실행될 때 객체와 배열이 저장되는 메모리 영역
 - Young Generation, Old Generation 등으로 나뉘고, 가비지 컬렉션(GC)과 밀접한 관련이 있다.
 - 내부적으로 트리 구조가 아니라, 단순히 “동적으로 할당되는 큰 메모리 공간”이라는 의미에서 “힙”이라고 부른다.

구체적으로 힙 메모리가 어떻게 구성이 되는지 알아보자.
크게 보면 young generation과 old generation으로 나뉘어져 있다.
이걸 단순히 보면 young generation은 초기 상태가 저장이 되었고 old generation은 오래된 상태가 저장이 되어있다. 이렇게 나눈 이유는 모두 GC의 효율때문이다.
그런데 이것들이 왜 GC의 효율때문인지는 생각할 필요가 있을 거 같다.

객체 즉 ```new```를 사용하게 되면 힙 메모리에 저장이 되어진다.
일단 지금 당장 생겨나기 때문에 GC는 이 객체를 지우지 않는다. 그러다 인터프리터 혹은 JIT가 
더 이상 이 객체를 해석하지 못하게 되면 GC는 이 객체를 제거를 하게 될거다.

이게 핵심인데 young generation과 old generation를 왜 나눈걸까?
굳이 안 나눠도 비효율적이라는 생각은 들지 않는다. 

잘 모르겠다..
일단 young generation부터 학습을 해보자.
알아본결과 young generation은 객체가 생성되는 공간으로 대부분의 객체들은 이곳에서 빠르게 소멸된다고 한다. 그렇다는 이야기는 빠르게 소멸되는 객체들과 오래사용하고 있는 객체들을 구분을 짓는다는 소리같다.

GC를 학습을 하게되면 조금더 자세하게 알 수 있을거 같은데 짧게 설명하자면 GC비용이 적다고 한다.
이걸로 봤을때 GC와 young generation은 뭔가 밀접한 관계가 있는게 분명하다.
또 GC유형도 마이너다. 결국 하나의 GC가 돌지 않는다는 그런 느낌으로 흘러갈수 있을거 같다.

### Young Generation 구조
조금전까지 객체가 생성이 되어질때 young generation에 저장이 되었다고 했다.
조금더 자세하게 들어가보면 young generation은 크게 eden space과 Survivor space로 나뉘어져있다.

#### Even space
eden space은 객체들이 처음 생성되어지는 공간이다.
그런데 여기에서도 의문이 생긴다. 
young이랑 old를 나누는 이유가 효율성이라고 했는데
young을 eden, survivor space로도 나누는 이유도 효율성 때문인가?

내가 그림을 대충 그려보니 다음과 같다.

<img src="/assets/img/jvm/even.png" alt="young generation 구조 다이어그램" style="width: 60%; height: auto; display: block; margin: 0 auto;">

even과 survivor은 서로 다른 환경을 뜻하는것이기때문에 직접적으로 이동은 불가능하다.
마치 우리가 배포를 하는거라 생각하면 된다.

1. 로컬에서 파일을 생성한다.
2. 로컬에서 파일을 복사하여 원격 서버에 복사한다.
3. 원격서버에서 그 파일을 확인한다.

이것이 배포하는 방법인데 요거랑 굉장히 유사하다고 생각한다. <br>
다른점은 객체가 복사한 즉시 even쪽에서는 그 객체를 사용할 수 없게 되어진다. <br>
**즉, survivor에 존재한다는 뜻은 살아남았다는 뜻이 된다.**

#### survivor space

이렇게 이동한 객체는 survivor space으로 이동이 되어진다. 
말이 이동이지 정확히 말하면 even에 있는걸 survivor로 복사되어진다.
survivor에는 두 개의 영역으로 나뉘어지는데 survivor 0 과 survivor 1이 있다고 한다.

이것도 그림을 통해 그려보면

<img src="/assets/img/jvm/sub.png" alt="survivor space 구조 다이어그램" style="width: 60%; height: auto; display: block; margin: 0 auto;">
이렇게 그릴 수 있을거 같다.

swap을 거치면서 age가 증가를 하고 일정 수치가 되면 이 객체는 old generation으로 이동이 된다.
물론 복사할때마다 기존에 있던 객체들은 사용을 할 수 가 없다.
이것도 even이랑 마찬가지로 환경이 달라지는것이기때문에 일반적인 이동은 불가능하다.

지금까지 young generation에 대해 알아보았다.
이제 old generation에 대해 알아보자.

### Old Generation 구조

young generation에서 일정 수치의 age로 올라가게 되면 old generation으로 복사가 되어진다.
또 young애서는 minor gc를 사용하지만 old generation은 major gc가 사용이 되어진다.

어떻게 보면 이쯤 왔으면 객체를 단순한 쓰레기로 보는게 아니라 하나의 제품으로 보는느낌이 든다.
제품은 웬만해서는 버릴 수 없다고 생각한다.

### 요약
JVM의 힙 메모리는 자바 프로그램에서 객체와 배열이 저장되는 공간으로, 효율적인 가비지 컬렉션(GC)을 위해 Young Generation과 Old Generation으로 나뉩니다.
Young Generation은 객체가 처음 생성되는 영역(eden, survivor space)으로, 대부분의 객체가 이곳에서 빠르게 소멸합니다.
Young Generation에서 살아남은 객체는 Old Generation으로 이동하며, 이곳에서는 Major GC가 발생해 오래된 객체를 정리합니다.
이렇게 세대를 나누는 구조는 GC의 효율을 높이기 위한 설계입니다.

### next to
이제 JVM의 마지막 GC에 대해 좀 더 자세히 알아보자. 아마 GC의 종류를 통해 설명하지 않을까 싶다.
지금까지 학습한 내용으로 GC를 설명할 예정이다.