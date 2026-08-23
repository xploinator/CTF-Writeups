# Reverse Engineering - Medium

# Description

I made a new password authentication program that even shows you the password you entered saved in the database! Isn't that cool? 

---

## Writeup

I connected to the service using netcat:

```bash
nc candy-mountain.picoctf.net 58912
```

The program asked to:

- Set a password
- Enter the length of the password
- Provide a hash to log in

After entering a sample password (`hello`) and its length, the program returned some data. However, didn’t get any output after entering random numbers as hash.

Then i entered nothing when asked for the hash and got this output : 

```
system.out: heartbleed.c:69: main: Assertion `1 == 0' failed.
Aborted (core dumped)
```

Got the same output when entered alphabets as hash.

---

Next, inspected the provided binary `system.out` :

```
strings system.out
```

This revealed references to `heartbleed.c`, which suggested that the challenge might be based on a heartbleed style vulnerability.

---

The idea behind this vulnerability is that the program trusts the user-provided length. If a larger length is given than the actual input, the program may leak extra memory.

Tested this by:

- Providing no password
- Entering a large length value

This caused the program to return additional data from memory.

```
Please set a password for your account:

How many bytes in length is your password?
1000
You entered: 1000
Your successfully stored password:
10 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 105 85 98 104 56 49 33 106 42 104 110 33
-86 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
Enter your hash to access your account! :
    
system.out: heartbleed.c:69: main: Assertion `1 == 0' failed.
Aborted (core dumped)

```

---

The program leaked the following sequence of integers:

```
105 85 98 104 56 49 33 106 42 104 110 33 -86
```

These values represent **raw bytes from memory**, printed as signed integers. To recover the original string, we convert each byte into its corresponding ASCII character. I also took help from when i entered “hello” and got those raw bytes. It helped me to map letters to ASCII characters.

---

### Step 1: Convert integers to ASCII

Each number corresponds to an ASCII value:

| Decimal | Hex | ASCII |
| --- | --- | --- |
| 105 | 0x69 | i |
| 85 | 0x55 | U |
| 98 | 0x62 | b |
| 104 | 0x68 | h |
| 56 | 0x38 | 8 |
| 49 | 0x31 | 1 |
| 33 | 0x21 | ! |
| 106 | 0x6A | j |
| 42 | 0x2A | * |
| 104 | 0x68 | h |
| 110 | 0x6E | n |
| 33 | 0x21 | ! |
| -86 | 0x170 | 0xAA (adjacent storage value) |

Ignoring -86 as it was the adjacent storage value and it doesnt add any meaning to the string, this gives:

```
iUbh81!j*hn!
```

Now, with respect to the hint “How does the hashing algorithm work?” and this being a reverse engineering room, we will look into the source code by decompiling and analysing it using GHIDRA

To better understand how the hash was generated, I opened the binary in Ghidra and analyzed the functions. Key functions:

- `main`
- `make_secret`
- `hash`

The hashing logic was found in the `hash` function.

```c
long hash(byte *param_1)

{
  byte *local_20;
  long local_10;
  
  local_10 = 0x1505;
  local_20 = param_1;
  while( true ) {
    if (*local_20 == 0) break;
    local_10 = (long)(int)(uint)*local_20 + local_10 * 0x21;
    local_20 = local_20 + 1;
  }
  return local_10;
}
```

`make_secret` returns the hash value while placing a secret in v13.

```c
void make_secret(long param_1)

{
  long local_10;
  
  for (local_10 = 0; obf_bytes[local_10] != '\0'; local_10 = local_10 + 1) {
    *(byte *)(local_10 + param_1) = obf_bytes[local_10] ^ 0xaa;
  }
  *(undefined1 *)(param_1 + 0xc) = 0;
  hash(param_1);
  return;
}
```

I then recreated this logic in Python to compute hashes locally.

---

I passed the leaked string through the recreated hash function to generate the correct hash.

```
hash = 15237662580160011234
```

Then reconnected to the service and:

- Skipped password input
- Skipped length input
- Provided the computed hash

It’s not necessary to skip password and length but I found it convenient.

The program accepted the hash and returned the flag.

---

## Flag

```
picoCTF{d0nt_trust_us3rs}
```
