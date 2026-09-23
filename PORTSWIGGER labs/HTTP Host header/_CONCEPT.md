# HTTP Host Header Attacks

## 1. What is HTTP Host Header Attack?

**Short Definition:**
HTTP Host header attack হলো এমন একটি ওয়েব সিকিউরিটি ভালনারেবিলিটি যেখানে সার্ভার ইউজারের দেওয়া HTTP `Host` header-কে অন্ধভাবে বিশ্বাস করে (implicitly trusts), যার ফলে অ্যাটাকার এই হেডার ম্যানিপুলেট করে অ্যাপ্লিকেশনের বিহেভিয়ার পরিবর্তন করতে পারে।

**Detailed Explanation:**
HTTP/1.1 থেকে `Host` হেডার বাধ্যতামূলক। এটি নির্দেশ করে ক্লায়েন্ট কোন ডোমেইন বা ওয়েবসাইটে অ্যাক্সেস করতে চায়। আধুনিক ক্লাউড আর্কিটেকচার, ভার্চুয়াল হোস্টিং (Virtual hosting) এবং রিভার্স প্রক্সির কারণে একই IP অ্যাড্রেসে অনেকগুলো ওয়েবসাইট হোস্ট করা থাকে। ইন্টারমিডিয়ারি সিস্টেম (যেমন লোড ব্যালান্সার) এই `Host` হেডার দেখেই সিদ্ধান্ত নেয় রিকোয়েস্টটি কোন ব্যাক-এন্ড সার্ভারে রাউট (route) করতে হবে।
অনেক সময় অ্যাপ্লিকেশনগুলো এই হেডারকে নিরাপদ ভেবে এর ভ্যালু ব্যবহার করে বিভিন্ন কাজ করে (যেমন, ইমেইলে অ্যাবসলিউট URL তৈরি করা বা ক্যাশ কি জেনারেট করা)। অ্যাটাকার যদি এই হেডারের ভ্যালু পরিবর্তন করে নিজের কন্ট্রোল করা কোনো ডোমেইন বসিয়ে দেয়, তবে সে সার্ভারকে দিয়ে আনঅথোরাইজড কাজ করাতে পারে।

## 2. Why Does It Happen?

এর মূল কারণ হলো **flawed assumption**—ডেভেলপাররা ধরে নেন যে `Host` হেডার ইউজার কন্ট্রোল করতে পারে না। এর ফলে সার্ভার এই হেডারের ওপর implicit trust তৈরি করে এবং এর ভ্যালু ঠিকমতো validate বা escape করে না।
এছাড়া, অনেক সময় থার্ড-পার্টি টেকনোলজি বা ইন্টারমিডিয়ারি ইনফ্রাস্ট্রাকচারের insecure configuration-এর কারণেও এটি হয়। ওয়েবসাইট ওনাররা হয়তো জানেনও না যে তাদের ফ্রেমওয়ার্ক ডিফল্টভাবে `X-Forwarded-Host`-এর মতো ওভাররাইড (override) হেডার সাপোর্ট করে, যা অ্যাটাকাররা কাজে লাগায়।

## 3. How It Works

1. User (বা Attacker) একটি HTTP রিকোয়েস্ট পাঠায়।
2. Application রিকোয়েস্টটি রিসিভ করে।
3. Application সিকিউরিটি মিস্টেক করে `Host` হেডারের ভ্যালুকে আনট্রাস্টেড ইনপুট হিসেবে ভ্যালিডেট করে না।
4. Attacker রিকোয়েস্ট ইন্টারসেপ্ট করে `Host` হেডারের ভ্যালু পরিবর্তন করে (বা ওভাররাইড হেডার ইনজেক্ট করে)।
5. Server এই ম্যালিসিয়াস ভ্যালু ব্যবহার করে লজিক এক্সিকিউট করে (যেমন ম্যালিসিয়াস লিংক তৈরি করে বা রিকোয়েস্ট অন্য কোথাও রাউট করে)।
6. Security control বাইপাস হয়।
7. Attacker তার কাঙ্ক্ষিত লক্ষ্য অর্জন করে (যেমন অ্যাকাউন্ট টেকওভার বা ইন্টারনাল নেটওয়ার্ক অ্যাক্সেস)।

