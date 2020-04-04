---
title: "My Forked Version of msfrpc with Python3 Support"
date: 2018-12-07T16:11:00+07:00
author: "P. Boonyakarn"
description: "The Metasploit's msgrpc module utilized MessagePack, a binary serialized version of JSON, as a format and only requirement to use this feature is to load msgrpc module..."
---

The Metasploit's msgrpc module utilized MessagePack, a binary serialized version of JSON, as a format and only requirement to use this feature is to load msgrpc module:

```sh
$ msfconsole -x "load msgrpc ServerHost=127.0.0.1 ServerPort=55552 User=msf Pass='abc123'"
```
Trustwave's SpiderLabs built "msfrpc" to handle RPC authentication and message authorization as a module for Ruby and Python, but isn't getting any update since 2012.

The is a [forked version](https://github.com/pe3zx/msfrpc) intended to support Python 3.x with fixes and updates on calling logic. You can also find examples code in main function. Please let me know if you are in trouble with this project.

Full API documentation available [here](https://metasploit.help.rapid7.com/docs/rpc-api).