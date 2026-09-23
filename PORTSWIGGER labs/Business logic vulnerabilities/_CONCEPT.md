# Business Logic Vulnerabilities

## 1. What is Business Logic Vulnerability?

**Short Definition:**
Business logic vulnerability (বা Application logic vulnerability) হলো অ্যাপ্লিকেশনের ডিজাইন এবং ইমপ্লিমেন্টেশনের এমন কিছু ত্রুটি, যা অ্যাটাকারকে অ্যাপ্লিকেশনের স্বাভাবিক ফাংশন ব্যবহার করে অনাকাঙ্ক্ষিত (unintended) কাজ করার সুযোগ দেয়।

**Detailed Explanation:**
Business logic বলতে সেই নিয়মগুলোকে বোঝায় যা নির্ধারণ করে অ্যাপ্লিকেশনটি কীভাবে কাজ করবে এবং বিভিন্ন পরিস্থিতিতে কীভাবে রিঅ্যাক্ট করবে। এর মূল উদ্দেশ্য হলো ইউজারদের এমন কিছু করা থেকে বিরত রাখা যা বিজনেসের জন্য ক্ষতিকর বা অযৌক্তিক। যখন এই লজিক বা নিয়মে কোনো ত্রুটি থাকে, তখন অ্যাটাকার সেই নিয়মগুলোকে ফাঁকি দিতে পারে (circumvent)। এটি অটোমেটেড স্ক্যানার দিয়ে খুঁজে পাওয়া প্রায় অসম্ভব কারণ এর জন্য অ্যাপ্লিকেশনের নির্দিষ্ট কাজ বা business domain সম্পর্কে মানুষের বোঝার ক্ষমতা (human knowledge) প্রয়োজন হয়।

## 2. Why Does It Happen?

এই দুর্বলতাগুলো মূলত তৈরি হয় কারণ ডিজাইন এবং ডেভেলপমেন্ট টিম ইউজারদের আচরণ নিয়ে ভুল ধারণা (flawed assumptions) পোষণ করে। তারা মনে করে ইউজাররা শুধু ওয়েব ব্রাউজার দিয়ে স্বাভাবিকভাবে অ্যাপ্লিকেশন ব্যবহার করবে। এর ফলে তারা:

* ক্লায়েন্ট-সাইড কন্ট্রোলের ওপর অতিরিক্ত নির্ভর করে।
* অস্বাভাবিক ইউজার ইনপুট বা অপ্রত্যাশিত অবস্থা (unusual application states) সার্ভার-সাইডে হ্যান্ডেল করার কোড লেখে না।
* Overly complicated system তৈরি করে, যেখানে কোনো একজন ডেভেলপার হয়তো জানেন না তার তৈরি কম্পোনেন্টের সাথে অন্য কম্পোনেন্ট যুক্ত হলে কী সমস্যা হতে পারে।
* Assumption-গুলো ঠিকমতো ডকুমেন্ট করে না।

## 3. How It Works

1. Developer অ্যাপ্লিকেশনের জন্য কিছু strict business rules সেট করে (যেমন: ব্যালেন্স চেক, 2FA ভেরিফিকেশন)।
2. Developer ধরে নেয় ইউজাররা স্বাভাবিক সিকোয়েন্স ও নিয়ম মেনেই কাজ করবে।
3. Attacker intercepting proxy ব্যবহার করে সাধারণ ইউজারের মতো আচরণ না করে ইনপুট বা রিকোয়েস্ট পরিবর্তন করে।
4. Application সেই অস্বাভাবিক অবস্থা (unusual state) সঠিকভাবে handle করতে পারে না কারণ এর জন্য কোনো সার্ভার-সাইড লজিক লেখা নেই।
5. Security rule বা business constraint বাইপাস হয়ে যায়।
6. Attacker অপ্রত্যাশিত সুবিধা বা ডেটা এক্সেস পেয়ে যায়।

## 4. Simple Example

**Domain-Specific Flaw (Discount Logic):**
ধরা যাক একটি অনলাইন শপে ১০০০ ডলারের ওপর কেনাকাটা করলে ১০% ডিসকাউন্ট দেওয়া হয়।

