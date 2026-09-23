# Authentication


# Authentication Vulnerabilities

## 1. What is Authentication?

**Short Definition:**
Authentication হলো কোনো user বা client-এর আসল পরিচয় (identity) যাচাই করার একটি প্রক্রিয়া।

**Detailed Explanation:**
Websites সাধারণত ইন্টারনেটে সবার জন্য উন্মুক্ত থাকে। তাই কে আসল user তা নির্ধারণ করতে robust authentication mechanism দরকার। Authentication মূলত তিনটি factor-এর ওপর ভিত্তি করে কাজ করে:

1. **Knowledge factors (Something you know):** যা আপনি জানেন, যেমন—password বা security question-এর উত্তর।
2. **Possession factors (Something you have):** যা আপনার কাছে আছে, যেমন—mobile phone বা security token।
3. **Inherence factors (Something you are):** আপনার শারীরিক বৈশিষ্ট্য বা আচরণ, যেমন—biometrics।

**Authentication vs Authorization:**

* **Authentication** নিশ্চিত করে যে user আসলেই সে কি না (Identity verification)। যেমন: `Carlos123` নামের user-ই লগইন করছে কি না।
* **Authorization** নিশ্চিত করে যে লগইন করা user-এর কোনো নির্দিষ্ট কাজ করার অনুমতি আছে কি না (Permission verification)। যেমন: `Carlos123` অন্যের account ডিলিট করতে পারবে কি না।

## 2. Why Does It Happen?

Authentication vulnerabilities সাধারণত দুটি কারণে ঘটে:

1. **Weak Mechanisms:** Authentication ব্যবস্থা দুর্বল হওয়ার কারণে এটি brute-force attack প্রতিরোধ করতে পারে না।
2. **Logic Flaws / Poor Coding (Broken Authentication):** Implementation-এ logic flaw থাকলে attacker পুরোপুরি authentication mechanism বাইপাস করে ফেলতে পারে।

যেহেতু web security-তে authentication অত্যন্ত গুরুত্বপূর্ণ, তাই এখানে কোনো logic flaw থাকলে তা নিশ্চিতভাবেই বড় security issue তৈরি করে।

## 3. How It Works

1. User (বা attacker) application-এ login/authentication request পাঠায়।
2. Application সেই request এবং credentials প্রসেস করে।
3. Application-এর implementation-এ logic flaw বা brute-force protection-এ দুর্বলতা থাকে।
4. Attacker বিভিন্ন technique (যেমন: timing analysis, parameter manipulation, brute-forcing) ব্যবহার করে সেই দুর্বলতার সুযোগ নেয়।
5. Security control bypass হয়ে যায়।
6. Attacker অন্য user-এর account বা sensitive functionality-তে access পেয়ে যায়।

## 4. Simple Example

**Username Enumeration (Error Message):**

*Normal request (Valid user, Wrong password):*
Response: `Incorrect password.`

*Modified request (Invalid user, Wrong password):*
Response: `Invalid username.`

**Result:**
Attacker response-এর এই ছোট পার্থক্য দেখে সহজেই বুঝে যায় যে সিস্টেমে কোন username-টি আসল এবং কোনটি নয়। এর ফলে brute-force করা অনেক সহজ হয়ে যায়।

## 5. Technical Example

**Flawed 2FA Verification Logic:**

ধরা যাক, একটি website-এর 2FA সিস্টেমে প্রথম ধাপে username/password দেওয়ার পর server একটি cookie সেট করে।

```http
HTTP/1.1 200 OK
Set-Cookie: account=carlos

```

এরপর দ্বিতীয় ধাপে (2FA code submit করার সময়) request-টি যায় এভাবে:

```http
POST /login-steps/second HTTP/1.1
Host: vulnerable-website.com
Cookie: account=carlos

verification-code=123456

```

**Explanation:**
এখানে important অংশ হলো `Cookie: account=carlos`। Application এই cookie দেখেই সিদ্ধান্ত নিচ্ছে যে কোন account-এর 2FA verify করা হচ্ছে। Attacker চাইলে নিজের valid credentials দিয়ে প্রথম ধাপ পার হয়ে, দ্বিতীয় ধাপে request intercept করে cookie-এর value `account=victim-user` করে দিতে পারে। এরপর attacker যদি victim-এর 6-digit code brute-force করে বের করতে পারে, তবে সে victim-এর password না জেনেই account-এ লগইন করতে পারবে।

## 6. Attack Flow

**2FA Bypass Logic Flaw Attack Flow:**

