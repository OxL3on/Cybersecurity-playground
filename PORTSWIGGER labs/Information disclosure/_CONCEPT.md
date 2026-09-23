# Information Disclosure Vulnerabilities

## 1. What is Information Disclosure?

**Short Definition:**
Information disclosure (বা information leakage) হলো এমন একটি ওয়েব সিকিউরিটি ভালনারেবিলিটি যেখানে কোনো ওয়েবসাইট অনিচ্ছাকৃতভাবে (unintentionally) ইউজারদের কাছে sensitive information প্রকাশ করে দেয়।

**Detailed Explanation:**
যখন কোনো ওয়েবসাইট তার সিকিউরিটি বা কনফিগারেশনের ভুলের কারণে এমন কোনো তথ্য লিক করে যা সাধারণ ইউজারের দেখার কথা নয়, তখন তাকে Information disclosure বলে। লিক হওয়া এই তথ্যের মধ্যে থাকতে পারে:

* অন্য ইউজারদের ডেটা (যেমন: ইউজারনেম বা ফিনান্সিয়াল ইনফরমেশন)।
* সেনসিটিভ ব্যবসায়িক ডেটা।
* ওয়েবসাইট এবং এর ইনফ্রাস্ট্রাকচারের টেকনিক্যাল ডিটেইলস।

অনেক সময় লিক হওয়া টেকনিক্যাল তথ্যগুলো সরাসরি কোনো ক্ষতি করে না, কিন্তু একজন অ্যাটাকার এই তথ্যগুলোকে অন্যান্য বড় বা জটিল অ্যাটাক (high-severity attacks) সাজানোর জন্য "missing piece of the puzzle" বা স্টার্টিং পয়েন্ট হিসেবে ব্যবহার করতে পারে।

## 2. Why Does It Happen?

Information disclosure মূলত তিনটি প্রধান কারণে ঘটে:

* **Failure to remove internal content:** প্রোডাকশন এনভায়রনমেন্টে যাওয়ার আগে পাবলিক কন্টেন্ট থেকে ইন্টারনাল ডেটা (যেমন: ডেভেলপারদের লেখা ইনলাইন HTML কমেন্ট) মুছে ফেলতে ভুলে যাওয়া।
* **Insecure configuration:** ওয়েবসাইট বা থার্ড-পার্টি টেকনোলজির ভুল কনফিগারেশন। যেমন: প্রোডাকশনে ডিবাগিং (debugging) বা ডায়াগনস্টিক ফিচার চালু রাখা, অথবা ডিফল্ট কনফিগারেশনের কারণে অনেক বড় ও বিস্তারিত (verbose) এরর মেসেজ দেখানো।
* **Flawed design and behavior:** অ্যাপ্লিকেশনের ডিজাইনে ত্রুটি থাকা। যেমন: ইনভ্যালিড ইনপুট দিলে একেকবার একেক রকম এরর স্টেট বা রেসপন্স আসা, যা দেখে অ্যাটাকার ভ্যালিড ইউজারনেম বা অন্যান্য ডেটা অনুমান (enumerate) করতে পারে।

## 3. How It Works

1. Application প্রোডাকশন এনভায়রনমেন্টে রান করছে কিন্তু এতে কনফিগারেশন বা লজিক্যাল ভুল রয়েছে।
2. Attacker ওয়েবসাইটটি নরমালি ব্রাউজ করে অথবা ইচ্ছাকৃতভাবে অপ্রত্যাশিত ইনপুট (unexpected data) দেয়।
3. Application সেই অপ্রত্যাশিত ইনপুট হ্যান্ডেল করতে গিয়ে ফেইল করে বা ডিবাগিং ডেটা জেনারেট করে।
4. Server রেসপন্সে সেনসিটিভ ডেটা, স্ট্যাক ট্রেস (stack trace) বা ইন্টারনাল ফাইল পাথ রিফ্লেক্ট করে দেয়।
5. Attacker সেই রেসপন্স থেকে টেকনিক্যাল বা সেনসিটিভ ডেটা সংগ্রহ করে।
6. প্রাপ্ত ডেটা ব্যবহার করে অ্যাটাকার পরবর্তী বড় কোনো অ্যাটাকের জন্য ব্লুপ্রিন্ট তৈরি করে।