## 4. Simple Example (Password Reset Poisoning)

**Normal request:**
ইউজার পাসওয়ার্ড রিসেটের রিকোয়েস্ট করে। অ্যাপ্লিকেশন `Host` হেডার (`normal-website.com`) ব্যবহার করে ইমেইলে একটি লিংক পাঠায়:
`[https://normal-website.com/reset?token=0a1b2c3d](https://normal-website.com/reset?token=0a1b2c3d)`

**Modified request:**
অ্যাটাকার ভিকটিমের ইমেইল দিয়ে পাসওয়ার্ড রিসেট রিকোয়েস্ট করে এবং `Host` হেডার পরিবর্তন করে `evil-user.net` দেয়।

**Result:**
ভিকটিম সরাসরি ওয়েবসাইট থেকে একটি আসল ইমেইল পায়, কিন্তু লিংকের ডোমেইনটি অ্যাটাকারের হয়:
`[https://evil-user.net/reset?token=0a1b2c3d](https://evil-user.net/reset?token=0a1b2c3d)`
ভিকটিম লিংকে ক্লিক করলে, সিক্রেট টোকেনটি অ্যাটাকারের সার্ভারে চলে যায় এবং অ্যাটাকার সেই টোকেন দিয়ে ভিকটিমের পাসওয়ার্ড পরিবর্তন করে ফেলতে পারে।

## 5. Technical Example

**Injecting Host Override Headers:**

```http
GET /example HTTP/1.1
Host: vulnerable-website.com
X-Forwarded-Host: bad-stuff-here

```

**Explanation:**
অনেক সময় `Host` হেডার পরিবর্তন করলে রিকোয়েস্ট টার্গেটে পৌঁছায় না। এখানে মূল `Host` হেডার ঠিক রাখা হয়েছে যাতে রিকোয়েস্টটি সঠিক ব্যাক-এন্ডে পৌঁছায়। কিন্তু `X-Forwarded-Host` হেডার ইনজেক্ট করা হয়েছে। যদি ফ্রেমওয়ার্ক এই ওভাররাইড হেডারকে ট্রাস্ট করে, তবে সে `bad-stuff-here` কে আসল হোস্ট হিসেবে প্রসেস করবে এবং ভালনারেবিলিটি ট্রিগার হবে।

## 6. Attack Flow

Attacker
↓
Find functionality that uses the Host header (e.g., password reset, caching, internal routing)
↓
Intercept request and modify Host header or inject override headers
↓
Application implicitly trusts and processes the modified header
↓
Security control is bypassed
↓
Server generates a malicious link, caches a poisoned response, or misroutes the request

## 7. Important Conditions / Requirements

* `Host` হেডার ম্যানিপুলেট করে টার্গেট অ্যাপ্লিকেশনে পৌঁছানোর সুযোগ থাকতে হবে (Burp Suite ഇതിന് সহায়ক)।
* অ্যাপ্লিকেশনকে `Host` হেডারের ভ্যালু কোনো সেনসিটিভ লজিকে ব্যবহার করতে হবে।
* **Password reset poisoning:** ভিকটিমকে অবশ্যই ইমেইলে আসা ম্যালিসিয়াস লিংকে ক্লিক করতে হবে (বা অ্যান্টিভাইরাস স্ক্যানার দিয়ে ফেচ হতে হবে)।
* **Web cache poisoning:** টার্গেট সিস্টেমে ওয়েব ক্যাশ থাকতে হবে এবং ইনজেক্টেড পেলোড ক্যাশ কি (cache key) প্রিজার্ভ করে রেসপন্সে রিফ্লেক্ট হতে হবে।

## 8. Types / Variations

### A. Password Reset Poisoning

