# Race Conditions

## 1. What is Race Condition?

**Short Definition:**
Race condition হলো এমন একটি vulnerability যা তখন ঘটে যখন কোনো ওয়েবসাইট পর্যাপ্ত safeguards ছাড়াই একাধিক রিকোয়েস্ট একই সাথে (concurrently) প্রসেস করে।

**Detailed Explanation:**
সাধারণত ওয়েবসাইটগুলো একই ডেটাবেস শেয়ার করে একাধিক থ্রেডের (threads) মাধ্যমে concurrent রিকোয়েস্ট হ্যান্ডেল করে। যখন একাধিক থ্রেড একই সময়ে একই ডেটার সাথে ইন্টারঅ্যাক্ট করে, তখন একটি "collision" বা সংঘর্ষ তৈরি হয়, যা অ্যাপ্লিকেশনে unintended behavior বা অনাকাঙ্ক্ষিত ফলাফল সৃষ্টি করে। যে অতি ক্ষুদ্র সময়ের জন্য এই collision ঘটানো সম্ভব (যেমন ডেটাবেসের সাথে দুটি ইন্টারঅ্যাকশনের মাঝখানের এক সেকেন্ডের ভগ্নাংশ), তাকে "Race window" বলা হয়। একজন attacker অত্যন্ত সতর্কতার সাথে টাইমিং মিলিয়ে রিকোয়েস্ট পাঠিয়ে ইচ্ছাকৃতভাবে এই collision ঘটায় এবং লজিক বাইপাস করে।

## 2. Why Does It Happen?

Race condition ঘটার মূল কারণ হলো অ্যাপ্লিকেশন কোড লেখার সময় concurrency বা একসাথে একাধিক রিকোয়েস্ট আসার ঝুঁকির কথা মাথায় না রাখা।

* **TOCTOU (Time-of-check to time-of-use):** সিকিউরিটি চেক করা এবং সেই ডেটা ব্যবহার করে অ্যাকশন নেওয়ার মাঝখানে একটি সময়ের গ্যাপ থাকে।
* **Flawed assumption ("Requests are atomic"):** ডেভেলপাররা প্রায়ই ধরে নেন যে একটি রিকোয়েস্ট সম্পূর্ণ শেষ হওয়ার পরই আরেকটি রিকোয়েস্ট প্রসেস হবে। কিন্তু বাস্তবে "everything is multi-step"। একটি সাধারণ HTTP রিকোয়েস্ট প্রসেস হওয়ার সময়ও অ্যাপ্লিকেশন কিছু ক্ষণস্থায়ী বা লুকানো স্টেটের (sub-states) মধ্য দিয়ে যায়।

## 3. How It Works

