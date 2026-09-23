# Server-side Request Forgery (SSRF)

## 1. What is SSRF?

**Short Definition:**
Server-side request forgery (SSRF) হলো একটি web security vulnerability যা attacker-কে সার্ভার-সাইড অ্যাপ্লিকেশনকে ম্যানিপুলেট করে কোনো অনির্ধারিত (unintended) লোকেশনে HTTP রিকোয়েস্ট পাঠাতে বাধ্য করে।

**Detailed Explanation:**
সাধারণত একটি ওয়েব অ্যাপ্লিকেশন যখন এক্সটারনাল বা ইন্টারনাল কোনো সোর্স থেকে ডেটা ফেচ করে, তখন সে ইউজার বা অন্য সিস্টেমের দেওয়া URL ব্যবহার করে। SSRF-এর ক্ষেত্রে, attacker এই URL ম্যানিপুলেট করে। এর ফলে অ্যাপ্লিকেশনটি attacker-এর পছন্দমতো কোনো ইন্টারনাল সার্ভিস (যা সাধারণত বাইরের কারও অ্যাক্সেস করার কথা নয়) অথবা আর্বিট্রারি কোনো এক্সটারনাল সিস্টেমে রিকোয়েস্ট পাঠায়। এটি সফল হলে sensitive data (যেমন: authorization credentials) লিক হতে পারে এবং ইন্টারনাল নেটওয়ার্কের সিকিউরিটি কম্প্রোমাইজ হতে পারে।

## 2. Why Does It Happen?

SSRF ঘটার মূল কারণ হলো trust relationship এবং insecure input handling।
অনেক অ্যাপ্লিকেশন লোকাল মেশিন (`127.0.0.1` বা `localhost`) বা ইন্টারনাল নেটওয়ার্ক থেকে আসা রিকোয়েস্টগুলোকে অন্ধভাবে বিশ্বাস করে (implicitly trust)।
এর কারণ হতে পারে:

* Access control চেকগুলো হয়তো ফ্রন্ট-এন্ড বা অন্য কোনো কম্পোনেন্টে থাকে। যখন লোকাল সার্ভার থেকে সরাসরি রিকোয়েস্ট যায়, তখন সেই চেকগুলো বাইপাস হয়ে যায়।
* Disaster recovery-এর জন্য অ্যাডমিনিস্ট্রেটরদের লগইন ছাড়াই লোকাল মেশিন থেকে অ্যাক্সেস দেওয়া থাকতে পারে।
* অ্যাডমিনিস্ট্রেটিভ ইন্টারফেস হয়তো ভিন্ন কোনো পোর্টে চলে যা বাইরের ইউজারদের জন্য ব্লক করা, কিন্তু ইন্টারনাল রিকোয়েস্টের জন্য ওপেন।
ডেভেলপাররা যখন ইউজারের দেওয়া URL কোনো ভ্যালিডেশন ছাড়া রিকোয়েস্ট জেনারেট করতে ব্যবহার করেন, তখনই এই trust relationship-কে অ্যাবিউজ করা সম্ভব হয়।

## 3. How It Works

1. User (Attacker) অ্যাপ্লিকেশনকে একটি URL বা ইনপুট প্রোভাইড করে।
2. Application সেই ইনপুটটি গ্রহণ করে ব্যাক-এন্ডে একটি HTTP রিকোয়েস্ট তৈরি করে।
3. Application ইনপুটটিকে ঠিকমতো ভ্যালিডেট বা স্যানিটাইজ করে না।
4. Attacker ইনপুট পরিবর্তন করে ইন্টারনাল কোনো সিস্টেমের ঠিকানা (`localhost` বা `192.168.x.x`) বসিয়ে দেয়।
5. Server সেই ইন্টারনাল সিস্টেমে রিকোয়েস্ট পাঠায়।
6. ইন্টারনাল সিস্টেম দেখে রিকোয়েস্টটি ট্রাস্টেড সার্ভার থেকে এসেছে, তাই সে Security control বাইপাস করে রেসপন্স দিয়ে দেয়।
7. Attacker সেই রেসপন্স (বা তার প্রভাব) দেখতে পায়।

