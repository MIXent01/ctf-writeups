# Larping 101

**Category:** Forensics | **Points:** 612

We get a `.pptx` file and description "To larp, one must become the larper.. What do you think of my presentation? It feels like it might be missing something so maybe you can tell me what it is?"

I confirmed it was a `.pptx` file and looked at presentation and possible hidden text. I found nothing.

I extracted the files from this file with `7z e challenge.pptx -oout` and used `grep -R "CIT{" out/` on the extracted files. It got me a flag from `transitions.xml` file.
