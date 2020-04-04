---
title: "Integrating BinExport with GhiDra"
date: 2020-03-07T16:49:00+07:00
author: "P. Boonyakarn"
description: "เมื่อวันที่ 2 มีนาคม ที่ผ่านมา ผู้พัฒนาโครงการหลักของ BinDiff ได้มีการประกาศเวอร์ชันใหม่ของ BinDiff ในรุ่นที่ 6 โดยในรุ่นนี้นั้นมีหนึ่งในฟีเจอร์สำคัญซึ่งช่วยให้ผู้ใช้งาน Ghidra สามารถบันทึกข้อมูลในโปรเจคที่ทำการวิเคราะห์โดยใช้ BinExport และนำฐานข้อมูลในรูปแบบ BinExport มาหาความแตกต่างด้วย BinDiff ได้"
---

เมื่อวันที่ 2 มีนาคม ที่ผ่านมา ผู้พัฒนาโครงการหลักของ BinDiff ได้มี[การประกาศเวอร์ชันใหม่ของ BinDiff ในรุ่นที่ 6](https://twitter.com/AdmVonSchneider/status/1234178304837668869) โดยในรุ่นนี้นั้นมีหนึ่งในฟีเจอร์สำคัญซึ่งช่วยให้ผู้ใช้งาน Ghidra สามารถบันทึกข้อมูลในโปรเจคที่ทำการวิเคราะห์โดยใช้ BinExport และนำฐานข้อมูลในรูปแบบ BinExport มาหาความแตกต่างด้วย BinDiff ได้

BinDiff มีบทบาทสำคัญในการเป็นเครื่องมือที่ทรงพลังในการช่วยให้ Reverse engineer หาความแตกต่างของไฟล์ไบนารีได้ อย่างไรก็ตามข้อจำกัดอย่างหนึ่งในการใช้งาน BinDiff คือการเปลี่ยนฟอร์แมตของไฟล์ไบนารีที่เราทำการวิเคราะห์ให้อยู่ในรูปแบบที่ BinDiff สามารถนำมาใช้ได้ด้วย BinExport ซึ่งแต่เดิมนั้นถูกพัฒนาให้เป็นคอมโพเนนต์ของ Disassembler อย่าง IDA Pro เท่านั้น การที่ BinExport จะต้องทำงานด้วย IDA Pro ในลักษณะปลั๊กอิน/คอมโพเนนต์ของ IDA Pro ทำให้ผู้ใช้ที่ต้องการจะใช้งาน BinDiff จะต้องครอบครอง [license ของ IDA Pro ที่สามารถใช้ปลัํกอิน/คอมโพเนนต์](https://www.hex-rays.com/order/)ได้

การมาของ Ghidra ซึ่งนอกจากฟังก์ชัน Decompiler ที่สามารถใช้ทดแทน [Hex-Rays Decompiler](https://www.hex-rays.com/products/decompiler/) ที่ต้องเสียเงินเช่นเดียวกับ IDA Pro ได้ ในขณะเดียวกัน Ghidra ก็ยังมาพร้อมกับฟีเจอร์ [Version Tracking](https://ghidra.re/courses/GhidraClass/Intermediate/VersionTracking_withNotes.html#VersionTracking.html) ซึ่งสามารถใช้เพื่อหาความเหมือนและแตกต่างของไฟล์ไบนารีที่ทำการวิเคราะห์อยู่ด้วย

เมื่อเปรียบเทียบประสบการณ์การใช้งานระหว่าง BinDiff และ Version Tracking ของ Ghidra นั้น Version Tracking ของ Ghidra ใช้วิธีการวิเคราะห์ทั้ง instruction และ data ซึ่งจะใช้เวลานานกว่ามากหากยอมให้ Ghidra ใช้ทุกอัลกอริธึมที่มีอยู่ในการค้นหาความเหมือนและแตกต่าง ส่งผลให้ผลลัพธ์ซึ่งได้มาจาก Ghidra มีปริมาณที่เยอะ ใช้ทรัพยากรระบบจำนวนมากในการใช้งานจริง และจำเป็นต้องทำการคัดแยกออกมาตามอัลกอริธึมที่จะใช้เปรียบเทียบจึงจะสามารถเห็นความแตกต่างได้ Ghidra ยังคงมีข้อด้อยในเรื่องของการค้นหาไฟล์ PDB ที่จำเป็นต้องหาด้วยตนเองและโหลดเข้าโปรแกรมก่อนเริ่มทำ Auto-Analysis และใช้ฟีเจอร์ Version Tracking ด้วย

## Compile and Build BinExport

![](https://1.bp.blogspot.com/-jZGUs6We5hI/XmNrrO7aA1I/AAAAAAAAVHk/kZ5WiqghyS04EpMsjAPYtLZb4chbh7nggCLcBGAsYHQ/s1600/Untitled.png)

การคอมไพล์และติดตั้ง BinExport รวมไปถึง BinDiff เพื่อใช้งาน Ghidra สามารถทำได้ตามขั้นตอนดังต่อไปนี้

1. ดาวโหลดและติดตั้ง BinDiff เวอร์ชันล่าสุดจากเว็บไซต์ของ [zynamics](https://zynamics.com/software.html)
2. ติดตั้ง Dependencies ที่จำเป็นสำหรับการคอมไพล์ BinExport ได้แก่
   1. OpenJDK 11 หรือใหม่กว่า
   2. Ghidra 9.1.2 (ดาวโหลดได้จาก [ghidra-sre.org](http://ghidra-sre.org/))
   3. Gradle 5.6 หรือมากกว่า (ดาวโหลดได้จาก [services.gradle.org](http://services.gradle.org/))
3. ทำการแตกไฟล์ของ Ghidra ไปยังพาธที่ต้องการ จากนั้นทำการตั้งค่า environment variable ในชื่อ `GHIDRA_INSTALL_DIR` ให้ชี้ไปยังพาธของโครงการ Ghidra
4. ดาวโหลดโครงการ BinExport จาก [google/binexport](https://github.com/google/binexport/)
5. ในโครงการ binexport ให้เข้าไปที่พาธ `java/BinExport` จากนั้นเรียกใช้คำสั่ง `gradle` หรือ `gradle.ba`t` ตามลักษณะของระบบเพื่อเริ่มกระบวนการคอมไพล์
6. เมื่อกระบวนการคอมไพล์เสร็จสิ้น ไฟล์ปลั๊กอินสำหรับ Ghidra จะถูกเก็บอยู่ในพาธ `java/BinExport/dist` ในลักษณะชื่อไฟล์ `ghidra_9.1.2_PUBLIC_XXXXXXXX_BinExport.zip`

เมื่อได้ไฟล์ปลั๊กอินสำหรับ Ghidra แล้ว เราสามารถโหลดปลั๊กอินดังกล่าวได้จากหน้าต่าง Project ของ Ghidra โดยเข้าไปที่ File และ Install Extension จากนั้นกดเครื่องหมายบวกและเรื่องไฟล์ ZIP ของ BinExport ที่เราคอมไพล์ไว้ในตอนแรก ตรวจสอบให้แน่ใจว่าได้มีการเปิดใช้ปลั๊กอินแล้ว จากนั้นกด OK

## Create a BinExport Database

การสร้างไฟล์ BinExport สามารถทำได้จากหน้า Project ของ Ghidra ตามขั้นตอนดังต่อไปนี้

1. ตรวจสอบให้แน่ใจว่าไบนารีที่จะทำการวิเคราะห์ได้ผ่านการใช้ฟีเจอร์ Auto-analysis แล้ว โดยแนะนำให้มีการใช้ Analyzer ชื่อ **Aggressive Instruction Finder** ด้วย
2. ที่หน้า Project view คลิกขวาที่ไบนารีที่ต้องการแล้วเลือก Export
3. ที่หน้าต่าง Export filename ให้เลือกฟอร์แมตเป็น Binary Export (v2) for BinDiff จากนั้นกด OK
   
ไฟล์ BinExport จะถูกบันทึกในตำแหน่งที่เราได้ระบุเอาไว้

## Diff BinExport with BinDiff

เมื่อเราได้ไฟล์ BinExport สำหรับไบนารีที่เราต้องการเปรียบเทียบมาแล้ว ขั้นตอนต่อไปคือการใช้ BinDiff เพื่อวิเคราะห์ไฟล์ BinExprt เหล่านี้และสร้าง Workspace ที่สามารถนำไปตรวจสอบกับ BinDiff UI ได้ ตามขั้นตอนดังต่อไปนี้

- เรียกใช้คำสั่ง `bindiff` โดยระบุพารามิเตอร์เป็น `bindiff(.exe) Primary.BinExport Secondary.BinExport`
- ไฟล์ `Primary_vs_Secondary.BinDiff` จะถูกสร้างขึ้นที่ไดเรกทอรีปัจจุบัน ให้เราทำการเปิด BinDiff UI และสร้าง Workspace ใหม่ จากนั้นเลือก Diff และ Add Exisiting Diff เพื่อโหลดไฟล์ `Primary_vs_Secondary.BinDiff` เข้าไป เป็นอันเสร็จสิ้นกระบวนการ