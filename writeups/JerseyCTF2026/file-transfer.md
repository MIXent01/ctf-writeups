# file-transfer
**Category:** Forensics | **Difficulty:** Hard

### Challenge Description
> Our network security solution has alerted us to some suspicious traffic from a user's workstation. Can you help us figure out what is going on? 
> This is the 3rd time this month something happened with this user, we really need to improve our password policies...

---

## 1. Initial PCAP Analysis
Opening the provided `.pcap` file in Wireshark reveals two main streams of interesting traffic:
1. **Suspicious TCP Traffic (Port 55544):** Packets 1-18 show communication between `10.1.2.210` and `10.1.2.211`. Small payloads are transmitted using `PSH` flags. This looks like a Command and Control (C2) or exfiltration channel.
2. **SMB Traffic (Port 445):** Starting from packet 19, there is traffic between the workstation (`10.1.2.210`) and a file server (`10.1.2.200`). We observe an NTLM authentication attempt for the user `IT640\operator1`, followed by completely encrypted SMB3 traffic.

To proceed, we need to decrypt the SMB3 traffic. The description hints that the user has a weak password ("improve our password policies...").

## 2. Extracting and Cracking the NTLMv2 Hash
We need to extract the NTLM challenge and response to format a hash for cracking.

* **Packet 27 (NTLMSSP_CHALLENGE):** We extract the **Server Challenge**: `a83c46425815db34`
* **Packet 28 (NTLMSSP_AUTH):** We extract the user credentials:
  * Username: `operator1`
  * Domain: `IT640`
  * NTProofStr: `e705d3efd451d9eff0b3005233f7b573` (First 16 bytes of the NTLM Response)
  * Blob: The remaining hex string of the NTLM response.

We format this into a string that `John the Ripper` or `Hashcat` can understand:
`Username::Domain:ServerChallenge:NTProofStr:Blob`

**Our final hash (`hash.txt`):**
```text
operator1::IT640:a83c46425815db34:e705d3efd451d9eff0b3005233f7b573:01010000000000001366cc6ceecddc012eec3d0097740ebd0000000002000a0049005400360034003000010012005700530032003000320035002d00530031000400260063006f00720070002e00690074003600340030002e0069006e007400650072006e0061006c0003003a005700530032003000320035002d00530031002e0063006f00720070002e00690074003600340030002e0069006e007400650072006e0061006c000500260063006f00720070002e00690074003600340030002e0069006e007400650072006e0061006c00070008001366cc6ceecddc0106000400020000000800500050000000000000000000000000200000324ad49cd8c06cd57b3e895fd7e1f8b498a54f8a43816223dfab71e7013c6ce0efb9d8a1f651758f475c106843bb63d6622ba533b447ac1d4cb80299edea1f940a0010000000000000000000000000000000000009001e0063006900660073002f00310030002e0031002e0032002e003200300030000000000000000000
```

We crack it using the `rockyou.txt` wordlist:
```bash
john --format=netntlmv2 hash.txt --wordlist=rockyou.txt
```
**Result:** The password is **`password`**.

## 3. Decrypting SMB3 and Extracting Malware
With the password recovered, we configure Wireshark to decrypt the traffic:
1. Navigate to **Edit -> Preferences -> Protocols -> NTLMSSP**.
2. Enter the **NTLM Password**: `password`.
3. Ensure **SMB2** decryption options are enabled.

The "Encrypted SMB3" packets instantly turn into readable Read/Write requests. We can observe the user downloading an executable.
We extract the file by navigating to **File -> Export Objects -> SMB** and saving **`DaVinci.exe`**.

## 4. Reverse Engineering the Malware
Instead of fully decompiling the executable, we start with basic static analysis by running `strings`:
```bash
strings DaVinci.exe
```
Among standard Windows API calls and socket error messages, we spot several interesting hardcoded strings:
* `10.1.2.211` (The C2 server IP)
* `55544` (The C2 server port)
* `CMD-SEQ-A`, `CMD-SEQ-B`, `CMD-SEQ-C` (Command sequences)
* **`sorry_im_not_the_flag_:)`**

This 24-character string looks highly suspicious and is likely a multi-byte XOR key used to encrypt the C2 traffic we saw at the very beginning of the PCAP.

## 5. Decrypting the C2 Traffic & Finding the Flag
We go back to the initial TCP stream on port 55544. Right-clicking one of the packets, we select **Follow -> TCP Stream** and view the data as **Raw/Hex**.

The extracted raw payload looks like this:
```hex
434d442d5345512d4104001b13...
434d442d5345512d4204010a15...
434d442d5345512d4304160c1a1d59120c1e2c4e181d2b1c481137034c03022c4e05530b1b17593300063a4e1b1c3a541a002c124c5f597f79132f3a01170b2c353d2a0c031d3c282c002c0d180e17034a5e1d0b5c06012b494b7f0b0c1c305402062b0017250637031847151c3a28360e2430023c5935431013135334083e305564473a1117047f57537f2d55280a070d172c3a3c140533534a2f2b1701122b061d031e181a3b5a1c1d2b040404
```

Notice the pattern: `CMD-SEQ-C` followed by the byte `04` (End of Transmission), followed by the encrypted payload. 

We isolate the encrypted payload from `CMD-SEQ-C` and input it into **CyberChef**.
1. Input: Hex string (`160c1a1d59...`)
2. Recipe: **XOR**
3. Key: `sorry_im_not_the_flag_:)` (Type: UTF8)

The output reveals a console command echoing the flag:
`echo jctf{Dah914znHQigIolS-j7xvL5XiYooM4Uce}`

---

## Flag
**`jctf{Dah914znHQigIolS-j7xvL5XiYooM4Uce}`**
