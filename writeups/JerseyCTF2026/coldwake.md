# Cold Wake

**Category:** Forensics | **Difficulty:** Medium

We get 3 pictures. `Exiftool` of the first one showed me first part of the flag. I used also `stegseek Tape* -wl rockyou.txt` and got passwords two `Tape2.jpg` and `Tape3.jpg`. Then I extracted hidden files with `steghide extract -sf Tape*` and got two new files. One was the picture of part of the flag and another was a recording of another part of the flag. 

