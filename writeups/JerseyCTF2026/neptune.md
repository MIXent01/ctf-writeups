# Neptune Authority (Forensics)

**Category:** Forensics | **Difficulty:** Medium

## Challenge Overview
* **Name:** Neptune Authority
* **Category:** Forensics / Network
* **Description:** An encrypted shutdown authorization was transmitted over the network. We need to recover the materials to decrypt the TLS exchange and find the flag.
* **Flag Format:** `jctf{xxxxxxxx}`

## Phase 1: Initial Reconnaissance
Upon opening the provided `.pcap` file in Wireshark, I observed two distinct types of communication involving the host `10.20.0.10`:
1.  **Unencrypted HTTP** on port **8080** with host `10.20.0.99`.
2.  **Encrypted TLS v1.2** on port **8443** with host `10.20.0.50`.

The protocol hierarchy suggested that the "materials" mentioned in the description were likely hidden in the unencrypted port 8080 traffic.

## Phase 2: Finding the Credentials
I filtered for HTTP traffic using `http`. I noticed several requests to a `/status` endpoint. While most returned a `404 Not Found`, I inspected the packet details of the first few requests.

In **Packet 11**, a custom HTTP header was discovered:
`X-Orbit-Note: oldorbit`

This string, `oldorbit`, appeared to be a password or a key intended for further decryption.

## Phase 3: Recovering the Encrypted Key
Continuing the analysis of the HTTP objects (**File -> Export Objects -> HTTP**), I found two interesting files being transferred:
* `ods.crt.enc`
* `ods.key.enc`

Based on the `.enc` extension and the challenge description, these were likely an encrypted certificate and an RSA private key. Using the command line `file` utility on the exported `ods.key.enc`, it was identified as **OpenSSL encrypted data with salted password**.

I attempted to decrypt the key using the password found in the HTTP headers:
```bash
openssl aes-256-cbc -d -in ods.key.enc -out ods.key
```
When prompted for the password, I entered `oldorbit`. The command succeeded, producing a valid RSA private key beginning with `-----BEGIN PRIVATE KEY-----`.

## Phase 4: Decrypting the TLS Traffic
With the private key in hand, I returned to Wireshark to decrypt the traffic on port 8443.
1.  Navigate to **Edit -> Preferences -> Protocols -> TLS**.
2.  Click on **RSA Keys List -> Edit**.
3.  Add a new entry:
    * **IP Address:** `10.20.0.50`
    * **Port:** `8443`
    * **Protocol:** `http`
    * **Key File:** `ods.key`

## Phase 5: Flag Extraction
After applying the key, Wireshark was able to decrypt the "Application Data" packets. I looked for the "shutdown authorization" mentioned in the description.

In **Packet 1041**, the decrypted TLS layer revealed a plaintext HTTP response:
```http
HTTP/1.1 200 OK
Content-Type: text/plain

STATUS: ESCALATION_ACTIVE
COUNTDOWN: ACTIVE
SHUTDOWN_CODE: 48173926
```

The challenge description stated that the shutdown authorization (the code) was the key to stopping the lockdown. Following the flag format `jctf{xxxxxxxx}`, I wrapped the 8-digit shutdown code to get the final flag.

**Flag:** `jctf{48173926}`
