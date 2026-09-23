# Access Control


# Access Control Vulnerabilities and Privilege Escalation

## 1. What is Access Control?

**Short Definition:**
Access control হলো এমন একটি ব্যবস্থা বা constraints যা নির্ধারণ করে কে বা কী (who or what) কোনো নির্দিষ্ট action করতে পারবে বা resource access করতে পারবে।

**Detailed Explanation:**
Web application-এর ক্ষেত্রে Access control মূলত authentication এবং session management-এর ওপর নির্ভরশীল।

* **Authentication** নিশ্চিত করে যে user আসলেই সে কি না (identity verification)।
* **Session management** নির্ধারণ করে পরবর্তী HTTP request-গুলো একই user-এর কাছ থেকে আসছে কি না।
* **Access control** সিদ্ধান্ত নেয় যে ওই authenticated user যে কাজটি করতে চাইছে, তার সেই অনুমতি আছে কি না।

Access control-এর design এবং management অত্যন্ত জটিল কারণ এটি প্রযুক্তিগত বাস্তবায়নের সাথে সাথে organizational এবং business rules-এর ওপর নির্ভরশীল। এই সিদ্ধান্তগুলো মানুষের দ্বারা নিতে হয় বলে এখানে ভুলের (errors) সম্ভাবনা অনেক বেশি থাকে, যা broken access control vulnerability-র সৃষ্টি করে।

## 2. Access Control Security Models

Access control security model হলো access control rules-এর একটি formally defined definition, যা নির্দিষ্ট technology বা platform-এর ওপর নির্ভরশীল নয়। সাধারণত নিচে উল্লেখিত model-গুলো ব্যবহার করা হয়:

* **Programmatic Access Control:**
এখানে user privilege-এর একটি matrix ডাটাবেস বা সমজাতীয় স্টোরেজে রাখা হয় এবং প্রোগ্রামের মাধ্যমে এই matrix চেক করে access control প্রয়োগ করা হয়। এটি roles, groups বা individual users-এর জন্য highly granular হতে পারে।
* **Discretionary Access Control (DAC):**
এখানে resource বা function-এর access নির্দিষ্ট user বা user group-এর ওপর নির্ভর করে। Resource-এর মালিক (Owner) চাইলে অন্য user-কে access permission assign বা delegate করতে পারে। এটি খুবই granular কিন্তু manage করা বেশ জটিল।
* **Mandatory Access Control (MAC):**
এটি একটি centrally controlled ব্যবস্থা, যেখানে resource বা file-এর access কঠোরভাবে নিয়ন্ত্রিত হয়। DAC-এর মতো এখানে resource-এর মালিক চাইলেই কাউকে access দিতে বা পরিবর্তন করতে পারে না। এটি সাধারণত military clearance-based system-এ দেখা যায়।
* **Role-Based Access Control (RBAC):**
এখানে বিভিন্ন role (যেমন: admin, purchase clerk) তৈরি করা হয় এবং সেই role-এ access privilege দেওয়া হয়। এরপর user-দের ওই নির্দিষ্ট role-এ assign করা হয়। এটি complex application-এ access control manage করার জন্য সবচেয়ে কার্যকর পদ্ধতি।

## 3. Types of Access Controls

* **Vertical Access Controls:**
এটি sensitive functionality-তে access শুধুমাত্র নির্দিষ্ট ধরনের user-দের মধ্যে সীমাবদ্ধ রাখে (যেমন: separation of duties)। উদাহরণস্বরূপ, একজন admin যেকোনো user-এর account delete করতে পারবে, কিন্তু সাধারণ user তা পারবে না।
* **Horizontal Access Controls:**
এটি একই ধরনের resource-এর ক্ষেত্রে নির্দিষ্ট user-কে শুধুমাত্র তার নিজের resource access করার অনুমতি দেয়। উদাহরণস্বরূপ, একটি banking application-এ একজন user শুধু নিজের account-এর transaction দেখতে পারবে, অন্যের নয়।
* **Context-dependent Access Controls:**
এটি application-এর বর্তমান অবস্থা (state) বা user-এর interaction-এর ওপর ভিত্তি করে access সীমাবদ্ধ করে। উদাহরণস্বরূপ, payment সম্পন্ন করার পর retail website-এ shopping cart পরিবর্তন করতে না দেওয়া।