## 4. Simple Example

ধরা যাক, একটি শপিং অ্যাপ্লিকেশনে স্টক চেক করার ফিচার আছে। স্টক চেক করার জন্য অ্যাপ্লিকেশনটি একটি ব্যাক-এন্ড REST API-তে রিকোয়েস্ট পাঠায়।

**Normal request:**

```http
stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1

```

**Modified request:**
Attacker URL-টি পরিবর্তন করে লোকাল সার্ভারের অ্যাডমিন প্যানেলের লিংক দিয়ে দেয়:

```http
stockApi=http://localhost/admin

```

**Result:**
সার্ভার `http://localhost/admin` থেকে ডেটা ফেচ করে ইউজারকে দেখায়। যেহেতু রিকোয়েস্টটি সার্ভার নিজে (লোকাল মেশিন) করেছে, তাই অ্যাডমিন প্যানেলের অ্যাক্সেস কন্ট্রোল বাইপাস হয়ে যায় এবং attacker অ্যাডমিন প্যানেলের ফুল অ্যাক্সেস পেয়ে যায়।

## 5. Technical Example

**Targeting another back-end system:**
অনেক সময় অ্যাপ্লিকেশন সার্ভার এমন কিছু ব্যাক-এন্ড সিস্টেমের সাথে কানেক্ট করতে পারে যেগুলোর Non-routable private IP address আছে (যেমন: `192.168.0.68`)। এই সিস্টেমগুলো নেটওয়ার্ক টপোলজির কারণে সুরক্ষিত থাকে বলে এদের নিজস্ব security posture অনেক দুর্বল হয় (যেমন: আনঅথেনটিকেটেড অ্যাক্সেস)।

**Request:**

```http
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://192.168.0.68/admin

```

**Explanation:**
এখানে `stockApi` প্যারামিটারটি গুরুত্বপূর্ণ। attacker এই প্যারামিটারে ইন্টারনাল IP বসিয়ে দিয়েছে। সার্ভার যখন এই IP-তে রিকোয়েস্ট করবে, তখন সে ইন্টারনাল অ্যাডমিন প্যানেলের অ্যাক্সেস পেয়ে যাবে যা বাইরের কারও পাওয়ার কথা ছিল না।

## 6. Attack Flow

Attacker
↓
Find an input field or parameter that processes a URL
↓
Modify the URL to point to an internal service (e.g., localhost/admin)
↓
Application processes the URL and issues a backend HTTP request
↓
Internal service trusts the request (because it comes from the server itself)
↓
Access control is bypassed
↓
Attacker receives sensitive internal data or performs unauthorized actions

## 7. Important Conditions / Requirements

* অ্যাপ্লিকেশনকে অবশ্যই ইউজার-প্রোভাইডেড URL বা ডেটা ব্যবহার করে ব্যাক-এন্ড থেকে HTTP রিকোয়েস্ট করার ক্ষমতাসম্পন্ন হতে হবে।
* সার্ভার এবং ইন্টারনাল ব্যাক-এন্ড সিস্টেমের মধ্যে একটি Trust relationship থাকতে হবে (যেমন লোকাল রিকোয়েস্টকে ট্রাস্ট করা)।

## 8. Types / Variations

### A. SSRF against the server itself

**What is it?** Attacker সার্ভারকে তার নিজের লুপব্যাক নেটওয়ার্ক ইন্টারফেস (loopback network interface)-এ রিকোয়েস্ট করতে বাধ্য করে।
**How does it work?** `127.0.0.1` বা `localhost` ব্যবহার করে রিকোয়েস্ট পাঠানো হয়।
**Impact:** লোকাল অ্যাডমিন প্যানেল বা ইন্টারনাল সার্ভিসের অ্যাক্সেস পাওয়া যায়।

### B. SSRF against other back-end systems