Attacker
↓
Log in with own valid credentials (Step 1)
↓
Application sets an account identification cookie
↓
Intercept the request for Step 2 (2FA code verification)
↓
Modify the account cookie to target a victim user
↓
Use Burp Intruder to brute-force the 6-digit verification code
↓
Application processes the valid code for the victim's account
↓
Logged in as the victim

## 7. Important Conditions / Requirements

* Brute-force সফল হওয়ার জন্য application-এ strict rate-limiting বা IP blocking না থাকা।
* Enumeration-এর জন্য response (status code, error message, time)-এ পার্থক্য থাকা।
* 2FA বাইপাসের জন্য multi-step process-এর মধ্যে state/session ঠিকমতো validate না করা।
* Password reset-এর ক্ষেত্রে URL parameter বা token user-controllable হওয়া।

## 8. Types / Variations

### A. Password-Based Login Vulnerabilities

**What is it?** Password-এর ওপর নির্ভরশীল সিস্টেমে brute-force বা credential guessing করে এক্সেস নেওয়া।
**Variations:**

* **Brute-forcing Usernames:** Predictable pattern (যেমন: `firstname.lastname@company.com`) বা `admin` ব্যবহার করা। Public profile বা HTTP response থেকে email/username leak হওয়া।
* **Brute-forcing Passwords:** Password policy বাইপাস করে user-রা সাধারণত পরিচিত শব্দের সাথে সংখ্যা বা স্পেশাল ক্যারেক্টার যুক্ত করে (e.g., `Mypassword1!`)।
* **Username Enumeration:** Status codes, Error messages, বা Response times-এর পার্থক্য দেখে valid username খুঁজে বের করা।
* **Credential Stuffing:** অন্য কোনো breach থেকে পাওয়া username:password pair ব্যবহার করে লগইন করার চেষ্টা করা।

### B. Flawed Brute-Force Protection

**What is it?** Application-এ protection থাকলেও logic flaw-এর কারণে তা বাইপাস করা যায়।
**Variations:**

* **IP Block Bypass:** অনেক সময় successful login হলে failed attempt-এর কাউন্টার reset হয়ে যায়। Attacker নিজের account-এ লগইন করে কাউন্টার reset করে ব্লক হওয়া থেকে বাঁচতে পারে।
* **Account Lock Bypass:** Account lock নির্দিষ্ট account-কে বাঁচায়, কিন্তু attacker যদি অনেকগুলো valid username-এর বিপরীতে মাত্র ৩টি কমন password (লক লিমিট ৩ হলে) ট্রাই করে, তবে কোনো account লক হবে না, কিন্তু attack সফল হতে পারে।
* **Multiple Credentials per Request:** Rate limit IP-এর ওপর ভিত্তি করে হলে, একটি request-এর ভেতরেই array হিসেবে অনেকগুলো password পাঠিয়ে limit বাইপাস করা।

### C. HTTP Basic Authentication

**What is it?** Browser নিজে থেকে `Authorization: Basic base64(username:password)` header পাঠিয়ে authenticate করে।
**Why vulnerable?** HSTS না থাকলে intercept হতে পারে। কোনো brute-force protection সাপোর্ট করে না। CSRF-এর বিরুদ্ধে কোনো প্রোটেকশন দেয় না।

### D. Multi-Factor Authentication (2FA) Vulnerabilities

**What is it?** 2FA সিস্টেমে logic flaw বা implementation error।
**Variations:**

* **Simple Bypass:** প্রথম ধাপ (password) পার হওয়ার পর সরাসরি logged-in page-এ চলে গেলে application যদি 2FA step complete হয়েছে কি না তা চেক না করে।
* **Flawed Verification Logic:** Cookie বা parameter manipulate করে অন্য user-এর 2FA পেজে চলে যাওয়া।
* **Brute-forcing 2FA Codes:** 4 বা 6-digit কোডের ওপর rate limit না থাকলে Burp Intruder দিয়ে সহজেই তা বের করে ফেলা।

### E. Keeping Users Logged In ("Remember Me")

**What is it?** Persistent cookie-এর মাধ্যমে বারবার লগইন করা থেকে বিরত রাখা।
**How it works:** Cookie যদি predictable হয় (e.g., `base64(username:timestamp)`) অথবা un-salted hash হয়, তবে attacker তা offline crack বা guess করে অন্যের account-এ ঢুকতে পারে।

### F. Resetting User Passwords

**What is it?** Password ভুলে গেলে তা রিসেট করার মেকানিজম।
**Variations:**