**What is it?** `Host` হেডার পরিবর্তন করে পাসওয়ার্ড রিসেট লিংক ম্যানিপুলেট করা।
**How it works:** অ্যাটাকারের দেওয়া ডোমেইন ইমেইলের লিংকে যুক্ত হয়। ভিকটিম ক্লিক করলে টোকেন চুরি হয়।

### B. Web Cache Poisoning via Host Header

**What is it?** `Host` হেডার ব্যবহার করে ক্যাশ করা রেসপন্সে ম্যালিসিয়াস পেলোড ইনজেক্ট করা।
**How it works:** `Host` হেডার যদি রেসপন্সে (যেমন স্ক্রিপ্ট ইমপোর্টে) রিফ্লেক্ট হয়, তবে অ্যাটাকার ম্যালিসিয়াস ডোমেইন দিয়ে রেসপন্স ক্যাশ করায়। এরপর অন্য ইউজাররা ওই পেজে আসলে ক্যাশ থেকে পয়জনড রেসপন্স পায়। এটি integrated application-level cache-এ ভালো কাজ করে।

### C. Routing-based SSRF (Host Header SSRF)

**What is it?** Load balancer বা reverse proxy-কে ম্যানিপুলেট করে রিকোয়েস্ট ইন্টারনাল বা আর্বিট্রারি সিস্টেমে রাউট করা।
**How it works:** ইন্টারমিডিয়ারি সিস্টেম যদি আনভ্যালিডেটেড `Host` হেডারের ওপর ভিত্তি করে রিকোয়েস্ট ফরোয়ার্ড করে, তবে অ্যাটাকার ইন্টারনাল IP (যেমন `192.168.0.0/16` রেঞ্জ) দিয়ে রিকোয়েস্ট ইন্টারনাল নেটওয়ার্কে পাঠিয়ে দিতে পারে।

### D. Host Header Authentication Bypass

**What is it?** `Host` হেডার পরিবর্তন করে ইন্টারনাল ইউজারদের জন্য রেস্ট্রিক্টেড ফাংশনালিটি অ্যাক্সেস করা।
**How it works:** কিছু সাইট `Host` হেডারের ওপর ভিত্তি করে অ্যাক্সেস কন্ট্রোল করে। `Host: localhost` দিলে হয়তো রেস্ট্রিক্টেড অ্যাডমিন প্যানেল অ্যাক্সেস করা যেতে পারে।

### E. Virtual Host Brute-forcing

**What is it?** একই সার্ভারে থাকা লুকানো ইন্টারনাল ওয়েবসাইট বা সাবডোমেইন খুঁজে বের করতে `Host` হেডার ব্রুট-ফোর্স করা।

### F. Connection State Attacks

**What is it?** একই কানেকশনে প্রথম রিকোয়েস্টটি নরমাল পাঠিয়ে ভ্যালিডেশন বাইপাস করা।
**How it works:** কিছু সার্ভার শুধু কানেকশনের প্রথম রিকোয়েস্টটি থরো (thorough) ভ্যালিডেট করে। প্রথমটি সেফ রেখে, একই কানেকশনে পরের রিকোয়েস্টটি ম্যালিসিয়াস `Host` হেডার দিয়ে পাঠালে তা বাইপাস হয়ে যায়।

## 9. Different Contexts

এই ভালনারেবিলিটি বিভিন্ন কনটেক্সটে কাজ করতে পারে:

* **HTTP Requests:** মূল `Host` হেডার।
* **Request Line:** Absolute URLs (`GET [https://vulnerable-website.com/](https://vulnerable-website.com/) HTTP/1.1`)।
* **HTTP Headers:** Override headers যেমন `X-Forwarded-Host`, `X-Host`, `X-Forwarded-Server`, `Forwarded`।
* **Intermediary Systems:** Load balancers, reverse proxies, CDNs।
* **Caching Systems:** Web caches।

## 10. Exploitation / Practical Understanding

