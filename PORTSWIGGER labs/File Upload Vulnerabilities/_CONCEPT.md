# File Upload Vulnerabilities

## 1. What is File Upload Vulnerability?

**Short Definition:**
File upload vulnerability হলো এমন একটি দুর্বলতা যেখানে কোনো ওয়েব সার্ভার ব্যবহারকারীর কাছ থেকে ফাইল আপলোড গ্রহণ করার সময় ফাইলের নাম, টাইপ, কন্টেন্ট বা সাইজ সঠিকভাবে ভ্যালিডেট (validate) করে না।

**Detailed Explanation:**
যখন কোনো অ্যাপ্লিকেশন ফাইল আপলোডের ক্ষেত্রে পর্যাপ্ত রেস্ট্রিকশন বা ভ্যালিডেশন প্রয়োগ করতে ব্যর্থ হয়, তখন একজন attacker একটি সাধারণ ইমেজ আপলোড ফাংশন ব্যবহার করে বিপজ্জনক ফাইল আপলোড করতে পারে। এর মধ্যে server-side script ফাইলও থাকতে পারে (যেমন: PHP, Java, Python), যা আপলোড করার পর execute করলে attacker সার্ভারে Remote Code Execution (RCE) করতে সক্ষম হয়। কিছু ক্ষেত্রে শুধু ক্ষতিকর ফাইল আপলোড করাই ড্যামেজ তৈরি করতে পারে, আবার কিছু ক্ষেত্রে আপলোড করা ফাইলটিকে HTTP রিকোয়েস্টের মাধ্যমে ট্রিগার করে এক্সিকিউট করতে হয়।

## 2. Why Does It Happen?

এই দুর্বলতাগুলো তৈরি হওয়ার মূল কারণগুলো হলো:

* **Flawed Validation:** ডেভেলপাররা এমন ভ্যালিডেশন মেকানিজম ব্যবহার করেন যা তারা robust মনে করেন, কিন্তু আসলে তা ত্রুটিপূর্ণ বা সহজে বাইপাস করা সম্ভব।
* **Blacklisting Mistakes:** অ্যাপ্লিকেশন প্রায়শই বিপজ্জনক ফাইল টাইপের (যেমন .php) ব্ল্যাকলিস্ট (blacklist) ব্যবহার করে। কিন্তু ফাইল এক্সটেনশন পার্স করার সময় নানা ধরনের অমিল (discrepancies) তৈরি হয়, অথবা কম পরিচিত বিপজ্জনক এক্সটেনশন ব্ল্যাকলিস্ট থেকে বাদ পড়ে যায়।
* **Trusting User Input:** ফাইল টাইপ চেক করার সময় অ্যাপ্লিকেশন এমন সব প্রপার্টি বা হেডারের (যেমন: Content-Type) ওপর নির্ভর করে যা attacker Burp Proxy বা Repeater ব্যবহার করে সহজেই ম্যানিপুলেট করতে পারে।
* **Inconsistent Configuration:** ওয়েবসাইটের বিভিন্ন ডিরেক্টরি এবং হোস্টের মধ্যে ভ্যালিডেশনের নিয়মে অসামঞ্জস্যতা (discrepancies) থাকে।

## 3. How It Works (How Servers Handle Static Files)

ওয়েব সার্ভার কীভাবে স্ট্যাটিক ফাইল প্রসেস করে, তার ওপর ভিত্তি করে এই অ্যাটাক কাজ করে।