* **Emailing Passwords:** Plaintext password ইমেইলে পাঠানো (Insecure channel)।
* **Predictable URL Parameters:** URL যদি `?user=victim` হয়, তবে যে কেউ অন্যের পাসওয়ার্ড রিসেট পেজে যেতে পারে।
* **Missing Token Validation:** Token দিয়ে পেজে ঢোকার পর form submit করার সময় যদি token আবার চেক না করা হয়, তবে attacker নিজের token দিয়ে পেজ লোড করে victim-এর account-এ form submit করতে পারে।
* **Password Reset Poisoning:** Dynamically URL জেনারেট করার সময় Host header manipulate করে token চুরি করা।

## 9. Different Contexts

Authentication vulnerabilities বিভিন্ন জায়গায় দেখা যেতে পারে:

* **Login Form:** Brute-force এবং username enumeration-এর জন্য।
* **Registration Form:** কোনো username আগে থেকেই আছে কি না তা চেক করে enumeration করা।
* **HTTP Headers:** HTTP Basic Auth এবং Cookies-এর ক্ষেত্রে।
* **Secondary Pages:** Password reset, password change, এবং 2FA verification pages।

## 10. Exploitation / Practical Understanding

**Username Enumeration via Response Timing:**
যদি application শুধু valid username-এর ক্ষেত্রেই password-এর hash চেক করে, তবে attacker একটি অত্যন্ত দীর্ঘ (excessively long) password পাঠাতে পারে। Username সঠিক হলে server সেই বিশাল password-টি hash করতে কিছুটা বেশি সময় নেবে। এই response time-এর পার্থক্য দেখে attacker নিশ্চিত হতে পারে যে username-টি valid।

**Account Lock Protection Bypass:**
Account lock থাকলে targeted brute-force (একজনের জন্য হাজারটা পাসওয়ার্ড) কাজ করে না। তখন technique পরিবর্তন করতে হয়। Attacker প্রথমে enumeration করে ১০০টি valid username বের করে। তারপর ৩টি সবচেয়ে কমন password সিলেক্ট করে (ধরি লিমিট ৪)। এরপর সে প্রতিটা username-এর জন্য ওই ৩টি password ট্রাই করে। এতে কোনো account লক হয় না, কিন্তু কোনো না কোনো user ওই কমন password ব্যবহার করায় attack সফল হয়ে যায়।

## 11. Step-by-Step Exploitation

**Exploiting Password Reset Broken Logic:**
Step 1 — Identify the input: নিজের account-এর password reset রিকোয়েস্ট পাঠান।
Step 2 — Test the behavior: Reset link-এ ক্লিক করে reset form পেজে যান।
Step 3 — Confirm the vulnerability: Form submit করার সময় রিকোয়েস্ট intercept করুন। দেখুন সেখানে token আছে কি না বা server token আবার validate করছে কি না।
Step 4 — Modify: রিকোয়েস্টে username-এর ফিল্ড থাকলে তা পরিবর্তন করে victim-এর username দিন বা URL থেকে token মুছে দিন।
Step 5 — Exploit: Request forward করুন। Server যদি শুধু submission দেখে password আপডেট করে দেয়, তবে victim-এর password পরিবর্তন হয়ে যাবে।

## 12. HTTP Requests / Responses

**HTTP Basic Authentication:**

```http
GET /admin HTTP/1.1
Host: insecure-website.com
Authorization: Basic YWRtaW46cGFzc3dvcmQ=

```

*Explanation:* এখানে `YWRtaW46cGFzc3dvcmQ=` হলো `admin:password`-এর Base64 encoded রূপ। এটি কোনো সত্যিকারের encryption নয়, শুধু encoding। Attacker খুব সহজেই এটি decode করে credentials পেয়ে যেতে পারে।

## 13. Payloads

* **Long Password Payload (Timing Attack):** একটি বিশাল character string (e.g., হাজার হাজার 'A')।
* *Purpose:* Server-এর hash function-কে ধীর করে দেওয়া, যাতে timing difference পরিষ্কারভাবে বোঝা যায়।


* **Account Cookie Modification (2FA Bypass):** `Cookie: account=victim-user`
* *Purpose:* 2FA verification context-কে নিজের থেকে victim-এর দিকে ঘুরিয়ে দেওয়া।


* **Predictable Remember-Me Cookie:** `Base64(carlos:1634567890)`
* *Purpose:* Predictable logic বুঝতে পেরে admin বা অন্য user-এর cookie জেনারেট করে account takeover করা।



## 14. Burp Suite Workflow

