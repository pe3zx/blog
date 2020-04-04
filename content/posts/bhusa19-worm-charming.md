---
title: "BHUSA19: Worm Charming - Harvesting Malware Lures for Fun and Profit"
date: 2019-10-13T15:08:00+07:00
author: "P. Boonyakarn"
description: "This is a summary for BlackHat USA 2019 talk, Worm Charming - Harvesting Malware Lures for Fun and Profit, by Pedram Amini from InQuest. The presentation of this talk is available here."
---

This is a summary for BlackHat USA 2019 talk, **Worm Charming - Harvesting Malware Lures for Fun and Profit**, by Pedram Amini from InQuest. The presentation of this talk is [available here](https://i.blackhat.com/USA-19/Wednesday/us-19-Amini-Worm-Charming-Harvesting-Malware-Lures-For-Fun-And-Profit.pdf).

- The main point of this talk is focusing around an idea to hunt for malware, especially a dropper in many forms such as PDF, Office, Java, Flash, scripts in archives which deliver via e-mail attachment or drive-by download attack.
- There are many places we can obtain new detail about an ongoing attack campaign and its malware. Online sandbox and threat intelligence platforms such as Hybrid Analysis, Any.run and VirusTotal already provide not only a sample but a way to efficiently hunt a new sample with its integration and search capabilities. Most threat research teams publish discovery on Twitter and their blog which can subscribe via RSS feed. You can also dig into your enterprise e-mail spool or mapping threat actor infrastructure manually in case it's a targeted attack.
- With those channels, there are OSS projects that can help us productively gather intel. [InQuest/ThreatIngestor](http://inquest/ThreatIngestor) can help extract and aggregate IOCs from threat feeds. You can build your own tool with [InQuest/python-iocextract](https://github.com/InQuest/python-iocextract) which already has the capability to extract numerous kinds of IOC from Twitter or a blog post. Intelligence Response team has already wrote [a review for ThreatIngestor](https://www.i-secure.co.th/2019/05/introduction-to-threatingestor/) in Thai, available at i-secure's Blog.
- One IOC can be used as a lead to other IOCs with your open-source intelligence tools and skills.
- Pedram gave an overview of the simple hunting process with VirusTotal Intelligence and Yara rules resulting in around 2,500 samples per day to examine. Yara rules mentioned in this section are extracted and available here.
- Some malware has unique properties and it's useful for hunting. For example, MIME or file magic evasions, interesting keywords like "shellcode", "exploit", "heapspray" and UTF-8 BOM (Byte Order Mark) trick with `0xEFBBBF` even new a new trick like RTF-Byte Nibble.

![](https://1.bp.blogspot.com/-ligS3VtBBWI/XaLbB6bL2RI/AAAAAAAATUM/p5BAzfWFi3I8hMmCzAsDc4aMofGjj5McwCLcBGAsYHQ/s1600/Untitled.png)

- A targeted attack usually leaves unseen artifacts and it needs intensive resources to accomplish the examination process. Sandboxed detonation, IOC extraction, and crawling will make it easier to do this, and also these tools.
    - Java: Jad-Retro, CFR, JDCore, FernFlower, Procyon
    - Flash: flasm, xxxswf, FFDec, SWFDump, JPEXS
    - PDF: PDFiD and pdf-parser, PeePDF, PDFtoText, ExifTool
    - Office: OLEDump, OLEFile, catdoc, xlsx2csv, docx2txt