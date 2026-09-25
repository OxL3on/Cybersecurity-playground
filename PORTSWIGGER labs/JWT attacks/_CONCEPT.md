# JSON Web Token (JWT) Attacks

## 1. What are JWTs?

**Short Definition:**
JSON Web Token (JWT) হলো সিস্টেমের মধ্যে ক্রিপ্টোগ্রাফিকভাবে সাইন (signed) করা JSON ডেটা আদান-প্রদান করার একটি স্ট্যান্ডার্ড ফরম্যাট।

**Detailed Explanation:**
JWT তাত্ত্বিকভাবে যেকোনো ধরনের ডেটা বহন করতে পারে। তবে এটি মূলত প্রমাণীকরণ (authentication), সেশন ম্যানেজমেন্ট (session handling) এবং অ্যাক্সেস কন্ট্রোলের (access control) অংশ হিসেবে ইউজারদের সম্পর্কে তথ্য ("claims") আদান-প্রদান করতে ব্যবহৃত হয়। ক্লাসিক সেশন টোকেনের বিপরীতে, সার্ভারের প্রয়োজনীয় সমস্ত ডেটা ক্লায়েন্ট-সাইডে, অর্থাৎ JWT-এর ভেতরেই স্টোর করা থাকে। এটি অত্যন্ত ডিস্ট্রিবিউটেড ওয়েবসাইটের জন্য জনপ্রিয়, যেখানে ইউজারদের একাধিক ব্যাক-এন্ড সার্ভারের সাথে নির্বিঘ্নে ইন্টারঅ্যাক্ট করতে হয়।

## 2. JWT Format

একটি JWT ৩টি অংশ নিয়ে গঠিত: header, payload এবং signature। এগুলো একে অপরের থেকে একটি ডট (dot) দিয়ে আলাদা করা থাকে। উদাহরণস্বরূপ: `header.payload.signature`।

* **Header:** এটি Base64url-এনকোড করা একটি JSON অবজেক্ট। এতে টোকেন সম্পর্কিত মেটাডেটা থাকে (যেমন: কোন অ্যালগরিদম দিয়ে সাইন করা হয়েছে)।
* **Payload:** এটিও Base64url-এনকোড করা JSON অবজেক্ট। এতে ইউজার সম্পর্কে আসল ডেটা বা "claims" থাকে (যেমন: ইউজারনেম, রোল, এক্সপায়ার ডেট)।
* **Signature:** যেহেতু Header এবং Payload সহজেই যে কেউ ডিকোড করে পড়তে পারে, তাই এর নিরাপত্তা সম্পূর্ণভাবে নির্ভর করে ক্রিপ্টোগ্রাফিক সিগনেচারের ওপর।

*Note:* "JWT" বলতে মূলত JSON Web Signature (JWS) টোকেন বোঝায়। আরেকটি রূপ হলো JSON Web Encryption (JWE), যেখানে ডেটা এনকোড করার পাশাপাশি এনক্রিপ্ট করা হয়।

## 3. JWT Signature

যে সার্ভার টোকেন ইস্যু করে, তারা সাধারণত header এবং payload-কে হ্যাশ (hash) করে সিগনেচার তৈরি করে। এই প্রক্রিয়ায় একটি গোপন সাইনিং কি (secret signing key) বা প্রাইভেট কি (private key) ব্যবহৃত হয়। এটি প্রমাণ করে যে টোকেনটি ইস্যু হওয়ার পর এর ভেতরের কোনো ডেটা পরিবর্তন করা হয়নি।

* Header বা Payload-এর একটি বাইটও পরিবর্তন করলে সিগনেচার আর মিলবে না (mismatched)।
* সার্ভারের গোপন কি (secret key) ছাড়া সঠিক সিগনেচার তৈরি করা সম্ভব নয়।

## 4. What are JWT Attacks?

**Short Definition:**
JWT অ্যাটাক হলো সার্ভারে মডিফাই করা বা জাল (forged) JWT পাঠানো, যাতে কোনো ক্ষতিকর উদ্দেশ্য হাসিল করা যায়।