* **Intruder:** Username enumeration, password brute-forcing, এবং 2FA code guess করার জন্য সবচেয়ে বেশি ব্যবহৃত হয়। List থেকে payload নিয়ে স্বয়ংক্রিয়ভাবে অনেকগুলো রিকোয়েস্ট পাঠাতে পারে।
* **Turbo Intruder (Extension):** 2FA verification code বা time-sensitive brute-forcing-এর ক্ষেত্রে অত্যন্ত দ্রুতগতিতে request পাঠানোর জন্য এটি ব্যবহার করা হয়।
* **Macros (Project Options):** 2FA brute-force করার সময় যদি application বারবার logout করে দেয়, তবে Macro সেট করে স্বয়ংক্রিয়ভাবে পুনরায় login step 1 পার হয়ে 2FA পেজে আসার কাজটি করানো যায়।

## 15. How to Identify / Detect

* **Status Codes:** Brute-force করার সময় যদি হাজারটা 200 OK-এর মধ্যে একটি 302 Redirect আসে, তবে তা successful login নির্দেশ করে।
* **Error Messages:** "Invalid username" বনাম "Incorrect password" অথবা ছোট টাইপিং মিস্টেক (অদৃশ্য ক্যারেক্টার) লক্ষ্য করা।
* **Response Length:** Response-এর সাইজে কোনো পরিবর্তন এলে তা সফল কাজ নির্দেশ করতে পারে।
* **Timing Difference:** বিশাল পাসওয়ার্ড দিয়ে response আসতে কত সময় লাগছে তা মাপা।
* **Skipping Steps:** URL পরিবর্তন করে সরাসরি `/my-account` পেজে যাওয়ার চেষ্টা করে দেখা 2FA বাইপাস হয় কি না।

## 16. Common Mistakes

* **Email-based 2FA:** এটি সত্যিকারের 2FA নয়। কারণ Email access করতেও শুধু password লাগে। এটি মূলত জ্ঞান (knowledge)-এর ওপর ভিত্তি করে দুবার ভেরিফাই করা।
* **SMS-based 2FA:** SIM swapping-এর মাধ্যমে SMS ইন্টারসেপ্ট করা যায়।
* **Base64 as Encryption:** Cookie-তে Base64 ব্যবহার করে ডেভেলপাররা ভাবতে পারে ডেটা নিরাপদ, কিন্তু এটি সহজেই ডিকোড করা যায়।
* **Un-salted Hashes:** Salt ছাড়া hash ব্যবহার করলে attacker offline-এ rainbow tables বা search engine দিয়ে সহজেই তা ক্র্যাক করতে পারে।

## 17. Limitations

* **Rate Limiting / IP Block:** সঠিকভাবে IP-based rate limiting থাকলে Intruder দিয়ে brute-force করা যায় না।
* **Account Lockout:** 특정 একটি account-এর ওপর password guessing attack ঠেকিয়ে দেয়।
* **Proper Token Validation:** Password reset-এর সময় server যদি token এবং user-এর relationship সঠিকভাবে মেলায়, তবে reset logic flaw কাজ করবে না।

## 18. Edge Cases / Important Details

* **Subtle Error Message Differences:** অনেক সময় error message দেখতে একই মনে হলেও, HTML source-এ স্পেস বা hidden character-এর কারণে তা আলাদা হতে পারে।
* **Credential Stuffing:** এটি account lockout পলিসি দিয়ে ঠেকানো যায় না, কারণ attacker প্রতিটা username-এ মাত্র একবার লগইন করার চেষ্টা করে।
* **Internal Attack Surface:** Low-privileged account compromise হলেও তা থেকে internal pages-এ এক্সেস পাওয়া যেতে পারে, যা নতুন vulnerability খুঁজে পেতে সাহায্য করে।

## 19. Impact

Authentication vulnerability সফলভাবে exploit হলে:

* Sensitive data এবং functionality-তে আনঅথোরাইজড এক্সেস পাওয়া যায়।
* High-privileged account (Admin) কম্প্রোমাইজ হলে পুরো অ্যাপ্লিকেশন এবং ইন্টারনাল ইনফ্রাস্ট্রাকচারের নিয়ন্ত্রণ নেওয়া সম্ভব।
* অতিরিক্ত attack surface ওপেন হয়, যা দিয়ে আরও বড় ধরনের exploit করা যায়।

## 20. Prevention / Mitigation

