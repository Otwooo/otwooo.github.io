---
layout: post
title: abex' crackme_#1 
subtitle : abex' crackme_#1 파일 분석 및 풀이
tags: [abex' crackme, Reversing]
author: Otwooo
comments : False
---

## 01. 파일 실행

<p align="center">
<img src="{{ site.baseurl }}assets\img\1_Untitled.jpg">
</p>

파일을 실행 시 다음과 같은 메세지를 확인할 수  있다.

<br>

<p align="center">
<img src="{{ site.baseurl }}assets\img\1_Untitled 1.png" height="161" width="300">
</p>

확인 버튼을 누르니 “This is not a CD-ROM Drive!” 메세지가 출력된다. 우리 컴퓨터의 HD를 CD-Rom으로 인식하게 해야하는 것 같다.


<br>
<br>


## 02. 디버깅 시작

### 02.1. 코드 분석

x32dbg를 이용하여 디스어셈 코드를 살펴보자.

<p align="center">
<img src="{{ site.baseurl }}assets\img\1_Untitled 2.png">
</p>

**EntryPoint** 주변에서 바로 `"Make me think your HD is a CD-Rom."`, `"Nah... This is not a CD-ROM Drive!"`, `"Ok, I really think that your HD is a CD-ROM! :p"` 등의 문자열들을 확인할 수 있다.

이는 abex’ crackme #1 파일이 바로 어셈블리어로 만들어져 **Stub Code**가 존재하지 않기 때문이다.

<br>

`0x00401026` 주소에서 `"Nah... This is not a CD-ROM Drive!"`, `"Ok, I really think that your HD is a CD-ROM! :p"`로 나눠지는 조건 분기문을 찾을 수 있다.

<p align="center">
<img src="{{ site.baseurl }}assets\img\1_Untitled 3.png">
</p>


{% highlight assembly %}
cmp eax esi 
je abex’ crackme1.40103D
{% endhighlight %}

`cmp op1, op2` : op1과 op2를 비교한다.

- 연산의 결과는 op1에 대입되지 않는다.
- 서로 같은 두 수를 빼면 결과가 0이 되어 ZF가 설정된다.

`je addr` : 직전에 비교한 두 피연산자가 같으면 점프한다. (jump if equl)

- ZF = 1 일시 점프한다.

<br>

{% highlight cpp %}
if (eax == esi) {
	jmp abex' crackme1.40103D 
}
{% endhighlight %}

즉 다음 조건 분기문은 다음과 같이 해석할 수 있다.

<br>

### 02.2. GetDriveTypeA 함수 분석

<p align="center">
<img src="{{ site.baseurl }}assets\img\1_Untitled 4.png">
</p>

즉, 윗부분에서 `GetDriveTypeA`의 반환값 `eax`와 `esi`의 값이 같아져야 위 조건 분기문에서 점프되어 `"Ok, I really think that your HD is a CD-ROM! :p"` 문자가 출력된다.

> 함수의 반환값은 eax에 저장된다.

<br>

구글링을 통해 [GetDriveTypeA](https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-getdrivetypea) 함수의 설명과 반환값을 확인할 수 있었다.

> Determines whether a disk drive is a removable, fixed, CD-ROM, RAM disk, or network drive.

| Return code/value | Description |
| --- | --- |
| DRIVE_UNKNOWN/0 | The drive type cannot be determined. |
| DRIVE_NO_ROOT_DIR/1 | The root path is invalid; for example, there is no volume mounted at the specified path. |
| DRIVE_REMOVABLE/2 | The drive has removable media; for example, a floppy drive, thumb drive, or flash card reader. |
| DRIVE_FIXED/3 | The drive has fixed media; for example, a hard disk drive or flash drive. |
| DRIVE_REMOTE/4 | The drive is a remote (network) drive. |
| DRIVE_CDROM/5 | The drive is a CD-ROM drive. |
| DRIVE_RAMDISK/6 | The drive is a RAM disk. |

제작자는 `GetDriveTypeA` 함수에서 디스크를 CD-ROM로 인식하고 5가 반환되면 `"Ok, I really think that your HD is a CD-ROM! :p"`가 출력되게 프로그래밍 한 것 같다.

<br><br>

### 03. 크랙

패치를 통해 프로그램을 크랙해보자.

<p align="center">
<img src="{{ site.baseurl }}assets\img\1_Untitled 5.png">
</p>

`cmp eax, esi`를 `cmp eax, eax`로 변경하여 무조건 ZF가 설정되고 아래 조건 분기문에서 점프되게 하면 될 것 같다. `우클릭 메뉴 - 어셈블`로 코드를 변경할 수 있다.

<p align="center">
<img src="{{ site.baseurl }}assets\img\1_Untitled 6.png">
</p>

<p align="center">
<img src="{{ site.baseurl }}assets\img\1_Untitled 7.png" height="161" width="300">
</p>

패치 후 실행해보니 성공적으로 `CD-ROM`으로 인식하는 것을 확인할 수 있다! `우클릭 메뉴 - 패치 - 파일 패치`를 통해 패치된 파일을 내보낼 수 있다.