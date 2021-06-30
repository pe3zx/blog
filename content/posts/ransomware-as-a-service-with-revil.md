---
title: "Ransomware as a Service With REvil (ESXi variant)"
date: 2021-06-29T20:55:33+07:00
author: "P. Boonyakarn"
description: สะท้อนความเคลื่อนไหวของตลาดบริการ Ransomware as a Service ผ่านการวิเคราะห์ REvil ใหม่ที่สามารถใช้เพื่อโจมตี ESXi ได้
---

หลังจาก DARKSIDE ประกาศยุติปฏิบัติการ และความเคลื่อนไหวที่ยังมีอยู่ของ DARKSIDE ในเว็บบอร์ดใต้ดินในช่วงไม่ถึงเดือนที่ผ่านมา ความเคลื่อนไหวในตลาดบริการของ Ransomware as a Service (RaaS) ก็มีอะไรน่าสนใจหลายอย่าง ส่วนหนึ่งที่ผมสามารถสังเกตเห็นได้อย่างชัดเจนคือความพยายามในการแข่งขันในบริการ Ransomware as a Service (RaaS) ให้รวมศูนย์และจบในตัวเองให้ได้ เพื่อให้บริการของตัวเองมีจุดเด่นเหนือกว่ากว่าบริการจากคู่แข่งซึ่งจะเป็นการการันตีผลประกอบการจากส่วนแบ่ง

ความเคลื่อนไหวบางส่วนที่สามารถรีแคปมาได้มีตามประเด็นดังนี้