**Why does it happen? (Root Cause):**
JWT স্পেসিফিকেশন বাই ডিজাইন খুবই ফ্লেক্সিবল। এর ফলে ডেভেলপাররা ইমপ্লিমেন্ট করার সময় অনেক ভুল করে ফেলেন। মূল সমস্যাগুলো দেখা দেয় যখন:

* সার্ভার টোকেনের সিগনেচার ঠিকমতো ভেরিফাই (verify) করে না।
* সার্ভারের গোপন কি (secret key) লিক হয়ে যায় বা ব্রুট-ফোর্স (brute-force) করে বের করা যায়।
* লাইব্রেরিগুলো ইনপুট ঠিকমতো হ্যান্ডেল না করার কারণে অ্যালগরিদম কনফিউশন (algorithm confusion) তৈরি হয়।

## 5. Attack Flow

```
Attacker
↓
Obtain a valid JWT from the application (e.g., after login)
↓
Decode the token to inspect the header and payload
↓
Identify a vulnerability in how the server handles or verifies the JWT
↓
Modify the payload (e.g., change `username` to `admin`)
↓
Exploit the vulnerability to bypass signature verification or forge a new valid signature
↓
Send the modified JWT back to the server
↓
Server trusts the token, granting the attacker unauthorized access
```

## 6. Important Conditions / Requirements

* অ্যাপ্লিকেশনকে অথেনটিকেশন বা সেশন ম্যানেজমেন্টের জন্য JWT ব্যবহার করতে হবে।
* অ্যাপ্লিকেশনের JWT ভেরিফিকেশন মেকানিজমে অবশ্যই কোনো লজিক্যাল বা কনফিগারেশন ফ্ল (flaw) থাকতে হবে।
* অ্যাটাকারকে টোকেন মডিফাই করে সার্ভারে পাঠানোর সুযোগ থাকতে হবে।

## 7. Types / Variations and Exploitation Techniques

### A. Exploiting Flawed JWT Signature Verification

সার্ভার সাধারণত ইস্যু করা টোকেনের কোনো তথ্য নিজের কাছে রাখে না। তাই সার্ভার যদি সিগনেচার ঠিকমতো ভেরিফাই না করে, তবে অ্যাটাকার টোকেনের বাকি অংশে যেকোনো পরিবর্তন করতে পারে।

**1. Accepting arbitrary signatures**

* **What is it?** অ্যাপ্লিকেশন সিগনেচার আদৌ চেক করে না।
* **How it works:** অনেক JWT লাইব্রেরিতে দুটি মেথড থাকে—একটি ভেরিফাই করার জন্য (`verify()`) এবং অন্যটি শুধু ডিকোড করার জন্য (`decode()`)। ডেভেলপার ভুল করে শুধু `decode()` মেথড ব্যবহার করলে সার্ভার সিগনেচার চেক না করেই পে-লোড (payload) গ্রহণ করে নেয়।
* **Exploitation:** পে-লোড মডিফাই করে (যেমন: `"username": "administrator"`) সিগনেচার পরিবর্তন না করেই (বা যেকোনো কিছু দিয়ে) পাঠিয়ে দেওয়া।

**2. Accepting tokens with no signature (None algorithm)**

* **What is it?** JWT হেডারের `alg` প্যারামিটার নির্দেশ করে কোন অ্যালগরিদম ব্যবহার করা হয়েছে। যদি এটি `none` সেট করা থাকে, তবে এটি একটি "unsecured JWT"।
* **How it works:** সার্ভার হেডারের `alg` প্যারামিটারটিকে অন্ধভাবে বিশ্বাস করে এবং সেই অনুযায়ী ভেরিফিকেশন অ্যালগরিদম ঠিক করে। `alg: none` থাকলে সার্ভার সিগনেচার চেকই করে না।
* **Exploitation:** হেডারে `"alg": "none"` (বা অবফাসকেটেড যেমন `NoNe`) সেট করা, পে-লোড মডিফাই করা এবং সিগনেচারের অংশটি পুরোপুরি মুছে ফেলা (তবে শেষের ডট `.` টি রেখে দিতে হবে)।

### B. Brute-forcing Secret Keys