অনেক সময় সরাসরি `Host` হেডার পরিবর্তন করলে রিকোয়েস্ট ব্লক হয়ে যায়। সেক্ষেত্রে নিচের টেকনিকগুলো ব্যবহার করতে হয়:

**1. Check for flawed validation:**

* **Port injection:** পোর্টের অংশ ভ্যালিডেট না হলে: `Host: vulnerable-website.com:bad-stuff-here`।
* **Subdomain logic flaws:** হোয়াইটলিস্ট বাইপাস করতে `notvulnerable-website.com` বা আগে থেকে হ্যাক করা সাবডোমেইন `hacked-subdomain.vulnerable-website.com` ব্যবহার করা।

**2. Send Ambiguous Requests:**
সিস্টেমের বিভিন্ন কম্পোনেন্ট (Front-end vs Back-end) `Host` হেডার আলাদাভাবে পার্স করলে discrepancy তৈরি হয়:

* **Inject duplicate Host headers:** দুটি `Host` হেডার দিন। ফ্রন্ট-এন্ড হয়তো প্রথমটি নেবে (টার্গেটে পৌঁছাবে), ব্যাক-এন্ড দ্বিতীয়টি নেবে (পেলোড এক্সিকিউট হবে)।
* **Supply an absolute URL:** রিকোয়েস্ট লাইনে ফুল URL এবং `Host` হেডারে পেলোড দিন। কোন সিস্টেম কোনটিকে প্রায়োরিটি দেয় তার ওপর ভিত্তি করে বাইপাস হতে পারে।
* **Add line wrapping:** স্পেস দিয়ে ইনডেন্ট (indent) করলে কোনো সার্ভার একে আগের হেডারের অংশ ভাবে (wrapped line), অন্য সার্ভার ইগনোর করে।

## 11. Step-by-Step Exploitation (Ambiguous Request - Duplicate Headers)

* **Step 1 — Identify the input:** Burp Suite ইন্টারসেপ্টর দিয়ে রিকোয়েস্ট ধরুন।
* **Step 2 — Test the behavior:** রিকোয়েস্টে একটি অতিরিক্ত (Duplicate) `Host` হেডার যোগ করুন।
* **Step 3 — Confirm the vulnerability:** দেখুন রিকোয়েস্ট ব্লক হচ্ছে নাকি কোনো একটি হেডারকে প্রায়োরিটি দিয়ে প্রসেস হচ্ছে।
* **Step 4 — Determine required conditions:** আইডেন্টিফাই করুন ফ্রন্ট-এন্ড প্রথমটিকে প্রায়োরিটি দেয় নাকি দ্বিতীয়টিকে।
* **Step 5 — Exploit the vulnerability:** প্রথম `Host` হেডারে আসল ডোমেইন দিন (যাতে রিকোয়েস্ট টার্গেটে পৌঁছায়) এবং দ্বিতীয়টিতে ম্যালিসিয়াস পেলোড দিন।
* **Step 6 — Verify the result:** রেসপন্সে দেখুন আপনার ম্যালিসিয়াস ভ্যালু রিফ্লেক্ট হয়েছে কি না বা লজিক চেঞ্জ হয়েছে কি না।

## 12. HTTP Requests / Responses

**Line Wrapping Discrepancy Example:**

```http
GET /example HTTP/1.1
    Host: bad-stuff-here
Host: vulnerable-website.com

```

*Explanation:* এখানে ইনডেন্টেশন (স্পেস) ব্যবহার করে Line wrapping করা হয়েছে। ফ্রন্ট-এন্ড ইনডেন্ট করা হেডারটিকে ইগনোর করে `vulnerable-website.com`-এ রিকোয়েস্ট পাঠাতে পারে। কিন্তু ব্যাক-এন্ড প্রথম হেডারটিকে প্রায়োরিটি দিয়ে `bad-stuff-here` কে প্রসেস করতে পারে, যা একটি discrepancy তৈরি করে।

## 13. Payloads