1. **RaaS พยายามอย่างยิ่งที่จะใช้ความเร็วในการเข้ารหัสเป็นจุดขาย** สอดคล้องกับประเด็น surviability ในงานวิจัย cryptovirology การสร้างข้อได้เปรียบเหนือคู่แข่งในประเด็นนี้อยู่ที่การปรับเปลี่ยนชุดอัลกอริธึมการเข้ารหัสโดยยังคงความเป็น Hybrid crypto เอาไว้ ตัวเลือกที่ DARKSIDE ใช้ในขณะนี้คือ ChaCha20 กับ RSA 4096-bit โดยใช้ประโยชน์จากความเป็น Stream cipher ของ ChaCha20 หลังจากเคยใช้ Salsa20 ในช่วงปี 2020 (ผมได้เคยเขียนประเด็นเรื่องการใช้ TEA ของมัลแวร์เรียกค่าไถ่ Phobos เอาไว้ด้วย [อ่านเพิ่มเติมได้ที่นี่](https://pandora.sh/posts/quick-note-on-phobos-ransomware/))
2. **การขยาย coverage เพื่อให้สามารถสร้างความเสียหายในวงกว้างที่สุดเท่าที่จะทำได้** เราจะเห็นตัวอย่างของการขยาย coverage ผ่านการทำให้มัลแวร์เรียกค่าไถ่รองรับการทำงานในอุปกรณ์และแพลตฟอร์มที่หลากหลายมากยิ่งขึ้น ในกรณีของ DARKSIDE จุดขายในส่วนนี้ถูกนำเสนอผ่านความสามารถในการเข้ารหัส NAS และ ESXi
3. **ความสามารถในการจัดการเงินค่าไถ่** บริการ RaaS พยายามจะทำให้การจัดการเงินค่าไถ่จบอย่างสมบูรณ์แบบในตัวเองผ่านการใช้งานบริการในกลุ่ม mixer/tumbler เพื่ออำพรางธุรกรรม คนใช้แพลตฟอร์ม RaaS สามารถถอนเงินได้ตามต้องการ (ไม่ระบุวิธีการ) โดยที่ตัวเลือกของสกุลเงินที่รองรับยังคงเป็น BTC และ XMR

เราต่างรู้กันดีว่าในกรณีของการโจมตี Colonial Pipeline โดย DARKSIDE นั้น FBI สามารถที่ยึดเงินค่าไถ่คืนมาได้โดยไม่ระบุวิธีการว่าทำได้อย่างไร ผมมีความสงสัยว่าประเด็นนี้อาจจะกระทบต่อจุดขายที่ 3 อยู่พอสมควร ไว้จะหาข้อมูลเพิ่มเติมในส่วนนี้ว่าตลาด RaaS มีปฏิกิริยาอย่างไรกับบริการของ DARKSIDE ที่โฆษณาออกมาในลักษณะนี้ครับ

อย่างไรก็ตาม แม้ DARKSIDE จะได้รับความสนใจเป็นอย่างมากในช่วงที่ผ่านมา การได้รับความสนใจก็ไม่ได้การันตีส่วนแบ่งในตลาด (หรือในบางทีอาจทำให้ส่วนแบ่งหายไปก็ได้) บริการ RaaS เจ้าอื่นอย่าง REvil ที่มีชื่อมานานกว่าก็ยังให้บริการอยู่ด้วยฟีเจอร์ที่คล้ายกัน

ในโพสต์นี้ ผมจะทำ quick overview กับตัวอย่างมัลแวร์เรียกค่าไถ่ REvil ซึ่งถูกออกแบบมาให้สำหรับการโจมตีสภาพแวดล้อมที่เป็น ESXi ไฟล์ตัวอย่างมัลแวร์ตัวนี้มาจากการรวบรวมของทีม VX underground สำหรับใครซึ่งสนใจจะเปรียบเทียบฟีเจอร์กับฝั่งของ DARKSIDE ผมขอแนะนำ[โพสต์จาก Trend Micro](https://www.trendmicro.com/en_us/research/21/e/darkside-linux-vms-targeted.html) ซึ่งได้มีการวิเคราะห์ไฟล์ของมัลแวร์เรียกค่าไถ่ DARKSIDE ที่ซัพพอร์ตการโจมตี ESXi ตามลิงค์ครับ

# Sample Info

ไฟล์ตัวอย่างของมัลแวร์ที่ถูกพูดถึงในโพสต์นี้มีค่าแฮช SHA256 คือ `ea1872b2835128e3cb49a0bc27e4727ca33c4e6eba1e80422db19b505f965bc4` ไฟล์ตัวอย่างของมัลแวร์ถูกคอมไพล์เป็น ELF 64-bit, dynamically linked, stripped ไม่มีลักษณะการ obfuscation/packing และมีการ ปรากฎ compilation banner คือ `GCC: (Ubuntu 4.8.4-2ubuntu1~14.04.4) 4.8.4`

# Analysis

REvil ในสายพันธุ์นี้เปิดช่องทางให้ผู้ใช้สามารถกำหนดลักษณะการทำงานได้ก่อนเริ่มการทำงาน ตัวเลือกซึ่งใช้เพื่อกำหนดลักษณะการทำงานมีดังนี้

- ระบุพาธในการเข้ารหัสผ่านพารามิเตอร์ `--path` หากไม่ระบุ มัลแวร์จะเข้ารหัสเฉพาะในพาธปัจจุบัน
- ระบุจำนวนเธรดผ่านพารามิเตอร์ `--threads` หากไม่ระบุ ค่าเริ่มต้นของมัลแวร์จะอยู่ที่ 50 เธรด
- ระบุ boolean flag ผ่านพารามิตเตอร์ `--slient` หากค่านี้เป็นจริง มัลแวร์เรียกค่าไถ่จะทำงานโดยไม่มีการหยุดการทำงานของ virtual machine

เนื่องจากโค้ดของมัลแวร์ไม่ได้มีมาตรการป้องกันการวิเคราะห์เป็นพิเศษ เราสามารถเห็นการทำงานเบื้องต้นคร่าว ๆ หากไล่วิเคราะห์ฟังก์ชันซึ่งถูกเรียกเป็นลำดับไปได้ดังนี้

![main](https://i.imgur.com/5Kcodmp.jpg#center)

- ส่วนของการอ่านค่าอากิวเมนต์ที่ถูกระบุเข้ามาตอนเริ่มการทำงานของมัลแวร์
- ส่วนของการอ่านการตั้งค่าซึ่งถูกฝังมาอยู่แล้วกับไฟล์มัลแวร์
- ส่วนของการจัดการกับ virtual machine ที่ยังทำงานอยู่ ในกรณีที่ไม่ได้มีการระบุ `--silent` 
- ส่วนของการสร้างเธรด และจัดการเธรดหลังจากเข้ารหัสเสร็จสิ้น
- ส่วนของการไล่พาธ ซึ่งประกอบไปด้วยส่วนของการสร้างกุญแจเข้ารหัสเฉพาะในแต่ละไฟล์ ส่วนของการเข้ารหัสไฟล์และส่วนที่ใช้เพื่อสร้าง ransom note ในแต่ละไดเรกทอรี

มัลแวร์เรียกค่าไถ่ของ REvil ในสายพันธุ์ที่พุ่งเป้าโจมตีสภาพแวดล้อม ESXi ยังคงสืบทอดความเหมือนกับมัลแวร์เรียกค่าไถ่ซึ่งปรากฎในสภาพแวดล้อมอื่น จุดหนึ่งซึ่งมีความเหมือนกันเป็นเอกลักษณ์คือส่วนการตั้งค่าของมัลแวร์ที่จะถูกฝังเอาไว้ในลักษณะของ JSON ตามรูปแบบด้านล่าง

```json
{
   "pk":"r58UPwgbaRk5py762WpY/rEsl1jd936THXwqUwID/iM=",
   "pid":"$2a$12$V3e/gZmP0hFlQhnJLAyOM.Fsb56ksfw0p42oLlNwf2Jou485ElO4K",
   "sub":"7987",
   "dbg":false,
   "et":0,
   "nbody":"LS0tPT09IFdlbGNvbWUuIEFnYWluLiA9PT0tLS0KClsrXSBXaGF0cyBIYXBwZW4/IFsrXQoKWW91ciBmaWxlcyBhcmUgZW5jcnlwdGVkLCBhbmQgY3VycmVudGx5IHVuYXZhaWxhYmxlLiBZb3UgY2FuIGNoZWNrIGl0OiBhbGwgZmlsZXMgb24geW91ciBzeXN0ZW0gaGFzIGV4dGVuc2lvbiB7RVhUfS4KQnkgdGhlIHdheSwgZXZlcnl0aGluZyBpcyBwb3NzaWJsZSB0byByZWNvdmVyIChyZXN0b3JlKSwgYnV0IHlvdSBuZWVkIHRvIGZvbGxvdyBvdXIgaW5zdHJ1Y3Rpb25zLiBPdGhlcndpc2UsIHlvdSBjYW50IHJldHVybiB5b3VyIGRhdGEgKE5FVkVSKS4KClsrXSBXaGF0IGd1YXJhbnRlZXM/IFsrXQoKSXRzIGp1c3QgYSBidXNpbmVzcy4gV2UgYWJzb2x1dGVseSBkbyBub3QgY2FyZSBhYm91dCB5b3UgYW5kIHlvdXIgZGVhbHMsIGV4Y2VwdCBnZXR0aW5nIGJlbmVmaXRzLiBJZiB3ZSBkbyBub3QgZG8gb3VyIHdvcmsgYW5kIGxpYWJpbGl0aWVzIC0gbm9ib2R5IHdpbGwgbm90IGNvb3BlcmF0ZSB3aXRoIHVzLiBJdHMgbm90IGluIG91ciBpbnRlcmVzdHMuClRvIGNoZWNrIHRoZSBhYmlsaXR5IG9mIHJldHVybmluZyBmaWxlcywgWW91IHNob3VsZCBnbyB0byBvdXIgd2Vic2l0ZS4gVGhlcmUgeW91IGNhbiBkZWNyeXB0IG9uZSBmaWxlIGZvciBmcmVlLiBUaGF0IGlzIG91ciBndWFyYW50ZWUuCklmIHlvdSB3aWxsIG5vdCBjb29wZXJhdGUgd2l0aCBvdXIgc2VydmljZSAtIGZvciB1cywgaXRzIGRvZXMgbm90IG1hdHRlci4gQnV0IHlvdSB3aWxsIGxvc2UgeW91ciB0aW1lIGFuZCBkYXRhLCBjYXVzZSBqdXN0IHdlIGhhdmUgdGhlIHByaXZhdGUga2V5LiBJbiBwcmFjdGlzZSAtIHRpbWUgaXMgbXVjaCBtb3JlIHZhbHVhYmxlIHRoYW4gbW9uZXkuCgpbK10gSG93IHRvIGdldCBhY2Nlc3Mgb24gd2Vic2l0ZT8gWytdCgpZb3UgaGF2ZSB0d28gd2F5czoKCjEpIFtSZWNvbW1lbmRlZF0gVXNpbmcgYSBUT1IgYnJvd3NlciEKICBhKSBEb3dubG9hZCBhbmQgaW5zdGFsbCBUT1IgYnJvd3NlciBmcm9tIHRoaXMgc2l0ZTogaHR0cHM6Ly90b3Jwcm9qZWN0Lm9yZy8KICBiKSBPcGVuIG91ciB3ZWJzaXRlOiBodHRwOi8vYXBsZWJ6dTQ3d2dhemFwZHFrczZ2cmN2NnpjbmpwcGtieGJyNndrZXRmNTZuZjZhcTJubXlveWQub25pb24ve1VJRH0KCjIpIElmIFRPUiBibG9ja2VkIGluIHlvdXIgY291bnRyeSwgdHJ5IHRvIHVzZSBWUE4hIEJ1dCB5b3UgY2FuIHVzZSBvdXIgc2Vjb25kYXJ5IHdlYnNpdGUuIEZvciB0aGlzOgogIGEpIE9wZW4geW91ciBhbnkgYnJvd3NlciAoQ2hyb21lLCBGaXJlZm94LCBPcGVyYSwgSUUsIEVkZ2UpCiAgYikgT3BlbiBvdXIgc2Vjb25kYXJ5IHdlYnNpdGU6IGh0dHA6Ly9kZWNvZGVyLnJlL3tVSUR9CgpXYXJuaW5nOiBzZWNvbmRhcnkgd2Vic2l0ZSBjYW4gYmUgYmxvY2tlZCwgdGhhdHMgd2h5IGZpcnN0IHZhcmlhbnQgbXVjaCBiZXR0ZXIgYW5kIG1vcmUgYXZhaWxhYmxlLgoKV2hlbiB5b3Ugb3BlbiBvdXIgd2Vic2l0ZSwgcHV0IHRoZSBmb2xsb3dpbmcgZGF0YSBpbiB0aGUgaW5wdXQgZm9ybToKS2V5OgoKCntLRVl9CgoKLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0KCiEhISBEQU5HRVIgISEhCkRPTlQgdHJ5IHRvIGNoYW5nZSBmaWxlcyBieSB5b3Vyc2VsZiwgRE9OVCB1c2UgYW55IHRoaXJkIHBhcnR5IHNvZnR3YXJlIGZvciByZXN0b3JpbmcgeW91ciBkYXRhIG9yIGFudGl2aXJ1cyBzb2x1dGlvbnMgLSBpdHMgbWF5IGVudGFpbCBkYW1nZSBvZiB0aGUgcHJpdmF0ZSBrZXkgYW5kLCBhcyByZXN1bHQsIFRoZSBMb3NzIGFsbCBkYXRhLgohISEgISEhICEhIQpPTkUgTU9SRSBUSU1FOiBJdHMgaW4geW91ciBpbnRlcmVzdHMgdG8gZ2V0IHlvdXIgZmlsZXMgYmFjay4gRnJvbSBvdXIgc2lkZSwgd2UgKHRoZSBiZXN0IHNwZWNpYWxpc3RzKSBtYWtlIGV2ZXJ5dGhpbmcgZm9yIHJlc3RvcmluZywgYnV0IHBsZWFzZSBzaG91bGQgbm90IGludGVyZmVyZS4KISEhICEhISAhISEA",
   "nname":"{EXT}-readme.txt",
   "rdmcnt":0,
   "ext":".rhkrc"
}
```

ความเป็นเอกลักษณ์ของลักษณะการตั้งค่าของมัลแวร์เรียกค่าไถ่ REvil นั้นทำให้การทำความเข้าใจการตั้งค่าเหล่านี้ไม่ใช่เรื่องยาก พารามิเตอร์โดยส่วนใหญ่ที่ปรากฎใน variant นั้นต่างเคยถูกวิเคราะห์มาก่อนแล้วโดยบริษัทด้านความปลอดภัย อาทิ [McAfee](https://www.mcafee.com/blogs/other-blogs/mcafee-labs/mcafee-atr-analyzes-sodinokibi-aka-revil-ransomware-as-a-service-what-the-code-tells-us/), [GoogleHeadedHacker](https://www.goggleheadedhacker.com/blog/post/sodinokibi-ransomware-analysis) และ [Intel471](https://intel471.com/blog/revil-ransomware-as-a-service-an-analysis-of-a-ransomware-affiliate-operation) ซึ่งสามารถอธิบายความหมายของการตั้งค่าในแต่ละพารามิเตอร์ได้ดังนี้

- `pk` เป็นค่าของ public key
- `pid` เป็นค่าระบุข้อมูลเกี่ยวกับผู้ใช้งานบริการ RaaS (Affiliate)
- `sub` เป็นค่าระบุ subaccount หรือ campaign ID
- `dbg` เป็น boolean flag สำหรับตรวจสอบว่ามัลแวร์กำลังทำงานใน debug mode หรือไม่
- `et` ใช้กำหนด encryption type
- `nbody` คือเนื้อหาของ ransom note ที่ผ่าน Base64 encoding algorithm
- `nname` คือชื่อไฟล์ ransom note ที่จะถูกเขียนไว้ที่พาธ `\`  โดยค่า `{EXT}` จะถูกแทนที่ด้วยค่าใน `ext`
- `rdmcnt` ค่าระบุจำนวนของไดเรกทอรีที่จะทำการเขียน ransom note ถ้าค่าเป็น 0 มัลแวร์จะทำการเขียน ransom note ให้กับทุกไดเรกทอรี
- `ext` คือค่าของนามสกุลไฟล์ที่จะนำมาต่อท้ายไฟล์ที่ถูกเข้ารหัสแล้ว

เมื่อพูดถึงจุดที่เหมือนกันไปแล้ว เรามาดูในจุดซึ่งแตกต่างกันบ้าง เนื่องจากไฟล์ตัวอย่างของมัลแวร์เรียกค่าไถ่นี้พุ่งเป้าการโจมตีที่ไปสภาพแวดล้อมใน VMware ESXi เพื่อทำให้ความเสียหายเกิดขึ้นมากที่สุดกับระบบ REvil ใน variant ได้มีการอิมพลีเมนต์ฟังก์ชันในการบังคับปิดการทำงานของ virtual machine เอาไว้ซึ่งจะทำงานหากไม่มีการระบุค่า `--silent` เมื่อเริ่มการทำงานมัลแวร์

![force_killing_vms](https://i.imgur.com/XC2ZZ60.jpg#center)

กระบวนการในส่วนนี้ไม่ได้มีอะไรซับซ้อน มัลแวร์สร้างคำสั่งสำหรับเรียกใช้ `pkill -9` กับค่าสตริง `vmx-*` จากนั้นนำมารวมด้วย `sprintf()` ก่อนนำไปเอ็กซีคิวต์ เกิดผลลัพธ์เป็นการบังคับปิดโปรเซสทุกโปรเซสซึ่งมีชื่อเริ่มต้นด้วยสตริง `vmx-*` ซึ่งส่วนใหญ่คือโปรเซสของ virtual machine นอกเหนือจากการใช้ฟังก์ชัน `pkill` เราจะเห็นมัลแวร์เรียกใช้คำสั่ง `esxcli` ด้านล่างเพื่อปิด virtual machine อีกรอบด้วย

```sh
esxcli --formatter=csv --format-param=fields=="WorldID,DisplayName" vm process list | awk -F "\"*,\"*" '{system("esxcli vm process kill --type=force --world-id=" $1)}
```

การวิเคราะห์มัลแวร์เรียกค่าไถ่ REvil ของ [Intel471](https://intel471.com/blog/revil-ransomware-as-a-service-an-analysis-of-a-ransomware-affiliate-operation) ระบุถึงการใช้ไลบรารีที่เกี่ยวข้องกับการเข้ารหัสของ REvil อยู่สองไลบรารี ไลบรารีหนึ่งเป็นไลบรารีของ [Curve25519](https://github.com/vstakhov/opt-cryptobox/tree/master/curve25519) ซึ่งปรากฎตรงกันกับฟังก์ชันในไฟล์ตัวอย่างของมัลแวร์ อีกไลบรารีหนึ่งเป็นของอัลกอริธึม [Salsa20](https://cr.yp.to/snuffle.html) แบบ [merged](https://cr.yp.to/snuffle/salsa20/merged/salsa20.c) ที่มีการใส่โค้ดแบบ inline ของ Salsa20 core เอาไว้ อย่างไรก็ตามดูเหมือนว่าโค้ดที่ถูกเอามาใช้ในการอิมพลีเมนต์นั้นจะมาจากโค้ดแบบ [ref](https://cr.yp.to/snuffle/salsa20/ref/salsa20.c) มากกว่า

![ECRYPT_encrypt_bytes](https://i.imgur.com/X6prfbP.jpg#center)

หลักฐานซึ่งใช้บ่งชี้การใช้โค้ดแบบ [ref](https://cr.yp.to/snuffle/salsa20/ref/salsa20.c) แทนที่แบบ [merged](https://cr.yp.to/snuffle/salsa20/merged/salsa20.c) นั้นสามารถเห็นได้ที่ฟังก์ชัน `0x40fe37` ฟังก์ชันนี้มีลักษณะที่มีอิมพลีเมนต์ฟังก์ชัน `ECRYPT_encrypt_bytes()` มา เราจะสังเกตได้ว่าโค้ดในแบบ ref จะมีการเรียกใช้ฟังก์ชันข้างในด้วย ซึ่งหากอ้างอิงจากโค้ดหลักนั้น ฟังก์ชันดังกล่าวคือ `salsa20_wordtobyte()` ภาพทั้งหมดจะเชื่อมต่อกันหากเราลองนำโค้ดแบบ ref มาลองคอมไพล์และ disassemble เป็นตัวอย่าง

ข้อควรระวังอย่างหนึ่งในการตรวจสอบมัลแวร์ประเภทนี้คือการยืนยันอัลกอริธึมที่มีการใช้งานจริง อัลกอริธึม Salsa20 และ ChaCha ต่างมีการใช้ค่าคงที่ใน Initial state เป็นค่าเดียวคือคำว่า `expand 32-byte k` เราอาจจะพบค่าอื่น เช่น `expand 16-byte k` ด้วย ดังนั้นการพบค่าคงที่เหล่านี้เพียงอย่างเดียวนั้นไม่สามารถใช้ยืนยันอัลกอริธึมที่มีการใช้ในมัลแวร์ได้ การยืนยันอัลกอริธึมที่ใช้ควรเกิดจากการตรวจสอบข้อแตกต่างระหว่าง 2 อัลกอริธึมแทน (อ้างอิง[วิกิพีเดีย](https://en.wikipedia.org/wiki/Salsa20#ChaCha_variant))

ประเด็นน่าสนใจสุดท้ายซึ่งอาจเป็นแนวทางในการปรับปรุงคุณภาพของมัลแวร์เรียกค่าไถ่ต่อไปได้นั้นอยู่ในกระบวนการเข้ารหัส อ้างอิงจากโค้ดซึ่งปรากฎในตัวอย่างไฟล์ของมัลแวร์ โค้ดของมัลแวร์เรียกค่าไถ่ REvil มีเงื่อนไขในการตรวจสอบไฟล์ที่จะเข้ารหัสเพียงแค่ 2 เงื่อนไข ได้แก่

- ดำเนินการตรวจสอบว่าไฟล์ที่จะทำการเข้ารหัสนั้นถูกเข้ารหัสไปแล้วหรือไม่ การตรวจสอบนี้ทำการตรวจสอบว่าในชื่อไฟล์มี substring ที่เป็นนามสกุลไฟล์ซึ่งจะมีอยู่ก็ต่อเมื่อไฟล์ถูกเข้ารหัสแล้วหรือไม่
- เงื่อนไขที่สองคือการพยายามเปลี่ยน mode ของไฟล์เป็น 600 และ 700 ซึ่งหากล้มเหลว ไฟล์นั้นจะไม่ถูกเข้ารหัส (อนุมานในประเด็นนี้ไว้ว่าเป็นเรื่องของสิทธิ์ของบัญชีที่ใช้รันมัลแวร์กับสิทธิ์ของไฟล์)

![file_check](https://i.imgur.com/OEx0C7N.jpg#center)

เนื่องจากตัวเลือกของการเข้ารหัสนั้นขึ้นอยู่กับผู้ใช้งาน ซึ่งหากไม่มีการกำหนดตัวเลือกในการเข้ารหัสนั้นมัลแวร์จะเข้ารหัสทั้งหมดในระบบ เมื่อเปรียบเทียบกับมัลแวร์เรียกค่าไถ่ DARKSIDE ซึ่งโจมตีสภาพแวดล้อม ESXi เหมือนกัน มัลแวร์เรียกค่าไถ่ DARKSIDE เลือกที่จะเข้ารหัสเฉพาะไฟล์ของ virtual machine เป็นค่าเริ่มต้น ซึ่งอาจทำให้กระบวนการเข้ารหัสและสร้างความเสียหายเป็นไปอย่างรวดเร็วขึ้น มัลแวร์เรียกค่าไถ่ DARKSIDE จะมีการเพิ่มเช็คในเรื่องของขนาดไฟล์และพื้นที่ดิสก์ที่เหลืออยู่อีกด้วย

ในกรณีของ REvil ในไฟล์ตัวอย่างซึ่งทำการวิเคราะห์นี้ มัลแวร์จะทำการสร้าง ransom note ด้วยข้อความด้านล่างกับทุกไดเรกทอรี และมีเอกลักษณ์สำคัญคือเพิ่มนามสกุลของไฟล์ที่ถูกเข้ารหัสด้วย `.rhkrc`

![in_action](https://i.imgur.com/8LTJQdF.gif#center)

```
---=== Welcome. Again. ===---

[+] Whats Happen? [+]

Your files are encrypted, and currently unavailable. You can check it: all files on your system has extension {EXT}.
By the way, everything is possible to recover (restore), but you need to follow our instructions. Otherwise, you cant return your data (NEVER).

[+] What guarantees? [+]

Its just a business. We absolutely do not care about you and your deals, except getting benefits. If we do not do our work and liabilities - nobody will not cooperate with us. Its not in our interests.
To check the ability of returning files, You should go to our website. There you can decrypt one file for free. That is our guarantee.
If you will not cooperate with our service - for us, its does not matter. But you will lose your time and data, cause just we have the private key. In practise - time is much more valuable than money.

[+] How to get access on website? [+]

You have two ways:

1) [Recommended] Using a TOR browser!
  a) Download and install TOR browser from this site: https://torproject.org/
  b) Open our website: http://aplebzu47wgazapdqks6vrcv6zcnjppkbxbr6wketf56nf6aq2nmyoyd.onion/{UID}

2) If TOR blocked in your country, try to use VPN! But you can use our secondary website. For this:
  a) Open your any browser (Chrome, Firefox, Opera, IE, Edge)
  b) Open our secondary website: http://decoder.re/{UID}