## 4. Simple Example

**Verbose Error Message:**
ধরা যাক, একজন ইউজার লগইন করার সময় ইচ্ছাকৃতভাবে ভুল ডেটা টাইপ দিল।
*Normal application response:* "Invalid input."
*Vulnerable application response:* "Syntax error in SQL query: SELECT * FROM users WHERE username = ..."

**Result:**
এখানে ওয়েবসাইটটি তার ডেটাবেস টেবিলের নাম (`users`) এবং কলামের নাম (`username`) লিক করে দিয়েছে, যা পরবর্তীতে SQL injection করার জন্য অ্যাটাকারকে দারুণভাবে সাহায্য করবে।

## 5. Technical Example

**Source code disclosure via backup files:**
সাধারণত `.php` ফাইল রিকোয়েস্ট করলে সার্ভার সেটি এক্সিকিউট করে এবং ব্রাউজারে শুধু HTML আউটপুট পাঠায়। কিন্তু টেক্সট এডিটরগুলো অনেক সময় কাজ করার সময় অরিজিনাল ফাইলের একটি ব্যাকআপ তৈরি করে (যেমন: এক্সটেনশনের শেষে `~` যুক্ত করে বা অন্য এক্সটেনশন দিয়ে)।

**Example Request:**
`GET /config.php~ HTTP/1.1`
`Host: example.com`

**Result:**
যেহেতু সার্ভার `.php~` এক্সটেনশনকে এক্সিকিউটেবল হিসেবে চেনে না, তাই সে ফাইলটি রান না করে এর ভেতরের সম্পূর্ণ সোর্স কোড (যাতে ডাটাবেসের পাসওয়ার্ড বা API key থাকতে পারে) প্লেইন টেক্সট হিসেবে রেসপন্সে দিয়ে দেয়।

## 6. Attack Flow

Attacker
↓
Interact with application using fuzzing or unexpected inputs
↓
Application handles input poorly / encounters an error
↓
Application returns verbose error, debug data, or exposes hidden files
↓
Attacker analyzes the response and extracts sensitive information
↓
Attacker uses this leaked information to craft a high-severity exploit

## 7. Important Conditions / Requirements

* ওয়েবসাইটটিতে এমন কোনো এন্ট্রি পয়েন্ট থাকতে হবে যেখানে ফাজিং (fuzzing) করা যায়।
* সার্ভার বা অ্যাপ্লিকেশনের কনফিগারেশনে ত্রুটি থাকতে হবে (যেমন: প্রোডাকশনে ডিবাগ মোড অন থাকা)।
* অ্যাটাকারকে রেসপন্সের ছোটখাটো পার্থক্য (যেমন: status code, response length, timing) লক্ষ্য করার মতো সতর্ক হতে হবে।

## 8. Types / Variations (Common Sources)

Information disclosure বিভিন্ন জায়গা থেকে হতে পারে। নিচে সাধারণ সোর্সগুলো আলোচনা করা হলো:

### A. Files for web crawlers

**What is it?** `/robots.txt` বা `/sitemap.xml` ফাইলগুলো ওয়েব ক্রলারদের ডিরেকশন দেওয়ার জন্য ব্যবহৃত হয়।
**How it works:** এই ফাইলগুলোতে অনেক সময় এমন সব ডিরেক্টরির নাম উল্লেখ থাকে (যাতে ক্রলার সেখানে না যায়), যেগুলো অত্যন্ত সেনসিটিভ। অ্যাটাকার সরাসরি এই ফাইলগুলো পড়ে সেই লুকানো ডিরেক্টরিগুলো খুঁজে বের করতে পারে।

### B. Directory listings