1. User একটি ফাইল আপলোড করে এবং পরবর্তীতে সেই ফাইলের জন্য HTTP রিকোয়েস্ট পাঠায়।
2. সার্ভার রিকোয়েস্টের পাথ পার্স করে ফাইলের এক্সটেনশন আইডেন্টিফাই করে।
3. সার্ভার এক্সটেনশন এবং পূর্বনির্ধারিত MIME type-এর ম্যাপিংয়ের ওপর ভিত্তি করে ফাইলের টাইপ নির্ধারণ করে।
4. **Execution Logic:**
* যদি ফাইলটি non-executable হয় (যেমন ইমেজ বা HTML), সার্ভার ফাইলের কন্টেন্ট সরাসরি রেসপন্সে পাঠিয়ে দেয়।
* যদি ফাইলটি executable হয় (যেমন PHP) এবং সার্ভার সেটি রান করার জন্য কনফিগার করা থাকে, তবে সার্ভার স্ক্রিপ্টটি রান করে এবং তার আউটপুট রেসপন্স হিসেবে পাঠায়।
* যদি ফাইলটি executable হয় কিন্তু সার্ভার সেটি রান করতে কনফিগার করা না থাকে, তবে সাধারণত error আসে। তবে মাঝে মাঝে সার্ভার ফাইলের কন্টেন্ট plain text হিসেবে দেখিয়ে দেয়, যা source code লিক (information disclosure) করতে পারে।



## 4. Simple Example

**Normal Request:**
একজন সাধারণ ইউজার তার প্রোফাইল পিকচার হিসেবে `avatar.jpg` আপলোড করে। সার্ভার সেটি `/images/avatar.jpg` ডিরেক্টরিতে সেভ করে।

**Modified Request:**
Attacker `avatar.jpg`-এর বদলে `shell.php` নামের একটি ফাইল আপলোড করে, যার ভেতরে ক্ষতিকর কোড রয়েছে। সার্ভারের ভ্যালিডেশন না থাকায় সেটি `/images/shell.php` হিসেবে সেভ হয়।

**Result:**
Attacker যখন `/images/shell.php` ব্রাউজ করে, তখন সার্ভার ওই PHP কোডটি রান করে এবং attacker সার্ভারের এক্সেস পেয়ে যায়।

## 5. Technical Example

**multipart/form-data request:**

```http
POST /images HTTP/1.1
Host: normal-website.com
Content-Length: 12345
Content-Type: multipart/form-data; boundary=---------------------------012345678901234567890123456

---------------------------012345678901234567890123456
Content-Disposition: form-data; name="image"; filename="exploit.php"
Content-Type: application/x-httpd-php

<?php echo system($_GET['command']); ?>
---------------------------012345678901234567890123456--

```

**Explanation:**

* এখানে `Content-Disposition` 헤ডার বলে দিচ্ছে যে এটি `image` ফিল্ডের ডেটা এবং এর নাম `exploit.php`।
* Attacker এখানে একটি PHP web shell পাঠাচ্ছে।
* সার্ভার যদি ফাইলের কন্টেন্ট বা আসল এক্সটেনশন ভ্যালিডেট না করে, তবে এই `exploit.php` সার্ভারে সেভ হবে এবং RCE ট্রিগার করবে।

## 6. Attack Flow

Attacker
↓
Find a file upload endpoint
↓
Upload a malicious server-side script (Web Shell)
↓
Bypass validation (Extension obfuscation / Content-Type manipulation)
↓
File is saved on the server
↓
Attacker requests the uploaded file's URL
↓
Server executes the file as code
↓
Remote Code Execution / Server Compromise

## 7. Important Conditions / Requirements

* অ্যাপ্লিকেশনকে ফাইল আপলোড অ্যালাউ করতে হবে।
* ফাইলটি এমন একটি ডিরেক্টরিতে সেভ হতে হবে যা পাবলিকলি এক্সেসিবল (URL এর মাধ্যমে রিকোয়েস্ট করা যায়)।
* সার্ভারকে আপলোড করা ফাইল টাইপটি (যেমন .php, .jsp) কোড হিসেবে এক্সিকিউট করার জন্য কনফিগার করা থাকতে হবে।
* ফাইলের নাম, এক্সটেনশন বা কন্টেন্ট ভ্যালিডেশন দুর্বল বা বাইপাস করার যোগ্য হতে হবে।