**What is it?** Attacker সার্ভারকে ইন্টারনাল নেটওয়ার্কের অন্য কোনো মেশিনে রিকোয়েস্ট করতে বাধ্য করে।
**How does it work?** প্রাইভেট IP অ্যাড্রেস (যেমন `192.168.0.x`) ব্যবহার করে রিকোয়েস্ট পাঠানো হয়।

### C. Blind SSRF

**What is it?** সার্ভার ব্যাক-এন্ড রিকোয়েস্টটি ঠিকই করে, কিন্তু তার রেসপন্স ফ্রন্ট-এন্ডে ইউজারের কাছে ফিরে আসে না।
**How does it work?** এটি ডিটেক্ট করতে Out-of-band (OAST) টেকনিক বা Burp Collaborator ব্যবহার করতে হয়।
**Impact:** সরাসরি ডেটা রিড করা যায় না, তবে ইন্টারনাল নেটওয়ার্ক স্ক্যান করে অন্যান্য ভালনারেবিলিটি (যেমন Shellshock) এক্সিকিউট করে Remote Code Execution (RCE) করা সম্ভব।

### D. Routing-based SSRF (Hidden Attack Surface)

**What is it?** Reverse proxy, Load balancer বা Analytics system-কে ম্যানিপুলেট করে রিকোয়েস্ট মিসরাউট (misroute) করানো।
**How does it work?** ইনভ্যালিড `Host` হেডার বা `Referer` হেডার ব্যবহার করে এই সিস্টেমগুলোকে অন্য কোনো ইন্টারনাল সিস্টেমে রিকোয়েস্ট পাঠাতে বাধ্য করা হয়।

## 9. Different Contexts

SSRF শুধু সাধারণ URL প্যারামিটারেই সীমাবদ্ধ নয়। এটি বিভিন্ন কনটেক্সটে হতে পারে:

* **Full URLs in requests:** যেমন `stockApi=http://...`
* **Partial URLs in requests:** যখন শুধু Hostname বা path ইনপুট হিসেবে নেওয়া হয়।
* **URLs within data formats:** XML ডেটা ফরম্যাট পার্স করার সময় (XXE injection থেকে SSRF)।
* **HTTP Headers:** `Referer` হেডার (Analytics software এটি পার্স করে রিকোয়েস্ট পাঠায়), `X-Forwarded-For`, বা `X-Wap-Profile` হেডার।

## 10. Exploitation / Practical Understanding

অ্যাটাকাররা সাধারণত বিভিন্ন ফিল্টার বাইপাস করার জন্য নিচের টেকনিকগুলো ব্যবহার করে:

**1. Circumventing Blacklist-based filters:**
যদি অ্যাপ্লিকেশন `127.0.0.1` বা `/admin` ব্লক করে, তবে:

* **Alternative IP representation:** `2130706433`, `017700000001`, বা `127.1` ব্যবহার করা।
* **Spoofed domain:** এমন ডোমেইন ব্যবহার করা যা `127.0.0.1` তে রিজলভ করে (যেমন `spoofed.burpcollaborator.net`)।
* **Obfuscation:** URL encoding বা Case variation ব্যবহার করা।
* **Redirects:** Attacker-এর নিজের কন্ট্রোলে থাকা একটি URL দেওয়া, যা পরে ইন্টারনাল URL-এ রিডাইরেক্ট করে। (অনেক সময় `http:` থেকে `https:` এ রিডাইরেক্ট করলে ফিল্টার বাইপাস হয়)।

**2. Circumventing Whitelist-based filters:**
যদি অ্যাপ্লিকেশন শুধু নির্দিষ্ট ডোমেইন (যেমন `expected-host`) অ্যালাউ করে, তবে URL পার্সিং ইনকনসিস্টেন্সি ব্যবহার করা যায়:

* Credentials syntax: `https://expected-host:fakepassword@evil-host`
* URL fragments: `https://evil-host#expected-host`
* DNS hierarchy: `[https://expected-host.evil-host](https://expected-host.evil-host)`
* Double URL encoding ব্যবহার করে পার্সারকে কনফিউজ করা।
* ওপরের সবগুলোর কম্বিনেশন ব্যবহার করা।

