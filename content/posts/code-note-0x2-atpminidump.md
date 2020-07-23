---
title: "Code Note 0x2: ATPMiniDump"
date: 2020-07-07T13:29:50+07:00
author: "P. Boonyakarn"
draft: true
description: "Code Note คือชุดของบล็อกและโพสต์ซึ่งจะนำโค้ดของโปรแกรมจากโครงการโอเพนซอร์สมาทำการวิเคราะห์และทำความเข้าใจ ให้ความหมายและข้อเสนอแนะ โดย Code Note ในฉบับที่ 0x2 จะเป็นการนำโค้ดจากโครงการ ATPMiniDump มาทำการวิเคราะห์และตรวจสอบ โดยจะนำผลลัพธ์ที่ได้ไปใช้ในการทดสอบและประเมินผลกับโซลูชัน Endpoint Detection and Response ต่อไป"
---

> **Code Note** คือชุดของบล็อกและโพสต์ซึ่งจะนำโค้ดจากโครงการโอเพนซอร์สมาทำการวิเคราะห์และทำความเข้าใจ ให้ความหมายและข้อเสนอตามจุดประสงค์ของแต่ละโครงการ

สวัสดีทุกท่านซึ่งเข้ามาอ่าน Code Note 0x2 ในเดือนกรกฎาคม 2020 ครับ 

ในชุดของบล็อก **Code Note** ครั้งที่สองนี้ เราจะมาดูแนวคิดของการโจมตีโปรเซส `lsass.exe` ซึ่งนำไปสู่การได้มาซึ่งข้อมูลสำหรับยืนยันตัวตนในสภาพแวดล้อมซึ่งใช้ระบบปฏิบัติการ Windows, แนวคิดและการทำงานของโปรแกรมซึ่งเป็นที่รู้จักกันดีในการนำแนวคิดเหล่านี้มาอิมพลีเมนต์และใช้งานอย่าง mimikatz และโครงการ ATPMiniDump ซึ่งเป็นโครงการที่นำแนวคิดของ mimikatz มาปรับปรุงเพื่อให้สามารถเข้าถึงข้อมูลสำหรับยืนยันตัวตนได้โดยผ่านการตรวจจับของซอฟต์แวร์อย่าง Microsoft Defender Advanced Threat Protection หรือ Microsoft Defender ATP

เนื่องจากหัวข้อซึ่งเราจะพูดถึงการวันนี้อยู่ในชุดของบล็อก **Code Note** ดังนั้นเราจะพูดถึงแนวคิดและการทำงานซึ่งได้มาจากการอ่านโค้ดของโครงการ ATPMiniDump เป็นสำคัญ ทั้งนี้แนวคิดของการโจมตีและการอิมพลีเมนต์แนวคิดนั้นล้วนแล้วแต่เป็นประเด็นที่น่าสนใจทั้งในมุมของการทำ Red teaming ซึ่งมีเป้าหมายสำคัญในการหลบหลีกการตรวจจับ และในมุมของฝั่ง Defense ซึ่งมีหน้าที่ในการเฝ้าระวังและตรวจหาพฤติกรรมการโจมตีเหล่านี้ให้ได้ ผมจะขอพูดถึงประเด็นเหล่านี้ในอนาคตหากมีโอกาสครับ

หัวข้อซึ่งเราจะพูดถึงในบล็อกมีดังนี้ครับ