## 4. Why Does It Happen?

Broken access control ঘটে কারণ access control design এবং management-এ human error থাকে। Developer-রা অনেক সময় obfuscation-কে (security by obscurity) security মনে করে। এছাড়া user-controlled input (যেমন: parameter, header, cookie) অন্ধভাবে trust করা এবং multi-step process-এর প্রতিটি ধাপে access verify না করার কারণে এই vulnerability তৈরি হয়।

## 5. Types of Access Control Vulnerabilities

এখানে মূলত Privilege Escalation-এর ধরন অনুযায়ী vulnerability-গুলোকে ভাগ করা হয়।

### A. Vertical Privilege Escalation

**What is it?** যখন একজন user এমন কোনো functionality access করতে পারে যার অনুমতি তার নেই। (যেমন: সাধারণ user যদি admin page access করে)।

**Types / Variations:**

1. **Unprotected Functionality:**
sensitive functionality-তে কোনো protection থাকে না। User সরাসরি URL ব্রাউজ করে (যেমন: `[https://insecure-website.com/admin](https://insecure-website.com/admin)`) সেখানে যেতে পারে। অনেক সময় এই URL-গুলো `robots.txt` ফাইলে leak হয় বা wordlist দিয়ে brute-force করে পাওয়া যায়।
2. **Unpredictable URLs (Security by Obscurity):**
URL-টি কঠিন বা unpredictable করে লুকিয়ে রাখা হয় (যেমন: `administrator-panel-yb556`)। কিন্তু এটি নিরাপদ নয় কারণ এটি JavaScript কোডে leak হতে পারে।
*Example:*
```html
<script>
    var isAdmin = false;
    if (isAdmin) {
        var adminPanelTag = document.createElement('a');
        adminPanelTag.setAttribute('href', 'https://insecure-website.com/administrator-panel-yb556');
        adminPanelTag.innerText = 'Admin panel';
    }
</script>

```


এখানে script-টি সবার জন্যই visible, তাই attacker URL-টি জেনে সরাসরি ব্রাউজ করতে পারে।
3. **Parameter-based Access Control Methods:**
Application লগইনের পর user-এর role নির্ধারণ করে তা hidden field, cookie বা URL parameter-এ রেখে দেয়।
*Example:* `[https://insecure-website.com/login/home.jsp?admin=true](https://insecure-website.com/login/home.jsp?admin=true)`
Attacker সহজেই এই `admin=true` পরিবর্তন করে admin functionality access করতে পারে।
4. **Platform Misconfiguration:**
Application layer-এ URL-ভিত্তিক access restrict করা হলেও তা bypass করা সম্ভব।
* *URL Override:* `X-Original-URL` বা `X-Rewrite-URL` header ব্যবহার করে original URL override করা।
* *Method Change:* `POST` method ব্লক করা থাকলে attacker `GET` method ব্যবহার করে actionটি perform করার চেষ্টা করতে পারে।


5. **URL-matching Discrepancies:**
Access control mechanism এবং application-এর URL parsing-এর পার্থক্যের সুযোগ নেওয়া।
* Capitalization (e.g., `/ADMIN/DELETEUSER`).
* Spring framework-এর `useSuffixPatternMatch` (e.g., `/admin/deleteUser.anything`).
* Trailing slash যুক্ত করা (e.g., `/admin/deleteUser/`).



### B. Horizontal Privilege Escalation

**What is it?** যখন একজন user নিজের account বা resource-এর বদলে অন্য user-এর একই ধরনের resource access করতে পারে।

**Types / Variations:**