* **What is it?** কিছু অ্যালগরিদম যেমন HS256 সিগনেচারের জন্য একটি সিমেট্রিক (symmetric) বা একক গোপন কি (secret key) ব্যবহার করে। এটি দুর্বল হলে ব্রুট-ফোর্স করা যায়।
* **How it works:** ডেভেলপাররা অনেক সময় ডিফল্ট বা সাধারণ পাসওয়ার্ডের মতো স্ট্রিং গোপন কি হিসেবে ব্যবহার করেন। অ্যাটাকার Hashcat-এর মতো টুল ব্যবহার করে অফলাইনে এই কি বের করে ফেলতে পারে।
* **Exploitation:**
1. একটি ভ্যালিড JWT এবং well-known secrets-এর ওয়ার্ডলিস্ট (wordlist) নিন।
2. Hashcat ব্যবহার করুন: `hashcat -a 0 -m 16500 <jwt> <wordlist>`
3. গোপন কি পেয়ে গেলে, Burp-এর JWT Editor-এ সেই কি দিয়ে নিজের ইচ্ছামতো তৈরি করা পে-লোড সাইন করে নিন।



### C. JWT Header Parameter Injections

JWS স্পেসিফিকেশন অনুযায়ী শুধু `alg` প্যারামিটারটি বাধ্যতামূলক। তবে হেডারে প্রায়ই আরও কিছু প্যারামিটার থাকে যা সার্ভারকে বলে দেয় ভেরিফাই করার জন্য কোন কি (key) ব্যবহার করতে হবে। অ্যাটাকার এগুলো ম্যানিপুলেট করে নিজের তৈরি করা কি (key) দিয়ে সাইন করা টোকেন সার্ভারকে গ্রহণ করাতে পারে।

**1. Injecting self-signed JWTs via the `jwk` parameter**

* **What is it?** `jwk` (JSON Web Key) প্যারামিটার সার্ভারকে টোকেনের ভেতরেই একটি পাবলিক কি এমবেড (embed) করার সুযোগ দেয়।
* **How it works:** কিছু সার্ভার ভুল করে হেডারের `jwk` প্যারামিটারে দেওয়া যেকোনো কি-কে ভেরিফিকেশনের জন্য ব্যবহার করে।
* **Exploitation:** অ্যাটাকার নিজের একটি RSA প্রাইভেট কি তৈরি করে। সেই কি দিয়ে টোকেন সাইন করে এবং হেডারের `jwk` প্যারামিটারে নিজের তৈরি পাবলিক কি এমবেড করে দেয়।

**2. Injecting self-signed JWTs via the `jku` parameter**

* **What is it?** `jku` (JSON Web Key Set URL) প্যারামিটার একটি URL দেয় যেখান থেকে সার্ভার ভেরিফাই করার জন্য কি (key) ফেচ করে।
* **How it works:** সার্ভার যদি ঠিকমতো URL ভ্যালিডেট না করে, তবে অ্যাটাকার নিজের সার্ভারের URL দিতে পারে।
* **Exploitation:** অ্যাটাকার নিজের সার্ভারে একটি JWK Set (public key) হোস্ট করে। হেডারের `jku` প্যারামিটারে সেই URL দেয় এবং প্রাইভেট কি দিয়ে টোকেন সাইন করে পাঠায়। (URL পার্সিং ফ্ল বা SSRF টেকনিক দিয়ে ফিল্টার বাইপাস করা যেতে পারে)।

**3. Injecting self-signed JWTs via the `kid` parameter**

* **What is it?** `kid` (Key ID) প্যারামিটার সার্ভারকে বলে দেয় একাধিক কি-এর মধ্যে কোনটি ব্যবহার করতে হবে। এটি সাধারণত একটি স্ট্রিং হয় (যেমন ডেটাবেস এন্ট্রি বা ফাইলের নাম)।
* **How it works:** যদি `kid` প্যারামিটারটি ডিরেক্টরি ট্রাভার্সাল (Directory Traversal)-এর জন্য ভালনারেবল হয়, তবে অ্যাটাকার সার্ভারের ফাইলসিস্টেমের যেকোনো ফাইলকে ভেরিফিকেশন কি হিসেবে ব্যবহার করতে বাধ্য করতে পারে।
* **Exploitation:** অ্যাটাকার `kid` হিসেবে `/dev/null` (যা একটি ফাঁকা ফাইল) দেয়: `"kid": "../../dev/null"`। এরপর সিমেট্রিক অ্যালগরিদম (যেমন HS256) ব্যবহার করে একটি ফাঁকা স্ট্রিং (empty string) দিয়ে টোকেন সাইন করে।

