---
title: "A Missing of Acrobat API JavaScript"
date: 2019-10-11T16:46:00+07:00
author: "P. Boonyakarn"
description: "One way to execute JavaScript with a PDF file is to rely on Acrobat API which already has a subset of useful API for red team engagement or adversary simulation when your target primarily uses Adobe products as a default PDF reader."
---

One way to execute JavaScript with a PDF file is to rely on [Acrobat API](https://www.adobe.com/content/dam/acom/en/devnet/acrobat/pdfs/js_api_reference.pdf) which already has a subset of useful API for red team engagement or adversary simulation when your target primarily uses Adobe products as a default PDF reader.

During the past engagement, the PDF files were sent to the targets with embedded JavaScript that will ask for permission to open a *check-in* URL. This process could simply be done with `app.launchURL()` function. It also let me update URLs during the preparation phase for mass phishing because it exposes the original URL in plaintext. We can identify an existing JavaScript embedded inside a PDF file with PDF analysis tools e.g. `PDFiD` and `pdf-parser`.

I decided to use this feature again for this year's engagement plus a trick to retrieve NTLM hash from the targets. With a trial version of Adobe Acrobat, I created the same JavaScript function and another object to make an external SMB request to the C&C. By the way, I failed to identify the embedded with the same tools. Both PDFiD and pdf-parser identify the file with and without JavaScript as no JavaScript.

So, if there's no plaintext or JS tag, the only way to hide this object is to compress it as a data stream and decompress when use. The compressed data stream will look like the image below.

![](https://1.bp.blogspot.com/-e45siGY5DWk/XaBJ_xtdD4I/AAAAAAAATTQ/KZz4pw61EDQkQNFMYo5slbh5M_UuV86sgCLcBGAsYHQ/s1600/2019-10-11_16-19-59.png)

`pdf-parser` has a feature to extract and decompress the specified object. All I need to do is to randomly find what's the exact object that has compressed JavaScript code and then run:

```
pdf-parser.py -o <object-id> -f file.pdf
```

The result will look like this:

![](https://1.bp.blogspot.com/-gyTKXCcECtc/XaBPZNsmtSI/AAAAAAAATTc/ukPNEuH_AaEXRcHOk9IpkUOdXSAoB9trQCLcBGAsYHQ/s1600/2019-10-11_16-44-48.png)


I still don't know what makes the JavaScript be compressed but it possibly happens because of features like PDF size reducing and optimization.

![](https://1.bp.blogspot.com/-GVpP_YCTg3w/XaFsL_ZAX-I/AAAAAAAATTo/vSpxRco89usFp0wYYGrybU1X__AHocB7gCLcBGAsYHQ/s1600/2019-10-12_12-59-40.png)