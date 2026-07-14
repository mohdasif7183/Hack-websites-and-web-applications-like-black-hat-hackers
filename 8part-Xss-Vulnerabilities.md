Cross-Site Scripting (XSS)

In this section, we begin exploring **Cross-Site Scripting (XSS)**, one of the most common web application vulnerabilities. Unlike SQL Injection, which targets the database or server, **XSS targets the users of the application** by injecting malicious JavaScript into web pages.

 What is Cross-Site Scripting (XSS)?

Cross-Site Scripting (XSS) is a vulnerability that allows an attacker to inject **JavaScript code** into a web page. When another user visits the affected page, the injected JavaScript is executed by **their web browser**.

Unlike server-side vulnerabilities, JavaScript is a **client-side scripting language**, meaning the code runs on the **victim's browser**, not on the web server.

As a result:

* The web server simply delivers the page containing the malicious JavaScript.
* The victim's browser executes the JavaScript automatically.
* Any actions performed by the script happen in the victim's browser under their current session.

> **Important:** Even if the injected JavaScript attempts to establish a reverse shell or perform other actions, those actions execute from the **victim's computer**, **not from the web server**.

---

 Types of Cross-Site Scripting (XSS)

There are **three main types of XSS vulnerabilities**:

 1. Stored (Persistent) XSS

Stored XSS occurs when the malicious JavaScript is permanently stored by the web application, usually inside a database.

Examples include:

* Comment sections
* Guestbooks
* Forums
* User profiles
* Message boards

When any user visits the affected page, the stored JavaScript is automatically executed in their browser.

**Characteristics:**

* Payload is stored on the server.
* Executes every time the vulnerable page is viewed.
* Can affect multiple users.

---

 2. Reflected XSS

Reflected XSS occurs when the malicious script is included in a specially crafted URL or request.

The payload is **not stored** on the server.

Instead:

1. The attacker creates a malicious URL.
2. The victim is persuaded to click the link.
3. The application reflects the input back into the webpage.
4. The browser executes the injected JavaScript.

**Characteristics:**

* Not stored in the database.
* Requires user interaction.
* Commonly delivered through phishing emails or malicious links.

---

 3. DOM-Based XSS

DOM-Based XSS occurs entirely inside the user's browser.

Instead of the web server processing the malicious input, JavaScript running in the browser updates the page using data from sources such as:

* URL parameters
* URL fragments (``)
* Client-side JavaScript variables

Since the payload never reaches the server, traditional server-side filtering may not detect it.

This type of XSS commonly appears in modern websites that dynamically update content **without refreshing the page**.

Examples include:

* Live search suggestions
* Dynamic page updates
* Single Page Applications (SPAs)
* Client-side rendering using JavaScript

**Characteristics:**

* Executes entirely in the browser.
* No communication with the server is required after the page loads.
* Can bypass server-side validation because the payload is never processed by the server.

---

 Client-Side Execution

Unlike SQL Injection, which executes SQL queries on the database server, XSS executes JavaScript on the victim's browser.

The flow is:

1. An attacker injects JavaScript into a vulnerable web application.
2. The web application serves the page to users.
3. A victim opens the page.
4. The victim's browser executes the JavaScript.
5. The attacker can perform actions within the victim's browser session.

---
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@


 Discovering Reflected Cross-Site Scripting (XSS)

In this exercise, we tested a web application for a **Reflected Cross-Site Scripting (XSS)** vulnerability. Similar to SQL Injection testing, we inspected user input fields and URL parameters to determine whether user-supplied data could be executed as JavaScript.

 Identifying Potential Injection Points

During reconnaissance, we looked for locations where user input is reflected back to the browser, such as:

* Text input fields
* Search boxes
* Login forms
* URL parameters (GET requests)

For example, the application accepts a parameter similar to:

```text
http://<target>/vulnerabilities/xss_r/?name=Adil
```

Since the application reflects the value of the **name** parameter back to the webpage, it becomes a potential XSS injection point.

---

 Testing the Input Field

We navigated to:

**DVWA → XSS (Reflected)**

The page contains a text field asking for a user's name.

Instead of entering normal text, we supplied the following JavaScript payload:

```html
<script>alert('XSS')</script>
```

After clicking **Submit**, the browser immediately displayed an alert dialog containing **XSS**.

This confirms that the application executed our JavaScript instead of treating it as plain text.

<img width="1440" height="745" alt="Screenshot 2026-07-14 at 9 04 48 PM" src="https://github.com/user-attachments/assets/8a087129-aceb-4c83-9d61-005d39d3d49b" />



---

 Testing Through the URL

Since the application uses the **GET** method, the input also appears in the URL.

The URL becomes similar to:

```text
http://<target>/vulnerabilities/xss_r/?name=<script>alert('XSS')</script>
```

When this URL is opened, the browser executes the injected JavaScript automatically and displays the alert box.

This demonstrates that the payload can be delivered through a specially crafted URL.

<img width="1438" height="728" alt="Screenshot 2026-07-14 at 9 06 01 PM" src="https://github.com/user-attachments/assets/3835288e-8f82-4602-af9b-526430384c7d" />

<img width="1440" height="818" alt="Screenshot 2026-07-14 at 9 05 50 PM" src="https://github.com/user-attachments/assets/2079d49b-5d15-400e-ae35-851d70a70d98" />


---

 Reflected XSS Attack Scenario

Unlike Stored XSS, the payload is **not saved** by the application.

Instead:

1. The attacker creates a URL containing the malicious JavaScript.
2. The attacker sends the URL to a victim (for example, through email or instant messaging).
3. The victim opens the link.
4. The web application reflects the input back into the page.
5. The victim's browser executes the injected JavaScript.

Because the payload is reflected from the request and not stored on the server, this is classified as **Reflected XSS**.

---

 Key Takeaways

* Reflected XSS occurs when user input is immediately returned by the application without proper sanitization.
* Both **text input fields** and **URL parameters** should be tested for XSS vulnerabilities.
* A simple payload such as:

```html
<script>alert('XSS')</script>
```

can be used to verify whether JavaScript execution is possible.

* If the browser executes the script, the application is vulnerable to Reflected XSS.
* Unlike Stored XSS, the payload is **not stored** on the server and only executes when a victim opens the malicious URL.
* In a real attack, an attacker would typically send the crafted URL to a victim through phishing or another social engineering technique, causing the victim's browser to execute the malicious JavaScript.

