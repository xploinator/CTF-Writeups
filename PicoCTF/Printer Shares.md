# General Skills - Easy

# Description

An important file was accidentally sent to a network printer. The goal is to retrieve it from the print server.

---

## Writeup

Started by checking connectivity to the given service:

```
nc -vz mysterious-sea.picoctf.net52864
```

The port was open, confirming that the service was reachable.

---

The hints pointed towards SMB, so began enumerating available shares using `smbclient`:

```
smbclient-L mysterious-sea.picoctf.net-p52864-N
```

```
Output
Sharename       Type      Comment
---------       ----      -------
shares          Disk      Public Share With Guests
IPC$            IPC       IPC Service (Samba 4.19.5-Ubuntu)
```

This listed the available shares. Among them:

- `IPC$` (IPC service)
- `shares` (disk, public)

The `shares` directory looked relevant since it was publicly accessible.

---

Connected to the share:

```bash
smbclient //mysterious-sea.picoctf.net/shares-p 52864-N
```

After connecting, listed the files:

```
ls
```

Found:

```
dummy.txt
flag.txt
```

---

There were two ways to go about this. Either downloading the file on my machine or viewing it in the terminal itself

Retrieved the flag file:

```
get flag.txt
```

Then viewed its contents

Or

View the file in the terminal

```
more flag.txt
```

---

## Flag

```
picoCTF{5mb_pr1nter_5h4re5_9fc5e085}
```