**3. Bypassing filters via Open Redirection:**
যদি ভ্যালিডেটেড বা ট্রাস্টেড ডোমেইনে একটি Open redirect ভালনারেবিলিটি থাকে, তবে সেটি ব্যবহার করে ইন্টারনাল নেটওয়ার্কে রিকোয়েস্ট রিডাইরেক্ট করা যায়।
*Example:* `path=[http://evil-user.net](http://evil-user.net)` এর বদলে `path=[http://192.168.0.68/admin](http://192.168.0.68/admin)` দেওয়া।

## 11. Step-by-Step Exploitation

**Exploiting Basic SSRF:**

* Step 1 — Identify the input: Burp Suite দিয়ে রিকোয়েস্টগুলো চেক করুন এবং দেখুন কোনো প্যারামিটারে URL বা ডোমেইন যাচ্ছে কি না।
* Step 2 — Test the behavior: সেই URL পরিবর্তন করে নিজের Burp Collaborator-এর লিংক দিন।
* Step 3 — Confirm the vulnerability: যদি Collaborator-এ HTTP বা DNS রিকোয়েস্ট আসে, তার মানে সার্ভার রিকোয়েস্ট পাঠাচ্ছে।
* Step 4 — Determine required conditions: এবার `[http://127.0.0.1/](http://127.0.0.1/)` বা `http://localhost/` দিয়ে টেস্ট করুন রেসপন্সে কী আসে।
* Step 5 — Exploit the vulnerability: যদি ব্লক হয়, তবে Blacklist/Whitelist বাইপাস টেকনিকগুলো (যেমন `127.1`) প্রয়োগ করুন।
* Step 6 — Verify the result: ইন্টারনাল অ্যাডমিন প্যানেল বা সেনসিটিভ ডেটা রেসপন্সে দেখা গেলে অ্যাটাক সফল।

## 12. HTTP Requests / Responses

**Routing-based SSRF via Invalid Host Header:**

```http
GET / HTTP/1.1
Host: uniqid.burpcollaborator.net
Connection: close    

```

*Explanation:* এখানে `Host` হেডারে Collaborator-এর লিংক দেওয়া হয়েছে। কিছু দুর্বল Reverse proxy এই হেডার দেখে রিকোয়েস্টটি ওই ডোমেইনে রাউট করে দেয়, যার ফলে একটি শক্তিশালী SSRF অ্যাটাক হয়।

**Apache HttpComponents URL Rewriting Flaw:**

```http
GET @burp-collaborator.net/ HTTP/1.1
Host: newrelic.com
Connection: close    

```

*Explanation:* Apache HttpComponents লাইব্রেরি রিকোয়েস্টের পাথ `/` দিয়ে শুরু হওয়া বাধ্যতামূলক করেনি। ফলে এই ইনপুটটি প্রসেস হয়ে `http://public-backend@burp-collaborator.net/` হয়ে যায় এবং রিকোয়েস্টটি Collaborator-এ চলে আসে।

## 13. Payloads

* **Alternative Localhost IPs:** `[http://127.1/](http://127.1/)`, `[http://2130706433/](http://2130706433/)`, `[http://017700000001/](http://017700000001/)`
* *Purpose:* Blacklist বাইপাস করে লোকালহোস্ট অ্যাক্সেস করা।


* **Whitelist Bypass Combinations:** `https://expected-host@evil-host` বা `https://evil-host#expected-host`
* *Purpose:* URL পার্সারকে বোকা বানিয়ে ম্যালিসিয়াস ডোমেইনে রিকোয়েস্ট পাঠানো।


* **Open Redirect Payload:** `stockApi=[http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin](http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin)`
* *Purpose:* ট্রাস্টেড ডোমেইনের রিডাইরেক্ট ফিচার ব্যবহার করে ইন্টারনাল সার্ভিসে ঢোকা।



## 14. Burp Suite Workflow