*Normal workflow:* ইউজার ১০০০ ডলারের প্রোডাক্ট কার্টে অ্যাড করে, ১০% ডিসকাউন্ট পায় এবং অর্ডার প্লেস করে।

*Attacker workflow:* অ্যাটাকার কার্টে ১০০০ ডলারের প্রোডাক্ট অ্যাড করে ডিসকাউন্ট ট্রিগার করে। এরপর অর্ডার প্লেস করার আগে কার্ট থেকে কিছু প্রোডাক্ট রিমুভ করে দেয়।

*Result:* অ্যাপ্লিকেশনটি অর্ডার প্লেস করার সময় নতুন করে চেক করে না যে কার্টের ভ্যালু এখনো ১০০০ ডলার আছে কি না। ফলে অ্যাটাকার কম টাকার অর্ডারেও ডিসকাউন্ট পেয়ে যায়।

## 5. Technical Example

**Failing to handle unconventional input (Funds Transfer):**

```php
$transferAmount = $_POST['amount'];
$currentBalance = $user->getBalance();

if ($transferAmount <= $currentBalance) {
    // Complete the transfer
} else {
    // Block the transfer: insufficient funds
}

```

*Explanation:*
এখানে অ্যাপ্লিকেশন চেক করছে ইউজারের ব্যালেন্স ট্রান্সফার অ্যামাউন্টের চেয়ে বেশি কি না। কিন্তু ডেভেলপার assumption করেছে যে ইউজার সবসময় পজিটিভ সংখ্যাই (যেমন ১০০) দেবে। অ্যাটাকার যদি `amount` প্যারামিটারে `-1000` দেয়, লজিক চেক করবে `-1000 <= $currentBalance`, যা সবসময় true হবে। ফলে ব্যালেন্স চেক বাইপাস হয়ে যাবে। উল্টো, অ্যাটাকারের অ্যাকাউন্ট থেকে ভিকটিমের অ্যাকাউন্টে টাকা যাওয়ার বদলে ভিকটিমের অ্যাকাউন্ট থেকে অ্যাটাকারের অ্যাকাউন্টে ১০০০ ডলার চলে আসতে পারে।

## 6. Attack Flow

Attacker
↓
Understand the application logic and business domain
↓
Identify flawed assumptions made by developers (e.g., relying on UI, sequence, mandatory params)
↓
Deviate from expected user behavior using an intercepting proxy
↓
Application fails to take appropriate steps for this unexpected state
↓
Business rules are circumvented
↓
Unintended action is executed (e.g., logic bypassed, funds manipulated)

## 7. Important Conditions / Requirements

* User input controllable হতে হবে (যাতে Intercepting proxy দিয়ে ডেটা পরিবর্তন করা যায়)।
* সার্ভার-সাইডে পর্যাপ্ত ইনপুট ভ্যালিডেশন বা স্টেট চেকিংয়ের ঘাটতি থাকতে হবে।
* অ্যাটাকারকে অ্যাপ্লিকেশনের Business domain বা functionality বুঝতে হবে।

## 8. Types / Variations

PortSwigger অনুযায়ী লজিক ফ্ল-গুলো মূলত ডেভেলপারদের করা সাধারণ কিছু ভুলের ওপর ভিত্তি করে কয়েক ভাগে ভাগ করা যায়:

### A. Excessive Trust in Client-Side Controls

**What is it?** অ্যাপ্লিকেশন ধরে নেয় ইউজার শুধু ওয়েব ইন্টারফেস দিয়ে রিকোয়েস্ট পাঠাবে, তাই সব ভ্যালিডেশন ক্লায়েন্ট-সাইডেই করা হয়।
**How it works:** অ্যাটাকার Burp Proxy দিয়ে ব্রাউজার থেকে রিকোয়েস্ট বের হওয়ার পর এবং সার্ভারে পৌঁছানোর আগে ডেটা পরিবর্তন করে দেয় (tampering), ফলে ক্লায়েন্ট-সাইড কন্ট্রোল পুরোপুরি অকেজো হয়ে যায়।

### B. Failing to Handle Unconventional Input

