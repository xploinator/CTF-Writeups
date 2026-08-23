# Forensics - Easy

# Description

The provided file contains a sequence of binary digits. The goal is to determine whether it encodes any meaningful data.

---

## Writeup

Started by downloading the file `digits.bin`. The contents appeared to be a long sequence of `0`s and `1`s.

Instead of treating it as plain text, assumed that the binary data might represent actual bytes.

Used a Perl one-liner to convert the bit string into raw bytes:

```
perl -lpe '$_=pack"B*",$_' digits.bin > output.file
```

---

After conversion, checked the file type:

```
file output.file
```

Output indicated that the file was a JPEG image.

Opened the file, which revealed an image containing the flag.

---

## Flag

```
picoCTF{h1dd3n_1n_th3_b1n4ry_8d00e35f}
```