## 8. Exploitation / Practical Understanding

File upload vulnerability এক্সপ্লয়েট করার বিভিন্ন প্র্যাকটিক্যাল টেকনিক নিচে বিস্তারিত আলোচনা করা হলো:

### A. Deploying a Web Shell

সবচেয়ে ভয়ংকর পরিস্থিতি হলো যখন আপনি একটি server-side script (যেমন PHP, Java, Python) আপলোড করে তা রান করতে পারেন। Web shell হলো এমন একটি ক্ষতিকর স্ক্রিপ্ট যা attacker-কে HTTP রিকোয়েস্টের মাধ্যমে সার্ভারে কমান্ড রান করার সুবিধা দেয়।

* **How to trigger:** ফাইলটি আপলোড করার পর সেটির URL-এ ব্রাউজ করে প্যারামিটার পাস করতে হয় (e.g., `?command=id`)।

### B. Flawed File Type Validation (Content-Type Restriction Bypass)

ব্রাউজার থেকে ফাইল আপলোড করার সময় ডেটা `multipart/form-data` ফরম্যাটে যায়। প্রতিটি ফাইলের জন্য একটি `Content-Type` হেডার থাকে যা ফাইলের MIME type বলে দেয়।

* **How it works:** সার্ভার অনেক সময় শুধুমাত্র এই `Content-Type` হেডারটি চেক করে।
* **Bypass:** Burp Repeater-এ রিকোয়েস্ট পাঠিয়ে `filename="exploit.php"` রেখে `Content-Type: image/jpeg` করে দিলে সার্ভার বিশ্বাস করে যে এটি একটি ইমেজ এবং ফাইলটি আপলোড করতে দেয়।

### C. Web Shell Upload via Path Traversal

সার্ভার হয়তো যে ডিরেক্টরিতে ফাইল আপলোড হয়, সেখানে স্ক্রিপ্ট এক্সিকিউট করা বন্ধ করে রেখেছে।

* **How it works:** আপনি যদি ফাইলটি অন্য কোনো ডিরেক্টরিতে (যেখানে এক্সিকিউশন অ্যালাউড) আপলোড করতে পারেন, তবে স্ক্রিপ্ট রান করানো সম্ভব।
* **Bypass:** `filename` প্যারামিটারে ডিরেক্টরি ট্রাভার্সাল (Directory Traversal) সিকোয়েন্স ব্যবহার করা। যেমন: `filename="../exploit.php"`।

### D. Insufficient Blacklisting & Overriding Server Configuration

সার্ভার হয়তো `.php` এক্সটেনশন ব্ল্যাকলিস্ট করে রেখেছে।

* **Bypass (Alternative Extensions):** কম পরিচিত এক্সটেনশন যেমন `.php5`, `.shtml` ইত্যাদি ব্যবহার করা।
* **Bypass (Config Override):** Apache সার্ভারে `.htaccess` বা IIS সার্ভারে `web.config` ফাইল আপলোড করা। এর মাধ্যমে আপনি সার্ভারকে নির্দেশ দিতে পারেন যেন সে নির্দিষ্ট কোনো কাস্টম এক্সটেনশনকে এক্সিকিউটেবল হিসেবে গণ্য করে।
* Apache Example: `AddType application/x-httpd-php .customExt`



### E. Obfuscating File Extensions

শক্তিশালী ব্ল্যাকলিস্ট বাইপাস করার জন্য এক্সটেনশন অবফাসকেট (obfuscate) করা যায়:

* **Multiple extensions:** `exploit.php.jpg` (সার্ভার পার্সিং অ্যালগরিদমের ওপর নির্ভর করে এটি বাইপাস হতে পারে)।
* **Trailing characters:** `exploit.php.` (কিছু কম্পোনেন্ট ডট বা স্পেস রিমুভ করে দেয়)।
* **URL encoding:** `exploit%2Ephp` (ভ্যালিডেশনের সময় ডিকোড না হলে, কিন্তু পরে সার্ভার-সাইডে ডিকোড হলে)।
* **Null bytes / Semicolons:** `exploit.asp;.jpg` বা `exploit.asp%00.jpg` (PHP/Java-তে ভ্যালিডেশন হলেও C/C++ এর লো-লেভেল ফাংশন null byte পেলে ফাইলের নাম ওখানেই শেষ বলে ধরে নেয়)।
* **Multibyte unicode characters:** `xC0 x2E`, `xC4 xAE` বা `xC0 xAE` যা UTF-8 হিসেবে পার্স হয়ে পরবর্তীতে ASCII-তে কনভার্ট হওয়ার পর `. (dot)` হয়ে যায়।
* **Recursive stripping bypass:** সার্ভার যদি `.php` রিমুভ করে দেয়, তবে `exploit.p.phphp` দিলে মাঝখানের অংশ রিমুভ হয়ে আবার `.php` তৈরি হবে।

### F. Flawed Validation of File Contents (Polyglot Web Shell)

অনেক সার্ভার ফাইলের `Content-Type` বিশ্বাস না করে ফাইলের আসল কন্টেন্ট বা ডাইমেনশন (ইমেজের ক্ষেত্রে) ভেরিফাই করে।

* **How it works:** সার্ভার ফাইলের সিগনেচার বা ম্যাজিক বাইটস চেক করে (যেমন JPEG ফাইল সবসময় `FF D8 FF` দিয়ে শুরু হয়)।
* **Bypass:** ExifTool-এর মতো টুল ব্যবহার করে একটি Polyglot JPEG ফাইল তৈরি করা যায়, যার মেটাডেটার (metadata) মধ্যে ক্ষতিকর PHP কোড লুকানো থাকে কিন্তু ফাইলটি একটি ভ্যালিড ইমেজ হিসেবেই আচরণ করে।

### G. Exploiting File Upload Race Conditions

আধুনিক ফ্রেমওয়ার্কগুলো ফাইল সরাসরি মেইন ফাইলসিস্টেমে আপলোড না করে স্যান্ডবক্সড টেম্পোরারি (temporary) ডিরেক্টরিতে রাখে, রেন্ডম নাম দেয় এবং ভ্যালিডেট করার পর মূল জায়গায় মুভ করে।

* **How it works:** ডেভেলপাররা অনেক সময় নিজেদের কাস্টম প্রসেসিং তৈরি করতে গিয়ে রেস কন্ডিশন (Race condition) তৈরি করে ফেলেন। যেমন: ফাইলটি মেইন ফাইলসিস্টেমে আপলোড করে অ্যান্টি-ভাইরাস দিয়ে চেক করানো এবং ক্ষতিকর হলে রিমুভ করা।
* **Bypass:** ফাইলটি আপলোড হওয়ার পর এবং ডিলিট হওয়ার মাঝখানের কয়েক মিলি-সেকেন্ড সময়ের মধ্যে যদি attacker ফাইলটির URL-এ রিকোয়েস্ট পাঠাতে পারে, তবে স্ক্রিপ্টটি এক্সিকিউট হয়ে যাবে।

### H. Race Conditions in URL-based File Uploads

যখন URL এর মাধ্যমে ফাইল আপলোড করা হয়, সার্ভার ইন্টারনেট থেকে ফাইলটি ফেচ করে লোকাল কপি বানায়।

* **Bypass:** টেম্পোরারি ডিরেক্টরির নাম যদি PHP এর `uniqid()` এর মতো pseudo-random ফাংশন দিয়ে তৈরি হয়, তবে তা ব্রুট-ফোর্স (brute-force) করা সম্ভব।
* **Timing Trick:** ফাইলের শুরুতে পেলোড রেখে শেষে বিশাল সাইজের প্যাডিং বাইট (padding bytes) যুক্ত করে একটি বড় ফাইল আপলোড করা, যাতে সার্ভারের প্রসেস করতে বেশি সময় লাগে এবং ব্রুট-ফোর্স করার জন্য উইন্ডো (time window) বড় হয়।