* **Burp Collaborator:** Blind SSRF ডিটেক্ট করার জন্য সবচেয়ে কার্যকর। এটি ইউনিক ডোমেইন জেনারেট করে দেয়। যদি টার্গেট সার্ভার থেকে ওই ডোমেইনে DNS বা HTTP রিকোয়েস্ট আসে, তবে Collaborator তা লগ করে দেখায়।
* **Collaborator Everywhere (Extension):** এটি অটোমেটিকভাবে ব্রাউজারের সব ট্রাফিকে (যেমন `Referer`, `X-Forwarded-For` হেডারে) Collaborator পেলোড ইনজেক্ট করে ব্যাক-এন্ড সিস্টেম (Analytics, Caches) ডিটেক্ট করতে সাহায্য করে।
* **Intruder:** ইন্টারনাল নেটওয়ার্কের IP অ্যাড্রেস স্পেস (যেমন `192.168.0.1` থেকে `192.168.0.255`) ব্রুট-ফোর্স বা সুইপ (sweep) করার জন্য ব্যবহৃত হয়।

## 15. How to Identify / Detect

* রিকোয়েস্টে URL বা আংশিক URL থাকলে তা পরিবর্তন করে Burp Collaborator-এর লিংক দিন।
* DNS বা HTTP ইন্টারঅ্যাকশন পেলে নিশ্চিত হওয়া যায় যে SSRF আছে।
* Blind SSRF-এর ক্ষেত্রে রেসপন্সে কিছু দেখা যায় না, তাই সম্পূর্ণভাবে OAST (Out-of-band) ডিটেকশনের ওপর নির্ভর করতে হয়।

## 16. Common Mistakes

* **Assuming no HTTP response means no SSRF:** অনেক সময় Collaborator-এ শুধু DNS লুকআপ আসে, কিন্তু HTTP রিকোয়েস্ট আসে না। বিগিনাররা মনে করে SSRF নেই। কিন্তু আসলে নেটওয়ার্ক-লেভেল ফিল্টারিংয়ের কারণে আউটবাউন্ড HTTP কানেকশন ব্লক থাকতে পারে, যদিও DNS অ্যালাউড। এটিও এক ধরনের Blind SSRF।
* **Relying only on `127.0.0.1`:** শুধু `127.0.0.1` ট্রাই করে ছেড়ে দেওয়া। Blacklist থাকলে Alternative IP বা এনকোডিং ট্রাই না করা বড় ভুল।

## 17. Limitations

* **Blind SSRF Limitations:** যেহেতু রেসপন্স দেখা যায় না, তাই সরাসরি ডেটা চুরি করা যায় না। তবে রিমোট কোড এক্সিকিউশন (যেমন Shellshock) করা যেতে পারে।
* **Network-level Filtering:** অনেক সময় ইন্টারনাল সার্ভার থেকে এক্সটারনাল ইন্টারনেটে HTTP রিকোয়েস্ট ব্লক করা থাকে।
* **Partial Control:** যদি শুধু ডোমেইন নেম কন্ট্রোল করা যায় কিন্তু পাথ কন্ট্রোল করা না যায়, তবে ফুল SSRF এক্সিকিউট করা কঠিন হতে পারে।

## 18. Edge Cases / Important Details

* **Pre-emptive caching (XSS to SSRF):** কিছু রিভার্স প্রক্সি বা ক্যাশ সার্ভার রেসপন্সের ভেতরের `<img>` ট্যাগ স্ক্যান করে রিসোর্স ফেচ করে। Attacker যদি XSS-এর মাধ্যমে `<img src="[http://internal-server.mil/fake.jpg](http://internal-server.mil/fake.jpg)"/>` ইনজেক্ট করে, তবে ক্যাশ সার্ভার সেই ইন্টারনাল ফাইলটি ফেচ করে ক্যাশে সেভ করে রাখবে, যা পরে Attacker দেখতে পাবে।
* **Referer Header Analytics:** অনেক ব্যাক-এন্ড অ্যানালিটিক্স সফটওয়্যার `Referer` হেডারের লিংকগুলোতে ভিজিট করে। এটি Blind SSRF-এর একটি চমৎকার অ্যাটাক সারফেস।

