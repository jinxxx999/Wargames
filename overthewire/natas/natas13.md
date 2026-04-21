# Natas Level 13 → Level 14

**Description:** Exploiting an **Insecure File Upload** vulnerability to achieve **Remote Code Execution (RCE)** by bypassing server-side content validation using a **Polyglot file** (JPEG + PHP).

## Vulnerability Analysis
In this level, the server used the `exif_imagetype()` function to verify that the uploaded file is a legitimate image.

```php
else if (! exif_imagetype($_FILES['uploadedfile']['tmp_name'])) {
    echo "File is not an image";
}
```
This function validates the file by reading its Magic Bytes (signatures). However, the server still trustingly takes the file extension from the user-controlled filename POST parameter and does not perform deep inspection of the file body beyond the header.

## The Attack (Signature Spoofing)
The goal was to create a file that satisfies the exif_imagetype() check while remaining an executable PHP script.

### Step 1: Crafting the Polyglog Payload
I created a file that contains both a valid JPEG signature and malicious PHP code. Using the printf command, I inserted the Magic Bytes FF D8 FF E0 at the very beginning.Command:
Bash

printf "\xff\xd8\xff\xe0<?php echo passthru('cat /etc/natas_webpass/natas14'); ?>" > shell.php

### Step 2: Intercept and Bypass
Using **Burp Suite**, I intercepted the request and changed the filename parameter from .jpg to .php. Since the file now started with the correct "Magic Bytes", it successfully passed the server-side exif_imagetype check.

## Result
After the successful upload, I navigated to the newly generated link (e.g., upload/xxxxx.php) in my browser. The server identified the file as PHP due to its extension and executed the malicious code, displaying the password for Natas 14.

## Security Recommendations (Mitigation)
Relying only on file signatures (Magic Bytes) is not enough to secure file uploads.
Polyglot Files: Files can be crafted to be valid in multiple formats simultaneously (e.g., a JPEG that is also a PHP script).

Defense in Depth: Proper security should include renaming files, storing them outside the web root, and using a whitelist of allowed extensions that cannot be overridden by the user.