* **Port Injection:** `Host: vulnerable-website.com:bad-stuff-here`
*Purpose:* ডোমেইন ঠিক রেখে পোর্টে পেলোড ইনজেক্ট করে ভ্যালিডেশন বাইপাস করা।
* **Override Headers:** `X-Forwarded-Host: evil-user.net`
*Purpose:* মূল `Host` হেডার ঠিক রেখে ফ্রেমওয়ার্কের ডিফল্ট ট্রাস্টকে কাজে লাগানো।
* **Malformed request line:** `GET @private-intranet/example HTTP/1.1`
*Purpose:* রিভার্স প্রক্সি একে `http://backend-server@private-intranet/example` বানিয়ে ফেলতে পারে, যা ইন্টারনাল সিস্টেমে ক্রেডেনশিয়াল হিসেবে পার্স হয়।

## 14. Burp Suite Workflow

* **Proxy / Repeater:** `Host` হেডার মডিফাই করতে এবং রিকোয়েস্ট টার্গেটে পৌঁছাচ্ছে কি না তা টেস্ট করতে। (Burp Suite `Host` হেডার এবং টার্গেট IP আলাদা রাখে, যা অন্যান্য টুলে সম্ভব হয় না)।
* **Intruder:** Virtual host ব্রুট-ফোর্স করার জন্য বা প্রাইভেট IP (যেমন `192.168.0.0/16` CIDR রেঞ্জ) স্ক্যান করার জন্য।
* **Collaborator:** Routing-based SSRF আইডেন্টিফাই করার জন্য। `Host` হেডারে Collaborator ডোমেইন দিলে যদি টার্গেট থেকে DNS লুকআপ আসে, তবে বোঝা যায় সার্ভার আর্বিট্রারি ডোমেইনে রিকোয়েস্ট রাউট করছে।

## 15. How to Identify / Detect

* Arbitrary `Host` হেডার দিয়ে দেখুন অ্যাপ্লিকেশন ডিফল্ট ফলব্যাক হিসেবে রেসপন্স করে কি না।
* `Host` হেডারের ভ্যালু রেসপন্সে HTML-এনকোডিং ছাড়া রিফ্লেক্ট হচ্ছে কি না বা স্ক্রিপ্ট ইমপোর্টে ব্যবহার হচ্ছে কি না তা চেক করা।
* পাসওয়ার্ড রিসেট রিকোয়েস্ট করে ইমেইলে আসা লিংক চেক করা।
* Burp Collaborator ডোমেইন দিয়ে DNS ইন্টারঅ্যাকশন চেক করা।

## 16. Common Mistakes

* Browser দিয়ে `Host` হেডার ম্যানিপুলেট করার চেষ্টা করা (এটি intercepting proxy ছাড়া সম্ভব নয়)।
* `Host` হেডার রিফ্লেকশন পেলেই তাকে সরাসরি XSS ভেবে এক্সপ্লয়েট করার চেষ্টা করা। (Host header দিয়ে ক্লায়েন্ট-সাইড XSS সাধারণত সরাসরি এক্সপ্লয়েটেবল হয় না, ওয়েব ক্যাশ পয়জনিং করতে হয়)।
* থার্ড-পার্টি ফ্রেমওয়ার্কের ডিফল্ট `X-Forwarded-Host` সাপোর্ট সম্পর্কে না জানা।

## 17. Limitations

* `Host` হেডার পরিবর্তন করলে অনেক সময় রিকোয়েস্ট টার্গেট অ্যাপ্লিকেশনে পৌঁছায় না (বিশেষ করে CDN থাকলে "Invalid Host header" এরর আসে)।
* Standalone cache-এর ক্ষেত্রে `Host` হেডার সাধারণত cache key-এর অংশ থাকে, তাই ক্যাশ পয়জনিং মূলত integrated/application-level cache-এ ভালো কাজ করে।
* Password reset poisoning-এ ভিকটিমকে অবশ্যই ম্যালিসিয়াস লিংকে ক্লিক করতে হবে।

## 18. Edge Cases / Important Details

