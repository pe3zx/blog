---
title: "SANS@MIC TALK: Tricking Modern Endpoint Security Products"
date: 2020-06-04T23:04:44+07:00
author: "P. Boonyakarn"
description: "สรุปประเด็นสำคัญจาก SANS@MIC TALK \"Tricking modern endpoint security products\" โดย Michel Coene"
---

ฟัง Webcast ออนไลน์ได้ที่ [SANS Institute](https://www.youtube.com/watch?v=xmNpS9mbwEc) และดาวโหลดสไลด์ฉบับเต็มได้ที่เว็บไซต์ของ [SANS Institute](https://www.sans.org)

Webcast นี้มีเป้าหมายในการพูดถึงวิธีในการข้ามผ่านการตรวจจับและเฝ้าระวังของซอฟต์แวร์กลุ่ม EDR โดยอาศัยเทคนิคในการดัดแปลงกระบวนการในการทำงานของโปรเซส ความสัมพันธ์ระหว่างโปรเซสซึ่งเป็น Parent และโปรเซสซึ่งเป็น Child และการลบร่องรอยต่างๆ ซึ่งซอฟต์แวร์กลุ่ม EDR จะใช้เป็นข้อมูลในการตรวจจับ กลยุทธ์ที่ได้ระบุมาสอดคล้องตามแนวทางของ [MITRE ATT&CK Enterprise Matrix](https://attack.mitre.org/matrices/enterprise/) ดังนี้
  - เทคนิค [Parent PID Spoofing (T1502)](https://attack.mitre.org/techniques/T1502/)
  - เทคนิค [Process Injection (T1055)](https://attack.mitre.org/techniques/T1055/)
  - เทคนิค [Process Hollowing (T1093)](https://attack.mitre.org/techniques/T1093/)

ทั้งนี้เทคนิค Command-line spoofing เป็นเทคนิคเดียวในการบรรยายที่ยังไม่ถูกระบุเป็นเทคนิคใน MITRE ATT&CK Enterprise Matrix

## Parent PID Spoofing

เทคนิค **Parent PID spoofing** มีเป้าหมายในการเปลี่ยน Parent process ของโปรเซสที่ต้องการเพื่อให้ความสัมพันธ์ระหว่าง Parent process และ Child process นั้นสามารถถูกเอาไปใช้ในการเฝ้าระวังและตรวจจับให้น้อยที่สุด หนึ่งในแนวทางของการทำ Parent PID spoofing คือการเรียกใช้ `CreateProcess` ซึ่งเปิดช่องทางให้ผู้ใช้งานสามารถระบุ Parent process ID ที่ต้องการได้ผ่านทางการสร้างค่า `LPPROC_THREAD_ATTRIBUTE_LIST` ด้วยฟังก์ชัน `InitializeProcThreadAttributeList` ซึ่งจะนำถูกนำมาใช้โดยแอตทริบิวต์ `STARTUPINFOEX` กับฟังก์ชัน `CreateProcess`

ตัวอย่างโค้ดด้านล่างคือโค้ดฉบับปรับปรุงบางส่วนจาก[โปรแกรม SelectMyParent โดย Didier Stevens](http://www.didierstevens.com/files/software/SelectMyParent_v0_0_0_1.zip) ซึ่งแสดงให้เห็นการนำค่า PPID ซึ่งผู้ใช้งานระบุเข้าไปเพื่อใช้ในการสร้างโปรเซสใหม่ภายใต้ PPID ดังกล่าว

```cpp
  dwPid = _tstoi(argv[2]);

  InitializeProcThreadAttributeList(NULL, 1, 0, &cbAttributeListSize);
  pAttributeList = (PPROC_THREAD_ATTRIBUTE_LIST) HeapAlloc(GetProcessHeap(), 0, cbAttributeListSize);

  InitializeProcThreadAttributeList(pAttributeList, 1, 0, &cbAttributeListSize);
  CurrentProcessAdjustToken();
  
  hParentProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPid);
  UpdateProcThreadAttribute(pAttributeList, 0, PROC_THREAD_ATTRIBUTE_PARENT_PROCESS, &hParentProcess, sizeof(HANDLE), NULL, NULL);

  sie.lpAttributeList = pAttributeList;
  CreateProcess(NULL, argv[1], NULL, NULL, FALSE, EXTENDED_STARTUPINFO_PRESENT, NULL, NULL, &sie.StartupInfo, &pi)
```

(**ความเห็น**) อย่างที่มีการระบุไว้ในสไลด์ เทคนิค Parent PID spoofing ในปัจจุบันนั้นเป็นเทคนิคซึ่งถูกนำไปอิมพลีเมนต์อย่างแพร่หลายแล้ว ผมแนะนำให้ค้นคว้าจาก[บล็อก Parent PID Spoofing โดย PenTest Lab](https://pentestlab.blog/2020/02/24/parent-pid-spoofing/) เพิ่มเติมเกี่ยวกับการใช้งาน

## Command-line Spoofing

Webcast นำเสนอการใช้เทคนิค Command-line Spoofing ผ่านการแก้ไข Process Environment Block ซึ่งมีสตรัคเจอร์ `RTL_USER_PROCESS_PARAMETERS` สำหรับการเก็บข้อมูลที่เกี่ยวข้องกับโปรเซสอยู่ในส่วน `ProcessParameters`

```cpp
typedef struct _PEB {
  BYTE                          Reserved1[2];
  BYTE                          BeingDebugged;
  BYTE                          Reserved2[1];
  PVOID                         Reserved3[2];
  PPEB_LDR_DATA                 Ldr;
  PRTL_USER_PROCESS_PARAMETERS  ProcessParameters;
  PVOID                         Reserved4[3];
  PVOID                         AtlThunkSListPtr;
  PVOID                         Reserved5;
  ULONG                         Reserved6;
  PVOID                         Reserved7;
  ULONG                         Reserved8;
  ULONG                         AtlThunkSListPtr32;
  PVOID                         Reserved9[45];
  BYTE                          Reserved10[96];
  PPS_POST_PROCESS_INIT_ROUTINE PostProcessInitRoutine;
  BYTE                          Reserved11[128];
  PVOID                         Reserved12[1];
  ULONG                         SessionId;
} PEB, *PPEB;
```

จุดน่าสนใจของสตรัคเจอร์ `RTL_USER_PROCESS_PARAMETERS` นั้นอยู่ที่ตัวแปร `ImagePathName` และ `CommandLine` ซึ่งเก็บข้อมูลแบบสตริงที่จะถูกส่งไปเป็นข้อมูลประกอบของโปรเซส

```cpp
typedef struct _RTL_USER_PROCESS_PARAMETERS {
  BYTE           Reserved1[16];
  PVOID          Reserved2[10];
  UNICODE_STRING ImagePathName;
  UNICODE_STRING CommandLine;
} RTL_USER_PROCESS_PARAMETERS, *PRTL_USER_PROCESS_PARAMETERS;
```

ด้วยไอเดียของการแก้ไขค่า `CommandLine` เพื่อซ่อนข้อมูลและปลอมแปลง การดำเนินการแก้ไขข้อมูลในส่วนนี้จึงสามารถทำได้และยังทำได้หลายวิธีการ อ้างอิงจากข้อมูลใน Webcast ฟีเจอร์ argue ของ Cobalt Strike มีลำดับการเรียกใช้ฟังก์ชันของระบบเพื่อแก้ไขค่าในพารามิเตอร์ `CommandLine` ดังนี้

1. สร้างโปรเซสขึ้นมาด้วยฟังก์ชัน `CreateProcess` โดยให้ทำการระบุ `CREATE_SUSPEND` ไปในฟังก์ชันเอาไว้ด้วยเพื่อให้โปรเซสหยุดการทำงานชั่วคราวทันทีที่เริ่มต้น
2. ทำการระบุตำแหน่งในหน่วยความจำของ PEB ด้วยฟังก์ชัน `NtQueryInformationProcess` โดยฟังก์ชันนี้จะทำการระบุหาตำแหน่งของ PEB และจัดเก็บไว้ในฟื้นที่ที่ระบุไว้ให้กับฟังก์ชัน