## 19. Database / Platform / Technology Differences

| Technology | Important Difference |
| --- | --- |
| **Apache HttpComponents** | এটি URL-এর পাথ `/` দিয়ে শুরু হওয়ার বাধ্যবাধকতা প্রয়োগ করেনি। ফলে `GET @burp-collaborator.net/` এর মতো পেলোড কাজ করে যায়। |
| **Internet Explorer/Edge** | কিছু নির্দিষ্ট ক্যারেক্টারকে ভিন্নভাবে পার্স করতে পারে, তবে SSRF-এর ক্ষেত্রে মূলত ব্যাক-এন্ড সার্ভারের URL পার্সিং লাইব্রেরির ওপর সবকিছু নির্ভর করে। |

## 20. Impact

একটি সফল SSRF অ্যাটাকের ফলে নিচের ঘটনাগুলো ঘটতে পারে:

* **Unauthorized Access:** ইন্টারনাল বা ব্যাক-এন্ড সিস্টেমে আনঅথরাইজড অ্যাক্সেস।
* **Data Leakage:** সেনসিটিভ ডেটা, ফাইল বা ক্লাউড ক্রেডেনশিয়াল লিক হওয়া।
* **Remote Command Execution (RCE):** ইন্টারনাল সার্ভারে থাকা অন্য কোনো ভালনারেবিলিটি (যেমন Shellshock) এক্সিকিউট করে সার্ভারের সম্পূর্ণ কন্ট্রোল নেওয়া।
* **Malicious Onward Attacks:** ভালনারেবল অ্যাপ্লিকেশনকে প্রক্সি হিসেবে ব্যবহার করে থার্ড-পার্টি সিস্টেমে অ্যাটাক করা, যা দেখলে মনে হবে অ্যাটাকটি টার্গেট অর্গানাইজেশন থেকেই এসেছে।

## 21. Prevention / Mitigation

SSRF প্রতিরোধের মূল আইডিয়া হলো সার্ভার-সাইড থেকে জেনারেট হওয়া রিকোয়েস্টগুলোকে কঠোরভাবে নিয়ন্ত্রণ করা।

* **Input Validation:** কখনোই ইউজারের দেওয়া ইনপুট (URL) সরাসরি রিকোয়েস্টে ব্যবহার করা উচিত নয়।
* **Strict Whitelisting:** শুধুমাত্র নির্দিষ্ট এবং ট্রাস্টেড ডোমেইনের একটি কড়াকড়ি হোয়াইটলিস্ট (Whitelist) মেইনটেইন করা।
* **Robust URL Parsing:** হোয়াইটলিস্ট ইমপ্লিমেন্ট করার সময় URL পার্সিং ইনকনসিস্টেন্সি (যেমন `@`, `#` দিয়ে বাইপাস) সম্পর্কে সচেতন থাকা এবং সিকিউর পার্সার ব্যবহার করা।
* **Network-level Defenses:** সার্ভার থেকে ইন্টারনেটে অপ্রয়োজনীয় আউটবাউন্ড কানেকশন (Outbound connections) ফায়ারওয়াল দিয়ে ব্লক করে দেওয়া।
* **Fixing Open Redirects:** অ্যাপ্লিকেশনে থাকা Open Redirection ভালনারেবিলিটিগুলো ফিক্স করা, কারণ এগুলো SSRF ফিল্টার বাইপাস করতে সাহায্য করে।

## 22. Vulnerable vs Secure Example

**Vulnerable Code Concept:**

```http
POST /fetch
url=http://localhost/admin

```

*Why vulnerable?* অ্যাপ্লিকেশন ইউজার-প্রোভাইডেড URL-কে সরাসরি রিকোয়েস্ট করার জন্য ট্রাস্ট করছে।

