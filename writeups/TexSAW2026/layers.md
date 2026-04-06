# Layers

**Category:** Forensics

We got `layers.zip` file with some zip files (`layer1.zip`, `layer2.zip` and `layer3.zip`) in it and `__MACOSX` folder. First, I verified their file signatures using `file` command to ensure they were ZIP archives. While `layer1.zip` was accessible, the second and the third archives were password-protected.

#### Layer 1 

The first ZIP contained a `.dmg` file. I extracted its contents using `7z`. Inside, I found numerous files, but one named `clue.txt` stood out. It contained the password for `layer2.zip`, which worked perfectly.

#### Layer 2 

After extracting `layer2.zip`, I got `.vhdx` file. I extracted it with `7z`. While inspecting the files, I noticed an Alternate Data Streams (ADS) named `report.txt:secret.bin`. Reading this file with `cat` I got text looking like base64 encoded text. After decoding it, I got the password for `layer3.zip`.

#### Layer 3

The final ZIP contained an `.img` file formatted with an `ext4` filesystem (Checked with `file` command). I ran `strings` command on the image nad saw references to a `flag.txt` file. However, after extracting `.img` file, `flag.txt` file was not there. Only 1 file was interesting: `Journal`. `exiftool`, `xxd` and `strings` did not show something interesting. However `binwalk` found `decompressed.bin` file, which was identified as ASCII text. Finally, I found the flag with `cat` command.