* **Take Care with Credentials:** সব সময় HTTPS ব্যবহার করুন এবং HTTP-কে HTTPS-এ redirect করুন। কোনোভাবেই HTTP response বা public profile-এ credentials/email leak হতে দেওয়া যাবে না।
* **Don't Count on Users:** শুধু strict password policy না দিয়ে `zxcvbn`-এর মতো real-time password checker ব্যবহার করুন।
* **Prevent Username Enumeration:** সঠিক বা ভুল যেকোনো username-এর ক্ষেত্রে সম্পূর্ণ এক (identical), generic error message দিন। Status code এবং response time-ও একই রাখতে হবে।
* **Robust Brute-force Protection:** Strict IP-based user rate limiting ব্যবহার করুন এবং নির্দিষ্ট লিমিটের পর CAPTCHA বাধ্যতামূলক করুন।
* **Triple-check Verification Logic:** 2FA বা multi-step প্রসেসে সেশন এবং স্টেট ঠিকমতো যাচাই করুন।
* **Secure Supplementary Functions:** Password reset বা "Remember me" কুকি মেইন লগইনের মতোই সিকিউর হতে হবে। Token-গুলো high-entropy এবং short-lived হতে হবে।
* **Proper 2FA:** SMS বা Email-এর বদলে dedicated authenticator app বা hardware token ব্যবহার করুন।

## 21. Vulnerable vs Secure Example

**Vulnerable Password Reset URL:**

```
http://vulnerable-website.com/reset-password?user=victim-user

```

**Why vulnerable?** Attacker সহজেই `user` প্যারামিটার পরিবর্তন করে অন্যের পাসওয়ার্ড রিসেট করতে পারে।

**Secure Password Reset URL:**

```
http://secure-website.com/reset-password?token=a0ba0d1cb3b63d13822572fcff1a241895d893f659164d4cc550b421ebdd48a8

```

**Why secure?** URL-এ কোনো user-এর তথ্য নেই। Server-এ এই high-entropy token-টি নির্দিষ্ট user-এর সাথে যুক্ত করা আছে এবং এটি সাময়িক সময়ের জন্য কাজ করবে।

## 22. Real Understanding

Authentication vulnerabilities মূলত তৈরি হয় যখন application user-এর দেওয়া ইনপুট বা স্টেট-কে অতিমাত্রায় বিশ্বাস করে, অথবা attacker-কে অতিরিক্ত তথ্য দিয়ে দেয় (enumeration)। Brute-force attack ঠেকানোর ব্যবস্থা না থাকলে attacker সময় নিয়ে সঠিক পাসওয়ার্ড বের করে ফেলে। আর logic flaw থাকলে attacker-কে পাসওয়ার্ড বের করতেও হয় না, সে শুধু application-এর flow পরিবর্তন করে বা session parameter (যেমন cookie) পরিবর্তন করে নিজেকে অন্য কেউ হিসেবে প্রমাণ করে ফেলে।

## 23. Mental Model

User/Attacker provides identity claim & credentials
↓
Application verifies credentials OR provides feedback (error/time)
↓
If no rate-limit / predictable feedback -> Enumeration & Brute-force
If flawed multi-step logic -> Attacker skips/modifies session token
↓
Authentication boundary is completely bypassed
↓
Full account access / Data compromised

## 24. Quick Revision

* **Definition:** User-এর পরিচয় যাচাই করার প্রক্রিয়া (Authentication) বাইপাস করা।
* **Cause:** Brute-force protection-এর অভাব বা verification implementation-এ logic flaw।
* **Main idea:** Attacker credentials গেস করে বা logic বাইপাস করে অন্যের account-এ এক্সেস নেয়।
* **Attack flow:** Username Enum -> Brute force / Credential stuffing / 2FA cookie bypass।
* **Important condition:** Response-এ পার্থক্য থাকা বা multi-step প্রক্রিয়ায় state validation না থাকা।
* **Main techniques:** Timing analysis, Cookie manipulation, Macro/Turbo Intruder brute-forcing, Password reset poisoning.
* **Detection:** Error message/status code observation, URL/Cookie parameter tampering.
* **Impact:** Account takeover, Server compromise, Data leak.
* **Prevention:** Generic messages, HTTPS, `zxcvbn`, strict IP rate-limiting, secure tokens, App-based 2FA.

## 25. Things to Remember

* Email-based 2FA আসল 2FA নয়, এটি শুধু "Something you know"-এর ডাবল চেক।
* SMS 2FA SIM swapping-এর কারণে ঝুঁকিপূর্ণ।
* Account lockout পলিসি Credential stuffing বা horizontal brute-force (অনেক ইউজার, অল্প পাসওয়ার্ড) আটকাতে পারে না।
* Username enumeration-এর জন্য লম্বা পাসওয়ার্ড দিয়ে response time-এর পার্থক্য মাপা যায়।
* Password reset token সাবমিট করার সময়ও validate করতে হয়, শুধু পেজ লোডের সময় নয়।
* IP block বাইপাস করতে attacker প্রতি কয়েকটা রিকোয়েস্টের পর নিজের account-এ সফলভাবে লগইন করতে পারে (যদি তাতে কাউন্টার রিসেট হয়)।

