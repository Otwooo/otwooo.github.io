---
layout: post
title: 스택 프레임(Stack Frame)
subtitle : 스택 프레임 개념 설명
tags: [Reversing]
author: Otwooo
comments : False
---

## 01. 스택 프레임이란?

**스택 프레임이**란 함수가 호출될 때 생성되고 함수 종료 시 소멸되는 공간으로 함수 내의 로컬 변수, 파라미터, 복귀 주소를 저장한다. 

<br>

스택 프레임의 시작과 끝은 다음과 같이 구성되어 있다.

{% highlight assembly %}
PUSH EBP
MOV EBP, ESP

...
...

MOV ESP, EBP
POP EBP
RETN
{% endhighlight %}

`PUSH EBP` : EBP의 값을 스택에 저장

`MOV EBP, ESP` : ESP의 값을 EBP에 저장

`MOV ESP, EBP` : EBP의 값을 ESP에 저장

`POP EBP` : EBP값 복원

`RETN` : 함수 종료

<br>

스택 포인터 역할을 하는 ESP(스택 포인터) 값을 함수 호출 시점에 EBP(베이스 포인터)에 저장한다. 이 때 ESP 값은 프로그램 실행 중에 수시로 변하기 때문에 함수의 변수, 파라미터, 복귀 주소를 EBP 기준으로 안전하게 접근할 수 있다. 

<br><br>

## 02. 스택프레임 자세히 살펴보기

### 02.1. 함수 호출 전 파라미터 입력

함수 호출 전에 먼저 파라미터가 입력될 수 있다. 

{% highlight assembly %}
mov eax,dword ptr ss:[ebp-8]
push eax
mov ecx,dword ptr ss:[ebp-4]
push ecx  
call stackframe.401000
{% endhighlight %}

파라미터 입력 예시 코드이다. 함수 호출 전에 바로 파라미터가 입력된다. EBP를 기준으로 파라미터가 저장되는 것을 확인할 수 있다.

<br>

### 02.2. 복귀 주소 설정

CALL 명령어가 실행되면 CPU는 자동적으로 **복귀 주소**를 스택에 저장한다.

<p align="center">
<img src="{{ site.baseurl }}assets\img\2_Untitled.png">
</p>

x64dbg로 살펴본 [Stackframe.exe](https://github.com/reversecore/book/tree/master/%EC%8B%A4%EC%8A%B5%EC%98%88%EC%A0%9C/01_%EA%B8%B0%EC%B4%88_%EB%A6%AC%EB%B2%84%EC%8B%B1/07_Stack_Frame/bin)의 add() 함수 호출 때의 스택 상태이다. **복귀 주소(return address)** `00401041`가 저장된 것을 확인할 수 있다.

<br>

실제 add() 함수 호출 다음 주소가 `00401041`인 것을 확인할 수 있다.

<p align="center">
<img src="{{ site.baseurl }}assets\img\2_Untitled 1.png">
</p>

<br>

### 02.3. 함수 프롤로그 (Prolog)

함수가 실행되면 먼저 **함수 프롤로그**가 수행된다.

{% highlight assembly %}
PUSH EBP
MOV EBP, ESP
{% endhighlight %}

EBP 값을 스택에 저장하고 이후 ESP 값을 EBP에 저장한다. 호출 함수에서 EBP와 ESP를 사용하기 전에 caller 함수(자신을 호출한 함수)의 EBP와 ESP 값을 미리 저장해두는 것이다.

<br>

### 02.4. 로컬 변수 세팅

함수 호출 후 로컬 변수에 대한 스택 메모리 영역을 확보하고 값들을 저장한다.

{% highlight assembly %}
SUB ESP, 8 ; 메모리 영역 확보
mov eax,dword ptr ss:[ebp+8] ; [EBP+8] = param a
mov dword ptr ss:[ebp-8],eax ; [EBP-8] = local x
mov ecx,dword ptr ss:[ebp+C] ; [EBP+C] = param b
mov dword ptr ss:[ebp-4],ecx ; [EBP-4] = local y 
{% endhighlight %}

ESP에 8을 빼면 메모리 영역이 줄어드는 것이 아니라 증가하는 이유는 스택이 거꾸로 자라기 때문이다.
변수까지 세팅을 완료하면 이제 함수의 역할을 수행한다.

<br>

### 02.5. 함수 에필로그 (Epilog)

함수가 종료되기 전 **함수 에필로그**가 수행된다.

{% highlight assembly %}
MOV ESP, EBP
POP EBP
RETN ; 함수 종료
{% endhighlight %}

EBP 값을 ESP에 저장하고 EBP 값을 복원한다. 함수 에필로그 과정으로 통해 함수 종료 후 caller 함수의 ESP, EBP 상태로 돌아갈 수 있다.

<br>

### 02.6. 함수 파라미터 제거 (스택 정리)

함수 호출 후 caller 함수로 돌아왔으면 마지막으로 스택을 정리해준다.

{% highlight assembly %}
ADD ESP, 8
{% endhighlight %}

<br><br>

## 03. 마무리

함수가 호출될 때 스택 프레임이 어떻게 생성되고 소멸되는지 과정에 대해 간략하게 살펴보았다. 

이 과정을 통해 복귀 주소 값이나 RETN 값을 조작하는 수많은 공격 기법이 존재하기 때문에 스택 프레임에 대해 정확히 이해하고 있어야한다.

<br><br>

---

> **Reference**
> 
> - [도서 리버싱 핵심원리](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=19577173)
> - [eli_ez3r Hacking Blog - 스택프레임(Stack Frame) 이란?](https://eliez3r.github.io/post/2019/10/16/study-system.Stack-Frame.html)