**What is it?** কোনো ডিরেক্টরিতে `index.html` বা ডিফল্ট পেজ না থাকলে ওয়েব সার্ভার সেই ডিরেক্টরির সব ফাইলের লিস্ট দেখিয়ে দেয়।
**How it works:** এর মাধ্যমে অ্যাটাকার খুব সহজেই ওই ডিরেক্টরির ভেতরের সেনসিটিভ ফাইল (যেমন: temporary files, crash dumps) দেখতে এবং ডাউনলোড করতে পারে।

### C. Developer comments

**What is it?** ডেভেলপমেন্টের সময় লেখা ইনলাইন HTML কমেন্ট।
**How it works:** প্রোডাকশনে যাওয়ার আগে এগুলো ডিলিট না করলে, অ্যাটাকার ব্রাউজারের "View Source" থেকে এই কমেন্টগুলো পড়ে হিডেন ডিরেক্টরি বা অ্যাপ্লিকেশন লজিকের ক্লু পেতে পারে।

### D. Error messages

**What is it?** অত্যন্ত বিস্তারিত (verbose) এরর মেসেজ।
**How it works:** এটি অ্যাপ্লিকেশনের প্যারামিটারে কী ধরনের ইনপুট আশা করা হচ্ছে তা বলে দেয়। এটি টেমপ্লেট ইঞ্জিন, ডেটাবেস টাইপ বা সার্ভারের ভার্সনও বলে দিতে পারে, যা দিয়ে অ্যাটাকার পাবলিক এক্সপ্লয়েট খুঁজতে পারে।

### E. Debugging data

**What is it?** ডেভেলপমেন্টের সুবিধার্থে তৈরি করা কাস্টম এরর মেসেজ বা লগ ফাইল।
**How it works:** প্রোডাকশনে এটি লিক হলে অ্যাটাকার সেশন ভেরিয়েবলের ভ্যালু, ব্যাক-এন্ডের ক্রেডেনশিয়াল, ক্রিপ্টোগ্রাফিক কি (key) বা ডিরেক্টরির নাম পেয়ে যেতে পারে।

### F. User account pages

**What is it?** ইউজার প্রোফাইল পেজে লজিক ফ্ল-এর কারণে অন্য ইউজারের ডেটা দেখা যাওয়া।
**Example Request:** `GET /user/personal-info?user=carlos`
**How it works:** অ্যাপ্লিকেশন হয়তো পুরো পেজটি অন্যের জন্য লোড করবে না, কিন্তু ইমেইল অ্যাড্রেস রেন্ডার করার লজিকটি হয়তো ঠিকমতো সেশন চেক করে না। ফলে অ্যাটাকার শুধু প্যারামিটার পরিবর্তন করে অন্যের ইমেইল দেখে ফেলতে পারে।

### G. Backup files

**What is it?** সোর্স কোডের ব্যাকআপ বা টেম্পোরারি ফাইল।
**How it works:** `.bak` বা `~` এক্সটেনশনের ফাইল রিকোয়েস্ট করে সোর্স কোড বা API key হাতিয়ে নেওয়া। এটি Insecure deserialization-এর মতো জটিল অ্যাটাকের পথ খুলে দেয়।

### H. Insecure configuration (HTTP TRACE)

**What is it?** সার্ভারে অপ্রয়োজনীয় বা ডায়াগনস্টিক ফিচার চালু থাকা।
**How it works:** HTTP `TRACE` মেথডটি ডায়াগনস্টিক কাজের জন্য। এটি ব্যবহার করলে সার্ভার রিকোয়েস্টটিকে হুবহু রেসপন্সে ইকো (echo) করে দেয়। এর মাধ্যমে রিভার্স প্রক্সির যোগ করা ইন্টারনাল অথেনটিকেশন হেডার লিক হতে পারে।

### I. Version control history (.git)

**What is it?** প্রোডাকশন সার্ভারে `.git` ফোল্ডার এক্সপোজড থাকা।
**How it works:** অ্যাটাকার `/.git` ব্রাউজ করে বা ডাউনলোড করে পুরো প্রজেক্টের ভার্সন কন্ট্রোল হিস্ট্রি (কমিট লগ, কোড স্নিপেট, হার্ড-কোডেড পাসওয়ার্ড) দেখতে পারে।

