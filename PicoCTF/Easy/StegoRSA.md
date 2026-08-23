# Cryptography/Steganography - Easy

# Description

A message has been encrypted using RSA. The public key is gone… but someone might have been careless with the private key. Can you recover it and decrypt the message?

---

## Writeup

Started by downloading the provided image. Since this is a steganography challenge, first checked the image for hidden data.

<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/4aa8a854-7046-4747-b55e-66fbf249eb15" />


---

Instead of using local tools, checked the metadata using an online tool. The metadata did not contain typical fields like author or filename, but the **comment field contained hex data**.

Copied the hex string and decoded it.

After decoding, it revealed a private key in the following format:

```
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCyi2qh7k1+l1Q7
xkVIM6mZY9swyl9433vHYRYMrBEvH+/DZws2stMKlFY9KgJbdZjx0qcgm2JQ6Tqp
Wip2IHFaRBnw3RYv8J5gAeLcXW0xrkqoQ+feWY3N7qHchF9jd/ADQuiT/DKFffCq
RQ8Rb9euTwJEYsx6CrqRMlEOPUlZiwPMsYCGX8Rt7tcg9fnCa29ZZoN0q33qaGQd
cUcds8yC+6Gwjd3un9pnyRFloUHNO2S6bRASWoJ80IT8Aw/UhGCYhpHvh4Qy8qED
6HvD5QORuOHVDMoQjdf3C5WXHgO4PtL1xuY6gKZSKltM5dhSSSRiD/2X/R3elAf9
hPDRs4NrAgMBAAECggEANBxs6wZap/ATLb8YyZIKljKG7x6h7u2Lew3jGZ+/BDoW
CLoyk6xt3FCfOwrf1UHlee85yFKRx3vLG1KtwfyGGQp3Z82fhC5+ixcB17+M90sf
jy0Cp+sLcGeN5obcMHP5IXqN12Nse3nenFO7qiMymDWHO956P8SR338IGVfZ6MAl
M3EWmbmBewGYsCn19K3EX9H4Sdb3DSHtydrO2/0JgGGDA+E98kIWTdLwhlPlyiEp
JvolP2OAEdF1UkVJzUR9db4GV4UNPcf3GUXJBcW9pHmvDnymrazets+CfzoGmZQg
ZON1GVpujVqsV1nvhGVhlrs47YDHuSA5Ve/p/rubgQKBgQDIU78/bl+RjmdIF9K9
aStSS2B1TalHL5lTb999RE8O5kqPqU1cJiMwhoG7dJMEcksIDpKAKlqTRA+q67oa
D3QUFd6ViFEgQYHr58P6WJqgsRlPai/zM014qgiZJdBJF5OzfiM87sdl92PklQc7
MArjKifr2B2Gbei6IdtjsSYgyQKBgQDkKfPwpd1b0eQEJQwG8JhS1dXEHgZgpUZo
NYm/sZ576CJpAKnx1jD1eKgKVaP5EnfCYlxhGNAVDXoyxfHb7qEEvqy/1eAjBZtB
Wkip80igoEjMqQUHHvGeUJGVWlScR3G1PfwuGl8Z64PYOvGv5J0JfOaumDw+HqfY
UmJaEuEwkwKBgQC7ntQL0I/pf3nz53wUsh9E4BvjQW09orLzll+2rvdsePt0OZie
qYljtVZj/vaCv5jOXveO2hwiuSgDaOvP5JFPDnx9iEKS90d7boH6Qmnv/m46FrX6
DR6N2JJc/TFqg45uGcFfHDPcqCsCtyEiqghIYf8pwCtG8EF7sqILaKrRIQKBgF3D
L9ARGWqGUqGxZ8PiU3aXEYXKoOxOfySL+9Oe9nYM6zcjYrNTRkNaFhRJJV1RzY1A
Rp5QSBKeuzzqQ34SDnGYuf0Ls1QxFaBBreLJa2s28zPHsZ0/hiN9EJbDzEl8wqms
k1mO1M4eDsxpTLDvzej8PwA452jPyEIJeQlzAL+pAoGBAKT76AQtOixOb23iF7gL
d9/bq/NCGbGYtyF0LB3GxLlRpdp5qIFmxwGy4hsdozntPLaaYgg+/yWLDdsZEP0G
sAGD7M6Ko7FdOLe1beSwp5Tv0tbZW5OcagyYSkHQTv2ihkvUszh4pEHmVu0CmvbI
Bt2UHx5Ojrvhm0U0QGU7c8F4
-----END PRIVATE KEY-----
```

---

Saved this key locally as a `.pem` file:

```
echo "-----BEGIN PRIVATE KEY-----
...
-----END PRIVATE KEY-----" > key.pem
```

---

With the private key available, used OpenSSL to decrypt the encrypted file:

```
openssl pkeyutl -decrypt -inkey key.pem -in flag.enc
```

This produced the decrypted output, which contained the flag.

---

## Flag

```
picoCTF{rs4_k3y_1n_1mg_a9a7c4c9}
```
