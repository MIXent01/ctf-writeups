# qaStaH nuq

**Category:** Forensics | **Difficulty:** Easy

We have to repair three corrupted files (`file1`, `file2`, `file3`). Files are identified as `data` by `file` command.

Based on challenge description `file1` was a PNG file. I changed magic bytes `python3 -c "with open('file1', 'r+b') as f: f.write(b'\x89\x50\x4e\x47\x0d\x0a\x1a\x0a')"`. 

We got first part of the flag and clue that `file2` was a JPEG format file. I changed the header with `python3 -c "with open('file2', 'r+b') as f: f.write(b'\xff\xd8\xff\xe0\x00\x10\x4a\x46\x49\x46')"` and I got another part of the flag and text "FILE TYPE BOX".

Initial analysis using `strings file3` revealed metadata from the x264 koder: `x264 - core 164 - H.264/MPEG-4 AVC codec...`

Examination of the first few bytes of file3 showed:
`00 00 00 20 70 66 79 74 69 73 6f 6d`

- 70 66 79 74 translates to ASCII pfyt.

- The standard MP4 "File Type Box" marker is ftyp (Hex: 66 74 79 70).

- The bytes were scrambled/anagrammed to prevent file recognition.

I repaired the file with `python3 -c "with open('file3', 'r+b') as f: f.write(b'\x00\x00\x00\x20\x66\x74\x79\x70')"` and got mp4 file with last part of the flag.
