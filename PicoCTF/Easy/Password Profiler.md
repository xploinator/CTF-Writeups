# General Skills - Easy

# Description

We intercepted a suspicious file from a system, but instead of the password itself, it only contains its SHA-1 hash. Using OSINT techniques, you are provided with personal details about the target. Your task is to leverage this information to generate a custom password list and recover the original password by matching its hash.Download the following files:

userinfo: Contains the personal details.

hash: Contains the SHA-1 hash of the password.

check_password: Script to test passwords against the hash.

---

## Writeup

Started by examining the provided files.  

The `user info` file contained personal details:

```
First Name: Alice
Surname: Johnson
Nickname: AJ
Birthdate: 15-07-1990
Partner's Name: Bob
Child's Name: Charlie
```

The hash file contained a SHA-1 hash:

```
968c2349040273dd57dc4be7e238c5ac200ceac5
```

Tried checking the hash using online SHA-1 lookup tools, but no match was found. This suggested that the password was likely custom and derived from the provided personal information.

---

Next, reviewed the provided script:

```python
import hashlib

HASH_FILE = "hash.txt"
WORDLIST_FILE = "alice.txt" # wordlist that was generated using CUPP

def load_hash():
    with open(HASH_FILE, "r") as f:
        return f.read().strip()

def crack_password(target_hash):
    with open(WORDLIST_FILE, "r", encoding="utf-8", errors="ignore") as f:
        for password in f:
            password = password.strip()
            if hashlib.sha1(password.encode()).hexdigest() == target_hash:
                return password
    return None

if __name__ == "__main__":
    target_hash = load_hash()
    result = crack_password(target_hash)
    if result:
        print(f"Password found: picoCTF{{{result}}}")
    else:
        print("No match found.")

```

From this, it was clear that:

- A wordlist is required
- Each word is hashed using SHA-1
- The correct password is the one matching the given hash

---

Since no wordlist was provided, generated one using **CUPP (Common User Passwords Profiler)**.

Ran CUPP in interactive mode:

```powershell
python cupp.py -i
```

Entered the target’s details (name, nickname, birthdate, etc.), which generated a custom wordlist based on the provided information.

---

After generating the wordlist, updated the script to point to the correct files:

```python
WORDLIST_FILE="alice.txt"
HASH_FILE="hash.txt"
```

Then executed the script:

```powershell
python3 check_password.py
```

The script iterated through the generated wordlist and found a matching hash.

---

## Flag

```
picoCTF{Aj_15901990}
```