* **SNI Validation:** কিছু সাইট TLS হ্যান্ডশেক-এর SNI-এর সাথে `Host` হেডার ম্যাচ করে, তবে পোর্ট চেক না করলে তা বাইপাস করা যেতে পারে।
* **Connection State:** প্রথম রিকোয়েস্টে থরো ভ্যালিডেশন হয়, একই কানেকশনের পরের রিকোয়েস্টগুলোতে হয় না।
* **Dangling Markup:** পাসওয়ার্ড রিসেট লিংক কন্ট্রোল করতে না পারলেও `Host` হেডার দিয়ে ইমেইলে HTML ইনজেক্ট করে dangling markup অ্যাটাক করা যেতে পারে (যদিও ইমেইল ক্লায়েন্ট JS রান করে না)।

## 19. Database / Platform / Technology Differences

| Technology | Important Difference |
| --- | --- |
| **Django Framework** | `ALLOWED_HOSTS` অপশন দিয়ে `Host` হেডার ভ্যালিডেট করার বিল্ট-ইন মেকানিজম আছে। |
| **Reverse Proxies / Front-ends** | Duplicate header, absolute URLs, বা line wrapping-এর ক্ষেত্রে বিভিন্ন সিস্টেম আলাদা আলাদা আচরণ করে (precedence differences), যা ambiguous request অ্যাটাক সম্ভব করে। |

## 20. Impact

Vulnerability সফলভাবে এক্সপ্লয়েট হলে নিচের ঘটনাগুলো ঘটতে পারে:

* **Account Takeover:** Password reset poisoning-এর মাধ্যমে।
* **Serving malicious content:** Web cache poisoning-এর মাধ্যমে অন্য ইউজারদের অ্যাটাক করা।
* **Gateway to internal network:** Routing-based SSRF দিয়ে ইন্টারনাল সিস্টেম অ্যাক্সেস করা।
* **Authentication bypass:** রেস্ট্রিক্টেড ইন্টারনাল ফাংশনালিটি অ্যাক্সেস করা।
* **Information disclosure:** ভার্চুয়াল হোস্ট ব্রুট-ফোর্স করে।

## 21. Prevention / Mitigation

এই ভালনারেবিলিটি প্রতিরোধের মূল আইডিয়া হলো `Host` হেডারের ওপর নির্ভরতা কমানো এবং প্রপার ভ্যালিডেশন করা:

* **Avoid Host header:** সার্ভার-সাইড কোডে `Host` হেডার ব্যবহার করা এড়িয়ে চলুন। Absolute URL-এর বদলে Relative URL ব্যবহার করুন।
* **Protect absolute URLs:** Absolute URL দরকার হলে কনফিগারেশন ফাইলে ম্যানুয়ালি ডোমেইন স্পেসিফাই করে দিন এবং সেই ভ্যালু ব্যবহার করুন।
* **Validate the Host header:** Permitted domain-এর একটি whitelist দিয়ে `Host` হেডার ভ্যালিডেট করুন। আনরিকগনাইজড হোস্ট রিজেক্ট করুন।
* **Don't support override headers:** `X-Forwarded-Host`-এর মতো হেডারগুলো ডিজঅ্যাবল করে রাখুন।
* **Whitelist for Routing:** লোড ব্যালান্সার বা রিভার্স প্রক্সিকে শুধু নির্দিষ্ট হোয়াইটলিস্টেড ডোমেইনেই রিকোয়েস্ট ফরোয়ার্ড করার জন্য কনফিগার করুন।
* **Separate Virtual Hosts:** পাবলিক এবং ইন্টারনাল-ওনলি ওয়েবসাইট একই সার্ভারে হোস্ট করবেন না।

## 22. Vulnerable vs Secure Example

**Vulnerable:**

```html
<a href="https://_SERVER['HOST']/support">Contact support</a>

```

*Why vulnerable?* এটি `Host` হেডার থেকে সরাসরি ডোমেইন নিয়ে লিংক জেনারেট করছে।

**Secure:**

```html
<a href="/support">Contact support</a>

```

