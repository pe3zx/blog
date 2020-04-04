---
title: "Quick Note on Phobos Ransomware"
date: 2019-04-11T20:22:00+07:00
author: "P. Boonyakarn"
description: "This is a quick note about Phobos ransomware analysis. If you're interesting, I left materials including the sample, required key file and x64dbg's comments on the repository here."
---

This is a quick note about Phobos ransomware analysis. If you're interesting, I left materials including the sample, required key file and x64dbg's comments on the repository [here](https://github.com/pe3zx/malware-analysis-phobos).

## A Separate 10-Bytes Key File

Retrieved USN journal on victim system shows that at almost the same time the malicious executable was dropped, there was a text file named `k.txt` dropped at the same location too. The text file was overwritten and deleted before the malicious process exiting.

This flow exists on program's entry point as shown below:

![](https://3.bp.blogspot.com/-vtVzUhzmpkU/XK8xvxauHvI/AAAAAAAASRw/ulO_AAB6UZQyHhvBjd_PV39wNwApIO78wCLcBGAs/s1600/2019-04-11_19-22-55.png)

If return value from `CreateFileA` with `OPEN_EXISTING` option is failed, the program will exit.

## Tiny Encryption Algorithm With Custom Packer

Looking closely to function at .text, we can see using of constants which relate to a key schedule constant of [Tiny Encryption Algorithm (TEA)](https://en.wikipedia.org/wiki/Tiny_Encryption_Algorithm) such as `0xC6EF3720` and `0x61C88647`. I didn't compare the whole section with the actual implementation of TEA, so there's a chance that this is TEA or modified version of TEA which utilizing the same constants.

![](https://4.bp.blogspot.com/-MgrglZ_RD9U/XK81NtB17uI/AAAAAAAASSA/qAQPfSDn3LEur75xMOZbcIeqfcFv-U5jACLcBGAs/s1600/2019-04-11_19-37-35.png)

The caller of the above function specifies an address of entry point, an address of `.data`, a length of data on the text file and its value as parameters, controlled by decrementing over `ESI` register which holds `0x2625A00`. This value embedded in `.data` segment at `0x2020`.

![](https://1.bp.blogspot.com/-L1OCdZ8Xboc/XK87pfMutHI/AAAAAAAASSQ/9av9AGJ7_jIozJI_U8Jc_wBpjY3mCtuMACLcBGAs/s1600/2019-04-11_20-05-00.png)

If we put a breakpoint after `jne` at `0x1437`, we could see changes on the `.text` segment:

![](https://1.bp.blogspot.com/-XByhTSgulQM/XK8-EEck8yI/AAAAAAAASSc/vrC4lq8Rgn0dXxOG_K6ossJ3W2egCl6cACLcBGAs/s1600/2019-04-11_20-08-40.png)

![](https://3.bp.blogspot.com/-1JorZ7v2SJs/XK8-X9yj9FI/AAAAAAAASSk/50ka21hZNsII4X9Sk-wgHdOWBXLBjjPEwCEwYBhgL/s1600/2019-04-11_20-15-09.png)

It seems like the 10-bytes is used with TEA to decrypt a garbage data and append to the `.text` section as a code. I didn't look further due to access violation exception.

## What Does It Look Like In Sandbox?

Not much. The only online sandbox provider that let me specify which submitted files to be executed is [ANY.RUN](https://app.any.run/). As you can see [here](https://app.any.run/tasks/c0903da9-e2b4-4a90-bdd4-3dbd09b017b6), the process was crashed, and because most of its operation doesn't rely on Windows API, the sandbox reported exact behavior information when compare to ProcMon.