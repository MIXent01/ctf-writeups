# Timeline 1

**Category:** Forensics | **Difficulty:** Medium

We get `partition4.img` file, which is partition (confirmed with `file` command). The name of challenge is `Timeline `, so I created `Sleuthkit MAC` timeline.

Firstly I used `fls -r -m / partition4.img > body.txt` and then `mactime -b body.txt -d > timeline.csv`. Next, I used `grep` with some `keywords`: `grep -i "flag\|pico\|ctf\|secret\|hidden" mac`. I found nothing, so I decided to open up it with some csv reader. I looked for `macb` files. We can see that `.ash_history` is created, which is wrong. Also we can see `etc/chat` file which is strange. I used `icat partition4.img 32716` to see it and it had something which looked like the flag. I wrapped it into picoCTF{} and it was wrong. So I decided to decrypt it with base64 and it was the flag.