- [Project Overview](#project-overview)
- [Project Background](#project-background)
  - [Attacking lsass.exe](#attacking-lsassexe)
  - [About ATPMiniDump](#about-atpminidump)
- [Code Analysis](#code-analysis)
  - [Functions](#functions)
    - [wmain Function](#wmain-function)
    - [GetPID Function](#getpid-function)
    - [IsElevated Function](#iselevated-function)
    - [SetDebugPrivilege Function](#setdebugprivilege-function)
  - [Notes](#notes)
    - [Loading External Function with Function Pointer](#loading-external-function-with-function-pointer)
- [Final Notes](#final-notes)

เครดิตซึ่งทำให้รีเสิร์ชชิ้นนี้เกิดขึ้นได้มีดังนี้ครับ

- โครงการ [ATPMiniDump](https://github.com/b4rtik/ATPMiniDump) โดย Matteo Malvica ซึ่งมีที่มาจากรีเสิร์ชของการพยายามหาวิธีการข้ามผ่านการตรวจจับเมื่อต้องดึงข้อมูลออกมาจากหน่วยความจำของโปรเซส lsass.exe ([ดูบล็อกต้นฉบับได้ที่นี่](https://www.matteomalvica.com/blog/2019/12/02/win-defender-atp-cred-bypass/))
- โครงการ [mimikatz](https://github.com/gentilkiwi/mimikatz) โดย Benjamin Delpy ซึ่งนำแนวคิดเกี่ยวกับความปลอดภัยใน Windows มาอิมพลีเมนต์เป็นโปรแกรมซึ่งสามารถใช้ได้จากทั่งฝังโจมตีและฝั่งป้องกัน
- หนังสือ [A Guide to Kernel Exploitation](https://www.amazon.com/Guide-Kernel-Exploitation-Attacking-Core/dp/1597494860) ที่เข้ามามีส่วนสำคัญในการอธิบายโค้ดและการเรียกใช้งาน Windows API ในระดับของ Kernel
- เว็บไซต์ [Geoff Chappell, Software Analyst](https://www.geoffchappell.com/index.htm) โดย Geoff Chappell ที่ช่วยอธิบายการใช้งาน Windows API ในระดับ Kernel ด้วยรายละเอียดที่มีมากกว่า [Microsoft Docs (MSDN)](https://docs.microsoft.com/en-us/)

# Project Overview

ข้อมูลจาก GitHub แสดงให้เห็นว่าโครงการนี้ถูกพัฒนาโดยใช้ภาษา C ทั้งหมด โดยโครงการนี้มีโครงสร้างของไฟล์และไดเรกทอรีตามแผนภาพด้านล่างครับ

```
.
├── ATPMiniDump
│   ├── ATPMiniDump.c
│   ├── ATPMiniDump.h
│   ├── ATPMiniDump.vcxproj
│   ├── ATPMiniDump.vcxproj.filters
│   ├── ATPMiniDump.vcxproj.user
├── ATPMiniDump.sln
├── LICENSE
└── README.md
```

การตรวจสอบไฟล์แต่ละรายการที่ปรากฎในโครงสร้างของโครงการปรากฎไฟล์ดังนี้

1. ไฟล์ในไดเรกทอรี `ATPMiniDump` เก็บโค้ดหลักของโครงการและไฟล์ประกอบอื่นๆ ได้แก่
   1. ไฟล์ `ATPMiniDump.c` จัดเก็บโค้ดการทำงานหลักของโครงการ
   2. ไฟล์ `ATPMiniDump.h` จัดเก็บการประกาศค่าและตัวแปรเฉพาะในลักษณะต่างๆ
   3. ไฟล์ `ATPMiniDump.vcxproj` จัดเก็บรายละเอียดในการ build โครงการสำหรับ Visual Studio
   4. ไฟล์ `ATPMiniDump.vcxproj.filters` จัดเก็บการตั้งค่าของไฟล์ที่จะถูกแสดงใน Visual Studio
2. ไฟล์ ATPMiniDump.sln จัดเก็บข้อมูลการตั้งค่าของ Solution สำหรับ Visual Studio

ในกรณีของการ build ผ่านโปรแกรม `cl.exe` ให้ใช้คำสั่งดังต่อไปนี้

```
cl.exe ATPMiniDump.c Advapi32.lib
```

กระบวนการ build สามารถทำได้ผ่าน Visual Studio เช่นกัน

# Project Background

## Attacking lsass.exe

โปรเซส `lsass.exe` นั้นเป็นโปรเซสของเซอร์วิสของระบบซึ่งมีชื่อเต็มว่า Local Security Authorithy Subsystem Service (LSASS) ซึ่งอิมพลีเมนต์แนวคิด Local Security Authorithy (LSA) เซอร์วิสของระบบเซอร์วิสนี้มีหน้าที่รับผิดชอบสำคัญในการบังคับใช้มาตรการรักษาความปลอดภัยที่กำหนดใดๆ กับระบบ รวมไปถึงมีหน้าในการพิสูจน์ตัวตนผู้ใช้งานทั้งในกรณีที่ผู้ใช้งานทำการเข้าถึงระบบผ่านทางหน้าจอเมื่อเปิดใช้งานระบบ หรือพิสูจน์ตัวตนในช่องทางทางเครือข่ายอื่นๆ เซอร์วิสนี้ยังทำหน้าที่ในการคุมสิทธิ์ในการเข้าถึงทรัพยากรของระบบผ่านการสร้างและจัดการ access token ด้วย (อ้างอิง [Wikipedia](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service))

ด้วยหน้าที่รับผิดชอบสำคัญของเซอร์วิสและโปรเซส `lsass.exe` ที่เกี่ยวข้องกับข้อมูลที่ใช้ในการพิสูจน์ตัวตนของผู้ใช้งาน โปรเซส `lsass.exe` ตกเป็นเป้าหมายในการโจมตีหลายรูปแบบและเป้าหมาย หนึ่งในรูปแบบและเป้าหมายในการโจมตีต่อโปรเซส `lsass.exe` ซึ่งเป็นที่รู้จักมากที่สุดรูปแบบหนึ่งคือการพยายามเข้าถึงข้อมูลใดๆ ที่โปรเซส `lsass.exe` กำลังประมวลผลอยู่ ค้นหาข้อมูลสำหรับยืนยันตัวตนไม่ว่าจะเป็นรหัสผ่านหรือค่าแฮชที่เก็บอยู่ในหน่วยความจำของโปรเซส และนำค่าหรือข้อมูลดังกล่าวออกมาใช้งาน

หนึ่งในโครงการซึ่งนำแนวคิดของการโจมตีและแสวงหาประโยชน์มาใช้งานและเป็นที่รู้จักอย่างกว้างขวางคือ [mimikatz](https://github.com/gentilkiwi/mimikatz) ซึ่งมีนักพัฒนาหลักคือ Bejamin *gentilkiwi* Delpy โครงการ mimikatz นำแนวคิดของการเข้าถึงโปรเซส `lsass.exe` รวมไปถึงโปรเซสที่เกี่ยวข้องกับการจัดการความปลอดภัยในระบบอื่นๆ เพื่อระบุหาข้อมูลสำหรับยืนยันตัวตนมาทำการอิมพลีเมนต์ให้สามารถใช้งานได้จริง ส่งผลให้ mimikatz กลายเป็นเครื่องมือยอดนิยมในการประเมินความปลอดภัยระบบและยังถูกใช้ในการโจมตีจริงโดย[หลายกลุ่มผู้โจมตี](https://attack.mitre.org/software/S0002/)อีกด้วย

ความนิยมของ mimikatz ในการใช้งานเพื่อโจมตีระบบจริงนั้นส่งผลให้เทคโนโลยีในฝั่งของการตรวจจับและป้องกันภัยคุกคามจำเป็นต้องขยับตาม มีเทคโนโลยีหลายรูปแบบเริ่มอิมพลีเมนต์แนวคิดมากมายภายใต้จุดประสงค์ที่ไม่แตกต่างกันมากนักคือการตรวจจับกิจกรรมใดๆ ในลักษณะที่ผิดปกติกับโปรเซส `lsass.exe` และขัดขวางกิจกรรมดังกล่าวนั้นในกรณีที่สามารถระบุได้จริงว่ากิจกรรมดังกล่าวนั้นมีจุดประสงค์ที่มุ่งร้ายจริง

## About ATPMiniDump

โครงการ ATPMiniDump ถูกพัฒนาออกมาภายใต้จุดประสงค์เพื่อให้โปรแกรม ATPMiniDump สามารถหลบหลีกการตรวจจับโดยเทคโนโลยีในฝั่งของการตรวจจับและป้องกันภัยคุกคามโดยเฉพาะอย่างยิ่ง Microsoft Defender ATP ได้ และยังสามารถทำงานจนบรรลุจุดประสงค์ของมันคือการสร้างไฟล์ที่มีข้อมูลจากหน่วยความจำของโปรเซส `lsass.exe` ได้โดยไม่ถูกขัดขวาง ทั้งนี้วิธีการที่ถูกใช้เพื่อข้ามผ่านการตรวจจับนี้ในปัจจุบันได้ถูกรายงานให้กับทางไมโครซอฟต์เพื่อทำการแก้ไขแล้ว

ในส่วนเริ่มต้นของบล็อก [Evading WinDefender ATP credential-theft: a hit after a hit-and-miss start](https://www.matteomalvica.com/blog/2019/12/02/win-defender-atp-cred-bypass/) นั้น ผู้พัฒนาโครงการ ATPMiniDump ได้อธิบายถึงสมมติฐานที่น่าสนใจเกี่ยวกับแนวคิดที่ใช้ในการตรวจจับกิจกรรมต้องสงสัยกับโปรเซส `lsass.exe` ว่า Microsoft Defender ATP อาจเฝ้าระวังพฤติกรรมต้องสงสัยโดยการตรวจสอบจำนวนของข้อมูลที่ถูกอ่านผ่านฟังก์ชัน `ReadProcessMemory()` เมื่อมีปลายทางของการเข้าถึงคือโปรเซส lsass.exe

ฟังก์ชัน `ReadProcessMemory()` เป็นฟังก์ชันซึ่งทำให้โปรเซสที่เรียกใช้ฟังก์ชันนี้สามารถคัดลอกข้อมูลซึ่งอยู่ในพื้นที่หน่วยความจำของโปรเซสอื่นในตำแหน่งที่ระบุมาเก็บไว้ในพื้นที่หน่วยความจำของโปรเซสตัวเองได้ ฟังก์ชัน `ReadProcessMemory()` เป็นฟังก์ชันสำคัญซึ่งถูกใช้ใน mimikatz ภายใต้ฟังก์ชัน `kull_m_memory_copy()` ซึ่งจะถูกเรียกใช้อยู่เสมอเมื่อ mimikatz จะทำการเข้าถึงข้อมูลในโปรเซส อาทิ จากฟังก์ชัน `kuhl_m_sekurlsa_getLogonData()`

```c
BOOL kull_m_memory_copy(OUT PKULL_M_MEMORY_ADDRESS Destination, 
    IN PKULL_M_MEMORY_ADDRESS Source, IN SIZE_T Length)
{
	...
	switch(Destination->hMemory->type)
	{
	case KULL_M_MEMORY_TYPE_OWN:
		switch(Source->hMemory->type)
		{
		...
		case KULL_M_MEMORY_TYPE_PROCESS:
			status = ReadProcessMemory(
                Source->hMemory->pHandleProcess->hProcess, // hProcess
                Source->address, // lpBaseAddress
                Destination->address, // lpBuffer
                Length, // nSize
                NULL // *lpNumberofBytesRead
            );
			break;
        ...
		}
		break;
    ...
```

โครงการ ATPMiniDump จึงเลือกใช้ฟังก์ชัน `PssCaptureSnapShot()` ซึ่งเป็นฟังก์ชันในการสร้าง snapshot ของโปรเซสแทนฟังก์ชัน `ReadProcessMemory()` การทำ process snapshotting นั้นเป็นกระบวนการในรวบรวมข้อมูลในหน่วยความจำของโปรเซสเพื่อจุดประสงค์ในการสร้างข้อมูล snapshot สำหรับการวิเคราะห์และแก้ไขปัญหาของระบบ ดังนั้นในกรณีของ ATPMiniDump นั้น ฟังก์ชัน `PssCaptureSnapShot()` อาจสามารถเรียกได้ว่าถูกนำมาใช้ในทางที่ผิด (abuse) นั่นเอง

```c
DWORD PssCaptureSnapshot(
  HANDLE            ProcessHandle,
  PSS_CAPTURE_FLAGS CaptureFlags,
  DWORD             ThreadContextFlags,
  HPSS              *SnapshotHandle
);
```

บล็อกของผู้พัฒนา ATPMiniDump จะแสดงถึงการวิเคราะห์ฟังก์ชัน `PssCaptureSnapShot()` เพื่อยืนยันว่าฟังก์ชันดังกล่าวสามารถถูกใช้เพื่อนำข้อมูลซึ่งอยู่ในหน่วยความจำของโปรเซสที่ต้องการออกมาด้วย โดยในของโปรเซส `lsass.exe` นั้น ฟังก์ชัน `PssCaptureSnapShot()` สามารถนำข้อมูลในหน่วยความจำรวมไปถึงรหัสผ่านและค่าแฮชซึ่งถูกเก็บไว้ที่สามารถนำเขียนลงไฟล์ได้ทันที จากนั้นไฟล์ซึ่งเก็บข้อมูลนี้ก็จะถูกนำมาวิเคราะห์โดยฟังก์ชัน `sekurlsa::minidump` ของ mimikatz เพื่อนำข้อมูลสำคัญที่ต้องการออกมาได้

# Code Analysis

ในส่วนถัดไป เราจะมาเริ่มการวิเคราะห์โค้ดของโครงการ ATPMiniDump โดยจะอ้างอิงการทำงานของฟังก์ชัน `wmain()` เป็นหลัก ผมจะแยกการทำงานของฟังก์ชันอื่นออกเป็นหัวข้อย่อยเพื่อให้สามารถอ่านได้เพิ่มเติมโดยไม่ขัดจังหวะการอ่านหัวข้อปัจจุบัน และจะมีการเพิ่มโน้ตในอ่านและทำความเข้าใจเป็นหัวข้อย่อยด้วยครับ

## Functions

### wmain Function

> ดูโค้ดของฟังก์ชัน `wmain()` แบบเต็มได้[ที่นี่](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.c#L122)
> 
> โค้ดในฟังก์ชัน `wmain()` มีการพยายามเรียกใช้ฟังก์ชันซึ่งอยู่ในไลบรารี DLL อื่น แนะนำให้อ่านหัวข้อ [Loading External Function with Function Pointer](#loading-external-function-with-function-pointer) เพิ่มเติมเพื่อความเข้าใจที่มากขึ้นครับ


### GetPID Function

> ดูโค้ดของฟังก์ชัน `GetPID()` แบบเต็มได้[ที่นี่](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.c#L16)
> 
> โค้ดในฟังก์ชัน `GetPID()` มีการพยายามเรียกใช้ฟังก์ชันซึ่งอยู่ในไลบรารี DLL อื่น แนะนำให้อ่านหัวข้อ [Loading External Function with Function Pointer](#loading-external-function-with-function-pointer) เพิ่มเติมเพื่อความเข้าใจที่มากขึ้นครับ

ฟังก์ชัน `GetPID()` เป็นฟังก์ชันที่จะส่งออกข้อมูลประเภท `BOOL` เป็นผลลัพธ์ ฟังก์ชันมีการระบุพารามิเตอร์เอาไว้โดยจะถูกเก็บไว้ในตัวแปร `pWinVerInfo` ซึ่งเป็นตัวแปรประเภท `PWIN_VER_INFO` ฟังก์ชัน `GetPID()` มีการถูกเรียกใช้งานเพียงหนึ่งครั้งจากโค้ดในฟังก์ชัน `wmain()` [บรรทัดที่ 182](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.c#L182) ตามตัวอย่างด้านล่าง

```c
if (!GetPID(pWinVerInfo)) {
  wprintf(L"	[!] Enumerating process failed.\n");
  exit(1);
}
```

ผมจะขอยกที่มาของอากิวเมนต์ไว้อธิบายในฟังก์ชัน `wmain()` ซึ่งเป็นส่วนของโค้ดที่กำหนดขึ้นมา ดังนั้นในการอธิบายฟังก์ชัน `GetPID()` เราจะสมมติโครงสร้างที่สมบูรณ์ของข้อมูลในประเภท `PWIN_VER_INFO` ขึ้นมาเพื่อให้เข้าใจการทำงานได้ง่ายขึ้นแทน โดยโครงสร้างของข้อมูลประเภท `PWIN_VER_INFO` ถูกกำหนดอยู่ในไฟล์ `ATPMiniDump.h` [ในบรรทัดที่ 49](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.h#L49) ตามโครงสร้างด้านล่าง

```c
typedef struct _WIN_VER_INFO {
	WCHAR chOSMajorMinor[8];
	DWORD dwBuildNumber;
	UNICODE_STRING ProcName;
	HANDLE hTargetPID;
	LPCSTR lpApiCall;
	INT SystemCall;
} WIN_VER_INFO, *PWIN_VER_INFO;
```

จากบทเรียนของเราในเรื่อง typedef (ข้ามไปอ่าน [Loading External Function with Function Pointer](#loading-external-function-with-function-pointer) ได้เพื่อความเข้าใจที่มากขึ้น) เราสามารถอธิบายการโค้ดด้านบนได้ดังนี้

- ทำการสร้างประเภทของข้อมูลใหม่ในชื่อ `_WIN_VER_INFO` จากประเภทข้อมูลแบบ `struct`
- ประเภทข้อมูลนี้จะมีโครงสร้างภายในคือ
  - เก็บค่า `chOSMajorMinor` แบบ wide character ขนาดอาเรย์ 8 ช่อง
  - เก็บค่า `dwBuildNumber` แบบ double word
  - เก็บค่า `ProcName` แบบสตริงที่ใช้ Unicode encoding
  - เก็บค่า `hTargetPID` ในลักษณะของ handle
  - เก็บค่า `lpApiCall` แบบพอยน์เตอร์แบบ 32 บิตซึ่งชี้ไปยังสตริงแบบ constant
  - เก็บค่า `SystemCall` แบบจำนวนเต็ม
- ชื่อของตัวแปรแบบ struct คือ `WIN_VER_INFO`
- ชื่อของพอยน์เตอร์ซึ่งชี้มายัง struct นี้คือ `*PWIN_VER_INFO`

ตอนนี้โครงสร้างด้านบนถูกส่งเข้ามาผ่านตัวแปรชื่อ `pWinVerInfo` ซึ่งมีการตั้งชื่อสมาชิกในโครงสร้างเอาไว้ให้สามารถตีความหมายได้ในตัวอยู่แล้วว่าสมาชิกแต่ละตัวจะเก็บข้อมูลอะไรเอาไว้

เริ่มต้นการทำงาน มีการ initialize ค่าในตัวแปร `pWinVerInfo` ก่อนหนึ่งครั้งที่ `pWinVerInfo->hTargetPID` โดยถูกกำหนดให้เป็นค่า `NULL` จากชื่อของฟังก์ชันหลักคือ `GetPID()` เราคงพอจะเดากันได้ว่าฟังก์ชันนี้น่าจะพยายามหาค่า `PID` ของโปรเซสใดโปรเซสหนึ่งซึ่งจะมาถูกเก็บเอาไว้ใน `pWinVerInfo->hTargetPID` ด้วยเป้าหมายนี้นั้น ลอจิคที่ดูสมเหตุสมผลคือการแสดงรายการของโปรเซสที่มีอยู่ในระบบ ณ ขณะนั้นทั้งหมด แล้วไล่หาค่า PID ด้วยปัจจัยบางอย่างก่อนที่จะนำมาเก็บไว้ที่ตำแหน่งดังกล่าว

หากใครได้อ่านฟังก์ชัน `wmain()` ก็อาจจะพอปะติดปะต่อได้ว่าตัวแปร `pWinVerInfo` ซึ่งเป็นค่านำเข้านั้นมีการกำหนดค่าไว้กับสมาชิก `pWinVerInfo->ProcName` ไว้อยู่แล้วว่าให้เป็นค่าใด ดังนั้นปัจจัยซึ่งจะนำมาใช้หาค่า `PID` จึงมีความเป็นไปได้สูงว่าจะมีการใช้ `pWinVerInfo->ProcName` ครับ ส่วนหากใครยังไม่ได้อ่านก็ไม่เป็นไร ยังไปต่อกันได้อยู่ครับ :)

ต่อมามีการใช้ท่า function pointer ในการโหลด 3 ฟังก์ชันกลุ่ม Windows Kernel API มาจาก `ntdll.dll` โดย 3 ฟังก์ชันนั้นคือ `ZwQuerySystemInformation()` ซึ่งมักถูกใช้สำหรับหาข้อมูลเกี่ยวกับระบบ, ฟังก์ชัน `NtAllocateVirtualMemory()` สำหรับการจองพื้นที่ในหน่วยความจำของโปรเซส และฟังก์ชัน `NtFreeVirtualMemory()` สำหรับการคืนพื้นที่หน่วยความจำที่จองมา

```c
_ZwQuerySystemInformation ZwQuerySystemInformation = (_ZwQuerySystemInformation) GetProcAddress(GetModuleHandle(L"ntdll.dll"), "ZwQuerySystemInformation");
if (ZwQuerySystemInformation == NULL) {
		return FALSE;
}

_NtAllocateVirtualMemory NtAllocateVirtualMemory = (_NtAllocateVirtualMemory) GetProcAddress(GetModuleHandle(L"ntdll.dll"), "NtAllocateVirtualMemory");
if (NtAllocateVirtualMemory == NULL) {
  return FALSE;
}

_NtFreeVirtualMemory NtFreeVirtualMemory = (_NtFreeVirtualMemory) GetProcAddress(GetModuleHandle(L"ntdll.dll"), "NtFreeVirtualMemory");
if (NtFreeVirtualMemory == NULL) {
  return FALSE;
}
```

ฟังก์ชัน `ZwQuerySystemInformation()` เป็นฟังก์ชันแรกซึ่งถูกเรียกใช้ตามโค้ดด้านล่าง โดยก่อนที่จะมีการเรียกใช้ฟังก์ชันนี้นั้น โค้ดได้มีการสร้างและกำหนดค่าให้กับตัวแปร `uReturnLength` ซึ่งเป็นตัวแปรประเภท `ULONG` ให้มีค่าเป็น 0 ด้วย

```c
ULONG uReturnLength = 0;
NTSTATUS status = ZwQuerySystemInformation(
  SystemProcessInformation, // SystemInformationClass
  0, // SystemInformation
  0, // SystemInformationLength
  &uReturnLength // ReturnLength
);
if (!status == 0xc0000004) {
  return FALSE;
}
```

ฟังก์ชัน `ZwQuerySystemInformation()` มีพารามิเตอร์ทั้งหมด 4 ตัว ซึ่งมีความหมายดังนี้

- พารามิเตอร์ `SystemInformationClass` ประเภท `SYSTEM_INFORMATION_CLASS` เป็นพารามิเตอร์สำหรับระบุประเภทของข้อมูลของระบบที่จะฟังก์ชันนี้ในการดึง โดยค่าของพารามิเตอร์จะต้องเป็นค่าใดค่าหนึ่งจากประเภทของข้อมูล [`SYSTEM_INFORMATION_CLASS`](https://www.geoffchappell.com/studies/windows/km/ntoskrnl/api/ex/sysinfo/class.htm)
- พารามิเตอร์ `SystemInformation` ประเภท `PVOID` เป็นพารามิเตอร์สำหรับระบุตัวแปรพอยน์เตอร์ซึ่งชี้ไปยังตำแหน่งของบัฟเฟอร์ที่ฟังก์ชันจะเก็บข้อมูผลลัพธ์เมื่อทำงานเสร็จสิ้น
- พารามิเตอร์ `SystemInformationLength` ประเภท `ULONG` เป็นพารามิเตอร์สำหรับระบุขนาดของบัฟเฟอร์ที่พารามิเตอร์ `SystemInformation` ชี้ไปโดยมีหน่วยเป็นไบต์
- พารามิเตอร์ `ReturnLength` ประเภท `PULONG` เป็นพารามิเตอร์สำหรับระบุตัวแปรพอยน์เตอร์ซึ่งชี้ไปยังตำแหน่งบัฟเฟอร์ที่ฟังก์ชันจะเก็บข้อมูลขนาดของผลลัพธ์การทำงานจริง พารามิเตอร์นี้จะถูกใช้เพื่อตรวจสอบว่าการกำหนดค่าในพารามิเตอร์ `SystemInformationLength` นั้นสอดคล้องกับค่าในพารามิเตอร์ `ReturnLength` หรือไม่ ในกรณีที่ค่าที่ระบุในพารามิเตอร์ `SystemInformationLength` ไม่เพียงพอ ฟังก์ชันจะมีการส่งค่ากลับไปฟังก์ชันซึ่งเรียกใช้ว่ามีการทำงานที่ผิดพลาด

การเรียกใช้ฟังก์ชัน `ZwQuerySystemInformation()` มีการระบุอากิวเมนต์ที่น่าสนใจเข้าไปเพียงอากิวเมนต์เดียวคือที่ตำแหน่งพารามิเตอร์ `SystemInformationClass` ซึ่งถูกระบุค่าเป็น `SystemProcessInformation` ซึ่งประเภทของข้อมูลของระบบเมื่อถูกระบุหรือถูกเรียกหา ค่าที่เราจะได้รับจะมีลักษณะเป็นอาเรย์ซึ่งเก็บข้อมูลของโปรเซสเอาไว้

> **SystemProcessInformation**
>
> An array of SYSTEM_PROCESS_INFORMATION structures, one for each process running in the system.
>
> These structures contain information about the resource usage of each process, including the number of handles used by the process, the peak page-file usage, and the number of memory pages that the process has allocated. 
> 
> — *[ZwQuerySystemInformation function parameters](https://docs.microsoft.com/en-us/windows/win32/sysinfo/zwquerysysteminformation#parameters)*

ทั้งนี้ถ้าเราสังเกตดูให้ดีในการเรียกใช้ฟังก์ชัน `ZwQuerySystemInformation()` พารามิเตอร์ `SystemInformation` ซึ่งควรถูกระบุเพื่อจัดเก็บข้อมูลกลับไม่มีการระบุค่าใด ๆ เอาไว้เลย พารามิเตอร์ที่ดูเหมือนจะมีการรับข้อมูลมีเพียงพารามิเตอร์เดียวคือ `uReturnLength` ซึ่งรับค่าขนาดของผลลัพธ์ออกมาเก็บไว้ จากนั้นจึงมีการตรวจสอบด้วยค่าประเภท `NTSTATUS` เพื่อให้แน่ใจว่าฟังก์ชันทำงานไม่ล้มเหลว อย่างไรก็ตามเฉลยของการเรียกใช้ฟังก์ชัน `ZwQuerySystemInformation()` ก็อยู่ไม่ใกล้ไม่ไกลจากโค้ดส่วนปัจจุบัน ซึ่งก็คืออีกไม่กี่บรรทัดต่อจากนี้ครับ

หลังจากมีการเรียกใช้ `ZwQuerySystemInformation()` จนได้ค่าขนาดของผลลัพธ์ของประเภทข้อมูล `SystemProcessInformation` มาเก็บไว้ที่ `uReturnLength` แล้ว เราจะเห็นการใช้ค่าใน `uReturnLength` ในโค้ดด้านล่าง

```c
LPVOID pBuffer = NULL;
SIZE_T uSize = uReturnLength;
status = NtAllocateVirtualMemory(
  GetCurrentProcess(), // ProcessHandle
  &pBuffer, // *BaseAddress
  0, // ZeroBits
  &uSize, // RegionSize
  MEM_COMMIT, // AllocationType
  PAGE_READWRITE // Protect
);
if (status != 0) {
  return FALSE;
}

status = ZwQuerySystemInformation(
  SystemProcessInformation,  // SystemInformationClass
  pBuffer, // SystemInformation
  uReturnLength, // SystemInformationLength
  &uReturnLength // ReturnLength
  );
	if (status != 0) {
		return FALSE;
	}
```

ค่า `uReturnLength` ที่เก็บค่าขนาดของประเภทข้อมูล `SystemProcessInformation` ซึ่งเป็นผลลัพธ์ของการฟังก์ชัน `ZwQuerySystemInformation()` ทุกจัดเก็บไว้ในตัวแปรใหม่ประเภท `SIZE_T` ชื่อ `uSize` พร้อมๆ กับการสร้างตัวแปรใหม่ในประเภท `LPVOID` ชื่อ `pBuffer` ซึ่งถูกกำหนดค่าเป็น `NULL` ตัวแปรใหม่ทั้งสองนี้ถูกนำมาใช้เป็นพารามิเตอร์ของฟังก์ชัน `NtAllocateVirtualMemory()` ซึ่งปรากฎในลำดับต่อมา โดยฟังก์ชันถูกเตรียมพร้อมในลักษณะของ function pointer เอาไว้อยู่แล้วซึ่งทำให้เราสามารถใช้งานฟังก์ชันนี้ได้ทันที

ฟังก์ชัน `NtAllocateVirtualMemory()` เป็นฟังก์ชันสำหรับการจองและจัดการพื้นที่ในหน่วยความจำของโปรเซส โดยพารามิเตอร์ที่มีความสำคัญต่อกการทำงานของโปรแกรมมีดังต่อไปนี้

- พารามิเตอร์ `ProcessHandle` เป็นพารามิเตอร์ซึ่งมีไว้ให้เราระบุโปรเซสที่เราจะใช้ฟังก์ชัน `NtAllocateVirtualMemory()` เข้าไปจองหน่วยความจำ ในทีนี้เราจะเห็นการใช้ฟังก์ชัน `GetCurrentProcess()` เพื่อให้ได้มาซึ่ง process handle ของโปรเซสปัจจุบันซึ่งบ่งชี้ให้เห็นว่าการจองพื้นที่หน่วยความจำที่กำลังจะเกิดขึ้นนั้นอยู่ในขอบเขตของโปรเซสปัจจุบัน
- พารามิเตอร์ `*BaseAddress` ซึ่งเป็นตัวแปรแบบพอยน์เตอร์ที่จะเก็บตำแหน่งของหน่วยความจำซึ่งเป็นผลลัพธ์จากการทำงานของฟังก์ชัน `NtAllocateVirtualMemory()` ในที่นี้โค้ดมีการระบุตำแหน่งของตัวแปร `pBuffer` ซึ่งถูกสร้างไว้ก่อนหน้า
- พารามิเตอร์ `RegionSize` เป็นพารามิเตอร์ซึ่งใช้ในการระบุขนาดในหน่วยไบต์ของพื้นที่ในหน่วยความจำที่จะมีการจอง ในที่นี้เราจะเห็นการนำค่า `uSize` ซึ่งมีที่มาจากตัวแปร `uReturnLength` ของการเรียกฟังก์ชัน `ZwQuerySystemInformation()` มาใช้งาน

พารามิเตอร์อื่นๆ ซึ่งถูกระบุในการเรียกใช้ฟังก์ชัน อาทิ `AllocationType`, `ZeroBits` และ `Protect` เป็นพารามิเตอร์เสริมซึ่งใช้เพื่อระบุคุณลักษณะอื่นๆ ของพื้นที่หน่วยความจำที่ทำการจอง ผลลัพธ์ของการเรียกใช้ฟังก์ชัน `NtAllocateVirtualMemory()` จะถูกตรวจสอบว่าฟังก์ชันทำงานเสร็จสิ้นหรือไม่ หากไม่เสร็จสิ้นฟังก์ชันจะถูกหยุดการทำงาน แต่หากฟังก์ชันนี้ทำงานเสร็จสิ้น เราก็จะได้พื้นที่ใหม่ในหน่วยความจำของโปรเซสปัจจุบันตามขนาดที่เราระบุ และสามารถเรียกใช้ได้ผ่านตำแหน่งของหน่วยความจำซึ่งถูกเก็บอยู่ในตัวแปร `pBuffer`

ต่อมาฟังก์ชัน `ZwQuerySystemInformation()` จะถูกเรียกใช้เป็นครั้งที่สองด้วยจุดประสงค์ที่ต่างออกไป เนื่องจากในตอนนี้เรามีพื้นที่ว่างในหน่วยความจำพร้อมใช้งานแล้ว เราก็สามารถระบุพื้นที่ดังกล่าวให้กับฟังก์ชัน `ZwQuerySystemInformation()` เพื่อจัดเก็บข้อมูลต่อได้ สังเกตว่าธรรมชาติของภาษาซีนั้น ก่อนจะมีการดำเนินการเรียกหาข้อมูลใดๆ เราจำเป็นจะต้องจัดเตรียมพื้นที่เอาไว้จัดเก็บผลลัพธ์เสมอ ซึ่งนั่นคือสิ่งที่เราเห็นในการเรียกใช้ฟังก์ชัน `ZwQuerySystemInformation()` ครั้งแรกกับการเรัยกใช้ฟังก์ชัน `NtAllocationVirtualMemory()` ครับ

การเรียกใช้ฟังก์ชัน `ZwQuerySystemInformation()` ในครั้งที่สองจะสังเกตได้อย่างชัดเจนว่าตัวแปร `pBuffer` นั้นถูกใส่ไว้ในตำแหน่งของพารามิเตอร์ `SystemInformation` เพื่อเป็นตำแหน่งที่จะเก็บข้อมูลผลลัพธ์ของการทำงาน โดยเมื่อสิ้นสุดการทำงานแล้ว พื้นที่หน่วยความจำซึ่งถูกอ้างอิงด้วยตัวแปร `pBuffer` จะเก็บข้อมูลประเภท `SystemProcessInformation` หรืออธิบายอีกอย่างได้คือข้อมูลของรายละเอียดของโปรเซสที่กำลังทำงานอยู่ในระบบ (หากใครสนใจว่า `pBuffer` เก็บข้อมูลในลักษณะใด ก็สามาถดีบั๊กเพื่อดูค่าในตัวแปรนี้ได้ตามสะดวกครับ)

```c
_RtlEqualUnicodeString RtlEqualUnicodeString = (_RtlEqualUnicodeString) GetProcAddress(GetModuleHandle(L"ntdll.dll"), "RtlEqualUnicodeString");
if (RtlEqualUnicodeString == NULL) {
  return FALSE;
}
```

โค้ดส่วนต่อมาคือการสร้าง function pointer ให้กับฟังก์ชันชื่อ `RtlEqualUnicodeString()` ด้วยรูปแบบที่เราคุ้นเคยกันดี ฟังก์ชัน `RtlEqualUnicodeString()` มีการทำงานตรงตัวตามชื่อของมันการรับค่าอินพุตเข้าไปสองค่าในประเภท `PCUNICODE_STRING` หรือสตริงซึ่งใช้ยูนิโค้ด จากนั้นทำการเปรียบเทียบว่าสตริงทั้งสองค่านั้นเป็นสตริงเดียวกันหรือไม่ หากเป็นสตริงเดียวกันฟังก์ชัน `RtlEqualUnicodeString()` จะส่งผลลัพธ์เป็น `TRUE` และเป็น `FALSE` หากไม่เป็นสตริงเดียวกัน

```c
PSYSTEM_PROCESSES pProcInfo = (PSYSTEM_PROCESSES) pBuffer;
```

ในส่วนถัดมาเราจะเห็นโค้ดในลักษณะที่เหมือนกับการทำ function pointer แต่ในความคิดของผมนั้น มันจะสามารถเข้าใจได้ง่ายกว่าถ้าเราอธิบายโค้ดในบรรทัดด้านบนว่าเป็นการเอาค่าผลลัพธ์จากการทำงานของฟังก์ชัน `ZwQuerySystemInformation()` นั้นมาทำการแปลงให้อยู่ในประเภทข้อมูลที่ชื่อ `PSYSTEM_PROCESSES` และเก็บผลลัพธ์ที่ได้ไว้ในตัวแปรชื่อ `pProcInfo` การทำแบบนี้มีจุดประสงค์เพื่อให้เราสามารถเข้าถึงค่าใน `pBuffer` ได้อย่างเป็นรูปแบบตามโครงสร้างที่ประเภทข้อมูลนี้พึงมี เราสามารถดูรายละเอียดเพิ่มเติมของ `PSYSTEM_PROCESSES` ได้จาก[ไฟล์ `ATPMiniDump.h` ที่บรรทัดที่ 87 ครับ](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.h#L87)

```c
typedef struct _SYSTEM_PROCESSES {
	ULONG NextEntryDelta;
	ULONG ThreadCount;
	ULONG Reserved1[6];
	LARGE_INTEGER CreateTime;
	LARGE_INTEGER UserTime;
	LARGE_INTEGER KernelTime;
	UNICODE_STRING ProcessName;
	KPRIORITY BasePriority;
	HANDLE ProcessId;
	HANDLE InheritedFromProcessId;
} SYSTEM_PROCESSES, *PSYSTEM_PROCESSES;
```

เมื่อผ่านการแปลงประเภทแล้ว เราสามารถมโนต่อได้เลยว่าหากเราเรียกดูข้อมูลโดยใช้การอ้างเป็น `&pProcInfo->ProcessName` เราก็จะได้รายการของชื่อโปรเซสจากผลลัพ์ของ `SystemProcessInformation` นั่นเอง

ในตอนนี้เรามีรายการข้อมูลของโปรเซสที่ทำงานอยู่ในระบบอยู่แล้ว ด้วยจุดประสงค์ของฟังก์ชัน `GetPID()` คือการหาค่า process ID และค่านำเข้าของฟังก์ชันซึ่งคือ `pWinVerInfo->ProcName` เราน่าจะพอเดาสิ่งที่จะเกิดขึ้นต่อไปได้แล้วว่าสิ่งที่จะเกิดขึ้นต่อไปในการทำงานของฟังก์ชันนี้นั้นคือการนำข้อมูลใน `pProcInfo` ซึ่งเก็บรายการโปรเซสที่ทำงานอยู่ทั้งหมดในระบบมาเปรียบเทียบชื่อกับข้อมูลในตัวแปร `pWinVerInfo->ProcName` ด้วยฟังก์ชัน `RtlEqualUnicodeString()` ซึ่งถูกสร้างขึ้นมาแต่ยังไม่ถูกเรียกใช้ เมื่อมีเปรียบเทียบชื่อจนเจอแล้ว เราก็สามารถคาดเดาได้ว่าค่า `pProcInfo->ProcessId` ของโปรเซสที่มีชื่อตรงกับที่เราต้องการจะถูกย้ายมาเก็บไว้ใน `pWinVerInfo->hTargetPID` ที่มีการ initialize ค่าเป็น `NULL` เอาไว้ตั้งแต่เริ่มต้นฟังก์ชัน

โค้ดส่วนถัดไปอธิบายสิ่งที่เราคาดเดาไว้ด้านบนครับ

```c
do {
		if (RtlEqualUnicodeString(&pProcInfo->ProcessName, &pWinVerInfo->ProcName, TRUE)) {
			pWinVerInfo->hTargetPID = pProcInfo->ProcessId;
			break;
		}
		
		pProcInfo = (PSYSTEM_PROCESSES)(((LPBYTE)pProcInfo) + pProcInfo->NextEntryDelta);
} while (pProcInfo);
```

มีการเรียกใช้ `do-while` ในการไล่หาข้อมูลใน `pProcInfo` และเราจะเห็นการขยับค่าใน `pProcInfo` ด้วยการเอาตำแหน่งข้อมูลปัจจุบันมารวมระยะห่างของข้อมูลในรายการโปรเซสซึ่งถัดไปด้วย `pProcInfo->NextEntryDelta` ด้วย ในกรณีที่มีโปรเซสซึ่งมีชื่อตรงกับที่เราต้องการ ค่า process ID ก็จะถูกนำไปเก็บไว้ `pWinVerInfo->hTargetPID` แต่กรณีที่ไม่พบแล้วนั้น ค่าใน `pWinVerInfo->hTargetPID` ก็จะยังคงเป็นค่า `NULL` ตามที่ iniitalize ไว้ในตอนแรกเหมือนเดิม

```c
status = NtFreeVirtualMemory(GetCurrentProcess(), &pBuffer, &uSize, MEM_RELEASE);

if (pWinVerInfo->hTargetPID == NULL) {
  return FALSE;
}

return TRUE;
```

ส่วนสุดท้ายของฟังก์ชัน `GetPID()` คือการเรียกใช้ฟังก์ชัน `NtFreeVirtualMemory()` เพื่อจัดการคืนพื้นที่ของหน่วยความจำซึ่งถูกจองไว้ด้วยฟังก์ชัน `NtAllocateVirtualMemory()` หลังจากนั้นฟังก์ชันจะมีการตรวจสอบค่าผลลัพธ์ว่า `pWinVerInfo->hTargetPID` ยังมีค่าเป็น `NULL` อยู่หรือไม่ หากใช่ก็จะมีการส่งค่า `FALSE` ออก แต่หากไม่ก็จะมีการส่งหา `TRUE` ออก เป็นอันจับการทำงานของฟังก์ชัน `GetPID()` ครับ

### IsElevated Function

> ดูโค้ดของฟังก์ชัน `IsElevated()` แบบเต็มได้[ที่นี่](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.c#L80)

ฟังก์ชัน `IsEleveated()` เป็นฟังก์ชันที่จะส่งออกข้อมูลประเภท `BOOL` ไม่มีการรับอากิวเมนต์ใดๆ เข้ามาประมวลผลในฟังก์ชัน และจะส่งออกค่าออกเป็น `TRUE` หรือ `FALSE` เท่านั้น

จุดประสงค์ของฟังก์ชัน `IsElevated()` คือการตรวจสอบ access token ของโปรเซสปัจจุบันว่าได้มีการกำหนดประเภทของค่าใน access token ว่ามีการตั้งค่าที่ถูกต้องแล้วหรือไม่ โดยค่าใน access token ที่ฟังก์ชันนี้จะทำการตรวจสอบคือค่าใน struct ชื่อ `TOKEN_ELEVATION` ซึ่งค่าใน access token ที่ระบุสิทธิ์ของผู้ดูแลระบบ

ฟังก์ชันทำการตรวจสอบ access token ปัจจุบันที่โปรเซส `ATPMiniDump.exe` มีอยู่ผ่านฟังก์ชัน `OpenAccessToken()` โดยฟังก์ชัน `OpenAccessToken()` มีการรับค่า process handle ซึ่งเป็นเป้าหมายของการตรวจสอบ สิทธิ์และรูปแบบในการเข้าถึง access token ของโปรเซสดังกล่าวและพอยน์เตอร์ซึ่งชี้ไปยังตัวแปรที่จะใช้เพื่อจัดเก็บตำแหน่งของ access token handle โดยในกรณีนี้เราจะสามารถสังเกตได้ว่าฟังก์ชันมีการใช้ process handle ที่เป็นผลลัพธ์จากการเรียกใช้ฟังก์ชัน `GetCurrentProcess()` ซึ่งจะได้ผลลัพธ์เป็น process handle ของโปรเซสปัจจุบัน

```c
if (!OpenProcessToken(
        GetCurrentProcess(), // ProcessHandle
        TOKEN_QUERY | TOKEN_ADJUST_PRIVILEGES, // DesiredAccess
        &hToken // TokenHandle
        )
    ) {
    TOKEN_ELEVATION Elevation = { 0 };
    DWORD cbSize = sizeof(TOKEN_ELEVATION);
}
```

ในกรณีที่ฟังก์ชัน `OpenProcessToken()` ทำงานเสร็จสิ้นและไม่เกิดข้อผิดพลาด ฟังก์ชันจะมีการสร้างตัวแปร `Elevation` จาก struct `TOKEN_ELEVATION` และมีการสร้างตัวแปร `cbSize` โดยมีการกำหนดค่าให้จัดเก็บขนาดของ struct `TOKEN_ELEVATION` เอาไว้ในรูปแบบ DWORD

หากสงสัยว่าตัวแปรสองตัวนี้ถูกสร้างขึ้นมาเพราะอะไร โค้ดด้านล่างคือคำตอบครับ

```c
if (GetTokenInformation(
      hToken, // TokenHandle
      TokenElevation, // TokenInformationClass
      &Elevation, // TokenInformation
      sizeof(Elevation), // TokenInformationLength
      &cbSize // ReturnLength
      ) 
    ) {
  fRet = Elevation.TokenIsElevated;
}
```

ฟังก์ชัน `GetTokenInformation()` คือฟังก์ชันถัดมาซึ่งถูกเรียกใช้ หน้าที่ของฟังก์ชันนี้คือการเรียกหาข้อมูลแบบระบุประเภทซึ่งอยู่ใน access token โดยเราสามารอธิบายการทำงานของฟังก์ชัน `GetTokenInformation()` ด้วยพารามิเตอร์ที่ถูกระบุมาไว้ก่อนแล้วได้ดังนี้

ให้ดึงข้อมูลจาก access token โดยที่

- ถ้าดึงมาได้แล้วให้เก็บตำแหน่งของ access token ไปที่ access token handler ชื่อ `hToken`
- ประเภทของข้อมูลที่เราจะดึงจาก access token คือประเภท `TOKEN_ELEVATION`
- ถ้าดึงมาได้แล้วให้ข้อมูลที่ดึงมาเก็บไว้ที่ตำแหน่งที่ของตัวแปร `Elevation`
- ขนาดของตัวแปรสำหรับเก็บข้อมูลซึ่งในที่นี้คือตัวแปร `Elevation` มีขนาดคือ `sizeof(Elevation)`
- ค่าที่ดึงมาจะต้องมีขนาดไม่เกิน `&cbSize` ซึ่งก็คือลิมิตขนาดของข้อมูลประเภท `TOKEN_ELEVATION`

หากอ่านตามความสำคัญของพารามิเตอร์ด้านบน เราจะพบว่าหากฟังก์ชันทำงานสำเร็จ เราสามารถดูข้อมูลหรือผลลัพธ์ที่แท้จริงได้จากตัวแปร `Elevation` ที่ถูกกำหนด struct เอาไว้แล้วให้สอดคล้องกับข้อมูลประเภท `TOKEN_ELEVATION`

อ้างอิงจาก [TOKEN_ELEVATION structure](https://docs.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-token_elevation) ในกรณีที่โปรเซสมีสิทธิ์ของผู้ดูแลระบบอยู่แล้ว ค่าภายใน struct `TOKEN_ELEVATION` ที่มีชื่อว่า `TokenIsElevated` จะต้องไม่เป็นศูนย์ ฟังก์ชัน `IsElevated()` จะใช้วิธีการส่งค่าภายใน `Elevation.TokenIsElevated` ออกไปเป็นผลลัพธ์ ทั้งนี้ในกรณีที่การทำงานของฟังก์ชันหลุดออกจากเงื่อนทั้ง `OpenProcessToken()` หรือ `GetTokenInformation()` ฟังก์ชัน `TokenIsElevated()` จะส่งค่าออกมาเป็น `FALSE` หรือ `0` ทันที

### SetDebugPrivilege Function

> ดูโค้ดของฟังก์ชัน `SetDebugPrivilege()` แบบเต็มได้[ที่นี่](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.c#L96)

ฟังก์ชัน `SetDebugPrivilege()` เป็นฟังก์ชันที่จะส่งออกข้อมูลประเภท `BOOL` ไม่มีการรับอากิวเมนต์ใดๆ เข้ามาประมวลผลในฟังก์ชัน และจะส่งออกค่าเป็น `TRUE` หรือ `FALSE` เท่านั้น

จุดประสงค์ของฟังก์ชัน `SetDebugPrivilege()` คือการกำหนดสิทธิ์ของโปรเซส `ATPMiniDump.exe` ให้มีสิทธิ์ `SeDebugPrivilege` ซึ่งเป็นสิทธิ์ของระบบปฏิบัติการที่ทำให้สามารถตรวจสอบและเปลี่ยนแปลงการทำงานของข้อมูลในหน่วยความจำของโปรเซสใดๆ ซึ่งมีบัญชีผู้ใช้งานอื่นเป็นเจ้าของ รวมไปถึงดำเนินการด้วยสิทธิ์ของระบบบางอย่างได้ กระบวนการกำหนดสิทธิ์จะทำโดยการแก้ไขค่าเกี่ยวกับสิทธิ์ที่โปรเซส `ATPMiniDump.exe` มีอยู่เพื่อให้ได้สิทธิ์ตามต้องการ โดยค่าดังกล่าวในสภาพแวดล้อมของระบบปฏิบัติการ Windows นั้นจะมีชื่อเรียกว่า access token

ค่า access token เป็นค่าซึ่งได้มาเมื่อผู้ใช้งานเข้าสู่ระบบโดยจะถูกถือครองโดยผู้ใช้งานและโปรเซสที่ถูกสร้างโดยผู้ใช้งาน ค่าที่อยู่ใน access token จะกำหนดสิทธิ์ที่ผู้ใช้งานและโปรเซสของผู้ใช้งานสามารถดำเนินการภายในระบบ ระบบจะทำการตรวจสอบ access token ดังกล่าวว่ามีสิทธิ์เพียงพอที่จะดำเนินการในเรื่องอย่างใดอย่างหนึ่งหรือไม่

ฟังก์ชันทำการตรวจสอบ access token ปัจจุบันที่โปรเซส `ATPMiniDump.exe` มีอยู่ผ่านฟังก์ชัน `OpenAccessToken()` เช่นเดียวกับที่ปรากฎในฟังก์ชัน `IsElevated()`

```c
if (!OpenProcessToken(
        GetCurrentProcess(), // ProcessHandle
        TOKEN_QUERY | TOKEN_ADJUST_PRIVILEGES, // DesiredAccess
        &hToken // TokenHandle
        )
    ) {
    return FALSE;
}
```

ค่าใน access token ประกอบไปด้วยข้อมูลหลายส่วนรวมไปถึงส่วนซึ่งจะใช้ในการระบุสิทธิ์คือค่า struct ที่มีชื่อว่า `SEP_TOKEN_PRIVILEGES` กระบวนการในลำดับต่อไปคือการสร้างตัวแปร `TokenPrivileges` จาก struct `TOKEN_PRIVILEGES` และจะมีการกำหนดค่าซึ่งจะส่งผลให้โปรเซสที่มี access token ปัจจุบันนั้นมีสิทธิ์ `SeDebugPrivilege` การกำหนดค่าให้กับสมาชิกในตัวแปร `TokenPrivileges` มีตามลักษณะของ struct ดังนี้

- ระบุจำนวนของรูปแบบ privilege ที่ไว้ที่ `TokenPrivileges.PrivilegeCount` ในที่นี้ค่าดังกล่าวจะถูกระบุเป็น `1` เนื่องจากรูปแบบของ privilege ที่จะถูกกำหนดลงไปนั้นมีอยู่รูปแบบเดียว
- ระบุประเภทของรูปแบบ privilege ที่จะกำหนด เราจะเห็น conditional operator ของภาษาซีในส่วนนี้คือเครื่องหมาย `?` การระบุประเภทดังกล่าวจะทำการกำหนดค่า `SE_PRIVILEGE_ENABLED` ซึ่งเป็นการกำหนดค่า privilege ในกรณีที่เงื่อนไข `TokenPrivileges.Privileges[0].Attribute` มีอยู่จริง (ซึ่งมีจริงอยู่แล้ว) ค่า `SE_PRIVILEGE_ENABLED` เมื่อถูกใส่ไปใน access token แล้วจะถือว่าโปรเซสที่มี access token มีสิทธิ์ `SeDebugPrivilege` ในทันที

```c
TokenPrivileges.PrivilegeCount = 1;
TokenPrivileges.Privileges[0].Attributes = TRUE ? SE_PRIVILEGE_ENABLED : 0;
```

นอกเหนือจากส่วนของค่าใน access token เกี่ยวกับสิทธิ์ ค่าภายใน access token ยังต้องประกอบไปด้วยค่าเฉพาะตามโครงสร้างของ access token ที่ระบบกำหนด ค่าเฉพาะดังกล่าวซึ่งถูกกำหนดให้นำมาใช้ตาม[โครงสร้างของ access token](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-token) ที่ระบบกำหนดคือค่า locally unique identifier (LUID) ซึ่งจะแตกต่างกันไปเมื่อมีการบูตระบบขึ้นมา โปรแกรมจะมีการเรียกใช้ฟังก์ชัน `LookupPrivilegeValueW()` เพื่อนำค่า LUID ดังกล่าวมาใส่ในแอตทริบิวต์ของตัวแปร `TokenPrivileges` ตามที่ระบบกำหนดไว้ ในกรณีที่การดำเนินในขั้นตอนนี้ไม่เสร็จสมบูรณ์ โปรแกรมจะหยุดการทำงานทันที

```c
LPWSTR lpwPriv = L"SeDebugPrivilege";
if (!LookupPrivilegeValueW(
        NULL, // lpSystemName
        (LPCWSTR)lpwPriv, // lpName
        &TokenPrivileges.Privileges[0].Luid // lpLuid
        )
    ) {
    CloseHandle(hToken);
    return FALSE;
}
```

เมื่อองค์ประกอบของ access token สมบูรณ์แล้ว โปรแกรมจะนำ access token ใหม่ไปใช้งานผ่านฟังก์ชัน `AdjustTokenPrivilges()` ซึ่งจะรับตำแหน่งของ access token handle ปัจจุบันของโปรเซสไป และรับค่า access token ใหม่จากตัวแปร `TokenPrivileges` ไปปรับใช้ ในกรณีที่การดำเนินในขั้นตอนนี้ไม่เสร็จสมบูรณ์ โปรแกรมจะหยุดการทำงานทันที


```c
if (!AdjustTokenPrivileges(
        hToken, // TokenHandle
        FALSE, // DisableAllPrivileges
        &TokenPrivileges,  // NewState
        sizeof(TOKEN_PRIVILEGES), // BufferLength
        NULL, // PreviousState
        NULL // ReturnLength
        )
    ) {
    CloseHandle(hToken);
    return FALSE;
}
```

เมื่อดำเนินการเสร็จสิ้น โปรแกรมจะดำเนินการปิด access token handle ที่มีการใช้งานอยู่ ก่อนจะส่งสถานะการทำงานกลับไปยังฟังก์ชันซึ่งทำการเรียกใช้ฟังก์ชัน `SetDebugPrivilege()`

## Notes

### Loading External Function with Function Pointer

จุดสังเกตที่น่าสนใจอย่างหนึ่งในโค้ดของโครงการ ATPMiniDump คือการเรียกใช้งาน Windows API ในระดับ kernel อยู่หลายส่วนตามตัวอย่างด้านล่างซึ่งเป็นโค้ดในฟังก์ชัน `wmain()`

```c
_RtlGetVersion RtlGetVersion = (_RtlGetVersion) GetProcAddress(GetModuleHandle(L"ntdll.dll"), "RtlGetVersion");

if (RtlGetVersion == NULL) {
    return FALSE;
}
```

โค้ดด้านบนมีจุดมุ่งหมายในการพยายามเรียกใช้ฟังก์ชัน `RtlGetVersion()` จากไลบรารี `ntdll.dll` ที่ป็น kernel-mode API โดยในขณะเดียวกันก็มี user-mode API ที่ชื่อ `GetVersionEx()` ซึ่งมีลักษณะและรูปแบบผลลัพธ์ใกล้เคียงกัน

อย่างไรก็ตามเนื่องจากไลบรารี `ntdll.dll` ซึ่งเก็บ kernel-mode API หลายรายการเอาไว้ให้เรียกใช้ได้นั้นไม่ได้เป็นไลบรารีพื้นฐานที่มักถูกเรียกใช้หรือมีความพร้อมใช้งานโดยทั่วไป ดังนั้นการจะทำให้ฟังก์ชัน `RtlGetVersion()` สามารถถูกเรียกใช้ได้โดยโค้ดในฟังก์ชัน `wmain()` จึงจำเป็นต้องมีการเพิ่มขั้นตอนหรือกลไกในการเตรียมฟังก์ชัน `RtlGetVersion()` ให้สามารถเรียกใช้ได้ขึ้นมา อย่างไรก็ตามก่อนที่เราจะพูดถึงกลไกลในส่วนนี้ การได้รู้เหตุผลว่าทำไมเราถึงต้องมีกลไกนี้อาจจะช่วยให้เราเข้าใจการทำงานในโค้ดตรงส่วนนี้มากขึ้นครับ

```c
_RtlGetVersion RtlGetVersion = (_RtlGetVersion) GetProcAddress(GetModuleHandle(L"ntdll.dll"), "RtlGetVersion");
```

หากเราพยายามทำความเข้าใจโค้ดด้านบน เราจะเห็นลำดับการทำงานตามขั้นตอนดังนี้

1. เพื่อที่จะใช้ฟังก์ชัน `RtlGetVersion()` ในไลบรารี `ntdll.dll` ที่ยังไม่สามารถเรียกใช้ได้ทันที โค้ดจึงต้องทำการสร้างการเชื่อมโยงไปยังไลบรารี `ntdll.dll` ก่อนด้วยฟังก์ชัน `GetModuleHandle()` พร้อมระบุไลบรารีที่จะเรียกหา
2. ผลลัพธ์ของการใช้ `GetModuleHandle(L"ntdll.dll")` จะทำให้ได้ handle ที่เก็บตำแหน่งที่อยู่ของโมดูลที่เราเรียกในหน่วยความจำ โดย handle นี้คือจุดที่ทำให้เราเชื่อมโยงไปยังไลบรารีที่เราต้องการได้
3. หลังจากได้ตำแหน่งของไลบรารีหรือโมดูล `ntdll.dll` ในหน่วยความจำมาแล้ว โค้ดมีการเรียกใช้ฟังก์ชัน `GetProcAddress()` ซึ่งทำหน้าที่ในการหาตำแหน่งจองฟังก์ชันในไลบรารีหรือโมดูล โดยในที่นี้ฟังก์ชัน `GetProcAddress()` จะถูกใช้เพื่อเรียกหาตำแหน่งของฟังก์ชัน `RtlGetVersion()` ซึ่งอยู่ในไลบรารี `ntdll.dll`

ผลลัพธ์ที่ได้จากการเรียกใช้ฟังก์ชัน `GetModuleHandle()` และฟังก์ชัน `GetProcAddress()` คือตำแหน่งของฟังก์ชัน `RtlGetVersion()` ในหน่วยความจำ อย่างไรก็ตามเพื่อให้โค้ดของโปรแกรมสามารถเรียกใช้ฟังก์ชัน `RtlGetVersion()` ได้ การรู้เพียงแค่ว่าฟังก์ชัน `RtlGetVersion()` นั้นอยู่ที่ไหนนั้นไม่เพียงพอ เพราะโดยทั่วไปนั้นโค้ดซึ่งจะทำหน้าที่เป็นฟังก์ชันจะต้องมีส่วนประกอบส่วนอื่นอีกที่จำเป็น อาทิ ประเภทของผลลัพธ์ที่ฟังก์ชันจะส่งออกมาเมื่อทำงานจนเสร็จสิ้น หรืออากิวเมนต์ จำนวนอากิวเมนต์และประเภทอากิวเมนต์นำเข้าที่ฟังก์ชันดังกล่าวจะรับเข้ามาประมวลผล เราจึงจำเป็นต้องเพิ่ม**กลไก**ที่จะเข้ามามีบทบาทเพื่อช่วยรวบรวมโครงสร้างของฟังก์ชันให้สมบูรณ์

```c
_RtlGetVersion RtlGetVersion = (_RtlGetVersion) 0x12345678
```

ถ้าเราสมมติว่าผลลัพธ์ของ `GetModuleHandle()` และ `GetProcAddrss()` นั้นคือ `0x12345678` ซึ่งก็คือตำแหน่งของ `RtlGetVersion()` ในหน่วยความจำ สิ่งที่เกิดขึ้นต่อมาคือการเปลี่ยนการทำ type casting ค่า `0x12345678` ให้เป็นประเภทเฉพาะที่เรากำหนดไว้ก่อนแล้วซึ่งในที่นี้คือ `_RtlGetVersion` ก่อนจะนำผลลัพธ์ที่ได้ไปกำหนดให้กับตัวแปร `RtlGetVersion` ที่เป็นตัวแปรประเภท `_RtlGetVersion` เช่นกัน

พอถึงจุดนี้เราอาจจะงงว่า `_RtlGetVersion` ซึ่งเป็นประเภทของตัวแปรมันมาจากไหนและมีที่มาอย่างไร คำตอบสำหรับคำถามนี้จะอยู่ในไฟล์ [`ATPMiniDump.h` ที่บรรทัดที่ 114](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.h#L114) ครับ

```c
typedef NTSTATUS(NTAPI *_RtlGetVersion)(
	LPOSVERSIONINFOEXW lpVersionInformation
);
```

คำว่า `typedef` เป็นคีย์เวิร์ดในภาษาซีซึ่งทำให้เราสามารถกำหนดชื่อ (alias) ให้กับประเภทของข้อมูลได้ในไวยากรณ์คือ `typedef type-definition alias` เช่น `typedef int length` คือการสร้างประเภทใหม่ชื่อ `length` จากประเภทจำนวนเต็ม (`int`) ดังนั้นจากโค้ดในไฟล์ `ATPMiniDump.h` ด้านบน เราสามารถอธิบายมันได้เป็นการสร้างประเภทของข้อมูลใหม่ในชื่อ `_RtlGetVersion` จากประเภท `NTSTATUS` ซึ่งเป็นประเภทของข้อมูลหนึ่งในระบบปฏิบัติการ Windows อย่างไรก็ตามการใช้ `typedef` ในลักษณะนี้ไม่ได้เป็นเพียงแค่การสร้างชื่อใหม่กับประเภทของข้อมูลเดิม แต่เป็นการใช้ `typedef` เพื่อสร้างสิ่งที่เราจะเรียกมันว่า **function pointer**

**Function pointer** เป็นเหมือนเงาของฟังก์ชันที่แท้จริง เมื่อ function pointer ถูกเรียกใช้งานมันจะรับอากิวเมนต์ที่มีประเภทและจำนวนตามที่ฟังก์ชันซึ่งมันชี้อยู่ต้องการ ก่อนจะส่งอากิวเมนต์เหล่านั้นไปยังตำแหน่งในหน่วยความจำซึ่งฟังก์ชันที่แท้จริงอยู่ ในส่วนของวิธีการลำดับการส่งและการล้างอากิวเมนต์นั้น กระบวนการเหล่านี้จะถูกกำหนดด้วย [calling convention](https://en.wikipedia.org/wiki/X86_calling_conventions) ซึ่งจะถูกระบุไว้ที่ตัว function pointer เช่นเดียวกัน

ดังนั้นจากโค้ด `typedef` ด้านบน เราจะเห็นการสร้างประเภทของข้อมูลใหม่ในชื่อ `_RtlGetVersion` จากประเภท `NTSTATUS` โดยมี `NTAPI` ซึ่งเป็น calling convetion ที่มีรากเหง้าคือ `__stdcall` มาประกอบและช่วยให้รู้ว่าพารามิเตอร์หรืออากิวเมนต์ควรถูกจัดเรียงและถูกจัดการอย่างไร ส่วนสุดท้ายคือการรับพารามิเตอร์ซึ่งเราจะเห็นว่าใน `typedef` ของเรานั้นจะมีการรับพารามิเตอร์ `lpVersionInformation` ซึ่งมีประเภทเป็น `LPOSVERSIONINFOEX` เข้าไป

```c
_RtlGetVersion RtlGetVersion = (_RtlGetVersion)GetProcAddress(GetModuleHandle(L"ntdll.dll"), "RtlGetVersion");

if (RtlGetVersion == NULL) {
		return FALSE;
}
```

ผลสุดท้ายตัวแปร `RtlGetVersion` ในประเภท `_RtlGetVersion` ก็สามารถที่จะถูกใช้ได้เสมอกับเรานำเข้าฟังก์ชัน `RtlGetVersion()` จาก `ntdll.dll` มาใช้งานเอง หากใครเห็นลักษณะโค้ดในด้านบนจากในโครงนี้อีก รวมไปถึงเงื่อนไขซึ่งถูกเพิ่มเข้ามาเพื่อช่วยตรวจสอบว่ากระบวนการนำเข้าฟังก์ชันจากไลบรารีข้างนอกนั้นสำเร็จไหม ให้คิดทันทีว่านี่เป็นลักษณะของการใช้ function pointer ครับ

# Final Notes

- แม้ว่าไฟล์ไบนารีของโปรแกรม mimikatz จะถูกตรวจจับและถูกระบุเป็นโปรแกรมอันตรายได้โดยเทคโนโลยีตรวจจับมัลแวร์โดยส่วนใหญ่ การสร้างไฟล์ข้อมูลที่มีการจัดเก็บข้อมูลในหน่วยความจำของโปรเซส `lsass.exe` มักเป็นเทคนิคที่สามารถใช้เพื่อหลบหลีกการตรวจจับได้ โดยกระบวนการ process memory dumping ยังสามารถทำได้ผ่านทางเครื่องมือทั่วไปของระบบ อาทิ โปรแกรม Task Manager หรือโปรแกรมอีกหลายโปรแกรมในชุดโปรแกรม Sysinternals ของ Microsoft ด้วย
- อย่างไรก็ตามเงื่อนไขสำคัญของการทำ process memory dumping ในลักษณะนี้คือความจำเป็นของการต้องสร้างไฟล์ในระบบ ดังนั้นการตรวจหาพฤติกรรมผิดปกติโดยการตรวจหาไฟล์ที่ต้องสงสัยว่าจะเป็นไฟล์ที่เกิดขึ้นจากโปรเซส `lsass.exe` ผ่านเทคนิคต่างๆ เช่น การทำ binary pattern matching ด้วย Yara ก็อาจสามารถช่วยระบุความผิดปกติได้