### I. Uploading Files Using PUT

ওয়েব ইন্টারফেসে আপলোড ফাংশন না থাকলেও সার্ভার যদি `PUT` রিকোয়েস্ট সাপোর্ট করে, তবে সরাসরি রিকোয়েস্ট পাঠিয়ে ফাইল আপলোড করা সম্ভব।

### J. Exploiting File Uploads without RCE

RCE না পেলেও অন্যান্য অ্যাটাক করা যায়:

* **Malicious Client-Side Scripts (Stored XSS):** HTML বা SVG ফাইল আপলোড করা যেখানে `<script>` ট্যাগ আছে। অন্য ইউজাররা এটি দেখলে তাদের ব্রাউজারে স্ক্রিপ্ট রান করবে (একই অরিজিন থেকে সার্ভ করা হলে)।
* **Parsing Vulnerabilities:** সার্ভার যদি XML-বেসড ফাইল (যেমন `.doc` বা `.xls`) পার্স করে, তবে XXE injection করা যেতে পারে।

## 9. Step-by-Step Exploitation

Step 1 — Identify the upload function: ওয়েবসাইটে ফাইল আপলোড করার অপশন খুঁজে বের করুন।
Step 2 — Test the behavior: একটি সাধারণ ফাইল (যেমন `.jpg`) আপলোড করে দেখুন সেটি কোথায় এবং কীভাবে সেভ হচ্ছে।
Step 3 — Test RCE: একটি সিম্পল ওয়েব শেল (e.g., `shell.php`) আপলোড করার চেষ্টা করুন।
Step 4 — Determine required conditions (Bypass Validation): যদি ব্লক হয়, তবে `Content-Type` পরিবর্তন করুন, এক্সটেনশন অবফাসকেট করুন, বা ডিরেক্টরি ট্রাভার্সাল ব্যবহার করুন।
Step 5 — Exploit the vulnerability: ফাইলটি সফলভাবে আপলোড হলে সেটির পাথে (e.g., `/images/shell.php?command=whoami`) নেভিগেট করুন।
Step 6 — Verify the result: রেসপন্সে কমান্ডের আউটপুট দেখা গেলে RCE কনফার্মড।

## 10. HTTP Requests / Responses

**Uploading Web Shell using PUT Method:**

```http
PUT /images/exploit.php HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-httpd-php
Content-Length: 49

<?php echo file_get_contents('/path/to/file'); ?>

```

*Explanation:* `PUT` মেথড ব্যবহার করে সরাসরি সার্ভারের নির্দিষ্ট পাথে ফাইল তৈরি করার রিকোয়েস্ট পাঠানো হয়েছে।

**Content-Type Response Clue:**
সার্ভার ফাইল সার্ভ করার সময় `Content-Type` রেসপন্স হেডার পাঠায়। এটি দেখে বোঝা যায় সার্ভার ফাইলটিকে কী হিসেবে ট্রিট করছে।

```http
HTTP/1.1 200 OK
Content-Type: text/plain

<?php echo system($_GET['command']); ?>

```

*Explanation:* সার্ভার PHP ফাইল রান করার জন্য কনফিগার করা নেই, তাই সে কোডটিকে `text/plain` হিসেবে আউটপুট দিয়ে দিয়েছে।

## 11. Payloads