**What is it?** বিজনেস রুল অনুযায়ী ইনপুটের কিছু লিমিট বা ধরন থাকে। কিন্তু আন-এক্সপেক্টেড ডেটা আসলে কী হবে তার কোনো লজিক না থাকা।
**Example:** অনলাইনে প্রোডাক্ট অর্ডারের সময় ইনপুট হিসেবে নেগেটিভ ভ্যালু (-10) দেওয়া বা বিশাল বড় টেক্সট স্ট্রিং দেওয়া। সার্ভার-সাইডে এটি হ্যান্ডেল না করলে অপ্রত্যাশিত কাজ হতে পারে।

### C. Making Flawed Assumptions About User Behavior

এখানে বেশ কয়েকটি সাব-ক্যাটাগরি আছে:

1. **Trusted users won't always remain trustworthy:** শুরুতে strict control থাকলেও অ্যাপ্লিকেশন ধরে নেয় ইউজার সবসময় বিশ্বস্ত থাকবে, তাই পরবর্তীতে lax (শিথিল) চেকিং করা হয়। এর ফলে inconsistent security controls তৈরি হয়।
2. **Users won't always supply mandatory input:** ডেভেলপার ভাবে mandatory প্যারামিটার সব সময় আসবে। অ্যাটাকার যদি প্যারামিটারের নাম এবং ভ্যালু পুরোপুরি মুছে দিয়ে রিকোয়েস্ট পাঠায়, তবে সার্ভার হয়তো অন্য কোড পাথ (code path) এক্সিকিউট করতে পারে।
3. **Users won't always follow the intended sequence:** ওয়ার্কফ্লোতে স্টেপগুলো পরপর আসবে বলে ধরে নেওয়া। যেমন: 2FA-এর ক্ষেত্রে লগইন করার পর ভেরিফিকেশন পেজে না গিয়ে forced browsing-এর মাধ্যমে সরাসরি ড্যাশবোর্ডে চলে যাওয়া।

### D. Domain-Specific Flaws

**What is it?** বিজনেস ডোমেইনের নিজস্ব নিয়মে ত্রুটি।
**Example:** অনলাইন শপে ডিসকাউন্ট লজিক (যা আগে উদাহরণে দেওয়া হয়েছে) বা সোশ্যাল মিডিয়ায় প্রচুর ফলোয়ার তৈরি করার মতো ডোমেইন-নির্দিষ্ট সুবিধা নেওয়া। এর জন্য ডোমেইন সম্পর্কে গভীর জ্ঞান থাকা প্রয়োজন।

### E. Providing an Encryption Oracle

**What is it?** অ্যাপ্লিকেশন যখন ইউজার-কন্ট্রোলড ইনপুট এনক্রিপ্ট করে তার ciphertext ইউজারকেই আবার দেখায়।
**Impact:** অ্যাটাকার এই "encryption oracle" ব্যবহার করে নিজের ইচ্ছামতো ডেটা এনক্রিপ্ট করে নিতে পারে এবং অ্যাপ্লিকেশনের অন্য সেনসিটিভ ফাংশনে সেই এনক্রিপ্টেড ইনপুট পাঠাতে পারে। যদি decryption oracle-ও থাকে, তবে তা আরও ক্ষতিকর।

### F. Email Address Parser Discrepancies

**What is it?** অ্যাপ্লিকেশনের বিভিন্ন অংশে ইমেইল অ্যাড্রেস পার্স (parse) করার নিয়মে অসামঞ্জস্যতা থাকা।
**Impact:** অ্যাটাকার এনকোডিং টেকনিক ব্যবহার করে এমন ইমেইল অ্যাড্রেস তৈরি করতে পারে যা প্রাথমিক ভ্যালিডেশন পাস করে যায়, কিন্তু সার্ভারের পার্সিং লজিক তাকে অন্যভাবে ইন্টারপ্রেট করে। এর মাধ্যমে রেস্ট্রিক্টেড ডোমেইনের ইমেইল ভান করে আনঅথোরাইজড এক্সেস পাওয়া যেতে পারে।

## 9. Different Contexts

Business logic vulnerabilities অ্যাপ্লিকেশনের বিভিন্ন জায়গায় হতে পারে:

* **Transactions:** পেমেন্ট বা ফান্ড ট্রান্সফারের প্যারামিটার।
* **Workflows/Sequences:** মাল্টি-স্টেপ প্রসেস (যেমন: Checkout, 2FA, Password Reset)।
* **Data Types:** Number field, string field, email field।
* **Encryption Inputs:** যেখানে অ্যাপ্লিকেশন ডেটা এনক্রিপ্ট/ডিক্রিপ্ট করে।

## 10. Exploitation / Practical Understanding

এই দুর্বলতাগুলো এক্সপ্লয়েট করার জন্য Payload মুখস্থ করার কিছু নেই, বরং অ্যাপ্লিকেশন লজিক নিয়ে এক্সপেরিমেন্ট করতে হয়:

* **Unconventional Input:** exceptionally high বা low numeric input দিন, অপ্রত্যাশিত ডেটা টাইপ দিন। লক্ষ্য করুন লিমিট ক্রস করলে কী হয় বা ইনপুট নরমালাইজ হচ্ছে কি না।
* **Parameter Removal:** একটি করে প্যারামিটার রিমুভ করুন। শুধু value নয়, প্যারামিটারের নামটিও ডিলিট করে রিকোয়েস্ট পাঠান।
* **Sequence Manipulation (Forced Browsing):** মাল্টি-স্টেপ প্রসেস সম্পন্ন না করে নির্দিষ্ট URL-এ ডিরেক্ট রিকোয়েস্ট পাঠান (যেমন GET/POST রিকোয়েস্ট দিয়ে)। আগের স্টেপে ফিরে যান বা একই স্টেপ একাধিকবার অ্যাক্সেস করুন।

*Observation:* এ ধরনের টেস্ট করার সময় অ্যাপ্লিকেশন অনেক সময় exception বা error দিতে পারে। Error message বা debug info খুব সতর্কতার সাথে পড়তে হবে, কারণ এগুলো থেকে ব্যাক-এন্ডের গুরুত্বপূর্ণ তথ্য লিক হতে পারে।

## 11. Step-by-Step Exploitation (Sequence / Workflow Bypass)

Step 1 — Identify the intended sequence: অ্যাপ্লিকেশনের স্বাভাবিক ফ্লো চিহ্নিত করুন (যেমন: Login → 2FA → Dashboard)।
Step 2 — Test the behavior: স্বাভাবিকভাবে প্রথম স্টেপ (Login) কমপ্লিট করুন।
Step 3 — Confirm the assumptions: অ্যাপ্লিকেশন আপনাকে 2FA পেজে রিডাইরেক্ট করবে।
Step 4 — Determine required conditions: লক্ষ্য করুন ড্যাশবোর্ডে যাওয়ার রিকোয়েস্ট বা URL কী।
Step 5 — Exploit the vulnerability: Forced browsing ব্যবহার করে 2FA পেজ স্কিপ করে সরাসরি Dashboard-এর URL-এ রিকোয়েস্ট পাঠান।
Step 6 — Verify the result: যদি সার্ভার-সাইডে 2FA কমপ্লিট হয়েছে কি না তা চেক করার লজিক না থাকে, তবে আপনি সরাসরি ড্যাশবোর্ডে অ্যাক্সেস পেয়ে যাবেন।

## 12. Burp Suite Workflow

* **Proxy (Intercept):** ক্লায়েন্ট-সাইড ভ্যালিডেশন বাইপাস করে ব্রাউজার থেকে যাওয়া ডেটা (যেমন প্রাইস, কোয়ান্টিটি) ম্যানিপুলেট করতে।
* **Repeater:** একই রিকোয়েস্ট বারবার মডিফাই করে পাঠাতে। যেমন: প্যারামিটার রিমুভ করা, unconventional ভ্যালু ইনপুট দেওয়া, এবং forced browsing-এর মাধ্যমে সিকোয়েন্স ব্রেক করা।

## 13. How to Identify / Detect