### D. JWT Algorithm Confusion Attacks

* **What is it?** অ্যাটাকার সার্ভারকে ধোঁকা দিয়ে তার উদ্দেশ্য করা অ্যালগরিদমের বদলে অন্য একটি অ্যালগরিদম দিয়ে সিগনেচার ভেরিফাই করায়।
* **How it works:** সার্ভার সাধারণত RS256 (অ্যাসিমেট্রিক) ব্যবহার করে যেখানে একটি প্রাইভেট কি দিয়ে সাইন করা হয় এবং পাবলিক কি দিয়ে ভেরিফাই করা হয়। অনেক লাইব্রেরির জেনেরিক `verify()` মেথড থাকে যা টোকেনের `alg` হেডার দেখে অ্যালগরিদম ঠিক করে। ডেভেলপাররা ভুল করে ধরে নেন যে সবসময় RS256 আসবে এবং তারা সবকিছুর জন্যই সার্ভারের পাবলিক কি পাস করে দেন।
* **Exploitation:**
1. সার্ভারের পাবলিক কি জোগাড় করতে হবে (যেমন `/.well-known/jwks.json` থেকে বা `jwt_forgery.py` টুল দিয়ে existing টোকেন থেকে বের করে)।
2. টোকেনের হেডারে `"alg": "HS256"` (সিমেট্রিক) সেট করতে হবে।
3. সার্ভারের ওই একই পাবলিক কি-কে সিমেট্রিক "গোপন কি (secret)" হিসেবে ব্যবহার করে টোকেন সাইন করতে হবে। সার্ভার যখন ভেরিফাই করবে, সেও পাবলিক কি-কে secret হিসেবে ধরে HS256 অ্যালগরিদম দিয়ে ভেরিফাই করবে এবং টোকেনটি ভ্যালিড মনে করবে।



## 8. HTTP Requests / Responses

**Vulnerable Request Example (Unverified Signature / None Algorithm):**

*Original Token:* `eyJhbGciOiJSUzI1NiJ9.eyJ1c2VybmFtZSI6IndpZW5lciJ9.signature_here`
(Decoded Header: `{"alg":"RS256"}`, Payload: `{"username":"wiener"}`)

*Modified Request by Attacker:*

```http
GET /admin HTTP/1.1
Host: vulnerable-website.com
Cookie: session=eyJhbGciOiJub25lIn0.eyJ1c2VybmFtZSI6ImFkbWluaXN0cmF0b3IifQ.

```

*(Decoded Header: `{"alg":"none"}`, Payload: `{"username":"administrator"}`)*
*Explanation:* অ্যাটাকার `alg` কে `none` করেছে, ইউজারনেম `administrator` করেছে এবং সিগনেচার মুছে দিয়েছে, কিন্তু শেষের ডট `.` রেখে দিয়েছে। সার্ভার এটি গ্রহণ করলে অথেনটিকেশন বাইপাস হবে।

## 9. Payloads

* **None Algorithm Payload:** `{"alg": "none"}`
* *Purpose:* সার্ভারকে সিগনেচার ভেরিফাই করা থেকে বিরত রাখা।


* **Path Traversal in `kid`:** `{"kid": "../../../../../dev/null"}`
* *Purpose:* ভেরিফিকেশন কি হিসেবে সার্ভারের একটি ফাঁকা ফাইল ব্যবহার করতে বাধ্য করা, যাতে ফাঁকা স্ট্রিং দিয়ে সাইন করা যায়।


* **Embedded JWK (`jwk` header):**
```json
{
  "kid": "my-key-id",
  "alg": "RS256",
  "jwk": { "kty": "RSA", "e": "AQAB", "n": "..." }
}

```