* **Versatile PHP Web Shell:**
`<?php echo system($_GET['command']); ?>`
*Purpose:* URL প্যারামিটারের মাধ্যমে যেকোনো সিস্টেম কমান্ড রান করানো। (e.g., `?command=id`).
* **File Read PHP Web Shell:**
`<?php echo file_get_contents('/path/to/target/file'); ?>`
*Purpose:* সার্ভারের ফাইল সিস্টেম থেকে কোনো নির্দিষ্ট ফাইলের কন্টেন্ট পড়া।
* **Directory Traversal Filename:**
`filename="../exploit.php"`
*Purpose:* রেস্ট্রিক্টেড আপলোড ডিরেক্টরি থেকে বের হয়ে অন্য ফোল্ডারে ফাইল সেভ করা।
* **Recursive Stripping Bypass:**
`exploit.p.phphp`
*Purpose:* সার্ভার যদি `.php` রিমুভ করে, তবে এটি আবার `.php` হয়ে যাবে।

## 12. Burp Suite Workflow

* **Repeater:** ফাইল আপলোডের `multipart/form-data` রিকোয়েস্ট ধরে `Content-Type` হেডার ম্যানিপুলেট করতে এবং ফাইলের নাম বা এক্সটেনশন পরিবর্তন করে বারবার টেস্ট করতে ব্যবহৃত হয়।
* **Proxy:** ব্রাউজার থেকে যাওয়া আপলোড রিকোয়েস্ট ইন্টারসেপ্ট করার জন্য।
* **Intruder:** অবফাসকেশনের বিভিন্ন এক্সটেনশন (যেমন `.php`, `.php5`, `.php%00.jpg`) বা URL-based আপলোডের ক্ষেত্রে `uniqid()` ডিরেক্টরি নাম ব্রুট-ফোর্স করার জন্য।

## 13. How to Identify / Detect

* বিভিন্ন এক্সটেনশন (`.php`, `.jsp`, `.html`, `.svg`) আপলোড করার চেষ্টা করুন।
* রিকোয়েস্টে `Content-Type` পরিবর্তন করে দেখুন ভ্যালিডেশন বাইপাস হচ্ছে কি না।
* `OPTIONS` রিকোয়েস্ট পাঠিয়ে দেখুন সার্ভার `PUT` মেথড সাপোর্ট করে কি না।
* ফাইল আপলোড করার পর ফাইলের লোকেশন বের করে সেটি ব্রাউজ করার চেষ্টা করুন।

## 14. Common Mistakes

* **Blacklisting:** ডেভেলপারদের বিপজ্জনক এক্সটেনশনের ব্ল্যাকলিস্ট ব্যবহার করা। এটি কখনোই নিরাপদ নয় কারণ অ্যাটাকার অবফাসকেশন বা অজানা এক্সটেনশন দিয়ে তা বাইপাস করতে পারে।
* **Trusting User Input:** `Content-Type` হেডার বা ফাইলের এক্সটেনশনকে অন্ধভাবে বিশ্বাস করা।
* **Inconsistent Validation:** শুধুমাত্র ফ্রন্টএন্ড বা ক্লায়েন্ট-সাইডে ভ্যালিডেশন করা।
* **No Sandboxing:** ভ্যালিডেট করার আগেই ফাইলটিকে সরাসরি সার্ভারের মেইন ফাইলসিস্টেমে রাখা।

## 15. Limitations

* ফাইল আপলোড সফল হলেও সার্ভারে যদি ওই নির্দিষ্ট এক্সটেনশন এক্সিকিউট করার পারমিশন না থাকে, তবে RCE সম্ভব নয়।
* ফাইল আপলোডের পর ফাইলের নাম যদি সার্ভার স্ট্রং রেন্ডমাইজেশন (strong randomization) করে এবং আপনি পাথের নাম না জানেন, তবে ফাইলটি ট্রিগার করা অসম্ভব হতে পারে।
* যদি ফাইলটি ভিন্ন কোনো অরিজিন (origin) বা স্যান্ডবক্সড ডোমেইন থেকে সার্ভ করা হয়, তবে Stored XSS এর মতো ক্লায়েন্ট-সাইড অ্যাটাক কাজ করবে না।

## 16. Edge Cases / Important Details