1. **Parameter Tampering (IDOR):**
*Example:* `[https://insecure-website.com/myaccount?id=123](https://insecure-website.com/myaccount?id=123)`
এখানে `id=123` পরিবর্তন করে `id=124` দিলে অন্যের account access করা যায়।
2. **Unpredictable IDs (GUIDs):**
যদি ID incrementing number না হয়ে GUID হয়, তবে তা guess করা কঠিন। কিন্তু এই GUID-গুলো application-এর অন্য কোথাও (যেমন: reviews বা user messages) leak হতে পারে, যা ব্যবহার করে attacker অন্যের resource access করতে পারে।
3. **Data Leakage in Redirect:**
Application access deny করে login page-এ redirect করলেও, সেই 302 Redirect response-এর body-তে কাঙ্ক্ষিত sensitive data থেকে যেতে পারে।

### C. Horizontal to Vertical Privilege Escalation

**What is it?** Horizontal escalation ব্যবহার করে বেশি ক্ষমতাসম্পন্ন কোনো user (যেমন Admin)-এর account compromise করা।
**How it works:** Attacker প্রথমে horizontal escalation-এর মাধ্যমে (e.g., parameter `id` পরিবর্তন করে) একজন admin-এর account page-এ ঢোকে। সেখান থেকে admin-এর password capture করা, password reset করা বা সরাসরি privileged function access করার মাধ্যমে এটি Vertical Privilege Escalation-এ রূপ নেয়।

### D. Insecure Direct Object References (IDOR)

**What is it?** IDOR হলো access control vulnerability-র একটি সাবক্যাটাগরি, যেখানে application user-supplied input ব্যবহার করে সরাসরি object access করে।

**Contexts / Variations:**

1. **Direct reference to database objects:**
URL-এ থাকা parameter সরাসরি database query-তে index হিসেবে ব্যবহৃত হয়।
*Example:* `?customer_number=132355`
2. **Direct reference to static files:**
Server-side filesystem-এর static file সরাসরি URL দিয়ে access করা।
*Example:* `[https://insecure-website.com/static/12144.txt](https://insecure-website.com/static/12144.txt)` (এখানে `12144` পরিবর্তন করে অন্যের চ্যাট ট্রান্সক্রিপ্ট পাওয়া যেতে পারে)।

### E. Access Control in Multi-step Processes

**What is it?** গুরুত্বপূর্ণ কাজগুলো কয়েকটি ধাপে (steps) ভাগ করা থাকে।
**How it works:** Application হয়তো প্রথম ও দ্বিতীয় ধাপে access control কঠোরভাবে চেক করে, কিন্তু ধরে নেয় যে তৃতীয় ধাপে কেউ এলে সে প্রথম দুই ধাপ পার হয়েই এসেছে। Attacker প্রথম দুই ধাপ বাদ দিয়ে সরাসরি তৃতীয় ধাপের request প্রয়োজনীয় parameter সহ পাঠিয়ে দিলে তা bypass হয়ে যায়।

### F. Referer-based Access Control

**What is it?** Application access control-এর ভিত্তি হিসেবে HTTP request-এর `Referer` header চেক করে।
**How it works:** যদি `/admin/deleteUser` পেজটি শুধু চেক করে যে request-টি `/admin` থেকে এসেছে কি না, তবে attacker নিজেই HTTP request-এ `Referer: [https://insecure-website.com/admin](https://insecure-website.com/admin)` বসিয়ে (forge করে) access পেয়ে যেতে পারে।

### G. Location-based Access Control

**What is it?** Geographical location-এর ওপর ভিত্তি করে access control।
**How it works:** Web proxies, VPNs বা client-side geolocation mechanism ম্যানিপুলেট করে এটি সহজেই বাইপাস করা যায়।

## 6. HTTP Requests / Responses (Technical Examples)

**Platform Misconfiguration Bypass (Header override):**

```http
POST / HTTP/1.1
Host: insecure-website.com
X-Original-URL: /admin/deleteUser
...

```

