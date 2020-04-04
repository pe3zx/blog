---
title: "Quick Note on Nanocore Tradecraft - A Double ZIP File"
date: 2019-11-12T13:39:00+07:00
author: "P. Boonyakarn"
description: "Trustwave SpiderLabs published a new technique used by Nanocore to bypass an email security gateway solution that inspects file attachment content. I found this technique simple but effective with the fact that there are many file parser implemented in a current security solution and many of it interprets content differently."
---

Trustwave SpiderLabs [published](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/double-loaded-zip-file-delivers-nanocore/) a new technique used by Nanocore to bypass an email security gateway solution that inspects file attachment content. I found this technique simple but effective with the fact that there are many file parser implemented in a current security solution and many of it interprets content differently.

The attachment hash is `9474e1517c98d4165300a49612888d16643efbf6`

As pointed out by Trustwave SpiderLabs, one of the factors that make it suspicious is that there are two "end of central directory (EOCD)" records on the file, where a valid ZIP file has only one record. The EOCD record responsible to indicate the end of the ZIP file, so in this scenario there are two endings.

![](https://1.bp.blogspot.com/--XIrj1H8_-I/XcpQNnJIu8I/AAAAAAAATZE/2k9L59Y3X_cxc7ZNCkWWKGKSMlNSsxtZwCLcBGAsYHQ/s1600/2019-11-12_13-24-22.png)

If we look into a valid ZIP file, we will find that there three or four main records, as noted in [Wikipedia](https://en.wikipedia.org/wiki/Zip_(file_format)#File_headers):

- `ZIPFILERECORD`
- `ZIPDIRENTR`
- `ZIPENDLOCATO`

For the ZIP file that has more than one compressed file inside, the file structure should be looked like this:

- `ZIPFILERECORD record[0]`
- `ZIPFILERECORD record[n]`
- `ZIPDIRENTRY dirEntry[0]`
- `ZIPDIRENTRY dirEntry[n]`
- `ZIPENDLOCATOR`

From the NanoCore sample, we can see that the attachment was built by concatenated two ZIP files together because there's no significant change made with offsets in each of `ZIPFILERECORD`. If we split the attachment into the two seperated ZIP file, it can be extracted without error.

To reproduce this technique, it can be done simply with system commands, for example with `copy /b`, `Get-Content`, or `cat`:

```ps
PS > Get-FileHash .\SHIPPING_MX00034900_PL_INV_pdf.zip -Algor SHA256 | Select Hash

Hash
----
E90B970C5E5DDF821D6F9F4D7D710D6DC01D59B517E8FB39DA726803DC52B5AD


PS > gc first.zip,second.zip -Enc Byte -Read 512 | sc new.zip -Enc Byte
PS > Get-FileHash .\new.zip -Algor SHA256 | Select Hash

Hash
----
E90B970C5E5DDF821D6F9F4D7D710D6DC01D59B517E8FB39DA726803DC52B5AD


PS >
```