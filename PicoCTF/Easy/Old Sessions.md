# Web Exploitation - Easy

# Description

Proper session timeout controls are critical for securing user accounts. If sessions never expire, an attacker using the same browser can reuse an existing session without needing credentials.

---

## Writeup

Started by opening the challenge and navigating to the provided link (“browse here”).

<img width="794" height="434" alt="image" src="https://github.com/user-attachments/assets/07d06fb6-3d5b-4fb3-8087-048b24ff6772" />


This redirected to a login page. Before interacting with it, checked the page source and inspected different tabs (Elements, Sources, Network), but nothing useful showed up.

The hints mentioned using the web inspector and checking cookies, so focused on that.

Tried logging in with random credentials, which redirected to a registration page. Created a new account and then logged in successfully.

<img width="758" height="711" alt="image" src="https://github.com/user-attachments/assets/a24de63c-cdd2-4f96-a15f-6d647bde1e25" />


After logging in, the site displayed some posts. One of them mentioned a strange page at:

```
/sessions
```

Before visiting it, opened the browser inspector and checked the **Application** tab.

Looked through:

- Local Storage → nothing useful
- Session Storage → nothing
- IndexedDB → nothing

Then checked **Cookies**, where a key named `session` was present along with a value. This appeared to be the session identifier for the logged-in user.

<img width="1642" height="383" alt="image" src="https://github.com/user-attachments/assets/31c98d24-a9fb-4c94-a944-9f9d04801755" />


Next, visited the hinted page `/sessions`:

This page listed multiple sessions, including one for the **admin** user. It showed:

```
1) session:x2ubW6kTrtsStj__qi2CGe3SNXL8XHjUoGzfd0KvKYo, {'_permanent': True, 'key': 'admin'}

2) session:t7GtrUYfDkFfc245qSTFfFnh9TyJ-geU1YpaemwC-DM, {'_permanent': True, 'key': 'admin123'}
```

The key observation was that the admin’s session value was visible.

Copied the admin session value and went back to the Application tab → Cookies. Replaced the current session value with the admin’s session value.

After refreshing the page, the session was now recognized as admin, and the flag was revealed.

---

## Flag

```
picoCTF{s3t_s3ss10n_3xp1rat10n5_51c526ab}
```