* *Purpose:* হেডারের ভেতরেই নিজের তৈরি করা পাবলিক কি দিয়ে দেওয়া যাতে সার্ভার সেটি দিয়ে ভেরিফাই করে।



## 10. Burp Suite Workflow

JWT নিয়ে কাজ করার জন্য Burp Suite-এ **JWT Editor** এক্সটেনশন ইনস্টল করা প্রয়োজন।

* **Viewing JWTs:** Inspector প্যানেলে গেলে JWT স্বয়ংক্রিয়ভাবে ডিকোড হয়ে দেখায়।
* **Editing JWTs:**
1. রিকোয়েস্ট Repeater-এ পাঠান।
2. JSON Web Token ট্যাবে গিয়ে Header এবং Payload এডিট করুন।


* **Generating & Adding Keys:**
1. JWT Editor Keys ট্যাবে গিয়ে `New RSA Key` বা `New Symmetric Key` তৈরি করুন।


* **Signing JWTs:**
1. Repeater-এর JSON Web Token ট্যাবে `Sign` বাটনে ক্লিক করে তৈরি করা কি (key) সিলেক্ট করুন। এটি নতুন ভ্যালু দিয়ে টোকেনটি রি-সাইন (re-sign) করে দেবে।


* **Automated Attacks (e.g., Embedded JWK):**
1. Repeater-এর JSON Web Token ট্যাবে `Attack` বাটনে ক্লিক করে `Embedded JWK` সিলেক্ট করুন এবং নিজের RSA কি বেছে নিন। এটি স্বয়ংক্রিয়ভাবে `jwk` এবং `kid` হেডার আপডেট করে দেবে।



## 11. How to Identify / Detect

* Burp Scanner (Professional 2022.5.1+) স্বয়ংক্রিয়ভাবে অনেক JWT ভালনারেবিলিটি ডিটেক্ট করতে পারে।
* ম্যানুয়ালি টেস্ট করার সময়:
* পে-লোডে সাধারণ পরিবর্তন করে সিগনেচার না পাল্টেই পাঠিয়ে দেখুন সার্ভার গ্রহণ করে কি না।
* `alg` হেডারকে `none` করে সিগনেচার মুছে দিয়ে দেখুন।
* হেডারে `jwk`, `jku`, বা `kid` প্যারামিটার আছে কি না তা লক্ষ্য করুন।



## 12. Common Mistakes

* **Forgetting the trailing dot:** `none` অ্যালগরিদম ব্যবহার করার সময় সিগনেচার মুছে দিলেও শেষের ডট `.` টি অবশ্যই রাখতে হবে, নইলে ফরম্যাট ইনভ্যালিড হবে।
* **Public Key Formatting in Algorithm Confusion:** অ্যালগরিদম কনফিউশন অ্যাটাকের ক্ষেত্রে সাইন করার জন্য যে পাবলিক কি ব্যবহার করা হবে, তা হুবহু সার্ভারে থাকা কি-এর মতো হতে হবে। কোনো স্পেস বা নিউলাইন (non-printing characters) বাদ পড়লে বা ফরম্যাট (যেমন X.509 PEM) না মিললে সিগনেচার মিলবে না।
* **Using `decode()` instead of `verify()`:** ডেভেলপারদের জন্য সবচেয়ে সাধারণ ভুল হলো সিগনেচার ভেরিফাই না করেই টোকেনের ডেটা ট্রাস্ট করা।

## 13. Limitations

* সার্ভার যদি শক্তিশালী সিমেট্রিক কি (secret key) ব্যবহার করে যা ব্রুট-ফোর্স করা সম্ভব নয়, এবং `alg` হেডার স্ট্রিক্টলি ভ্যালিডেট করে, তবে সিগনেচার ফোরজ (forge) করা প্রায় অসম্ভব।
* অ্যালগরিদম কনফিউশন অ্যাটাকের জন্য সার্ভারের পাবলিক কি প্রয়োজন হয়। কি (key) জানা না থাকলে বা existing টোকেন থেকে ডিরাইভ (derive) করতে না পারলে এই অ্যাটাক করা যায় না।

## 14. Edge Cases / Important Details

