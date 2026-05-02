# 📁 Week 8 – Arbitrary File Upload Vulnerabilities

> A practical guide to understanding and exploiting insecure file upload mechanisms in web applications.

---

## 🎯 Overview

File upload functionality is everywhere — from profile pictures to cloud storage and code repositories. However, when poorly implemented, it becomes one of the most dangerous attack vectors in web security.

This module focuses on how attackers exploit file upload features and how weak validation can lead to severe consequences like **Remote Code Execution (RCE)**.

---

## 📚 Task 1 – Introduction

This section covers foundational knowledge required before diving into exploitation.

### 🔗 Learning Resources


- Link 1: https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html
- Link 2: https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload
- Link 3:
- Link 4:
- Link 5:
- Link 6:
- Link 7:

---


## ⚠️ Task 2 – Understanding File Upload Vulnerabilities

### 🧠 Why File Uploads Matter

File uploads are essential in modern applications, but when insecure, they can allow attackers to:

- Upload malicious files (e.g., reverse shells)
- Execute arbitrary code on the server
- Deface or manipulate website content
- Inject malicious scripts (leading to further attacks)
- Leak sensitive information
- Host illegal or harmful content

👉 **Key Insight:**  
An unrestricted file upload feature can give attackers significant control over a system.

---

## 🧪 Exploitation Methodology

A structured approach to attacking file upload functionality:

### 1. 🔎 Enumeration
- Identify technologies used (frameworks, languages)
- Inspect HTTP headers (`server`, `x-powered-by`)
- Use tools like:
  - Burp Suite
  - Browser extensions (tech detection)
  - Directory brute-forcing tools

---

### 2. 📂 Locate Upload Points
- Find upload forms/pages
- Inspect source code
- Identify client-side filtering mechanisms

---

### 3. ✅ Upload a Benign File
- Test with a harmless file
- Observe:
  - Storage location
  - Access method
  - Naming conventions

💡 Establishes a baseline for further testing

---

### 4. 🧭 Discover File Location
- Attempt direct access (e.g., `/uploads/file.jpg`)
- Use brute-force tools if needed

---

### 5. 💣 Attempt Malicious Upload
- Upload a payload
- Analyze error responses
- Identify filtering mechanisms

---

### 6. 🧠 Identify Server-Side Protections

Test different filtering techniques:

- Extension Filtering
  - Detect blacklist vs whitelist behavior

- Magic Number Validation
  - Modify file headers

- MIME Type Filtering
  - Intercept requests and change content types

- File Size Restrictions
  - Upload progressively larger files

---

## 🧩 Key Takeaway

Attackers rely on:
- Enumeration
- Observation
- Iterative testing

Each failure provides useful information to refine the attack.

---

## 📌 Task 3 – Overwriting Existing Files

### 🧠 Concept

Attackers may overwrite existing files on a server to:
- Modify website content
- Replace legitimate assets
- Inject malicious files

---

### 🧪 Practical Exercise

🎯 Objective:
- Identify a file that can be overwritten
- Replace it using the upload feature

---

### ❓ Questions

1. What is the name of the image file which can be overwritten?  
✍️ Answer:  

📸 Add screenshot showing how the file was identified

---

2. Overwrite the image. What is the flag you receive?  
✍️ Answer:  

📸 Add screenshot showing successful overwrite and flag

---

## 🧠 Final Thoughts

- File upload vulnerabilities are high-impact and often underestimated
- Proper validation must include:
  - File type
  - Content inspection
  - Size restrictions
  - Secure storage practices
- Even small misconfigurations can lead to critical system compromise

---

## 🚀 Next Steps

- Practice bypassing filters in more advanced scenarios
- Explore real-world exploitation techniques
- Apply secure coding practices to defend against these vulnerabilities