Warning: secondary website can be blocked, thats why first variant much better and more available.

When you open our website, put the following data in the input form:
Key:


{KEY}


-----------------------------------------------------------------------------------------

!!! DANGER !!!
DONT try to change files by yourself, DONT use any third party software for restoring your data or antivirus solutions - its may entail damge of the private key and, as result, The Loss all data.
!!! !!! !!!
ONE MORE TIME: Its in your interests to get your files back. From our side, we (the best specialists) make everything for restoring, but please should not interfere.
!!! !!! !!!.
```

# Conclusion

จากการวิเคราะห์ไฟล์ตัวอย่างของมัลแวร์เรียกค่าไถ่ REvil และข้อมูลที่เรารู้เกี่ยวกับบริการ Ransomware as a Service ในฝั่งของกลุ่ม DARKSIDE เทคโนโลยีในฝั่งของเหล่าอาชญากรไซเบอร์กำลังพัฒนาและแข่งขันกันอย่างเข้มข้นเพื่อครอบครองอัตราส่วนแบ่งของผลประโยชน์เอาไว้ให้มากที่สุด มันบังคับให้ฝั่งของผู้ที่มีโอกาสเป็นเหยื่ออย่างเราต้องก้าวตามไปให้ทันตามศักยภาพที่เรามี หวังว่าการทำความเข้าใจข้อมูลในบล็อกนี้จะเกิดขึ้นก่อนที่ทุกอย่างจะสายเกินไปครับ