* **`cty` (Content Type) Header Injection:** যদি সিগনেচার ভেরিফিকেশন বাইপাস করা যায়, তবে `cty` হেডার ইনজেক্ট করে কন্টেন্ট টাইপ `text/xml` বা `application/x-java-serialized-object` করে দেওয়া যেতে পারে, যা XXE বা Insecure Deserialization অ্যাটাকের পথ খুলে দিতে পারে।
* **`x5c` Header Injection:** `x5c` প্যারামিটার ব্যবহার করে X.509 সার্টিফিকেট ইনজেক্ট করা যায়। এর জটিলতার কারণে পার্সিং করার সময় ভালনারেবিলিটি দেখা দিতে পারে (যেমন CVE-2017-2800)।

## 15. Impact

JWT অ্যাটাক সফল হলে এর ইমপ্যাক্ট সাধারণত অত্যন্ত ভয়ংকর (severe) হয়:

* **Authentication Bypass:** ইউজারনেম পরিবর্তন করে অন্য ইউজারের অ্যাকাউন্টে লগইন করা।
* **Privilege Escalation:** পে-লোডে `"isAdmin": true` করে অ্যাডমিন এক্সেস পাওয়া।
* **Account Takeover & Full Control:** নিজের ইচ্ছামতো ভ্যালিড টোকেন তৈরি করে পুরো অ্যাপ্লিকেশন এবং অন্যান্য ইউজারদের ডেটার পূর্ণ নিয়ন্ত্রণ নেওয়া।

## 16. Prevention / Mitigation

এই দুর্বলতাগুলো এড়াতে নিচের পদক্ষেপগুলো নেওয়া উচিত:

* **Use Up-to-date Libraries:** আধুনিক এবং সিকিউর JWT লাইব্রেরি ব্যবহার করা এবং এর সিকিউরিটি ইমপ্লিকেশন ভালো করে বোঝা।
* **Robust Signature Verification:** টোকেন রিসিভ করার পর অবশ্যই সিগনেচার ভেরিফাই করতে হবে। অপ্রত্যাশিত অ্যালগরিদম (যেমন `none` বা RSA-এর জায়গায় HMAC) ব্লক করতে হবে।
* **Strict Whitelisting:** `jku` হেডার ব্যবহার করলে অবশ্যই অনুমোদিত হোস্টের হোয়াইটলিস্ট (whitelist) ব্যবহার করতে হবে।
* **Secure `kid` Handling:** `kid` প্যারামিটারের মাধ্যমে যাতে পাথ ট্রাভার্সাল (Path Traversal) বা SQL ইনজেকশন না হতে পারে তা নিশ্চিত করতে হবে।
* **Strong Secrets:** সিমেট্রিক অ্যালগরিদমের ক্ষেত্রে এমন সিক্রেট ব্যবহার করতে হবে যা কোনোভাবেই ব্রুট-ফোর্স করা সম্ভব নয়।

**Additional Best Practices:**

* সবসময় টোকেনের একটি এক্সপায়ার ডেট (expiration date) সেট করা।
* URL প্যারামিটারের মাধ্যমে টোকেন পাঠানো এড়িয়ে চলা।
* টোকেনের আসল প্রাপক কে তা নির্দিষ্ট করতে `aud` (audience) ক্লেইম ব্যবহার করা।
* সার্ভারের যেন টোকেন রিভোক (revoke) বা বাতিল করার ক্ষমতা থাকে (যেমন লগআউটের সময়)।

## 17. Vulnerable vs Secure Example

**Vulnerable Validation (Node.js snippet):**

```javascript
const decoded = jwt.decode(token); 
if(decoded.username === 'admin') { /* grant admin access */ }

```

*Why vulnerable?* এটি শুধুমাত্র টোকেনটিকে ডিকোড করেছে কিন্তু এর সিগনেচার ভেরিফাই করেনি।

**Secure Validation:**

```javascript
jwt.verify(token, secretKey, { algorithms: ['RS256'] }, function(err, decoded) {
  if (err) { /* handle error */ }
  if(decoded.username === 'admin') { /* grant admin access */ }
});

```

