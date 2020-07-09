---
title: "FIN7 CrimeOps: ศึกษาปฏิบัติการของกลุ่มอาชญากรไซเบอร์ \"FIN7\" กลุ่มนักล่าระบบ Point of Sale (POS)"
date: 2020-07-09T14:08:26+07:00
author: "P. Boonyakarn"
description: "ศึกษาปฏิบัติการของกลุ่มอาชญากรไซเบอร์ FIN7 ทั้งในมุมกลยุทธ์ ปฏิบัติการและเทคนิคการโจมตีทางไซเบอร์ และทำความเข้าใจเหตุผลว่าทำไม FIN7 อาจเป็นกลุ่มอาชญากรไซเบอร์ที่ดีที่สุดในด้าน Operation management"
---

การศึกษาปฏิบัติการของกลุ่มอาชญากรไซเบอร์ FIN7 ซึ่งถูกจัดลำดับตามความซับซ้อนของปฏิบัติการและผลกระทบที่เกิดขึ้นให้อยู่ในกลุ่ม Advanced Persistent Threat หรือ APT นั้นเป็นงาน(อดิเรก)ซึ่งอยู่ในรายการสิ่งที่ต้องทำของผมมานานแล้วตั้งแต่ช่วงปี 2018 ซึ่งมีการเปิดเผยข้อมูลวงในของปฏิบัติการออกมาจาก [indictment ที่ออกโดยกระทรวงยุติธรรมสหรัฐฯ](https://www.justice.gov/opa/press-release/file/1084316/download) อย่างไรก็ตามเนื่องจากกิจกรรมของอาชญากรกลุ่มนี้ยังไม่ได้ปรากฎในประเทศไทยอ้างอิงจากข้อมูลสาธารณะที่เกี่ยวข้องกับภัยคุกคาม  (หรืออาจปรากฎแต่ยังไม่สามารถระบุหรือเชื่อมโยงได้) ปฏิบัติการและข้อมูลที่ถูกเปิดเผยออกมาก็เลยถูกบดบังด้วยกิจกรรมของกลุ่มอาชญากรทางไซเบอร์กลุ่มอื่นแทน

![CrimeOps: The operational art of cybercrime](https://i.imgur.com/3bHqfbG.png)

สิ่งที่ทำให้โพสต์นี้เกิดขึ้นมาได้นั้นมาจากแนวทางเดิมที่ผมทำมาโดยตลอดคือการใช้โพสต์หรือบล็อกเป็นที่สรุปความเข้าใจเมื่อได้เรียนรู้อะไรใหม่ สื่อที่ถูกนำมาสรุปในวันนี้นั้นมาจากงานสัมมนาออนไลน์ซึ่งจัดโดย [OPCDE](https://www.opcde.com/) ในชื่อ vOPCDE ภายใต้หัวข้อ [CrimeOps: The operational art of cybercrime](https://www.youtube.com/watch?v=E9F7WCO-pZM) บรรยายโดย [thegrugq](https://twitter.com/thegrugq) ในหัวข้อการสัมมนานี้นั้น grugq ใช้ TTP ของกลุ่ม FIN7 เป็นแกนหลักในการอธิบายสิ่งที่เรียกว่า crime operations หรือ **CrimeOps** ซึ่งเผยความคิดเห็นที่สะท้อนมุมที่น่าสนใจของ TTP ออกมาให้ได้ทำความเข้าใจเพิ่มเติมครับ

หัวข้อในบทความนี้มีดังต่อไปนี้ครับ

- [Levels of War - The Analytical Framework](#levels-of-war---the-analytical-framework)
- [Cyberizing The Analytical Framework](#cyberizing-the-analytical-framework)
- [Understanding FIN7 CrimeOps](#understanding-fin7-crimeops)
  - [FIN7 People](#fin7-people)
  - [FIN7 Process](#fin7-process)
  - [FIN7 Technology](#fin7-technology)
- [Excelling Operational Art](#excelling-operational-art)
- [How to Manage Cybercrimial Syndicate Efficiently — A Conclusion](#how-to-manage-cybercrimial-syndicate-efficiently--a-conclusion)

บทความนี้ใช้วิธีการเล่าแบบเรียงต่อกัน ดังนั้นการอ่านตามลำดับจะทำให้ได้รับความเข้าใจและประโยชน์สูงสุดครับ

## Levels of War - The Analytical Framework

แม้ว่าในการสัมมนานั้นผู้บรรยายจะเริ่มที่การเกริ่นความน่าสนใจจาก TTP ของ FIN7 ก่อน แต่ผมขอสลับโดยการมาเล่าถึง**วิธีการที่เราจะมองและให้ความหมาย**ที่จะนำไปสู่การเห็นความน่าสนใจจาก TTP ของ FIN7 ก่อน หลังจากนั้นเราค่อยจับ TTP มาเพื่อวิเคราะห์ผ่านเฟรมเวิร์คเหล่านี้ครับ

![Levels of Warfare](https://images.squarespace-cdn.com/content/v1/5497331ae4b0148a6141bd47/1462063537559-18HWZ7YRYO41RAM2UI3X/ke17ZwdGBToddI8pDm48kKnS1laj_EUY2RS4C9bjjNp7gQa3H78H3Y0txjaiv_0fHWpUEmQZkX684UWF7bhg2uXFJOxHRx9nL7VEpwjp7EsNDPIMYUqMpyMYI20OdjP7OqpeNLcJ80NK65_fV7S1UQiv0gy8B3nFCq0Cu5pOGuLUQVCMEYi5js33AwzsRrk-MW9u6oXQZQicHHG1WEE6fg/image-asset.png?format=600w)

อ้างอิงจาก[หลักการพื้นฐานเกี่ยวกับทฤษฎีการทำสงคราม](https://www.cc.gatech.edu/~tpilsch/INTA4803TP/Articles/Three%20Levels%20of%20War=CADRE-excerpt.pdf) สงครามจะมีระดับของการดำเนินการ (Levels of Warfare) อยู่ 3 ระดับ ได้แก่

1. **ระดับ Strategic** หรือ**ระดับยุทธศาสตร์**เป็นระดับของการสร้างคำถามว่าเราจะทำสงครามอย่างไร หรือแม้แต่คำถามว่าเราควรทำสงครามหรือไม่และเพื่ออะไรเกิดขึ้น มันเป็นระดับที่เราจะกำหนดเป้าหมายของการดำเนินการที่จะครอบคลุมประเด็นต่างๆ ทุกประเด็น
2. **ระดับ Operational** หรือ**ระดับยุทธการ**เป็นระดับที่การจัดการองค์ประกอบต่างๆ ให้สอดคล้องกับการทำให้เป้าหมายเป็นจริงนั้นเกิดขึ้น
3. **ระดับ Tactical** หรือ**ระดับยุทธวิธี**เป็นระดับที่การกระทำเพื่อให้เป็นไปตามเป้าหมายนั้นเกิดขึ้น

สมมติว่าเรากำลังเข้าสู่สงครามและศัตรูของเรามีกิจกรรมทางสงครามบางอย่างเกิดขึ้น เราสามารถใช้แนวคิดของ Levels of Warfare เข้าไปให้ความหมายและอธิบายการดำเนินการได้ไม่ต่างกับ [Lockheed Martin Cyber Kill Chain®](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html) หรือ [MITRE ATT&CK Enterprise Attack Framework](https://attack.mitre.org/) ที่เราคุ้นเคย

อย่างไรก็ตามเมื่อเรานำแนวคิดของ Levels of Warfare ไปใช้ แม้เราจะรู้ว่ากิจกรรมประเภทไหนควรอยู่ในระดับใด แต่ความยากและอุปสรรคที่แท้จริงนั้นอยู่ที่การมองหากิจกรรมเหล่านั้นเพื่อเอามาเติมและอธิบายให้ครบวงจร ที่**ระดับ Tactical** การระบุหาตัวบ่งชี้กิจกรรมที่เกิดขึ้นในระดับนี้นั้นไม่ใช่เรื่องที่ยาก บางทีมันอาจเป็นหลักฐานอะไรบางอย่างที่บ่งชี้ให้เห็นเทคนิคและเครื่องมือ เช่นเดียวกับใน**ระดับ Strategic** ที่หากเราศึกษาและค้นคว้าศัตรูอย่างเพียงพอ การระบุเป้าหมายที่ศัตรูต้องการทำให้ได้ก็ไม่ใช่เรื่องที่เป็นไปไม่ได้

ปัญหาแท้จริงในการหาตัวบ่งชี้ของกิจกรรมนั้นอยู่ที่**ระดับ Operational** ซึ่งเป็นปัญหาเดียวกับในขั้นตอน Weaponization ของ Lockheed Martin Cyber Kill Chain® ที่ตัวบ่งชี้และกิจกรรมนั้นเกิดขึ้นและอยู่ที่ฝั่งของศัตรูหรือผู้โจมตี การหาตัวบ่งชี้หรือกิจกรรมเพื่อยืนยันพฤติกรรมในระดับนี้ให้ได้นั้นจึงจำเป็นต้องอาศัยการเข้าถึงในลักษณะพิเศษที่ทำให้ได้มาซึ่งข้อมูลที่น่าเชื่อถือมา ดังนั้นงานในส่วนนี้จึงถูกแตกออกมาเป็นแขนงใหญ่ในด้านเกี่ยวกับการข่าวกรอง (intelligence) และรวมไปถึงการต่อต้านข่าวกรอง (counter intelligence) เพื่อทำให้เกิด operation security หรือ OPSEC ด้วย

## Cyberizing The Analytical Framework

เมื่อภัยคุกคามทางไซเบอร์ก้าวเข้ามาเป็น The Fifth Domain อย่างเป็นทางการต่อจาก Land, Sea, Air และ Space เราจึงจำเป็นต้องเอา Levels of War นั้นมาประยุกต์เพื่ออธิบายระดับของการดำเนินการในโดเมนของสงครามที่เป็นไซเบอร์ และอาชญากรรมทางไซเบอร์ได้ด้วย โดย grugq ได้ยกตัวอย่างการประยุกต์ไว้ดังนี้

1. **ระดับ Strategic** ในลักษณะของอาชญากรรมไซเบอร์ มันอาจเกิดขึ้นในลักษณะของการมองหาซึ่งวิธีการใหม่ในการทำเงินขึ้นมาหรือทำให้มากกว่าเดิม เช่น โจมตีกลุ่มเป้าหมายใหม่ซึ่งยังไม่ตกเป็นเป้าหมายของกลุ่มอื่นๆ หรืออาจมองหาศักยภาพเชิงเทคนิคหรือแนวคิดของเทคโนโลยีใหม่ๆ ซึ่งจะช่วยสร้างตลาดที่อาชญากรจะทำเงินขึ้นมาได้ เช่น แทนที่จะเอาข้อมูลไปขายเพื่อทำกำไร การจับข้อมูลเป็นตัวประกันเพื่อเรียกค่าไถ่อาจการันตีผลกำไรที่ดีกว่า
2. **ระดับ Operational** ในลักษณะของอาชญากรรมไซเบอร์ มักจะเกิดขึ้นในลักษณะของการเตรียมความพร้อมและทรัพยากรให้สามารถไปถึงเป้าหมายเชิงยุทธศาสตร์ที่ตั้งไว้ได้ เช่น ที่แฮกเกอร์ตั้งใจจะแฮกเพื่อขโมยข้อมูลออกมาขาย ข้อมูลนั้นจะต้องเป็นที่ต้องการของตลาดและหาผู้ซื้อได้ ซึ่งการที่จะได้ข้อมูลนั้นมาอาจต้องอาศัยการรคงอยู่และฝังตัวในระบบเป็นเวลานาน การเตรียมความพร้อมจึงอาจเกิดขึ้นในลักษณขะองการเตรียม infrastructure สำหรับการเข้าถึงระบบที่ถูกแฮกและมีมัลแวร์ฝังตัวอยู่ให้แนบเนียนและยากต่อการตรวจจับ
3. **ระดับ Tactical** ในลักษณะของอาชญากรรมไซเบอร์คือขั้นตอนที่การโจมตีเกิดขึ้นจริงเพื่อให้เป็นไปตามเป้าหมายเชิงยุทธศาสตร์ที่ตั้งเอาไว้โดยใช้ทรัพยากรที่เตรียมมาในการสนับสนุน เช่น เริ่มทำ social engineering เพื่อสร้างความน่าเชื่อถือก่อนจะส่งอีเมลแนบมัลแวร์ไปให้กับเป้าหมาย

แท้จริงแล้วการดำเนินการใน**ระดับ Operational** อาจสามารถถูกมองได้ว่ามีความสำคัญที่ต่ำ เพียงแต่การมีอยู่ของการดำเนินการใน**ระดับ Operational** จะช่วยการันตีประสิทธิภาพและอัตราความสำเร็จของปฏิบัติการโดยรวมให้เพิ่มมากยิ่งขึ้นตามปริมาณที่มีอยู่

> เพราะ**อาชญากรรมไซเบอร์คือธุรกิจ** ดังนั้นการจัดการอาชญากรรมไซเบอร์ให้มีประสิทธิภาพก็เหมือนกับการจัดการให้ธุรกิจสร้างผลกำไรและเติบโตได้อย่างมีประสิทธิภาพ

จุดเชื่อมโยงระหว่างแนวคิด Levels of War กับปฏิบัติการและอาชญากรรมของกลุ่ม FIN7 นั้นอยู่ในจุดที่กลุ่ม FIN7 ไม่ได้มีเป้าหมายที่แปลกใหม่ในระดับ Strategic และไม่ได้มีเทคนิคการโจมตีที่พิเศษในระดับ Tactical แต่**กลุ่มมี FIN7 มีแนวคิดของการทำปฏิบัติการที่ทำให้การทำอาชญากรรมสามารถถูกมองได้ว่าเป็นการทำธุรกิจหรือการเปิดบริษัท**ซึ่งเป็นจุดเด่นที่เกิดขึ้นในระดับ Operational

เราจะมาพูดถึงกลุ่ม FIN7 ในส่วนต่อไปครับ

## Understanding FIN7 CrimeOps

รายละเอียดเชิงลึกของปฏิบัติการและการดำเนินการซึ่งนำไปสู่อาชญากรรมของ FIN7 นั้นสามารถหาอ่านได้แบบเต็มๆ จาก [indictment ที่ออกโดยกระทรวงยุติธรรมสหรัฐฯ](https://www.justice.gov/opa/press-release/file/1084316/download) หรือจากบทวิเคราะห์ indictment อื่นๆ เช่น [The Wild Inner Workings of a Billion-Dollar Hacking Group](https://www.wired.com/story/fin7-wild-inner-workings-billion-dollar-hacking-group/) ดังนั้นในส่วนนี้รายละเอียดจะถูกขมวดปมเพื่ออธิบายตามแนวคิดธรรมดาๆ อย่าง People, Process, Technology ตามที่ grugq นำเสนอแทนครับ

> หากเป้าหมายของอาชญากรไซเบอร์คือเงิน การทำเงินให้ได้มากที่สุดก็หมายถึงการทำตามเป้าหมายได้สำเร็จสูงสุด

### FIN7 People

ความพิเศษและเป็นเอกลักษณ์ในการดำเนินการของกลุ่ม FIN7 นั้นคือการทำให้การดำเนินอาชญากรรมทางไซเบอร์เป็นเหมือนกับการทำบริษัทและธุรกิจทั่วไป ซึ่งอันที่จริงนั้นจะบอกว่าเป็น*เหมือน*ก็ไม่ได้เพราะกลุ่ม FIN7 มีการ**สร้างบริษัทด้าน cybersecurity ขึ้นมาบังหน้าปฏิบัติการอาชญากรรม**

กลุ่ม FIN7 ใช้วิธีการจดทะเบียนบริษัททั่วไปโดยอ้างว่าเป็นบริษัทที่ดำเนินงานทางด้าน cybersecurity แต่แท้จริงแล้วเป็นกิจการซึ่งบังหน้าปฏิบัติการที่แท้จริงคือการเจาะระบบและนำข้อมูลบัตรเครดิตและข้อมูลธุรกรรมออกมาขาย คนที่มีส่วนร่วมในปฏิบัติการบางส่วนนั้นก็มาจากการว่าจ้างในลักษณะทั่วไปในตำแหน่งของ penetration tester ซึ่งมีการประกาศรับ การสัมภาษณ์และการให้ทำงานซึ่งเป็นการเจาะระบบจริงๆ

grugq ให้ความเห็นว่าคนซึ่งถูกว่าจ้างเข้าไปทำงานในตำแหน่งดังกล่าวนั้นน่าจะรู้ว่ามีอะไรบางอย่างผิดปกติ เพราะกิจกรรมซึ่งทำจริงๆ นั้นแตกต่างไปจากการทดสอบเจาะระบบโดยทั่วไปอยู่พอสมควรในมุมของการไม่มีรีพอร์ตสรุปผลการทดสอบเจาะระบบ การฝังตัวอยู่ในระบบในระยะเวลาที่นานและการนำข้อมูลจริงออกมา

ด้วยลักษณะขององค์กรอาชญากรรมแบบบริษัท grugq จึงอธิบาย organization chart และตำแหน่งงานรวมไปถึงความรับผิดชอบในแต่ละตำแหน่งออกมาได้ตามรูปด้านล่าง

![FIN7 Organization Chart](https://i.imgur.com/cAnNtMY.png)

จาก organization chart ด้านบน เราจะสามารถเห็นหน้าที่ของแต่ละส่วนซึ่งสามารถอธิบายความรับผิดชอบได้ดังนี้

1. **กลุ่ม Management** มีหน้าที่รับผิดชอบคือ
   1. จัดการในเรื่องผลประโยชน์ของส่วนอื่นๆ ให้ได้รับอย่างเหมาะสม (เหมือนกับจัดการเงินเดือน)
   2. ว่าจ้างบุคลากรใหม่ ทำการสัมภาษณ์ และ มอบหมายงานหรือโปรเจคการโจมตีเป้าหมายให้กับบุคคลไปดำเนินการ
   3. ควบคุมการใช้งานทรัพยากรขององค์กรอาชญากรรม
   4. สนับสนุนทางด้านเทคโนโลยีแก่ผู้ปฏิบัติการที่จำเป็น
   5. (คาดว่า) มีส่วนร่วมกับการเลือกเป้าหมาย เพื่อให้การโจมตีเป้าหมายสร้างผลประโยชน์ที่เหมาะสมที่สุดแก่องค์กร
2. **กลุ่ม Support** มีหน้าที่รับผิดชอบคือ
   1. ดูแลและจัดการ infrastructure ที่ใช้ในปฏิบัติการให้มีความพร้อมใช้อยู่เสมอ
   2. สนับสนุนทางด้านเทคโนโลยีแก่ผู้ปฏิบัติการที่จำเป็น
   3. คอยประสานงานระหว่างผู้ปฏิบัติการที่ทำการโจมตีระบบ
3. **กลุ่ม Operator** มีหน้าที่รับผิดชอบคือการแฮกหรือโจมตีระบบตามคำสั่ง
4. **กลุ่ม Developer** มีหน้าที่รับผิดชอบในการพัฒนาศักยภาพในการโจมตีให้ตรวจจับได้ยากขึเน มีประสิทธิภาพขึ้น
5. **กลุ่ม Interpreter** มีหน้าที่รับผิดชอบในการสนันสนุนการหาและประเมินเป้าหมาย หรือการสรรหาบุคลากร กลุ่มนี้มีส่วนร่วมในการโจมตีบางครั้งหากเป้าหมายมีการใช้ภาษาต่างประเทศที่ผู้ปฏิบัติการไม่เข้าใจ

### FIN7 Process

grugq อธิบายภาพรวมกระบวนการดำเนินงานของกลุ่ม FIN7 เอาไว้ว่าเป็นลักษณะของ "การมีกระบวนการไว้เพื่อรับกลุ่มหรือบริษัทเป็นของนำเข้า และส่งออกมาไว้เป็นเงินสด" ซึ่งเป็นคำอธิบายที่ตรงตามลักษณะของกลุ่ม FIN7 โดย grugq ยังมีการอธิบายกระบวนการในการก่ออาชญากรรมออกมาได้ตามรูปด้านล่าง

![FIN7 Crime Process](https://i.imgur.com/Izp7kSf.png)

เราจะเห็นการแบ่งส่วนของการทำอาชญากรรมโดยกลุ่ม FIN7 ออกเป็น 2 ส่วนใหญ่ๆ ส่วนแรกคือกระบวนการที่เกิดขึ้นก่อนการโจมตีหรือ pre-exploitation และอีกกระบวนการจะเกิดขึ้นหลังจากที่กระบวนการโจมตีเกิดขึ้นและกลุ่ม FIN7 เข้าไปอยู่ในเครือข่าบเป้าหมายได้แล้ว ซึ่งเราจะเรียกมันว่า post-exploitation

grugq ให้ความเห็นในข้อมูลในส่วนนี้ค่อนข้าง straightforward และออกจะน่าเบื่อ เพราะมันเป็นวิธีการทั่วๆ ไปในการดำเนินการอาชญากรรมและเป็นภาพที่เราอาจมีโอกาสเห็นในระดับ Tactical ทั้งนี้จุดน่าสนใจจริงๆ จะอยู่ก่อนหน้ากระบวนการ Target selection และอยู่แทรกในระหว่างการดำเนินการโจมตี ได้แก่

- FIN7 มีการ**ทำ project management ผ่าน JIRA** โดยมองว่าการโจมตีหนึ่งปฏิบัติการนั้นเปรียบเสมือนกับโปรเจคการทำ penetration testing หนึ่งโปรเจค
- เนื่องจากความหลากหลายของสภาพแวดล้อมที่กลุ่ม FIN7 ต้องเจอและทำการโจมตี กลุ่ม FIN7 มีการทำสภาพแวดล้อมขึ้นมาและ**นำหลักการของ DevOps มาใช้** เพื่อให้แน่ใจว่าเครื่องมือและเทคนิคในการโจมตีนั้นจะประสบผลสำเร็จสูงสุด
- FIN7 มีการ**นำแนวคิด Agile มาใช้ทั้งในเรื่องของเครื่องมือที่ใช้ในการโจมตีและปฏิบัติกา**ร โดยอาจถูกนำมาใช้ในลักษณะของการทดสอบก่อนเอาไปใช้จริง ตรวจสอบผลลัพธ์ที่เกิดขึ้นและปรับปรุงแก้ไขอยู่ตลอดเวลา และมีการช่องทางให้ผู้ปฏิบัติการได้ coordination และ collaboration อยู่เสมอ

นอกเหนือจากแนวคิดเหล่านี้แล้ว FIN7 ยังมีการประยุกต์แนวคิดหนึ่งในเรื่องของ project management มาใช้อย่างจริงจังคือการทำ [portfolio management](https://en.wikipedia.org/wiki/Portfolio_manager) โดยเริ่มต้นจากการเปลี่ยนปฏิบัติการโจมตีหรืออาชญากรรมที่ตนเองจะก่อเป็นเสมือนกับหนึ่งโปรเจคงานหรือโครงการของบริษัท จากนั้นจึงมีการใช้เครื่องมืออย่าง JIRA ในการติดตามการใช้งานทรัพยากรของแต่ละโครงการ สถานะปัจจุบัน ปัญหาของการดำเนินการและรายละเอียดอื่นๆ มีการคาดว่า FIN7 มีการใช้ข้อมูลเหล่านี้ในการประเมินการดำเนินการแต่ละโครงการเพื่อทำให้เกิด [return on investment (ROI)](https://en.wikipedia.org/wiki/Return_on_investment) นั้นสูงสุด

### FIN7 Technology

แม้จะเป็นหัวข้อที่น่าสนใจที่สุดเพราะเราจะได้เห็น "อาวุธ" ในการทำอาชญากรรมที่ FIN7 ใช้งานจริง ผู้บรรยายให้ความเห็นในประเด็นนี้ว่าแท้จริงอาวุธเหล่านั้นก็ต่างเป็นเครื่องมือและซอฟต์แวร์ที่รู้จักกันทั่วไปในแวดวง cybersecurity และเชื่อได้ว่าอาวุธโดยส่วนใหญ่ที่กลุ่ม FIN7 ใช้นั้นสามารถหาดาวโหลดได้จาก GitHub ผสมกับซอฟต์แวร์พัฒนาเองบางส่วนซึ่งไม่ได้มีความซับซ้อน grugq ยังให้ความเห็นเพิ่มเติมว่า *"มันไม่ได้ดูเท่ เพียงแต่มันใช้งานจริงได้ ซึ่งแค่นั้นก็เพียงพอแล้ว"*

ในหัวข้อของเทคโนโลยีของกลุ่ม FIN7 จึงเน้นไปที่เทคโนโลยีที่มีไว้เพื่อสนับสนุนการทำงานแทน อาทิ กลุ่ม FIN7 มีการทำ project management ผ่าน JIRA และมีการใช้ทั้ง HipChat, Jabber XMPP และอีเมลในการพูดคุยกัน

หากใครสนใจเพิ่มเติมเกี่ยวเครื่องมือและซอฟต์แวร์ที่กลุ่ม FIN7 ใช้ ก็สามารถหาอ่านได้เพิ่มเติมจากข้อมูลที่รวบรวมไว้ใน [MITRE ATT&CK Framework](https://attack.mitre.org/groups/G0046/) ครับ

## Excelling Operational Art

จากรายละเอียดในมุม People, Process และ Technology เมื่อเรานำแนวคิดของ Levels of War มาใช้ในการวิเคราะห์ เราจะเห็นภาพของการดำเนินการในระดับ Operational ที่น่าสนใจได้ตามประเด็นดังนี้

- กระบวนการโจมตีสามารถเกิดขึ้นได้ในรูปแบบที่ไม่แตกต่างกัน ทำให้ FIN7 สามารถสลับเปลี่ยนคนในกลุ่มปฏิบัติการได้เสมอตามความต้องการ คนใหม่เข้ามาก็ให้เรียนรู้งานผ่าน knowledge base ได้ ไม่จำเป็นต้องมีการฝึกสอนเป็นพิเศษเพราะกระบวนการมีลักษณะคล้ายคลึงเดิมอยู่ตลอดเวลา
- ด้วยการจัดการแบบ portfolio management ที่มีประสิทธิภาพ FIN7 จึงสามารถ "รับงาน" ได้หลายงานพร้อมกัน การประสานงานและทำงานร่วมกันก็มีเครื่องมือในการรองรับ ซึ่งส่งผลทำให้รายได้จากอาชญากรรมที่เข้ามานั้นเพิ่มขึ้นด้วย
- การแบ่งหน้าที่ออกมาชัดเจนทำให้คนในตำแหน่งงานหนึ่งมีความรับผิดชอบและไม่ก้าวก่ายงานซึ่งกันและกัน ระดับบริหารจัดการในภาพรวม การใช้ทรัพยากรและเงิน ในขณะที่ระดับสนับสนุนและปฏิบัติการเน้นไปที่เรื่องเทคโนโลยีซึ่งทำให้มันสามารถถูกพัฒนาได้จริง
- 
  
เมื่อถึงจุดนี้ FIN7 อาจเป็นองค์กร(อาชญากรรม)ไซเบอร์ซึ่งมีวิธีการจัดการบริหารงานที่น่าสนใจและมีประสิทธิภาพได้ดีกว่าองค์กรอื่นๆ เลยด้วยซ้ำและแน่นอนว่าทำได้เงินได้ดีกว่าด้วย อย่างไรก็ตามบทวิเคราะห์ในส่วนนี้ยังไม่บ่งชี้ให้เห็นถึงปัญหาในการทำงานร่วมกันซึ่งมักจะมีในองค์กรทั่วไปและคาดว่าจะมีอยู่ในองค์กรอาชญากรรมอย่าง FIN7 รวมไปถึงประเด็นทางด้านคุณธรรมและจริยธรรมด้วยว่าประเด็นเหล่านี้จะถูกจัดการอย่างไรหากมีพนักงานคนใดๆ คนหนึ่งเกิดไม่เห็นด้วยขึ้นมากับปฏิบัติการที่มีอยู่

## How to Manage Cybercrimial Syndicate Efficiently — A Conclusion

> Crime pays when you move up the value chain into project management
> 
> — thegrugq

grugq สรุปประเด็นในการบรรยายนี้เอาไว้น่าสนใจ โดยเป็นการพูดถึงในมุมที่ว่า**หากคุณดูแลหรือดำเนินกิจการกับองค์กรอาชญากรรมไซเบอร์ นี่คือสิ่งที่คุณควรทำตามอย่าง FIN7** ซึ่งมีประเด็นดังต่อไปนี้

1. พยายามควบคุมให้การมี innovation ในเชิงเทคนิค เช่น มีโค้ดมัลแวร์ที่ซับซ้อนและถูกตรววจับได้ยาก หรือครอบครอง 0day exploit ราคาแพงมีความเหมาะสมกับการมี innovation ในเชิงธุรกิจ เพราะการก้าวไปให้ถึงเป้าหมายของการทำธุรกิจ(หรืออาชญากรรม)ก็ไม่จำเป็นต้องมีความซับซ้อนขนาดนั้นก็ได้
2. สร้างกระบวนการในการดำเนินงาน(หรือทำอาชญากรรม)ที่หยืดหยุ่นแต่ก็มั่นคง ชัดเจนในการส่งต่อและปฏิบัติตาม
3. มีหน้าที่และความรับผิดชอบที่ชัดเจน ซึ่งจะช่วยเพิ่มและจัดการ operation security ได้อย่างเหมาะสม
4. พยายามกระจายงาน(หรือการทำอาชญากรรม)ออกไปในกลุ่มคนให้ได้มากที่สุดเพื่อให้เกิดการทำงานแบบคู่ขนานซึ่งจะช่วยสร้างผลประโยชน์ที่มากขึ้น
5. ใช้ portfolio management ผ่านซอฟต์แวร์ project management ในการประเมินการใช้ทรัพยากรต่อโครงการ(หรือการโจมตีเป้าหมาย) ความคุ้มค่ารวมไปถึงการจัดการและบริหารคนต่องาน 
6. ใช้เทคโนโลยีและแนวคิดใหม่ๆ เข้ามาช่วยหากเป็นไปได้ เช่น ใช้ DevOps เพื่อประเมินการทำงานของมัลแวร์กับระบบจำลองของเป้าหมายในการลดความเสี่ยง หรือเน้นการทำงานแบบ Agile ในการพัฒนามัลแวร์หรือในปฏิบัติการ

ทั้งหมดนี้คือสรุปจากการบรรยายในหัวข้อ [CrimeOps: The operational art of cybercrime](https://www.youtube.com/watch?v=E9F7WCO-pZM) บรรยายโดย [thegrugq](https://twitter.com/thegrugq) และประเด็นของปฏิบัติการและเทคนิคจากกลุ่มอาชญากรไซเบอร์ FIN7 ครับ :)