* **Source Code Leakage:** সার্ভার যদি PHP রান না করে `text/plain` হিসেবে আউটপুট দেয়, তবে এটি দিয়ে ব্যাকএন্ডের সোর্স কোড বা সেনসিটিভ ডেটা লিক করানো যেতে পারে।
* **Reverse Proxies:** একই ডোমেইনে রিকোয়েস্ট পাঠালেও ব্যাকএন্ডে লোড ব্যালান্সার বা রিভার্স প্রক্সির কারণে আলাদা সার্ভার রিকোয়েস্ট হ্যান্ডেল করতে পারে, যাদের কনফিগারেশন ভিন্ন হতে পারে।
* **Polyglot Files:** এক্সিফ টুল (ExifTool) ব্যবহার করে ইমেজের মেটাডেটাতে কোড লুকালে সার্ভারের কন্টেন্ট ভ্যালিডেশন (যেমন ডাইমেনশন বা সিগনেচার চেক) বোকা বনে যায়।

## 17. Database / Platform / Technology Differences

| Technology | Important Difference |
| --- | --- |
| **Apache Server** | ডিরেক্টরি স্পেসিফিক কনফিগারেশন ওভাররাইড করার জন্য `.htaccess` ফাইল ব্যবহার করে। যেমন: `LoadModule` বা `AddType application/x-httpd-php .php` |
| **IIS Server** | কনফিগারেশন ওভাররাইড করার জন্য `web.config` ফাইল ব্যবহার করে। যেমন `<mimeMap fileExtension=".json" mimeType="application/json" />` |
| **C / C++ Parsing** | C/C++ এর লো-লেভেল ফাংশনগুলো null byte (`%00`) পেলে স্ট্রিংয়ের শেষ (termination) হিসেবে ধরে নেয়, যা PHP বা Java-তে হয় না। |
| **PHP** | URL-based আপলোডের ক্ষেত্রে টেম্পোরারি নাম জেনারেট করতে `uniqid()` ব্যবহার করলে তা ব্রুট-ফোর্স করা যায়। |

## 18. Impact

* **Remote Code Execution (RCE):** ওয়েব শেল ব্যবহার করে সার্ভারের পূর্ণ নিয়ন্ত্রণ নেওয়া।
* **Overwriting Critical Files:** ডিরেক্টরি ট্রাভার্সাল ব্যবহার করে ফাইলের নাম দিয়ে সিস্টেমের গুরুত্বপূর্ণ ফাইল ওভাররাইট করা।
* **Stored XSS:** HTML বা SVG ফাইল আপলোড করে ক্লায়েন্ট-সাইড অ্যাটাক করা।
* **XXE Injection:** `.doc` বা `.xls` ফাইলের পার্সিং দুর্বলতা ব্যবহার করা।
* **Denial of Service (DoS):** সাইজ ভ্যালিডেশন না থাকলে বিশাল সাইজের ফাইল আপলোড করে সার্ভারের ডিস্ক স্পেস পূর্ণ করে ফেলা।

## 19. Prevention / Mitigation

ফাইল আপলোড দুর্বলতা প্রতিরোধ করার জন্য নিচের প্র্যাকটিসগুলো অনুসরণ করা উচিত:

* **Whitelisting:** ব্ল্যাকলিস্টের বদলে সবসময় অনুমোদিত এক্সটেনশনের হোয়াইটলিস্ট (whitelist) ব্যবহার করুন।
* **Filename Validation:** নিশ্চিত করুন ফাইলের নামে যেন কোনো ডিরেক্টরি ট্রাভার্সাল সিকোয়েন্স (`../`) বা বিপজ্জনক সাবস্ট্রিং না থাকে।
* **Rename Files:** কলিশন (collision) বা ফাইল ওভাররাইট হওয়া এড়াতে আপলোড করা ফাইলের নাম রেন্ডমভাবে পরিবর্তন করে দিন।
* **Sandboxing:** পুরোপুরি ভ্যালিডেট না হওয়া পর্যন্ত ফাইলটিকে কখনোই সার্ভারের পার্মানেন্ট ফাইলসিস্টেমে আপলোড করবেন না।
* **Use Established Frameworks:** ফাইল আপলোড প্রসেসিংয়ের জন্য নিজে থেকে ভ্যালিডেশন মেকানিজম না লিখে প্রতিষ্ঠিত ফ্রেমওয়ার্ক ব্যবহার করুন।

