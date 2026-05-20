# Lab: Excessive trust in client-side controls

First, open Burp Suite and access the lab. Log in using the provided credentials wiener:peter.

After logging in, navigate to the home page. You will see a product priced at $1337, while your account balance is only $100, so you cannot purchase it normally.

Turn on Burp Intercept and click “Add to cart” on the product page. The request will be captured in Burp. You should see the following request:

```
POST /cart HTTP/2
Host: 0a0b00ff04458ad281b0993600f000b5.web-security-academy.net
Cookie: session=YR8shXWi04rq8I4BkmrmWWPZVsIDltBx
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Origin: https://0a0b00ff04458ad281b0993600f000b5.web-security-academy.net
Referer: https://0a0b00ff04458ad281b0993600f000b5.web-security-academy.net/product?productId=1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

productId=1&redir=PRODUCT&quantity=1&price=133700
```

Notice that the price parameter is being sent from the client side. This is the key flaw.

Modify the request by changing:
price=133700 → price=0

Forward the modified request.

Now go back to the cart page. You will see that the product price has been updated to 0, confirming that the server trusted client-side input without proper validation.

Proceed to checkout and complete the purchase to solve the lab.


# Lab: 2FA broken logic

First, I logged into my own account while running Burp Suite and observed the 2FA verification flow. During this process, I noticed that the verify parameter in the /login2 request determines which user’s account is being verified.

Next, I logged out and captured the GET /login2 request. I sent this request to Burp Repeater and modified the verify parameter to carlos. This allowed me to initiate the 2FA process for Carlos’s account.

After that, I logged in again using my own credentials and intentionally entered an incorrect 2FA code. I captured the resulting POST /login2 request and sent it to Burp Intruder for brute-forcing.

The captured request was:

```id="x92kd1"
POST /login2 HTTP/2
Host: 0a15008804d1938a81b1cf7f004400be.web-security-academy.net
Cookie: session=91ka4mE6bKYyNTzdH5XfSxjxC6Sjs3RU; verify=carlos
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 13
Origin: https://0a15008804d1938a81b1cf7f004400be.web-security-academy.net
Referer: https://0a15008804d1938a81b1cf7f004400be.web-security-academy.net/login2
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

mfa-code=1234
```

In Burp Intruder, I kept verify=carlos in the Cookie and brute-forced the mfa-code parameter. During the attack, I monitored the responses and identified a request that returned a 302 status code, indicating a successful login.

I opened that response in the browser and navigated to “My account,” which confirmed access to Carlos’s account and solved the lab.


# Lab: High-level logic vulnerability

First, log in using the provided credentials. After logging in, navigate to the store and locate the “Lightweight l33t leather jacket,” which costs $1337. Since your balance is only $100, you cannot purchase it directly.

To exploit the vulnerability, choose a cheaper product and add it to your cart while intercepting the request using Burp Suite.

The intercepted request looks like this:

```id="k29xqa"
POST /cart HTTP/2
Host: 0a9b003204cb6878807a30e300be00aa.web-security-academy.net
Cookie: session=UBW2tglBuwrs83Hfh9nqB3dTqt2QQejT
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 38
Origin: https://0a9b003204cb6878807a30e300be00aa.web-security-academy.net
Referer: https://0a9b003204cb6878807a30e300be00aa.web-security-academy.net/product?productId=7
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

productId=7&redir=PRODUCT&quantity=1
```

Notice that the quantity parameter is user-controlled. Modify this value to a negative number, for example:  quantity = -5

Forward the request. You will observe that the cart updates with a negative quantity and the total price decreases instead of increasing. Repeat this process by adjusting the quantity further into negative values until the total cart price drops below your available balance ($100).

Once the total is low enough, add the expensive jacket to your cart and proceed to checkout. The manipulated cart total allows you to complete the purchase and solve the lab.



# Lab: Inconsistent security controls

First, Try to browse to `/admin`. Although you are denied access, the error message reveals that only users with a `@dontwannacry.com` email address are allowed.

Navigate to the account registration page. Observe the hint telling DontWannaCry employees to use their company email address. Register with an arbitrary email in the format: `anything@your-email-id.web-security-academy.net` (you can find your unique email domain by clicking the "Email client" button). Go to the email client and click the confirmation link to complete the registration.

Log in with your new account and go to the "My account" page. Notice that you have the option to change your email address. Modify your email to an arbitrary `@dontwannacry.com` address (e.g., `hacker@dontwannacry.com`).

After saving the change, refresh the page or navigate to `/admin`. You will now have full access to the admin panel, where you can delete `carlos` to solve the lab.



# Lab: Flawed enforcement of business rules

First, log in to the lab using the provided credentials. At the bottom of the page, notice that there is a coupon code `NEWCUST5`. Sign up for the newsletter as well — you will receive another coupon code, `SIGNUP30`.

Add the leather jacket to your cart and proceed to the checkout page. Apply both coupon codes to receive a discount on your order. 

Try applying the same code twice in a row. Observe that the second attempt is rejected because the coupon has already been applied. However, if you alternate between the two codes — applying `NEWCUST5`, then `SIGNUP30`, then `NEWCUST5` again, and so on — you can bypass this restriction. Each time you alternate, the system mistakenly treats the code as a new, unused coupon.

Reuse the two codes alternately enough times to reduce your order total to less than your remaining store credit. Once the total is low enough, complete the order. The lab will be solved.