*Why secure?* এটি সিগনেচার ভেরিফাই করছে এবং স্পষ্টভাবে বলে দিচ্ছে যে শুধুমাত্র `RS256` অ্যালগরিদমই গ্রহণ করা হবে, যা অ্যালগরিদম কনফিউশন এবং `none` অ্যালগরিদম অ্যাটাক প্রতিরোধ করে।

## 18. Real Understanding

JWT মূলত ক্লায়েন্টের কাছে সার্ভারের দেওয়া একটি 'সিলমোহর যুক্ত পরিচয়পত্র'। সমস্যাটা তখনই হয় যখন সার্ভার সেই সিলমোহর (signature) ঠিকমতো যাচাই করে না, অথবা এমন ব্যবস্থা রাখে যেখানে ক্লায়েন্ট নিজেই বলে দেয় সিলমোহরটি আসল না নকল (যেমন `alg: none` বা `jwk` হেডার)। যেহেতু পুরো ডেটা ক্লায়েন্টের কাছে থাকে, তাই ভেরিফিকেশনের এই ছোটখাটো লজিক্যাল ভুলগুলো অ্যাটাকারকে সম্পূর্ণ নতুন ও ভ্যালিড পরিচয়পত্র (টোকেন) তৈরি করার ক্ষমতা দিয়ে দেয়।

## 19. Mental Model

Server issues JWT to Client (Header.Payload.Signature)
↓
Attacker decodes the token and understands the structure (claims)
↓
Attacker alters the claims (e.g., changes user to 'admin')
↓
Attacker exploits a flaw in verification:
- Sets `alg: none` and removes signature
- Injects own key via `jwk` or `jku`
- Uses path traversal in `kid` to sign with empty string
- Uses algorithm confusion to sign with server's public key
↓
Server receives the forged token, validates it incorrectly, and trusts the altered claims
↓
Authentication bypassed / Privilege escalated

## 20. Quick Revision

* **Definition:** সার্ভারে মডিফাই করা বা জাল (forged) JWT পাঠিয়ে অথেনটিকেশন বাইপাস করা।
* **Cause:** সার্ভারের সিগনেচার ভেরিফিকেশনে ত্রুটি বা দুর্বল সিক্রেট কি।
* **Main idea:** টোকেনের পে-লোড (claims) মডিফাই করে নিজেকে অন্য ইউজার বা অ্যাডমিন হিসেবে প্রমাণ করা।
* **Attack flow:** Intercept token -> Decode -> Modify payload -> Exploit signature verification flaw -> Send back.
* **Important condition:** সার্ভারের ভেরিফিকেশন লজিকে ত্রুটি থাকতে হবে।
* **Main techniques:** `alg: none`, `jwk`/`jku` injection, `kid` path traversal, Secret brute-forcing, Algorithm confusion.
* **Detection:** Burp Scanner, ম্যানুয়ালি সিগনেচার মুছে বা মডিফাই করে টেস্ট করা।
* **Impact:** Account takeover, Privilege escalation.
* **Prevention:** `verify()` মেথড ব্যবহার করা, অ্যালগরিদম ফিক্সড (hardcode) করে রাখা, স্ট্রং সিক্রেট ব্যবহার করা।

## 21. Things to Remember

* **Trailing Dot:** `none` অ্যালগরিদম ব্যবহার করলে সিগনেচার মুছে দিলেও শেষের ডট `.` রাখতে হবে।
* **Algorithm Confusion:** এই অ্যাটাকের জন্য সাইন করার সময় যে পাবলিক কি ব্যবহার করা হবে, তা হুবহু (byte-by-byte) সার্ভারের পাবলিক কি-এর মতো হতে হবে।
* **`kid` Path Traversal:** `/dev/null` ফাইল ব্যবহার করে ফাঁকা স্ট্রিং দিয়ে সিগনেচার তৈরি করা একটি অত্যন্ত স্মার্ট পদ্ধতি।
* **Hashcat:** অফলাইনে JWT সিক্রেট ব্রুট-ফোর্স করার জন্য `hashcat -a 0 -m 16500 <jwt> <wordlist>` কমান্ডটি ব্যবহৃত হয়।

