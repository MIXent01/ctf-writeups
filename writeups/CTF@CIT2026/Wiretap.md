# wiretap 

**Category:** Forensics | **Points:** 875

## Description

Ton, they're listening to the phones meet me down at the badabing.

## Solve

We get a `.wav` file. `Exiftool` didn't show anything interesting and `file` confirmed it was  a `.wav` file. Description of this ctf was "Ton, they're listening to the phones meet me down at the badabing."

It sounded like a phone number being dialed and then there was a strange sound, like from old dial-up modem or fax machine.

Firstly, I decoded the phone number (DTMF) and it gave me `11.44.111.55.555.55.555.11.999.99.888`. I tried to wrap as a flag, but it was wrong.

Then, I analyzed the modem signal with `minimodem --read 300 -f beep_beep_boop.wav`. It got me:

```
GET /SiliconValley/Heights/4721/diary.html HTTP/1.0
Host: www.geocities.com
User-Agent: Mozilla/4.0 (compatible; MSIE 4.01; Windows 95)
```

This is new CTF and GeoCities is an old URL, however I tried to find `diary.html` on the Wayback Machine, but it lead to a dead end.

I decided to force `minimodem` to listen to the answer channel of the phone line with `minimodem --read 300 -M 2225 -S 2025 -f beep_beep_boop.wav` it was right. I got html site and opened it. There was a flag.