* অটোমেটেড ভালনারেবিলিটি স্ক্যানার দিয়ে এগুলো ধরা কঠিন। Manual testing সবচেয়ে কার্যকর।
* Burp Repeater দিয়ে প্রতিটি প্যারামিটার মুছে দিয়ে বা ভ্যালু পরিবর্তন করে অ্যাপ্লিকেশনের response observe করতে হবে।
* ইউজার ইন্টারফেসের বাইরে গিয়ে অ্যাপ্লিকেশনকে আন-এক্সপেক্টেড স্টেটে ফেলার চেষ্টা করে error বা response-এর পার্থক্য চেক করতে হবে।

## 14. Common Mistakes

* ডেভেলপারদের ক্লায়েন্ট-সাইড কন্ট্রোলের ওপর অতিরিক্ত বিশ্বাস করা।
* ইউজার সবসময় mandatory input ফিল্ড পূরণ করবেই—এমন ধারণা করা।
* মাল্টি-স্টেপ প্রসেসে ইউজার সব সময় সঠিক স্টেপ (sequence) ফলো করবে—এটা ধরে নিয়ে সার্ভার-সাইডে স্টেট চেক না করা।

## 15. Limitations

* Automated vulnerability scanner দিয়ে এগুলো সহজে বের করা যায় না।
* অ্যাপ্লিকেশনের business domain সম্পর্কে যথেষ্ট জ্ঞান (human knowledge) না থাকলে অনেক সময় বিপজ্জনক লজিক ফ্ল-ও চোখ এড়িয়ে যেতে পারে বা এর ইমপ্যাক্ট বোঝা যায় না।

## 16. Edge Cases / Important Details

* **Multiple Functions in One Script:** অনেক সময় একই সার্ভার-সাইড স্ক্রিপ্টে অনেকগুলো ফাংশন থাকে। একটি প্যারামিটার মুছে দিলে সেটি অন্য কোড পাথ (code path) এক্সিকিউট করতে পারে যা ইউজারের অ্যাক্সেস করার কথা নয়।
* **Obscure Domains:** খুব অপরিচিত বা জটিল ডোমেইনের ক্ষেত্রে বাগ মিস করার সম্ভাবনা বেশি থাকে। তাই ডকুমেন্টেশন পড়া বা subject-matter expert-দের সাথে কথা বলা জরুরি।

## 17. Impact

* Impact সাধারণত trivial থেকে শুরু করে high-severity পর্যন্ত হতে পারে, যা নির্ভর করে কোন ফাংশনালিটিতে ফ্ল আছে তার ওপর।
* **Authentication Flaws:** Privilege escalation বা authentication bypass হতে পারে।
* **Financial Flaws:** ফিন্যান্সিয়াল ট্রানজেকশনে ত্রুটি থাকলে বিজনেসের massive loss, চুরি বা ফ্রড হতে পারে।
* অনেক সময় অ্যাটাকারের নিজের সরাসরি লাভ না হলেও সে বিজনেসের বড় ধরনের ক্ষতি (damage) করতে পারে।

## 18. Prevention / Mitigation

এই দুর্বলতাগুলো প্রতিরোধের মূল মন্ত্র হলো: **ডেভেলপার এবং টেস্টারদের ডোমেইন সম্পর্কে পরিষ্কার ধারণা থাকা এবং ইউজার বিহেভিয়ার নিয়ে কোনো implicit assumption না করা।**

* **Server-side State Validation:** সার্ভার-সাইডে স্টেট এবং ইনপুট সেন্সিবল কি না, তা যাচাই করার লজিক ইমপ্লিমেন্ট করতে হবে।
* **Documentation:** সব ট্রানজেকশন এবং ওয়ার্কফ্লোর জন্য clear design document এবং data flow মেইনটেইন করতে হবে এবং কোথায় কী assumption করা হচ্ছে তা নোট করে রাখতে হবে।
* **Clear Code:** কোড পরিষ্কারভাবে লিখতে হবে যেন লজিক সহজেই বোঝা যায়। জটিল ক্ষেত্রে ক্লিয়ার ডকুমেন্টেশন থাকা বাধ্যতামূলক।
* **Analyze Dependencies:** কোনো কম্পোনেন্ট অন্য কোডে কীভাবে ব্যবহৃত হচ্ছে এবং আন-ইউজুয়াল ম্যানিপুলেশনে তার side-effects কী হতে পারে, তা লক্ষ্য করতে হবে।