*Explanation:* এখানে front-end access control `POST /` request-কে allow করে, কিন্তু server-side-এ `X-Original-URL` header-টি process হয়ে request-টি restrict থাকা `/admin/deleteUser` endpoint-এ চলে যায়।

## 7. Limitations / Important Conditions

* **User input controllable হতে হবে:** Parameter-based, Referer-based বা Header-based attack-এর ক্ষেত্রে user-এর input modify করার সুযোগ থাকতে হবে।
* **GUID leakage:** Unpredictable ID-র ক্ষেত্রে attack তখনই সম্ভব, যখন ID অন্য কোনো সোর্স থেকে পাওয়া বা leak হয়।
* **Platform discrepancy:** URL-matching attack সফল হওয়ার জন্য application framework এবং access control layer-এর URL interpretation-এর মধ্যে পার্থক্য (discrepancy) থাকতে হবে।

## 8. Impact

Broken access control-এর impact অত্যন্ত ভয়াবহ হতে পারে:

* Unauthorized data access (Information disclosure).
* Account takeover (Horizontal/Vertical escalation).
* Privilege escalation (Admin access পাওয়া).
* Data modification বা deletion.

## 9. Prevention / Mitigation

Access control vulnerability প্রতিরোধ করতে Defense-in-depth approach নিতে হবে:

1. **Never rely on obfuscation:** শুধু URL বা parameter লুকিয়ে রাখলে (obscurity) কাজ হবে না।
2. **Deny access by default:** Publicly accessible না হলে সব resource-এর access default-ভাবে deny রাখতে হবে।
3. **Single mechanism:** পুরো application-এ access control প্রয়োগের জন্য একটিমাত্র (single application-wide) mechanism ব্যবহার করতে হবে।
4. **Code-level declaration:** Developer-দের জন্য mandatory করতে হবে যেন তারা কোড লেভেলে প্রতিটি resource-এর access ডিক্লেয়ার করে।
5. **Audit and test:** Access control ঠিকমতো কাজ করছে কি না তা নিশ্চিত করতে নিয়মিত অডিট ও টেস্ট করতে হবে।

## 10. Real Understanding / Mental Model

User-controlled input (e.g., parameter `id=2`, `admin=true`, `Referer` header)
↓
Application assumes the input/state is trustworthy or relies on previous steps
↓
Attacker modifies the input or skips steps
↓
Application fails to verify the actual authorization for that specific action
↓
Security boundary is bypassed (Horizontal/Vertical Privilege Escalation)

## 11. Quick Revision

* **Definition:** Access control নির্ধারণ করে কে কোন কাজ করতে পারবে।
* **Security Models:** DAC, MAC, RBAC, Programmatic.
* **Vertical Escalation:** User তার অনুমতি নেই এমন কাজ করা (e.g., user থেকে admin)।
* **Horizontal Escalation:** User অন্যের ডেটা access করা (e.g., user A থেকে user B)।
* **IDOR:** User-supplied input সরাসরি object/database access করতে ব্যবহৃত হওয়া।
* **Main techniques:** Parameter tampering, Header override (`X-Original-URL`), Forging Referer, Skipping multi-step process, URL matching tricks.
* **Prevention:** Deny by default, use a single mechanism, never rely on obfuscation, strictly validate privileges on every step.

## 12. Things to Remember

* JavaScript বা HTML source code-এ লুকানো URL চেক করতে হবে।
* 302 Redirect পেলেও response body চেক করতে হবে, সেখানে sensitive data leak হতে পারে।
* Multi-step process-এ সরাসরি শেষ ধাপের request পাঠিয়ে দেখতে হবে।
* URL bypass করার জন্য `X-Original-URL` header, trailing slash `/`, বা `.anything` (Spring framework) ব্যবহার করে দেখতে হবে।
* Client-side parameter (যেমন: `role=1`, `admin=true`) কখনোই trust করা যাবে না।
* `Referer` header ক্লায়েন্ট কন্ট্রোল করতে পারে, তাই এর ওপর ভিত্তি করে access control করা ইনসিকিউর।
