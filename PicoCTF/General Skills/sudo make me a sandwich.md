# General Skills

## Description

Can you read the flag? I think you can.

Flag Format - picoCTF{}

---

## Writeup

Started by connecting to the instance using the provided SSH credentials.

```
ssh -p 58672 ctf-player@green-hill.picoctf.net
```

After logging in, listed the files in the current directory:

```
ls
```

Found a `flag.txt` file. Tried to read it directly:

```
cat flag.txt
```

Got a permission denied error, so direct access wasn’t allowed.

Since the hint mentioned `sudo`, checked what commands were permitted:

```
sudo-l
```

The output showed that the user could run `/bin/emacs` as root without a password.

Knowing that Emacs can be used to open files with elevated privileges, used it to access the flag:

```
sudo emacs flag.txt
```

This opened the file with root permissions, allowing the contents to be read.

---

## Flag

```
picoCTF{ju57_5ud0_17_d8e1a280}
```
