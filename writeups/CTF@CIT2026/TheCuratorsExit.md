# The Curator's Exit

**Category:** Osint | **Points:** 934

We get a `.pdf` file protected with password and description: "We've received a communication from Interpol, containing a PDF. We are unsure of the contents, but believe it contains information on one of the thieves from the Louvre Heist. Your job is to find out as much as you can about this individual and send the information back to the authorities.". Firstly, I tried some simple passwords like date of the heist, but it didn't work. 

I decided to use `pdf2john` and `johntheripper` programs. Firstly: `pdf2john VF0000000011-Enc.pdf >> hash.txt` and then `john --wordlist=rockyou.txt hash.txt` and I found the password `cherell`.


