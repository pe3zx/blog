---
title: "Story of Unknown ATM Malware, Thai's Bank and Attribution"
date: 2019-06-27T20:52:00+07:00
author: "P. Boonyakarn"
description: "According to my session for #MiSSConf(SP5), I decided to find some well-known case as an example and it will be used to analyze with MITRE ATT&CK for pre and post-incident assessment. One of the most well-known security incident cases I picked is relating to the heist of Government Savings Bank in Thailand 2016 in which I found many interesting points with the malware. In this post, I will briefly explain those points as a note for myself and a teaser for my talk."
---

According to my session for [#MiSSConf(SP5)](https://missconf.github.io/SP5.html), I decided to find some well-known case as an example and it will be used to analyze with [MITRE ATT&CK](https://attack.mitre.org/) for pre and post-incident assessment. One of the most well-known security incident cases I picked is relating to the heist of Government Savings Bank in Thailand 2016 in which I found many interesting points with the malware. In this post, I will briefly explain those points as a note for myself and a teaser for my talk.

![](https://1.bp.blogspot.com/-7jzweTXug04/XRTCTQdyUbI/AAAAAAAATGs/nber0uVdUicbaNoDVoKGmozMfcMnRjRUgCLcBGAs/s1600/APT-based%2BSecurity%2BAssessment%2Band%2BDetection.png)

On the first week of August 2016, many headlines published about a security incident with one of a financial institution in Thailand, Government Savings Bank or GSB. The incident involved the use of unknown malware with many ATMs around the country to forcefully dispense the money.

Based on [the analysis by FireEye](https://www.fireeye.com/blog/threat-research/2016/08/ripper_atm_malwarea.html), they discovered a piece of malware uploaded to VirusTotal by an IP address in Thailand. It was uploaded a couple of minutes before the media reports about the heist. The analysis gives malware the name "RIPPER". The original of MD5 which packed by UPX is `15632224b7e5ca0ccb0a042daf2adc13`.

Back in today, [Threat Group Cards: A Threat Actor Encyclopedia](https://www.thaicert.or.th/downloads/files/A_Threat_Actor_Encyclopedia.pdf) from ThaiCERT points out that the group responsible for the attack is **Cobalt Group**. By the way, there's no detail about Cobalt Group and ATM RIPPER malware as much as I can find on the Internet. So my big question is, how can they attribute the attack to the threat actor group?

I decided to play a little bit with the malware sample by searching malware with the same attributes. In order to do that, I used the YARA rule posted by sadfud on KernelMode.info to find other samples with Hybrid-Analysis's "YARA search" feature.

![](https://1.bp.blogspot.com/-QooFWmNP-f0/XRTFvzuvd4I/AAAAAAAATG4/rO-oXD-QCooxs5MklQUQ9c7xoV-cOJQcACLcBGAs/s1600/APT-based%2BSecurity%2BAssessment%2Band%2BDetection%2B%25281%2529.png)

As seen in the picture above, the oldest submission is the one from August 29, 2016, done after FireEye's analysis. The malware is identified as "Malware.STk.Generic" type. There's no older sample than that. We can infer that the malware hasn't been used before. More recent versions were also discovered, they were identified as "Backdoor.ATMripper".

I don't have any access to private threat intelligence services except [Autofocus from Palo Alto Networks](https://www.paloaltonetworks.com/products/secure-the-network/autofocus) which also shows the same result. It may have more information and more effective way to find other samples older than that.

NCR published their advisory which explained that the malicious actor may breach internal bank network to install the malware. Later information by the Royal Thai Police force was mentioning about a suspicious group of European/Russian guys who probably behind the attack. Given these facts, I tried to find threat actor groups whose behaviors similar to what reported with the attack. Based on [Groups section of MITRE ATT&CK](https://attack.mitre.org/groups/), there are only two threat actors who have similar attributes from the facts.

![](https://1.bp.blogspot.com/-Zt_8evQAwCs/XRTJOnu5PuI/AAAAAAAATHE/dSXooK4vymUcd8vx_ToRbtgjnkvS32kIwCLcBGAs/s1600/Untitled.png)

This conclusion might be wrong because of [confirmation bias](https://en.wikipedia.org/wiki/Confirmation_bias). But at least we have a list of TTPs we can use for compromised assessment (post-incident assessment).