## 19. Vulnerable vs Secure Example

**Vulnerable:**

```php
if ($transferAmount <= $currentBalance) {
    // Complete the transfer
}

```

*Why vulnerable?* এখানে ধরে নেওয়া হয়েছে ইনপুট সবসময় পজিটিভ হবে। নেগেটিভ ভ্যালু দিলে লজিক বাইপাস হয়ে যায়।

**Secure:**
(Source-এ explicit secure code না থাকলেও, prevention logic অনুযায়ী)

```php
if ($transferAmount > 0 && $transferAmount <= $currentBalance) {
    // Complete the transfer
}

```

*Why secure?* সার্ভার-সাইডে explicitly চেক করা হচ্ছে ইনপুটটি সেন্সিবল (পজিটিভ) কি না।

## 20. Real Understanding

Business logic vulnerability আসলে কোনো হ্যাকিং ম্যাজিক নয়; এটি হলো অ্যাপ্লিকেশনের "বিশ্বাস"-এর (trust) দুর্বলতা। অ্যাপ্লিকেশন যখন ইউজারকে অতিরিক্ত বিশ্বাস করে এবং ধরে নেয় যে সে নিয়ম ভাঙবে না, তখনই সমস্যাটি তৈরি হয়। অ্যাটাকার কোনো স্পেশাল পেলোড দেয় না, সে শুধু অ্যাপ্লিকেশনের নিয়মগুলো ভাঙে (যেমন স্টেপ স্কিপ করা, প্যারামিটার মুছে দেওয়া, অদ্ভুত ভ্যালু দেওয়া)। অ্যাপ্লিকেশন যখন সেই অপ্রত্যাশিত পরিস্থিতি হ্যান্ডেল করতে পারে না, তখনই অ্যাটাকার সফল হয়।

## 21. Mental Model

Developer defines strict business rules assuming standard user behavior.
↓
Application relies on client-side controls or implicit sequence assumptions.
↓
Attacker uses proxy to manipulate input, skip steps, or send unconventional values.
↓
Application lacks explicit server-side logic to handle this specific unexpected state.
↓
Business constraints are bypassed → Unintended action succeeds (Fraud, Bypass).

## 22. Quick Revision

* **Definition:** অ্যাপ্লিকেশনের ডিজাইন/নিয়মে ত্রুটি যা অ্যাটাকারকে অনাকাঙ্ক্ষিত কাজ করতে দেয়।
* **Cause:** ইউজার বিহেভিয়ার নিয়ে ডেভেলপারদের ভুল ধারণা (flawed assumptions)।
* **Main idea:** বিজনেস রুল বা লজিক বাইপাস করা।
* **Attack flow:** Identify assumptions → Violate them (tamper input, skip steps) → Exploit.
* **Important condition:** Server-side ভ্যালিডেশনের অভাব এবং proxy দিয়ে ইনপুট কন্ট্রোল করার সুযোগ।
* **Detection:** Manual testing, Burp Proxy/Repeater দিয়ে parameter remove বা forced browsing করা।
* **Impact:** Trivial থেকে শুরু করে Authentication bypass বা Massive financial loss।
* **Prevention:** Clear documentation, server-side validation, never trust client-side controls.

## 23. Things to Remember

* Automated scanner দিয়ে Logic flaws সহজে ধরা যায় না; manual test আবশ্যিক।
* Client-side controls-এর ওপর কখনোই পুরোপুরি নির্ভর করা যাবে না।
* টেস্টিংয়ের সময় প্যারামিটারের ভ্যালু এবং নাম—দুটোই মুছে দিয়ে দেখতে হবে রেসপন্স কী আসে।
* Forced browsing করে নির্দিষ্ট sequence (যেমন 2FA) ভাঙার চেষ্টা করতে হবে।
* টেস্টিংয়ের সময় আসা error message বা debug information খুব মনোযোগ দিয়ে পড়তে হবে।
* Encryption oracle থাকলে তা দিয়ে অন্য সেনসিটিভ ডেটা এনক্রিপ্ট করা যায় কি না চেক করতে হবে।