## 9. Different Contexts

* **Source Code:** ইনলাইন কমেন্ট বা হার্ড-কোডেড API কি।
* **HTTP Headers:** `TRACE` মেথড ব্যবহার করে ইন্টারনাল হেডার লিক।
* **Filesystem:** Directory listing বা backup files।
* **Application Logic:** রেসপন্স টাইমিং বা ডিফারেন্ট এরর কোড।

## 10. Exploitation / Practical Understanding

**Engineering informative responses:**
অ্যাটাকাররা বসে থাকে না। তারা অ্যাপ্লিকেশনকে ম্যানিপুলেট করে এরর জেনারেট করতে বাধ্য করে। যেমন, কোনো প্যারামিটারে ইচ্ছাকৃতভাবে ইনভ্যালিড ডেটা (অ্যালফাবেটের জায়গায় স্পেশাল ক্যারেক্টার) দিয়ে স্ট্যাক ট্রেস বা ডিবাগ রেসপন্স বের করে আনে।

**Fuzzing:**
ফাজিং হলো স্বয়ংক্রিয়ভাবে প্রচুর আন-এক্সপেক্টেড ডেটা পাঠানো। রেসপন্সে সরাসরি কিছু না এলেও, রেসপন্স প্রসেস করার টাইমের সামান্য পার্থক্য (timing difference) বা ভিন্ন এরর কোড দেখেও অ্যাপ্লিকেশন লজিক বোঝা যায়।

## 11. Step-by-Step Exploitation

Step 1 — Identify parameters: অ্যাপ্লিকেশন কোথায় ইনপুট নেয় তা বের করুন।
Step 2 — Fuzz the inputs: Burp Intruder ব্যবহার করে ইনভ্যালিড বা অপ্রত্যাশিত ডেটা (fuzz strings) পাঠান।
Step 3 — Monitor responses: রেসপন্সের স্ট্যাটাস কোড, লেন্থ বা টাইমিংয়ের পার্থক্য লক্ষ্য করুন।
Step 4 — Look for exposed paths: `/robots.txt`, `/sitemap.xml` বা `/.git` ডিরেক্টরি ম্যানুয়ালি চেক করুন।
Step 5 — Extract data: এরর মেসেজ, স্ট্যাক ট্রেস বা ব্যাকআপ ফাইল থেকে সেনসিটিভ তথ্য সংগ্রহ করুন।
Step 6 — Pivot: এই লিক হওয়া তথ্য ব্যবহার করে অন্য কোনো ক্রিটিকাল ভালনারেবিলিটি (যেমন IDOR বা SQLi) এক্সপ্লয়েট করুন।

## 12. HTTP Requests / Responses

**TRACE Method Request:**

```http
TRACE / HTTP/1.1
Host: vulnerable-website.com

```

**Response:**

```http
HTTP/1.1 200 OK
Content-Type: message/http

TRACE / HTTP/1.1
Host: vulnerable-website.com
X-Internal-Auth-Token: secret_token_123

```

*Explanation:* `TRACE` মেথডের কারণে সার্ভার তার কাছে আসা রিকোয়েস্টটি হুবহু রেসপন্স করে দিয়েছে, এবং দেখা যাচ্ছে রিভার্স প্রক্সি একটি ইন্টারনাল অথেনটিকেশন হেডার (`X-Internal-Auth-Token`) যুক্ত করেছিল যা এখন অ্যাটাকারের কাছে লিক হয়ে গেছে।

## 13. Payloads

* **Backup file extensions:** `~`, `.bak`, `.old`
* *Purpose:* সোর্স কোড প্লেইন টেক্সট হিসেবে পড়া।


* **HTTP TRACE Method:** `TRACE / HTTP/1.1`
* *Purpose:* ইন্টারনাল হেডার বা রিভার্স প্রক্সির ডেটা লিক করা।


* **Version control paths:** `/.git/`
* *Purpose:* সোর্স কোডের হিস্ট্রি এবং কমিট লগ দেখা।


