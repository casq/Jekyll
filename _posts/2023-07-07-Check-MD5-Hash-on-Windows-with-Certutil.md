---
layout: post
title:  "Check the MD5 hash of a file on Windows with certutil"
date:   2023-07-07 11:00:00 +0100
categories: Utilities
---

To calculate the MD5 hash of a file on Windows, to check an exe for example, you can use certutil:

    certutil -hashfile \Path\to\file.exe MD5


Result:

    C:\Users\andy>certutil -hashfile c:\Users\andy.davies\Downloads\R-4.3.1-win.exe MD5
    MD5 hash of c:\Users\andy\Downloads\R-4.3.1-win.exe:  
    cbb6da78aef13dca33afd6a2e0ce7cdb  
    CertUtil: -hashfile command completed successfully.  
