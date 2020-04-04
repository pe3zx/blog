---
title: "BHUSA19: ClickOnce and You're In - When Appref-ms Abuse is Operating as Intended"
date: 2019-10-14T16:17:00+07:00
author: "P. Boonyakarn"
description: "This a summary for BlackHat USA 2019 talk, ClickOnce and You're In - When Appref-ms Abuse is Operating as Intended, by William J. Burke IV from U.S. Department of Homeland Security. The presentation of this talk is available here."
---

This a summary for BlackHat USA 2019 talk, **ClickOnce and You're In - When Appref-ms Abuse is Operating as Intended**, by William J. Burke IV from U.S. Department of Homeland Security. The presentation of this talk is [available here](https://i.blackhat.com/USA-19/Wednesday/us-19-Burke-ClickOnce-And-Youre-In-When-Appref-Ms-Abuse-Is-Operating-As-Intended.pdf).

- The first thing I want to point out is how the research was conducted and its objectives. The main objective of this research is to develop a new method for initial access and code execution with the following requirements (which might be related to operation requirements)
    - Must utilize Cobalt Strike framework for command & control
    - Must evade Windows Defender
    - Must work on both Windows 10 and Windows 7 environments and
    - Must not use previously disclosed delivery mechanisms
- According to the main objective, the researcher develops an interesting test environment and variables which consist of both attacker and victim environment. Here are points I found interesting
    - There are anti-malware solutions installed on the victim's side. Windows Defender for Windows 10 and Symantec Antivirus for Windows 7. Even it's not EDR or next-gen AV solutions, there's still a bunch of work needed to do to evade the security protection.
    - The payload used in this research is known, signed (required by .NET framework 4.5), delivered via e-mail as the `.appref-ms` filetype.
- Microsoft ClickOnce with `.appref-ms` is the only application focused on this research. By the way, the researcher discovered other file extensions and executables that were still permitted for use as an OLE file (Object Linking & Embedding).
- Microsoft ClickOnce is an application deployment technology that allows users to create self-updating applications with minimal user interaction. When an application is published as "Online & Offline Availability", it can be installed by the user remotely from a file share with a `.appref-ms` file generated and available with the user's start menu.
- The core technology of an application deployed by Microsoft ClickOnce is C#. Because Microsoft ClickOnce will prompt the user with an application install prompt and `.appref-ms` file will need to be generated, there are changes that need to be made to C# code to hide the execution of the payload and its malicious activity, such as deletion of start menu items (`.appref-ms` file) and lure the user with false errors.
- Another possible scenario for payload delivery is to use `.appref-ms` file for OLE delivery via phishing. It's also possible to deliver `.application` file (the file which will run the installation via Microsoft's ClickOnce) directly to target via hyperlink or HTA. For HTA, *`Dfshim.dll` is the library that ClickOnce uses to manage application downloads and updates. Through a Visual Basic script, `rundll32.exe` can be called to invoke dfshim.dll and run the application*
- `.appref-ms` file could also be used to establish persistence on the compromised system by placing in startup folder as part of the deployment process.
- Due to an ability to check into the deployment server for a new version of installed application and run any updates without user interaction, the attacker can just deliver a non-malicious application in the initial installation phase to establishing a foothold, then update with a malicious one.
- Because the domain for application deployment and a C&C domain name is different. The attacker has given a backup channel for command and control and deploys the application if it's detected with an ability to push a malicious application with a new C&C.