1. User (Attacker) অত্যন্ত সূক্ষ্ম টাইমিং ব্যবহার করে একই সাথে একাধিক রিকোয়েস্ট পাঠায়।
2. Application রিকোয়েস্টগুলোকে আলাদা আলাদা থ্রেডে প্রসেস করা শুরু করে।
3. Thread A একটি ভ্যালিডেশন চেক করে (যেমন: ডিসকাউন্ট কোড আগে ব্যবহৃত হয়েছে কি না)।
4. Thread A ডেটাবেস আপডেট করার আগেই (race window-এর মধ্যে), Thread B একই ভ্যালিডেশন চেক করে।
5. যেহেতু ডেটাবেস এখনো আপডেট হয়নি, Thread B-ও চেক পাস করে যায়।
6. Security control বাইপাস হয়ে যায়।
7. Attacker একই কাজ (যেমন: একই ডিসকাউন্ট কোড দু'বার ব্যবহার) একাধিকবার করতে সক্ষম হয়।

## 4. Simple Example

**Limit Overrun Race Condition (Discount Code):**
ধরা যাক, একটি অনলাইন স্টোরে ওয়ান-টাইম ডিসকাউন্ট কোড ব্যবহারের সুযোগ আছে।
*Normal process:* কোড চেক করা হয় -> অর্ডারে ডিসকাউন্ট দেওয়া হয় -> ডেটাবেস আপডেট করে কোডটি 'ব্যবহৃত' মার্ক করা হয়।
*Modified request (Attack):* Attacker একই সাথে দুটি রিকোয়েস্ট পাঠায়।
*Result:* ডেটাবেস আপডেট হওয়ার আগের সেই ক্ষুদ্র Race window-তে দুটি রিকোয়েস্টই চেক পাস করে ফেলে এবং একই অর্ডারে দু'বার ডিসকাউন্ট অ্যাপ্লাই হয়ে যায়।

## 5. Technical Example

**Single-endpoint Collision (Password Reset):**
ধরা যাক, একটি পাসওয়ার্ড রিসেট মেকানিজম ইউজার আইডি এবং রিসেট টোকেন ইউজারের সেশনে স্টোর করে।
*Request 1:* `POST /reset` (username=victim)
*Request 2:* `POST /reset` (username=attacker)

যদি রিকোয়েস্ট দুটি একই সেশন থেকে একই সাথে পাঠানো হয়, তবে এমন একটি collision হতে পারে যেখানে ফাইনাল স্টেট দাঁড়ায়:

```
session['reset-user'] = victim
session['reset-token'] = 1234

```

এখানে সেশনে ভিকটিমের ইউজার আইডি স্টোর হয়েছে, কিন্তু ভ্যালিড রিসেট টোকেনটি অ্যাটাকারের ইমেইলে চলে গেছে।

## 6. Attack Flow

Attacker
↓
Predict potential collisions (Identify security controls, objects, and how state is stored)
↓
Probe for clues (Benchmark normal behavior vs Parallel requests)
↓
Observe deviations (Changes in responses, timings, or second-order effects like emails)
↓
Overcome network/server jitter using Single-packet attack or Connection warming
↓
Cause an intentional collision during the race window
↓
Exploit unintended sub-states or bypass business logic limits

## 7. Important Conditions / Requirements

* রিকোয়েস্টগুলো প্রসেস হওয়ার মাঝখানে অবশ্যই একটি "Race window" (সময়ের গ্যাপ) থাকতে হবে।
* অন্তত দুটি রিকোয়েস্ট একই ডেটা বা রেকর্ডের ওপর কাজ করতে হবে (Collision potential)।
* Network jitter এড়িয়ে রিকোয়েস্টগুলোকে ঠিক একই সময়ে সার্ভারে পৌঁছানোর ব্যবস্থা থাকতে হবে (যেমন Single-packet attack)।

## 8. Types / Variations

### A. Limit Overrun Race Conditions

**What is it?** অ্যাপ্লিকেশন লজিকের কোনো লিমিট ক্রস করা। এটি TOCTOU (Time-of-check to time-of-use)-এর একটি সাবটাইপ।
**Examples:** গিফট কার্ড একাধিকবার রিডিম করা, একটি প্রোডাক্টে একাধিক রেটিং দেওয়া, ব্যালেন্সের চেয়ে বেশি টাকা ট্রান্সফার করা, রেট-লিমিট বা CAPTCHA বাইপাস করা।

### B. Hidden Multi-step Sequences (Sub-states)

**What is it?** একটি রিকোয়েস্ট প্রসেস হওয়ার সময় অ্যাপ্লিকেশন সাময়িক কিছু স্টেটের (sub-states) মধ্য দিয়ে যায়।
**How it works:** যেমন, লগইন করার সময় অ্যাপ্লিকেশন সাময়িকভাবে সেশনে ভ্যালিড লগইন স্টেট তৈরি করে কিন্তু MFA এনফোর্স করার আগেই। এই অল্প সময়ের গ্যাপে অন্য রিকোয়েস্ট পাঠিয়ে MFA বাইপাস করা সম্ভব হতে পারে।

### C. Multi-endpoint Race Conditions

**What is it?** একসাথে একাধিক আলাদা এন্ডপয়েন্টে রিকোয়েস্ট পাঠিয়ে লজিক ব্রেক করা।
**Example:** একটি রিকোয়েস্টে পেমেন্ট ভ্যালিডেট করা হচ্ছে, এবং ঠিক সেই সময়েই (অর্ডার কনফার্ম হওয়ার আগে) অন্য একটি রিকোয়েস্ট দিয়ে কার্টে নতুন আইটেম অ্যাড করা হচ্ছে।

### D. Single-endpoint Race Conditions

**What is it?** একই এন্ডপয়েন্টে আলাদা ভ্যালু দিয়ে একসাথে রিকোয়েস্ট পাঠানো।
**Example:** একই সাথে দুটি ভিন্ন ইমেইলে ইমেইল চেঞ্জ বা পাসওয়ার্ড রিসেট রিকোয়েস্ট পাঠানো। ইমেইল পাঠানো সাধারণত ব্যাকগ্রাউন্ড থ্রেডে হয়, ফলে race condition-এর সম্ভাবনা বেড়ে যায়।

### E. Partial Construction Race Conditions

**What is it?** যখন কোনো অবজেক্ট একাধিক ধাপে ডেটাবেসে তৈরি হয়, তখন মাঝখানের সময়ে অবজেক্টটি uninitialized (যেমন null বা ফাঁকা) থাকে।
**How it works:** Attacker ইনপুটে এমন ভ্যালু দেয় যা ওই uninitialized ভ্যালুর সাথে ম্যাচ করে। যেমন, API key ইনিশিয়ালাইজ হওয়ার আগে ফাঁকা (empty array বা null) ভ্যালু দিয়ে অথেনটিকেটেড রিকোয়েস্ট করা।

### F. Deferred Collisions

**What is it?** এখানে রিকোয়েস্টের সাথে সাথেই কলিশন হয় না। অ্যাপ্লিকেশন ব্যাকগ্রাউন্ডে ব্যাচ প্রসেস করার সময় (যেমন ২০ মিনিট পর) কলিশন ঘটে। এর জন্য সিঙ্ক্রোনাইজড রিকোয়েস্টের প্রয়োজন হয় না।

### G. Time-sensitive Attacks

**What is it?** যখন অ্যাপ্লিকেশন সিকিউরিটি টোকেন জেনারেট করার জন্য Cryptographically secure random string-এর বদলে High-resolution timestamp ব্যবহার করে।
**How it works:** একই সময়ে রিকোয়েস্ট পাঠালে সার্ভার একই টাইমস্ট্যাম্প পায় এবং একই টোকেন দুটি ভিন্ন রিকোয়েস্টের জন্য জেনারেট করে।

## 9. Different Contexts

* **Databases:** সাধারণত Limit overrun এবং TOCTOU ভালনারেবিলিটিগুলো ডেটাবেস লেয়ারে হয়।
* **Sessions:** সেশন ভেরিয়েবল আপডেট করার সময় (বিশেষ করে যখন সেশন ব্যাচ আপডেট না করে রিয়েল-টাইমে আপডেট হয়)।
* **Background Threads:** ইমেইল পাঠানো বা অন্যান্য হেভি টাস্ক সাধারণত ব্যাকগ্রাউন্ড থ্রেডে হয়, যা Race condition-এর জন্য অত্যন্ত ভালো টার্গেট।

## 10. Exploitation / Practical Understanding

**Single-packet Attack (Bypassing Network Jitter):**
Race condition অ্যাটাকের সবচেয়ে বড় বাধা হলো Network jitter (প্যাকেট পৌঁছাতে সময়ের হেরফের)। HTTP/2 এর সুবিধা নিয়ে ২০-৩০টি রিকোয়েস্টের ডেটা একটিমাত্র TCP প্যাকেটে পাঠানো যায়। প্রথমে রিকোয়েস্টের মূল অংশ পাঠিয়ে দেওয়া হয় এবং শেষ বাইট আটকে রাখা হয়। এরপর একটিমাত্র প্যাকেটে সবগুলোর শেষ ফ্রেম একসাথে পাঠানো হয়। এতে সার্ভারে সবগুলো রিকোয়েস্ট ঠিক একই মিলি-সেকেন্ডে প্রসেস শুরু হয়। HTTP/1-এর ক্ষেত্রে "last-byte sync" টেকনিক ব্যবহার করা হয়।

**Connection Warming (Bypassing Back-end Delays):**
Multi-endpoint অ্যাটাকে ফ্রন্ট-এন্ড সার্ভার যখন ব্যাক-এন্ডের সাথে নতুন কানেকশন তৈরি করে, তখন একটি delay হয়। এই delay দূর করার জন্য মূল অ্যাটাক রিকোয়েস্টগুলোর আগে কিছু সাধারণ রিকোয়েস্ট (যেমন হোমপেজে GET রিকোয়েস্ট) পাঠিয়ে কানেকশন "Warm up" করে নেওয়া হয়।

**Abusing Rate or Resource Limits:**
Multi-endpoint অ্যাটাকে যদি একটি এন্ডপয়েন্ট দ্রুত প্রসেস হয়ে যায়, তবে প্রচুর ডামি রিকোয়েস্ট পাঠিয়ে সার্ভারের Rate limit ট্রিগার করা যায়। এতে সার্ভার-সাইডে একটি delay তৈরি হয়, যা দুটি ভিন্ন এন্ডপয়েন্টের Race window মেলানোর জন্য কাজে লাগে।

## 11. Step-by-Step Exploitation (Methodology)

Step 1 — **Predict potential collisions:** এমন সিকিউরিটি ক্রিটিকাল এন্ডপয়েন্ট খুঁজুন যা একই ডেটাবেস রেকর্ড বা সেশন এডিট (append নয়) করে।
Step 2 — **Probe for clues (Benchmark):** Burp Repeater-এ রিকোয়েস্টগুলো "Send group in sequence" দিয়ে পাঠিয়ে সাধারণ বিহেভিয়ার নোট করুন।
Step 3 — **Probe for clues (Parallel):** এবার "Send group in parallel" ব্যবহার করে সবগুলো রিকোয়েস্ট একসাথে পাঠান (Single-packet attack)।
Step 4 — **Look for clues:** রেসপন্সের স্ট্যাটাস কোড, প্রসেসিং টাইম বা ইমেইল কন্টেন্টের কোনো অস্বাভাবিক পরিবর্তন (Deviation) বা second-order effect লক্ষ্য করুন।
Step 5 — **Prove the concept:** যদি কোনো sub-state বা কলিশনের ক্লু পান, তবে অপ্রয়োজনীয় রিকোয়েস্ট বাদ দিয়ে শুধুমাত্র দুটি রিকোয়েস্ট দিয়ে টাইমিং পারফেক্ট করে অ্যাটাকটি সফল করুন।
Step 6 — **Verify the result:** লজিক বাইপাস বা অপ্রত্যাশিত এক্সেস পাওয়া গেছে কি না তা চেক করুন।

## 12. HTTP Requests / Responses

**Array Injection for Partial Construction:**

Ruby on Rails-এ `param[key]` বা PHP-তে `param[]` দিয়ে empty array বা nil ভ্যালু পাঠানো যায়, যা partial construction race condition-এ ডাটাবেসের uninitialized ভ্যালুর সাথে ম্যাচ করাতে কাজে লাগে।

Example API Request during Race Window:

```http
GET /api/user/info?user=victim&api-key[]= HTTP/2
Host: vulnerable-website.com

```

*Explanation:* এখানে `api-key[]=` পাঠানো হয়েছে। অবজেক্ট ক্রিয়েট হওয়ার সময় API key যখন uninitialized (null বা empty) থাকে, তখন এই empty array-টি সেই uninitialized স্টেটের সাথে ম্যাচ করে গিয়ে অথেনটিকেশন বাইপাস করতে পারে।

## 13. Payloads

* **PHP Array Syntax:** `param[]=foo` (Equivalent to `param = ['foo']`)
* **PHP Multiple Array:** `param[]=foo&param[]=bar` (Equivalent to `param = ['foo', 'bar']`)
* **PHP Empty Array:** `param[]=` (Equivalent to `param = []`)
* **Ruby on Rails Nil Value:** `param[key]=` (Results in `params = {"param"=>{"key"=>nil}}`)

## 14. Burp Suite Workflow

* **Repeater (Group Send Options):**
* `Send group in sequence (single connection)`: Connection warming এবং timing attack-এর জন্য। এটি TCP কানেকশন তৈরির jitter কমায়।
* `Send group in parallel`: Race condition ট্রিগার করার জন্য। Burp স্বয়ংক্রিয়ভাবে HTTP/1-এর জন্য Last-byte sync এবং HTTP/2-এর জন্য Single-packet attack ব্যবহার করে।


* **Turbo Intruder:**
* অনেক বেশি রিকোয়েস্ট, staggered request timing বা জটিল অ্যাটাকের জন্য এটি ব্যবহৃত হয়।
* HTTP/2 Single-packet attack করার জন্য পাইথন স্ক্রিপ্টে `engine=Engine.BURP2` এবং `concurrentConnections=1` সেট করে `gate` এর মাধ্যমে রিকোয়েস্টগুলো কিউ (queue) করে একসাথে রিলিজ করতে হয়।


```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                            concurrentConnections=1,
                            engine=Engine.BURP2
                            )
    for i in range(20):
        engine.queue(target.req, gate='1')
    engine.openGate('1')

```



## 15. How to Identify / Detect

* **Anomalies/Clues:** রেসপন্সের যেকোনো ছোটখাটো পরিবর্তনই একটি ক্লু।
* **Request Processing Time:** রিকোয়েস্টের প্রসেসিং টাইম যদি প্রত্যাশার চেয়ে অনেক কম হয়, তবে বুঝতে হবে ডেটা ব্যাকগ্রাউন্ড থ্রেডে পাস করা হচ্ছে (যা Race condition-এর জন্য ভালো)।
* **Second-order Effects:** রেসপন্সে কিছু না বুঝলেও পরে ইমেইলে (যেমন, অন্য ইমেইলে কনফার্মেশন কোড যাওয়া) বা অ্যাপ্লিকেশনের বিহেভিয়ারে কোনো পরিবর্তন দেখা গেলে।
* **Benchmarking:** বেঞ্চমার্ক না করলে আপনি বুঝতেই পারবেন না কোনটি স্বাভাবিক রেসপন্স আর কোনটি রেস কন্ডিশনের ক্লু।

## 16. Common Mistakes

* Network jitter-এর কারণে রিকোয়েস্ট ঠিক সময়ে না পৌঁছানোকে "Vulnerable নয়" ভেবে ভুল করা। (অবশ্যই Single-packet attack বা parallel group send ব্যবহার করতে হবে)।
* "Requests are atomic" - এই ভুল ধারণা রাখা। প্রতিটি রিকোয়েস্ট অ্যাপ্লিকেশনকে বিভিন্ন ক্ষণস্থায়ী sub-states এর মধ্য দিয়ে নিয়ে যায়।
* বেঞ্চমার্কিং (Benchmarking) না করেই সরাসরি প্যারালাল রিকোয়েস্ট পাঠানো।

## 17. Limitations

* Single-packet attack শুধুমাত্র HTTP/2 সাপোর্ট করে এমন সার্ভারেই কাজ করে (HTTP/1 এর জন্য Last-byte sync লাগে)।
* PHP-এর মতো কিছু এনভায়রনমেন্টে Default session locking থাকে। একই সেশন থেকে একসাথে রিকোয়েস্ট দিলে তা sequential প্রসেস হয়। এক্ষেত্রে অ্যাটাক করতে হলে আলাদা আলাদা সেশন ব্যবহার করতে হয়।
* Static file-এর ক্ষেত্রে Single-packet attack কাজ নাও করতে পারে।
* Multi-endpoint race condition-এ সার্ভার-সাইড প্রসেসিং টাইমের ভিন্নতা অ্যাটাককে কঠিন করে তোলে।

## 18. Edge Cases / Important Details

* **Deferred Collisions:** এগুলো আইডেন্টিফাই করা খুব কঠিন কারণ তাৎক্ষণিক কোনো ক্লু পাওয়া যায় না। সেকেন্ড-অর্ডার ক্লু (যেমন ২০ মিনিট পর আসা ইমেইল) লক্ষ্য করতে হয়।
* **Time-sensitive Vulnerabilities:** Race condition না পেলেও, রিকোয়েস্টগুলোকে ঠিক একই সময়ে ডেলিভার করতে পারলে টাইমস্ট্যাম্প-বেসড সিকিউরিটি টোকেনগুলো একই জেনারেট করানো সম্ভব হতে পারে।
* **Object Masking:** রেস কন্ডিশন ব্যবহার করে একটি অবজেক্ট তৈরি করা যা পরবর্তীতে অন্য কোনো উচ্চ-ক্ষমতাসম্পন্ন অবজেক্ট দিয়ে রিপ্লেস বা মাস্ক হয়ে যায়।

## 19. Database / Platform / Technology Differences

| Technology | Important Difference |
| --- | --- |
| **PHP** | ডিফল্টভাবে Session locking ব্যবহার করে। অর্থাৎ, একই সেশন আইডি দিয়ে একাধিক রিকোয়েস্ট আসলে PHP একটি রিকোয়েস্ট শেষ না হওয়া পর্যন্ত অন্যটিকে অপেক্ষা করায়। |
| **Ruby on Rails / Devise** | Devise ফ্রেমওয়ার্কে ইমেইল পাঠানোর সময় ডেটাবেস থেকে টেমপ্লেট ভেরিয়েবল রিড করা এবং ব্যাকগ্রাউন্ড থ্রেডে ইমেইল পাঠানোর মধ্যে একটি race window তৈরি হতে পারে। |
| **HTTP/1 vs HTTP/2** | HTTP/1-এ "Last-byte sync" কাজ করে (প্রায় 4ms spread)। HTTP/2-এ "Single-packet attack" ব্যবহার করে সম্পূর্ণ Network jitter এড়ানো যায় (1ms spread)। |

## 20. Impact

Race condition-এর ইমপ্যাক্ট অ্যাপ্লিকেশনের ফাংশনালিটির ওপর নির্ভর করে:

* **Financial Loss:** গিফট কার্ড বা ব্যালেন্স উইথড্র লিমিট ওভাররান করা।
* **Privilege Escalation / Authentication Bypass:** পাসওয়ার্ড রিসেট বা MFA মেকানিজমে সাব-স্টেট অ্যাবিউজ করা।
* **Data Corruption:** একই ডাটাবেস রেকর্ডে ওভাররাইট করা বা ভুল কনফার্মেশন কোড জেনারেট করা।

## 21. Prevention / Mitigation

এই দুর্বলতাগুলো এড়াতে ডেটাবেস এবং কোড লেয়ারে কঠোর ব্যবস্থা নিতে হবে:

* **Make state changes atomic:** State changes-কে atomic করার জন্য datastore-এর concurrency ফিচার ব্যবহার করুন (যেমন Database transactions)।
* **Datastore Integrity:** Datastore-এর নিজস্ব কনসিস্টেন্সি ফিচার (যেমন Column uniqueness constraints) ব্যবহার করুন।
* **Avoid Mixing Data Sources:** বিভিন্ন সোর্স (যেমন ইনস্ট্যান্স ভেরিয়েবল এবং ডেটাবেস) থেকে ডেটা নিয়ে প্রসেস করা থেকে বিরত থাকুন।
* **Do not use one storage layer to secure another:** ডেটাবেস লিমিট ওভাররান ঠেকাতে সেশনের ওপর নির্ভর করবেন না।
* **Session Consistency:** সেশন ভেরিয়েবলগুলো আলাদা আলাদা আপডেট করার বদলে ব্যাচ (batch) আপডেট করুন। ORM-এর ক্ষেত্রে ট্রানজেকশন ব্যবহার নিশ্চিত করুন।
* **Push State Client-side:** কিছু আর্কিটেকচারে সার্ভার-সাইড স্টেট পরিহার করে JWT-এর মতো এনক্রিপশন ব্যবহার করে স্টেট ক্লায়েন্ট-সাইডে পুশ করা যেতে পারে (তবে JWT-এর নিজস্ব ঝুঁকি মাথায় রাখতে হবে)।

## 22. Vulnerable vs Secure Example

**Vulnerable Pseudo-code (MFA Bypass):**

```python
session['userid'] = user.userid
if user.mfa_enabled:
    session['enforce_mfa'] = True
    # generate and send MFA code to user

```

*Why vulnerable?* `session['userid']` সেট হওয়ার পর এবং `enforce_mfa` সেট হওয়ার মাঝখানের সময়ে একটি sub-state তৈরি হয় যেখানে ইউজারের ভ্যালিড সেশন থাকে কিন্তু MFA এনফোর্সড থাকে না।

**Secure Concept:**
*Why secure?* পুরো স্টেট ট্রানজিশনটি একটি সিঙ্গেল, অ্যাটমিক অপারেশনে করতে হবে অথবা এমনভাবে ডিজাইন করতে হবে যেন কোনো অসম্পূর্ণ বা middle-state-এ ইউজার কোনোভাবেই ক্রিটিকাল এন্ডপয়েন্ট অ্যাক্সেস করতে না পারে।

## 23. Real Understanding

Race condition মানেই শুধু "একসাথে দুইবার ডিসকাউন্ট কোড ব্যবহার করা" নয়। এর প্রকৃত রূপ বুঝতে হলে একটি কথা মনে রাখতে হবে: "With race conditions, everything is multi-step." একটি সাধারণ HTTP রিকোয়েস্টও সার্ভারের ভেতরে অনেকগুলো ছোট ছোট ধাপে সম্পন্ন হয়। এই ধাপগুলোর মাঝখানে অতি ক্ষুদ্র সময়ের যে গ্যাপ (race window) তৈরি হয়, অ্যাটাকার ঠিক সেই সময়েই লজিক ব্রেক করার চেষ্টা করে। Single-packet attack এই অ্যাটাককে ল্যাব এনভায়রনমেন্টের বাইরে এনে রিয়েল ওয়ার্ল্ডে নির্ভরযোগ্যভাবে (reliably) এক্সপ্লয়েট করার সুযোগ করে দিয়েছে।

## 24. Mental Model

Attacker identifies a critical endpoint acting on a shared resource (e.g., Session, Database)
↓
Attacker crafts parallel requests using HTTP/2 Single-packet attack (or HTTP/1 Last-byte sync)
↓
Requests travel together and arrive at the exact same millisecond, dodging network jitter
↓
Server spawns multiple threads to process them concurrently
↓
Thread A transitions the application into a temporary "sub-state" (e.g., Check passed, but DB not updated)
Thread B simultaneously interacts with the same resource during this tiny race window
↓
Collision occurs! Thread B exploits the sub-state or bypasses the limit
↓
Unintended action succeeds (Limit overrun, MFA bypass, etc.)

## 25. Quick Revision

* **Definition:** একই সময়ে একাধিক রিকোয়েস্ট প্রসেস হওয়ার কারণে তৈরি হওয়া লজিক ফ্ল।
* **Cause:** Concurrency হ্যান্ডেল না করা এবং TOCTOU গ্যাপ।
* **Main idea:** অতি ক্ষুদ্র সময়ের গ্যাপে (Race window) রিকোয়েস্ট পাঠিয়ে লজিক বাইপাস করা।
* **Attack flow:** Predict -> Benchmark -> Probe (Parallel) -> Observe clues -> Exploit.
* **Important condition:** রিকোয়েস্টগুলো ঠিক একই সময়ে সার্ভারে প্রসেস হতে হবে।
* **Main techniques:** Single-packet attack, Limit overrun, Hidden sub-states, Connection warming.
* **Detection:** Burp Repeater-এর 'Send group in parallel' দিয়ে স্ট্যাটাস কোড বা টাইমিংয়ের পার্থক্য খোঁজা।
* **Impact:** Privilege escalation, Account takeover, Financial manipulation.
* **Prevention:** Database transactions (Atomic operations), Uniqueness constraints.

## 26. Things to Remember

* **Single-packet attack:** HTTP/2-এর একটি ফিচার ব্যবহার করে ২০-৩০টি রিকোয়েস্ট একটি TCP প্যাকেটে পাঠানো যায়, যা Network jitter পুরোপুরি দূর করে।
* **Clues are everything:** টাইমিং, স্ট্যাটাস কোড, বা ইমেইলের বডির যেকোনো ছোটখাটো পরিবর্তনই একটি বিশাল Race condition-এর ক্লু হতে পারে।
* **PHP Session Locking:** PHP ডিফল্টভাবে সেশন লক করে রাখে। তাই রেস কন্ডিশন টেস্ট করতে হলে আলাদা আলাদা সেশন আইডি ব্যবহার করতে হবে।
* **Connection Warming:** ব্যাক-এন্ড সার্ভারের কানেকশন ডিলে দূর করতে মূল অ্যাটাকের আগে একটি ডামি রিকোয়েস্ট পাঠানো হয়।

