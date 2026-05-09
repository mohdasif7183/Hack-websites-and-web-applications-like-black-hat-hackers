 Introduction to SQL Injection

In this lecture and the next few lectures, we will learn about a very common web vulnerability called **SQL Injection (SQLi)**.

Before learning how SQL Injection works or how to exploit it, we first need to understand what **SQL** and **databases** are.

---

 What is a Database?

Most modern websites use a **database** to store information.
Only very small or simple websites may not use one.

Databases are used to store:

* Usernames
* Passwords
* Blog posts
* News articles
* Images
* Comments
* Credit card details
* And almost all website data

The website communicates with the database to:

* Read data
* Insert new data
* Update existing data
* Delete data

---

 What is SQL?

SQL stands for **Structured Query Language**.

It is the language used by web applications to communicate with databases.

Using SQL, applications can:

* Retrieve information
* Add new records
* Modify existing records
* Delete records

---

 Understanding Databases in Practice

In this example, we log into the database installed on the **Metasploitable 2** machine.

This is **not hacking**.
We are only exploring how databases work.

We log into MySQL:

```bash id="iq1jlwm"
mysql -u root
```

* `-u root` → login as root user
* Metasploitable has no root password (intentionally vulnerable)

---

 Viewing Databases

To see all available databases:

```sql id="suxb3e"
show databases;
```

Example databases:

* `information_schema`
* `mysql`
* `dvwa`
* `owasp10`
* `metasploit`

Each web application usually has its **own database**.

For example:

* DVWA uses the `dvwa` database
* OWASP WebGoat uses another database


---

 Selecting a Database

To use a specific database:

```sql id="vqv55v"
use owasp10;
```

---

 Viewing Tables

Each database contains **tables**.

Tables store related information.

Example:

* `accounts`
* `blogs`
* `credit_cards`

To display tables:

```sql id="ng13fm"
show tables;
```

---


 Viewing Data Inside a Table

To view all data from the `accounts` table:

```sql id="e9dnwy"
select * from accounts;
```

* `select` → retrieve data
* `*` → select everything
* `from accounts` → from the accounts table

This may display:

* Usernames
* Passwords
* User IDs
* Admin status

---



 Example Output

| ID | Username | Password     | Admin |
| -- | -------- | ------------ | ----- |
| 1  | admin    | adminpass    | yes   |
| 2  | adrian   | somepassword | no    |

---

 Important Concepts

 Database

Stores website information

 Table

Stores specific categories of data

 Columns

Fields inside a table
Example:

* username
* password
* email

 SQL Queries

Commands used to interact with databases

---

 Goal of SQL Injection

Normally, only website administrators can access the database directly.

In upcoming labs, we will learn how attackers exploit SQL Injection vulnerabilities to:

* Access databases
* Read sensitive data
* Modify records
* Delete information
* Gain full database control


Why SQL Injection is Dangerous

SQL Injection (SQLi) is one of the most dangerous web vulnerabilities because it gives attackers direct access to the website’s database. Since databases store important information like usernames, passwords, emails, credit cards, and private user data, a single SQL Injection vulnerability can expose the entire application.

Many popular websites have suffered from SQL Injection vulnerabilities because they are easy to create accidentally and difficult to secure properly. In many cases, attackers do not even need a reverse shell or PHP upload because accessing the database already gives them everything they need.

An attacker can use SQL Injection to read sensitive information, modify data, create admin accounts, delete records, or sometimes even upload malicious files and gain full control over the server. This is why SQL Injection is considered one of the most critical vulnerabilities in web security.