*Why secure?* এটি Relative URL ব্যবহার করছে, তাই `Host` হেডারের ওপর কোনো নির্ভরতা নেই।

## 23. Real Understanding

HTTP Host header attacks ঘটে কারণ ওয়েব ইনফ্রাস্ট্রাকচার (ফ্রন্ট-এন্ড, প্রক্সি) এবং ব্যাক-এন্ড অ্যাপ্লিকেশন একই ডেটাকে (`Host` হেডার) ভিন্নভাবে ট্রাস্ট করে বা পার্স করে। অ্যাটাকার যখন এই হেডার ম্যানিপুলেট করে, সে আসলে সার্ভারকে ধোঁকা দিয়ে ভুল ডোমেইনের জন্য লজিক এক্সিকিউট করায়। এর ফলে সার্ভার হয় ভুল লিংকে ইমেইল পাঠায়, ভুল ডেটা ক্যাশ করে, অথবা ইন্টারনাল নেটওয়ার্কে রিকোয়েস্ট রাউট করে দেয়। এটি মূলত ইনফ্রাস্ট্রাকচার এবং অ্যাপ্লিকেশনের মধ্যকার miscommunication এবং implicit trust-এর ফল।

## 24. Mental Model

Client sends HTTP Request with Host Header
↓
Intermediaries (Load Balancers/Proxies) route based on Host Header
↓
Backend Application relies on Host Header for logic (Links, Cache, Routing)
↓
Attacker modifies Host Header or uses Overrides (Ambiguous requests)
↓
Application blindly trusts modified Header
↓
Vulnerability executed (Poisoned Reset, SSRF, Cache Poison)

## 25. Quick Revision

* **Definition:** `Host` হেডার ম্যানিপুলেট করে সার্ভার-সাইড লজিক এক্সপ্লয়েট করা।
* **Cause:** `Host` হেডারকে আনট্রাস্টেড ইনপুট হিসেবে বিবেচনা না করা (Implicit trust)।
* **Main idea:** সার্ভারকে দিয়ে ভুল ডোমেইনে লিংক জেনারেট করানো বা রিকোয়েস্ট রাউট করানো।
* **Attack flow:** Modify Host header -> Bypass Front-end -> Backend processes malicious Host.
* **Important condition:** অ্যাপ্লিকেশনকে `Host` হেডারের ভ্যালু কোনো লজিক বা আউটপুটে ব্যবহার করতে হবে।
* **Main techniques:** Duplicate headers, Absolute URLs, X-Forwarded-Host, Non-numeric ports, Line wrapping.
* **Detection:** Arbitrary `Host` হেডার বা `X-Forwarded-Host` ইনজেক্ট করে রেসপন্স ও বিহেভিয়ার চেক করা।
* **Impact:** Account Takeover, Routing SSRF, Cache Poisoning.
* **Prevention:** Relative URLs ব্যবহার করা, Whitelisting, Override headers ব্লক করা।

## 26. Things to Remember

* `Host` হেডার অ্যাটাক দিয়ে সরাসরি XSS এক্সপ্লয়েট করা যায় না, এর জন্য Web Cache Poisoning কাজে লাগাতে হয়।
* Burp Suite `Host` হেডার এবং টার্গেট IP আলাদা রাখে, যা এই অ্যাটাক টেস্ট করার জন্য অত্যন্ত জরুরি।
* SSRF-এর ক্ষেত্রে `Host` হেডার ম্যানিপুলেট করে ইন্টারনাল নেটওয়ার্ক (`192.168.0.0/16` CIDR) স্ক্যান করা যায়।
* Password reset poisoning-এ টার্গেট ভিকটিমকে অবশ্যই ইমেইলের ম্যালিসিয়াস লিংকে ক্লিক করতে হবে।
* অনেক থার্ড-পার্টি ফ্রেমওয়ার্ক ডিফল্টভাবেই `X-Forwarded-Host` সাপোর্ট করে, যা ওনাররা জানেনও না।