* **Fuzzing characters:** `'`, `"`, `<`, `>`
* *Purpose:* অ্যাপ্লিকেশন লজিকে এরর তৈরি করে স্ট্যাক ট্রেস বা ভার্বোস মেসেজ দেখা।



## 14. Burp Suite Workflow

* **Burp Scanner:** এটি স্বয়ংক্রিয়ভাবে ব্যাকআপ ফাইল, ডিরেক্টরি লিস্টিং, প্রাইভেট কি (private keys), ইমেইল অ্যাড্রেস এবং ক্রেডিট কার্ড নম্বর খুঁজে বের করতে পারে।
* **Burp Intruder:** প্যারামিটারে ফাজিং করার জন্য। `Grep Matching` দিয়ে দ্রুত `error`, `SQL`, বা `invalid` শব্দগুলো রেসপন্সে খোঁজা যায় এবং `Grep Extraction` দিয়ে রেসপন্স থেকে স্পেসিফিক ডেটা এক্সট্রাক্ট করা যায়।
* **Logger++ (BApp Store):** সমস্ত টুলের রিকোয়েস্ট/রেসপন্স লগ করে এবং অ্যাডভান্সড ফিল্টার দিয়ে ইন্টারেস্টিং এন্ট্রিগুলো হাইলাইট করতে সাহায্য করে।
* **Engagement Tools:**
* **Search:** যেকোনো কিওয়ার্ড (regex সহ) খোঁজার জন্য।
* **Find comments:** রেসপন্স থেকে দ্রুত ডেভেলপারদের লেখা ইনলাইন কমেন্টগুলো এক্সট্রাক্ট করার জন্য।
* **Discover content:** ভিজিবল নয় এমন হিডেন ডিরেক্টরি বা ফাইলগুলো ব্রুট-ফোর্স করে বের করার জন্য।



## 15. How to Identify / Detect

* রিকোয়েস্টে ইনভ্যালিড ডেটা দিয়ে দেখুন এরর মেসেজে ফ্রেমওয়ার্কের নাম বা ভার্সন আসে কি না।
* ব্রাউজারের সোর্স কোডে বা Burp-এর "Find comments" টুল দিয়ে ইনলাইন কমেন্ট চেক করুন।
* ম্যানুয়ালি `/robots.txt`, `/sitemap.xml` এবং `/.git` পাথগুলো ব্রাউজ করুন।
* রেসপন্স টাইম এবং লেন্থের পার্থক্য দেখে ইনভিজিবল ডেটা অনুমান করার চেষ্টা করুন।

## 16. Common Mistakes

* "Tunnel vision" তৈরি হওয়া। অর্থাৎ, শুধু একটি নির্দিষ্ট অ্যাটাক (যেমন XSS বা SQLi) খুঁজতে গিয়ে চোখের সামনে থাকা সেনসিটিভ কমেন্ট বা হিডেন ফাইলের দিকে নজর না দেওয়া।
* মনে করা যে, "এরর মেসেজ তো আর কোনো অ্যাকচুয়াল ডেটা লিক করছে না, তাই এটি সেফ।" অথচ একটি ভিন্ন এরর মেসেজ (যেমন "Username exists" বনাম "Invalid username") অ্যাটাকারকে ইউজারনেম এনুমেরেট করতে সাহায্য করে।

## 17. Limitations

এই ভালনারেবিলিটির সবচেয়ে বড় লিমিটেশন হলো, লিক হওয়া তথ্য সব সময় সরাসরি ক্ষতিকর হয় না।
যেমন, একটি ওয়েবসাইট যদি তার ফ্রেমওয়ার্কের ভার্সন লিক করে এবং সেই ভার্সনটি যদি ফুলি প্যাচড (fully patched) হয়, তবে এই ইনফরমেশন ডিসক্লোজার অ্যাটাকারের তেমন কোনো কাজেই আসবে না। তবে ভার্সনটি যদি পুরোনো হয়, তবে এটি একটি ডেভাস্টেটিং (devastating) অ্যাটাকের কারণ হতে পারে।