## 20. Real Understanding

ফাইল আপলোড ভালনারেবিলিটি মূলত 'বিশ্বাস' (trust) এর একটি বড় ভুল। সার্ভার যখন ধরে নেয় যে ইউজার একটি ছবিই আপলোড করছে এবং সেই ফাইলের নামের এক্সটেনশন বা Content-Type হেডারকে সত্য বলে মেনে নেয়, তখনই অ্যাটাকার তার ফায়দা লোটে। অ্যাটাকার আসলে ফাইল আপলোড মেকানিজম হ্যাক করে না, সে ফাইল আপলোডের প্রক্রিয়াটি ব্যবহার করে সার্ভারে নিজের কোড প্ল্যান্ট (plant) করে এবং পরবর্তীতে সার্ভারকে দিয়ে সেই কোড রান করায়।

## 21. Mental Model

User uploads a file (claims it's an image via headers/extension)
↓
Server performs weak validation (trusts Content-Type, blacklists only .php)
↓
Attacker obfuscates extension (.php5, .php.jpg) or sends a Polyglot file
↓
File is saved to a web-accessible directory
↓
Attacker navigates to the file URL
↓
Server recognizes the executable logic and runs it
↓
Remote Code Execution (Web Shell)

## 22. Quick Revision

* **Definition:** সার্ভার যখন ফাইলের নাম, টাইপ বা সাইজ ঠিকমতো ভ্যালিডেট না করে আপলোড করতে দেয়।
* **Cause:** ইনপুটকে বিশ্বাস করা, ব্ল্যাকলিস্ট ব্যবহার করা এবং রেস কন্ডিশন।
* **Main idea:** সার্ভারে ক্ষতিকর স্ক্রিপ্ট (Web Shell) আপলোড করে তা রান করানো।
* **Attack flow:** Upload Web Shell -> Bypass Validation -> Access URL -> RCE.
* **Important condition:** আপলোডেড ফাইলটি পাবলিকলি এক্সেসিবল এবং এক্সিকিউটেবল হতে হবে।
* **Main techniques:** Content-Type bypass, Path Traversal, Extension Obfuscation, Polyglot files, Race Conditions.
* **Detection:** বিভিন্ন এক্সটেনশন টেস্ট করা, Content-Type পরিবর্তন করা।
* **Impact:** RCE, XSS, XXE, DoS, Server Compromise.
* **Prevention:** Extension Whitelisting, Renaming files, Sandboxing, Framework validation.

## 23. Things to Remember

* Content-Type হেডার ইউজার কন্ট্রোল করতে পারে, তাই এটিকে কখনো বিশ্বাস করা উচিত নয়।
* ব্ল্যাকলিস্ট সবসময় বাইপাস করা সম্ভব (e.g., .php5, null bytes, unicode obfuscation)।
* সার্ভারে এক্সিকিউট করতে না পারলেও HTML/SVG আপলোড করে Stored XSS করা সম্ভব।
* রেস কন্ডিশনের ক্ষেত্রে, অ্যান্টি-ভাইরাস ফাইল রিমুভ করার আগের মিলি-সেকেন্ড সময়ের মধ্যেই ফাইল এক্সিকিউট করা যায়।
* Apache-তে `.htaccess` এবং IIS-এ `web.config` আপলোড করে সার্ভার কনফিগারেশন ওভাররাইড করা যায়।

