# Join Team 

The challenge was based on an unrestricted file upload vulnerability leading to Remote Code Execution (RCE).

First, I explored the website and found a CV upload feature in the **WorkForUs** page. Uploading a `.txt` file returned the message:

```text
Only PDF Document Allowed
```

This indicated that the application validated uploaded file types.

I created a PHP payload:

```php
<?php echo "test"; ?>
```

Then I intercepted the upload request using [Burp Suite](https://portswigger.net/burp?utm_source=chatgpt.com) and modified:

* the filename from `.php` to `.pdf`
* the `Content-Type` to `application/pdf`

The upload succeeded, meaning the validation could be bypassed.

By checking the server response, I found that uploaded files were stored inside the `data/` directory. Visiting the uploaded file URL executed the PHP code successfully, confirming Remote Code Execution.

To read the source code safely, I used:

```php
<?php system("base64 index.php"); ?>
```

After decoding the output, I found the flag inside the PHP source:

```php
$_ENV['flag'] = 'hiddenenvironflag';
```

Flag:

```text
hiddenenvironflag
```

The source code also revealed a Local File Inclusion vulnerability:

```php
include str_replace('_','.',$page[0]);
```
