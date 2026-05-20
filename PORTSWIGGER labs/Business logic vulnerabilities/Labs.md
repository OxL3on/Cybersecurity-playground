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



# Lab: Low-level logic flaw

First, with Burp Suite running, log in to the lab and attempt to buy the leather jacket. The order is rejected because your store credit is insufficient. In the Proxy history, study the order process, then send the `POST /cart` request to Burp Repeater.

In Burp Repeater, notice that you can only add a 2‑digit quantity with each request. Send this request to Burp Intruder. In Intruder, set the `quantity` parameter to `99`. In the Payloads side panel, select the payload type **Null payloads**. Under Payload configuration, choose **Continue indefinitely**. Start the attack.

While the attack is running, go to your cart and refresh the page every few seconds. Monitor the total price. Eventually, you will notice that the price suddenly becomes a large negative integer and begins counting upward toward zero. This happens because the total price has exceeded the maximum value allowed for an integer in the back‑end language (2,147,483,647 cents). The value wraps around to the minimum possible value (-2,147,483,648 cents).

Clear your cart. Using only the leather jacket, it is not mathematically possible to make the wrapped‑around price land between $0 and your remaining $100 credit (the jacket’s price is 133,700 cents). Instead, create the same Intruder attack again, but this time under Payload configuration, generate exactly **323 payloads**. Click **Resource pool**, add the attack to a pool with **Maximum concurrent requests = 1**, then start the attack.

After the Intruder attack finishes, go back to the `POST /cart` request in Burp Repeater and send a single request for **47 jackets**. The total price should now be -$1221.96 (i.e., -122,196 cents). Use Burp Repeater to add a suitable quantity of another cheap item to your cart so that the total falls between $0 and $100. Finally, place the order — the lab is solved.



# Lab: Inconsistent handling of exceptional input

Try to browse to `/admin`. Although you don't have access, the error message reveals that only DontWannaCry users are allowed.

Go to the account registration page. Observe the hint telling DontWannaCry employees to use their company email address. From the button in the lab banner, open the email client and make a note of the unique ID in your email server domain (format: `@YOUR-EMAIL-ID.web-security-academy.net`).

Go back to the lab and register with an exceptionally long email address in the format:  
`very-long-string@YOUR-EMAIL-ID.web-security-academy.net`  
The `very-long-string` should be at least 200 characters long.  
Go to the email client — you will receive a confirmation email. Click the link to complete registration. Log in and go to the "My account" page. Notice that your email address has been truncated to 255 characters.

Log out and return to the account registration page. Register a new account with another long email address, but this time include `dontwannacry.com` as a subdomain, as follows:  
`very-long-string@dontwannacry.com.YOUR-EMAIL-ID.web-security-academy.net`  
Carefully choose the length of `very-long-string` so that the `"m"` at the end of `@dontwannacry.com` is exactly the 255th character.

Go to the email client and click the confirmation link. Log in to your new account. Because the application server truncated the stored email address to 255 characters, your account now appears to have a valid `@dontwannacry.com` address (verify this on the "My account" page). You now have access to the admin panel. Navigate to the admin panel and delete `carlos` to solve the lab.



# Lab: Weak isolation on dual-use endpoint

Log in and go to your account page. Change your password while intercepting the request with Burp. Send the request to Repeater.

Notice that if you remove the `current-password` parameter, you can change the password without knowing the old one. Also, the `username` parameter controls which user gets changed.

Change `username` to `administrator` and send the request. Now log out and log back in as `administrator` with your new password. Go to the admin panel and delete `carlos` — lab solved.


# Lab: Insufficient workflow validation

Log in and buy a cheap item you can afford. In Burp's proxy history, notice that after placing the order, the `POST /cart/checkout` request redirects you to a confirmation page. Send the `GET /cart/order-confirmation?order-confirmation=true` request to Burp Repeater.

Now add the expensive leather jacket to your cart. In Burp Repeater, resend the saved order confirmation request. The order completes without deducting any store credit — lab solved.


# Lab: Authentication bypass via flawed state machine

Log in while intercepting with Burp. On the login page, you must select a role before reaching the home page. 

Turn on intercept, log in, and forward the `POST /login` request. The next request is `GET /role-selector` — drop this request. Now browse directly to the lab's home page. Your role defaults to administrator, giving you admin access. Go to the admin panel and delete `carlos` — lab solved.


# Lab: Infinite money logic flaw

