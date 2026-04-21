# Cool Car

**Category:** Steganography | **Points:** 925

We get a `.png` file. I found nothing interesting in `file`, `strings`, `grep` and `zsteg`. Only interesting maybe was this part of `exiftool` output:

```
X-random-00                     : mhfTqX6RDOY7VexDEarS1ydOVgDFVHezSsrrWfdDpuYBjkr/Kq664fCDwpozGrGYXUC70POmQ4eyERqD1tFVmdEsti5bMwzR77ckCFbE
X-random-01                     : iK6dPGHKaBUcCu6UbHB0K04cVgJOTzopefEy047ZHkUeJyIp4HD+VMYUn2/R8XF/AycsNC+dLPIJaDKS+SuTXqtp4FC0n1zTJBuTPCrjAgT197zF+vOx
X-random-02                     : 0ryC80KZTeyYq1H7Qtq+S8Hf8oUiUkXY+lfANmdHyY2HKqvp0KVwEr/I8UDk66NjdS2QWPIiWVYW+cw4q+8/GKzbkL/15yZekA1CrT
X-random-03                     : AVUxKYR8qHNsmIrbzk4tsDOBDhVDoaAe2YRtnv553h/0SnsnEBPEKWY9CdWcK0Jy0FUPkF/EKfn2rLLR/bJnULcACo54ONM
X-random-04                     : d7tQ1cCTD4up09I/TzlcxYyZqm56KixRv/phHwihan0mww53SmAFHyE
X-random-05                     : abWWi8YgCnMLX4pwK2Mw6ASvLj300KPBiM5VdSd3ww0/wQnGM
```

I decided to left it for later. I decided to check Bit Planes and it showed something. When I checked Planes 0 I saw many strange texts, but only on `alpha plane 0` there was strange text on the middle: `Q0lUezRWdTF1MXpofQ==`. It looked like base64 crypted text, so I decrypted it and got the flag.