**Secure Concept:**
*Why secure?* অ্যাপ্লিকেশন শুধু একটি আইডি (যেমন `id=1`) নেবে এবং সার্ভার-সাইডে হার্ডকোডেড ট্রাস্টেড URL-এর সাথে সেই আইডি ম্যাপ করে রিকোয়েস্ট পাঠাবে। ইউজার সরাসরি কোনো URL ইনপুট দিতে পারবে না।

## 23. Real Understanding

SSRF হলো মূলত সার্ভারকে বোকা বানিয়ে নিজের কাজ হাসিল করা। ব্রাউজার থেকে সরাসরি ইন্টারনাল সার্ভারে ঢোকা যায় না কারণ ফায়ারওয়াল ব্লক করে দেয়। কিন্তু পাবলিক ফেসিং ওয়েব সার্ভারটির সেই ইন্টারনাল সার্ভারে যাওয়ার পারমিশন আছে। অ্যাটাকার তাই পাবলিক সার্ভারটিকে একটি ম্যালিসিয়াস URL ধরিয়ে দেয়। পাবলিক সার্ভার যখন সেই URL-এ রিকোয়েস্ট করে, তখন ইন্টারনাল সিস্টেম ভাবে, "ওহ, রিকোয়েস্টটি তো আমাদের নিজস্ব সার্ভার থেকে এসেছে, তাহলে নিশ্চয়ই সে ট্রাস্টেড।" এই অন্ধ বিশ্বাসের কারণেই SSRF এত ভয়ংকর।

## 24. Mental Model

Attacker cannot reach Internal Network directly (Blocked by Firewall)
↓
Attacker finds a vulnerable parameter on the Public Web Server that takes a URL
↓
Attacker injects an Internal URL (e.g., `192.168.0.x/admin`)
↓
Public Web Server implicitly trusts the input and makes the HTTP request
↓
Internal Server receives request from Public Web Server (Trusted Source)
↓
Internal Server grants access and returns sensitive data
↓
Public Web Server passes the data back to the Attacker

## 25. Quick Revision

* **Definition:** সার্ভার-সাইড অ্যাপ্লিকেশনকে দিয়ে অনির্ধারিত স্থানে রিকোয়েস্ট করানো।
* **Cause:** ইনসিকিউর URL হ্যান্ডলিং এবং ইন্টারনাল সিস্টেমের ট্রাস্ট রিলেশনশিপ।
* **Main idea:** পাবলিক সার্ভারকে প্রক্সি হিসেবে ব্যবহার করে ইন্টারনাল নেটওয়ার্কে ঢোকা।
* **Attack flow:** Find URL input -> Inject internal IP -> Bypass filters -> Access internal data.
* **Important condition:** সার্ভারকে অবশ্যই ইউজার ইনপুট দিয়ে HTTP রিকোয়েস্ট জেনারেট করতে হবে।
* **Main techniques:** Basic SSRF (`localhost`), Blind SSRF, Whitelist/Blacklist bypass, Open Redirect bypass.
* **Detection:** OAST (Burp Collaborator) দিয়ে DNS/HTTP পিংব্যাক চেক করা।
* **Impact:** Unauthorized access, Data leakage, Remote Command Execution (RCE).
* **Prevention:** Strict Whitelisting, Robust URL validation, Network-level outbound filtering.

## 26. Things to Remember

* শুধু `127.0.0.1` ট্রাই করে ছেড়ে দেওয়া যাবে না; Alternative IP (`2130706433`, `127.1`) বা এনকোডিং ট্রাই করতে হবে।
* Blind SSRF-এ যদি শুধু DNS লুকআপ আসে কিন্তু HTTP রিকোয়েস্ট না আসে, তবে বুঝতে হবে নেটওয়ার্ক লেভেলে HTTP ব্লক করা আছে। এটিও একটি ফাইন্ডিং।
* `Referer` হেডার এবং Analytics সিস্টেমগুলো Blind SSRF-এর জন্য চমৎকার টার্গেট।
* Open Redirect থাকলে যেকোনো কড়া Whitelist ফিল্টারও বাইপাস করা সম্ভব হতে পারে।