## 18. Edge Cases / Important Details

* **Timing Differences:** কখনো কখনো অ্যাপ্লিকেশন কোনো এরর মেসেজ দেয় না, কিন্তু রিকোয়েস্ট প্রসেস করতে সামান্য বেশি বা কম সময় নেয়। এই টাইমিং ডিফারেন্সও এক ধরনের ইনফরমেশন ডিসক্লোজার যা অ্যাপ্লিকেশনের ভেতরের অবস্থা (internal state) বলে দেয়।
* **Third-party defaults:** অনেক সময় ডেভেলপাররা থার্ড-পার্টি টেকনোলজি ব্যবহার করেন কিন্তু এর ডিফল্ট কনফিগারেশন চেঞ্জ করেন না, যা ডিফল্টভাবেই অনেক সেনসিটিভ ডেটা লিক করে।

## 19. Database / Platform / Technology Differences

| Technology | Important Details |
| --- | --- |
| **Git** | ভার্সন কন্ট্রোল সিস্টেম। ডিফল্টভাবে `.git` ফোল্ডারে প্রজেক্টের সব কমিট হিস্ট্রি থাকে। |
| **Text Editors (Vim/Nano)** | এডিট করার সময় অনেক এডিটর স্বয়ংক্রিয়ভাবে `~` এক্সটেনশন দিয়ে ব্যাকআপ ফাইল তৈরি করে। |
| **HTTP/Web Servers** | ডিফল্ট কনফিগারেশনে প্রায়ই Directory Listing বা `TRACE` মেথড এনাবল করা থাকে। |

## 20. Impact

এর ইমপ্যাক্ট দুই ধরনের হতে পারে:

* **Direct Impact:** যদি ক্রেডিট কার্ড ডিটেইলস, ইউজারনেম বা ফাইন্যান্সিয়াল ডেটা লিক হয়, তবে এটি অত্যন্ত ক্রিটিকাল এবং সরাসরি ক্ষতিকর।
* **Indirect Impact:** যদি টেকনিক্যাল ডেটা, ডিরেক্টরি স্ট্রাকচার বা ফ্রেমওয়ার্ক ভার্সন লিক হয়, তবে তা সরাসরি ক্ষতি করে না। কিন্তু অ্যাটাকার এই তথ্যগুলো ব্যবহার করে অন্য কোনো বড় অ্যাটাক (যেমন Insecure Deserialization বা SQLi) ডিজাইন করতে পারে।

## 21. Prevention / Mitigation

Information disclosure প্রতিরোধ করতে নিচের পদক্ষেপগুলো নেওয়া উচিত:

* **Awareness:** টিমের সবাইকে সেনসিটিভ তথ্যের গুরুত্ব সম্পর্কে সচেতন করা। অনেক সময় হার্মলেস মনে হওয়া তথ্যও অ্যাটাকারের জন্য দারুণ কাজের হতে পারে।
* **Audit Code (QA/Build):** প্রোডাকশনে যাওয়ার আগে বিল্ড প্রসেসের অংশ হিসেবে স্বয়ংক্রিয়ভাবে ডেভেলপার কমেন্ট এবং ডিবাগ কোড মুছে ফেলার (strip) ব্যবস্থা করা।
* **Generic Error Messages:** অ্যাপ্লিকেশনে সব সময় জেনেরিক (সাধারণ) এরর মেসেজ ব্যবহার করা। অ্যাটাকারকে অ্যাপ্লিকেশনের বিহেভিয়ার সম্পর্কে কোনো ক্লু দেওয়া যাবে না।
* **Disable Debugging:** প্রোডাকশন এনভায়রনমেন্টে অবশ্যই সব ধরনের ডিবাগিং এবং ডায়াগনস্টিক ফিচার বন্ধ রাখতে হবে।
* **Secure Configuration:** যেকোনো থার্ড-পার্টি টেকনোলজি ব্যবহারের আগে এর কনফিগারেশন এবং সিকিউরিটি ইমপ্লিকেশন ভালো করে বোঝা এবং অপ্রয়োজনীয় ফিচারগুলো ডিজেবল করে দেওয়া।

