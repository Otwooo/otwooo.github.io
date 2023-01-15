---
layout: post
title: abex' crackme_#2
subtitle : abex' crackme_#2 파일 분석 및 풀이
tags: [abex' crackme, Reversing]
author: Otwooo
comments : False
---


## 01. 파일 실행

<p align="center">
<img src="{{ site.baseurl }}assets\img\3_Untitled.png">
</p>
파일을 실행하면 다음과 같이 `Name`과 `Serial`을 입력하는 창이 나온다. 

<br>

<p align="center">
<img src="{{ site.baseurl }}assets\img\3_Untitled 1.png">
</p>
아무 값이나 입력해보자. 네단어 이상 입력해야하는 것 같다.

<br>

<p align="center">
<img src="{{ site.baseurl }}assets\img\3_Untitled 2.png">
</p>
네 단어 이상 입력하면 다음과 같이 `Wrong serial!` 알림창이 뜬다.

이 외에도 `About` 버튼을 누르면 파일 정보가 뜨고 `Quit` 버튼을 누르면 프로그램이 종료된다. 리버싱을 통해 `Name`값과 `Serial`값을 알아내야하는 것 같다.

<br><br>

## 02. 디버깅 시작

### 02.1. Detect It Easy

<p align="center">
<img src="{{ site.baseurl }}assets\img\3_Untitled 3.png">
</p>

Visual Basice으로 프로그램이 개발되었고 따로 패킹은 안된 것을 확인할 수 있다.

<br>

### 02.2. 시리얼 검증 코드 분석

x32dbg를 이용하여 디스어셈 코드를 살펴보자.

<p align="center">
<img src="{{ site.baseurl }}assets\img\3_Untitled 4.png">
</p>

`00401000` 주소에서 **EntryPoint**를 확인할 수 있다. 다음으로 Push를 통해 `401E14` 주소값을 스택에 넣고 `ThunRTMain` 함수를 호출한다. 

> **ThunRTMain()**
>
비주얼 베이직 기반 프로그램에서 실행되는 함수로`RT_MainStruct` 구조체를 인자로 받는다.
VB엔진은 RT_MainStruct 구조체를 가지고 프로그램의 실행에 필요한 모든 정보를 얻는다.
> 

<br>

<p align="center">
<img src="{{ site.baseurl }}assets\img\3_Untitled 5.png">
</p>

`401E14` 주소에서 구조체로 보이는 문자를 확인할 수 있다. `(우클릭 메뉴 - 덤프에서 따라가기 - 주소)`

<br>

<p align="center">
<img src="{{ site.baseurl }}assets\img\3_Untitled 6.png">
</p>

`ThunRTMain()` 함수 내부로 들어온 모습이다. 메모리 주소가 완전히 달라졌는데 이는 `MSVBVM60.dill` 모듈의 주소 영역이다. `MSVBVM60.dill`는 VB전용 엔진이다.

직접 하나하나씩 분석하기는 힘드니 문자열 검색 기능을 사용해보자. `(우클릭 메뉴 - 다음을 찾기 - 모든 모듈 - 문자열 참조)`

<br>

<p align="center">
<img src="{{ site.baseurl }}assets\img\3_Untitled 7.png">
</p>

파일 실행 과정에서 볼 수 있었던 `Wrong serial!` 문자가 있다.

`Wrong serial!` 문자를 따라간 모습이다. 아마 윗부분에서 키를 검증하는 분기문을 찾을 수 있을 것 같다.

<br>

<p align="center">
<img src="{{ site.baseurl }}assets\img\3_Untitled 8.png">
</p>

예상대로 위로 올라가니 분기문을 확인할 수 있다.

<br>

<p align="center">
<img src="{{ site.baseurl }}assets\img\3_Untitled 9.png">
</p>

분기문에서 점프가 되지 않아야 `Yep, this key is right!` 출력된다. 

<br>

`je` : 직전에 비교한 두 피연산자가 같으면 점프 (jump if equl)

- ZF가 1로 설정되어 있어야 점프된다.

`test op1, op2` : op1과 op2를 비교

- 연산의 결과는 op1에 대입되지 않는다.
- op1, op2가 같으면 ZF가 설정된다.

<br>

즉 `vbavarTstEq()` 함수에서 실제 시리얼값과 입력된 시리얼값을 비교하고 이에 따라 참, 거짓 코드로 분기하게 되는 것 같다.

<br>

그러면 `vbavarTstEq()` 함수의 인자로 들어오는 값을 찾으면 시리얼 값을 찾을 수 있을 것 같다. `vbavarTstEq()`함수에 BreakPint(F2)를 걸고 실행(F9)시켜보자.

<p align="center">
<img src="{{ site.baseurl }}assets\img\3_Untitled 10.png">
</p>
`Name : abcd / Serial : 12345678` 로 값을 채우고 실행

<br>

하지만 함수의 인자로 들어간 edx, eax 값의 메모리 주소를 확인해보면 이상하게 아무 값도 들어가 있지 않다.  `(우클릭 메뉴 - 덤프에서 따라가기 - 주소)`

<p align="center">
<img src="{{ site.baseurl }}assets\img\3_Untitled 11.png">
</p>

> **VA의 문자열**
VB의 문자열은 가변 길이 문자열 타입을 사용한다. 따라서 바로 문자열이 나타나지 않고 16바이트 크기의 데이터가 나타난다. 이 데이터 실제 문자열 주소가 포함되어 있다. 
> 

<br>

메모리 영역에서 `우클릭 메뉴 - 주소`로 주소에 존재하는 문자열을 확인할 수 있다.

<p align="center">
<img src="{{ site.baseurl }}assets\img\3_Untitled 12.png">
</p>

여기서 내가 입력한 `12345678` 문자와 `C5C6C7C8` 문자를 확인할 수 있다. 따라서 `C5C6C7C8` 문자를 시리얼 값으로 추측할 수 있다.

<br>

## 03. 크랙

<p align="center">
<img src="{{ site.baseurl }}assets\img\3_Untitled 13.png">
</p>

예상대로 `Name : abcd / Serial : C5C6C7C8`를 입력한 후 `Check` 버튼을 누르면 맞았다는 메시지박스가 뜬다.

<br>

---
> **Reference**
> 
> - [도서 리버싱 핵심원리](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=19577173)
> - [POSIX - abex' crackme 2 분석](https://posix.tistory.com/78)