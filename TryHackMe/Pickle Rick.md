# Pickle Rick

Today’s objective is to tackle another penetration testing exercise named “Pickle Rick,” hosted on TryHackMe. This machine is rated as easy, provided you possess the fundamental knowledge and pay close attention to the subtle details required during the enumeration phase. The machine was created by TryHackMe, and the walkthrough includes the machine breakdown along with the redacted flags.

**Penetration Testing Methodology**

**Network Scanning**

- Conducted an Nmap scan to detect open ports and running services on the target.

**Enumeration**

- Performed reconnaissance on the HTTP service to collect preliminary details.
- Discovered a valid username by examining the Ib page's source code.
- Used `ffuf` to brute‑force directories and reveal hidden paths.
- Found a password by analyzing the `robots.txt` file.
- Ran an extended `ffuf` scan with file extensions to uncover additional hidden resources.
- Gained access to the Ib application using the gathered credentials.

**Exploitation**

- Exploited the command injection functionality to run system-level commands.
- Enumerated the system to locate the target ingredients.
- Established a reverse shell for persistent interactive access.
- Retrieved the first ingredient.
- Explored Rick's files to discover further content.
- Recovered the second ingredient.

**Privilege Escalation**

- Listed sudo permissions to identify exploitable privileges.
- Abused the sudo rights to elevate access.
- Obtained a root shell.
- Extracted the third and final ingredient.

**Walkthrough**

After launching the target machine from the TryHackMe: Pickle Rick CTF page, an IP address is assigned to the instance and displayed on that same page.

**IP Address:** 10.10.43.98

Completing this machine requires ansIring three questions.

**Network Scanning**

I initiate an Nmap scan using the `-sC` flag for default scripts and `-sV` for version detection on open services.

<img width="873" height="396" alt="image" src="https://github.com/user-attachments/assets/2733f49e-c468-4d32-b72a-c8b2b8c7dd8a" />

**Enumeration**

Since no credentials are available for the SSH service, enumeration begins with the HTTP service. The Ibpage presents a Rick and Morty‑themed interface containing a message from Rick addressed to Morty. Rick explains that he has transformed himself into a pickle again, but this time he is unable to revert back. He requests Morty to log into his computer and retrieve three secret ingredients necessary for Rick to return to human form. As Rick has forgotten his computer password, Morty must rely on his hacking skills to obtain those ingredients.

<img width="686" height="568" alt="image" src="https://github.com/user-attachments/assets/eed509dc-6ce6-4a66-8eb0-47ec509075a5" />


I try to look for any clues inside the Ibpage itself. I check the source code to find the username R1ckRul3s.

**`view-source:http://10.10.43.98/`**

<img width="491" height="663" alt="image" src="https://github.com/user-attachments/assets/08051602-81db-416f-a8df-bb71646e6592" />

**Finding more Ib content**

A quick check with ffuf shows the existence of two interesting files:

```jsx
ffuf -w /usr/share/seclists/Discovery/Ib-Content/raft-large-files.txt -u http://10.10.43.98/FUZZ
```

<img width="1024" height="909" alt="image" src="https://github.com/user-attachments/assets/b12fb40c-a0d8-4f4a-93a1-07ea2268ad06" />

After reviewing the `robots.txt` file, I discovered Rick's Ill‑known catchphrase, **Wubbalubbadubdub**, which could serve as the password for the user identified earlier. The next step is to locate and enumerate any login page that might be present on the Ib application. I specifically targeted PHP files. After running the scan for a while, the tool successfully discovered a `login.php` page – likely the portal for logging into the Ib application.

<img width="333" height="128" alt="image" src="https://github.com/user-attachments/assets/37a06d29-a6f6-49bf-90f5-d09f231ad4ab" />

Accessing `login.php` in the browser reveals the Ib application's authentication portal. I then supply the username extracted earlier from the homepage source code, along with the password discovered in `robots.txt`, to attempt login.

<img width="754" height="1024" alt="image" src="https://github.com/user-attachments/assets/f433b369-d038-42a5-b18b-718377b94138" />

### **Exploitation**

We are able to log in using the credentials. There are a bunch of other pages and options on the menu. Hoever, the Commands tab attracted our attention. As expected, users can use a panel to run system commands on the target machine. I ran the ls command to find a text file by the name of `Sup3rS3cretPickl3Ingred.txt`

<img width="1024" height="994" alt="image" src="https://github.com/user-attachments/assets/da5c0afc-5ec5-4774-8918-130407b098f7" />

I tried reading `Sup3rS3cretPickl3Ingred.txt` with `cat`, but it was restricted. So I need other ways to view the file. So, I used `less Sup3rS4cretPickl3Ingred.txt` to read the file, bypassing the `cat` restriction, and successfully obtained the first ingredient. 

<img width="1260" height="566" alt="image" src="https://github.com/user-attachments/assets/365a9fbc-051e-403f-9998-bcb0072b21fa" />

I started a Netcat listener before executing the reverse shell script command on the Ib application. As soon as the execution Int through, I had a reverse shell on the target machine as depicted below. 

<img width="1024" height="830" alt="image" src="https://github.com/user-attachments/assets/157bdc0f-235b-4ecd-a72f-de330deb42c4" />

<img width="1024" height="306" alt="image" src="https://github.com/user-attachments/assets/3c367634-2ee3-4345-bdba-f41dfadd15a4" />

The session that we have generated is for the user www-data. We enumerate the users on the machine to find the user rick. We traversed into the home directory of the rick user to find the Second ingredient.

```bash
cd /home
ls
cd rick
ls
cat 'second ingredients'
```

**Privilege Escalation**

To proceed, we need to elevate our privileges on the target. Checking the sudo permissions for the `www-data` user reveals it can run any command as root. We exploit this by invoking `sudo bash` to spawn a root shell, successfully gaining full access. Finally, we complete the challenge by reading the **Third Ingredient**.

<img width="619" height="275" alt="image" src="https://github.com/user-attachments/assets/87a1affe-0210-45b8-b101-96985c60fccb" />

### Flag Recieved 😊