## 22. Vulnerable vs Secure Example

**Vulnerable Error Handling:**

```html
<p>Error: Table 'shop.users' doesn't exist in query: SELECT * FROM shop.users WHERE id = 'A'</p>

```

*Why vulnerable?* এটি ডেটাবেস টেবিলের নাম এবং স্ট্রাকচার বলে দিচ্ছে।

**Secure Error Handling:**

```html
<p>An unexpected error occurred. Please try again later.</p>

```

*Why secure?* এটি জেনেরিক। অ্যাটাকার এখান থেকে কোনো টেকনিক্যাল ক্লু পাচ্ছে না।

## 23. Real Understanding

Information disclosure মূলত কোনো সরাসরি "হ্যাক" নয়, এটি হলো অ্যাপ্লিকেশনের "মুখ ফসকে কথা বলে ফেলা"। যখন অ্যাপ্লিকেশনকে কোনো অপ্রত্যাশিত পরিস্থিতিতে ফেলা হয়, সে ভয় পেয়ে বা কনফিগারেশনের ভুলে তার ভেতরের কথা (সোর্স কোড, এরর লগ, ব্যাকআপ) বলে দেয়। একজন দক্ষ অ্যাটাকার এই ছোট ছোট লিক হওয়া কথাগুলো একত্র করে একটি বড় অ্যাটাকের ব্লুপ্রিন্ট তৈরি করে।

## 24. Mental Model

Application holds sensitive logic & data
↓
Developer makes a configuration mistake or leaves internal artifacts (comments, backups)
↓
Attacker probes the application (Fuzzing, viewing source, directory brute-forcing)
↓
Application responds with internal hints (Stack traces, timing differences, `.git` files)
↓
Attacker collects these puzzle pieces
↓
Attacker uses the extracted info to craft a direct, high-severity exploit

## 25. Quick Revision

* **Definition:** ওয়েবসাইটের অনিচ্ছাকৃতভাবে সেনসিটিভ বা টেকনিক্যাল ডেটা লিক করা।
* **Cause:** ডিবাগিং চালু রাখা, কমেন্ট ডিলিট না করা, ভার্বোস এরর মেসেজ এবং ইনসিকিউর কনফিগারেশন।
* **Main idea:** লিক হওয়া ডেটা দিয়ে পরবর্তী বড় অ্যাটাক সাজানো।
* **Attack flow:** Fuzzing -> Read Verbose Error -> Extract Data -> Exploit.
* **Important condition:** অ্যাপ্লিকেশনে কনফিগারেশন ত্রুটি থাকতে হবে।
* **Main techniques:** Directory listing, Fuzzing, Fetching `.git` or `~` backup files.
* **Detection:** Burp Scanner, Intruder (Grep Matching), Engagement Tools (Find comments).
* **Impact:** Direct (CC data leak) or Indirect (Stepping stone for RCE).
* **Prevention:** Generic errors, strip comments during build, disable debug mode.

## 26. Things to Remember

* টেস্টিংয়ের সময় "Tunnel vision" তৈরি হতে দেওয়া যাবে না। একটি বাগ খুঁজতে গিয়ে অন্য জায়গায় লিক হওয়া ডেটা মিস করা যাবে না।
* এরর মেসেজে কোনো টেক্সট না থাকলেও, রেসপন্স টাইম বা স্ট্যাটাস কোডের পার্থক্যও এক ধরনের Information disclosure.
* `TRACE` মেথড রিকোয়েস্টকে ইকো করে দেয়, যা রিভার্স প্রক্সির ইন্টারনাল হেডার লিক করতে পারে।
* `.git` ডিরেক্টরি এক্সপোজড থাকলে পুরো প্রজেক্টের সোর্স কোড হিস্ট্রি বের করে ফেলা সম্ভব।
* Burp-এর "Engagement tools" (যেমন Find comments) ইনফরমেশন গ্যাদারিংয়ের জন্য খুবই চমৎকার।

