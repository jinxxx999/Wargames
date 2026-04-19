# Natas Level 11 → Level 12
## Description
This level demonstrates a **Weak Cryptography vulnerability** using a XOR cipher. The server stores user preferences (background color and a "show password" flag) in an encrypted cookie. Since the XOR cipher is symmetrical and the plaintext is partially known, the secret key can be recovered.

## Vulnerability Analysis
The provided PHP source code revealed the following:

## Data Structure: 
An array {"showpassword":"no", "bgcolor":"#ffffff"} is converted to JSON.

## Encryption Flow: 
JSON -> XOR with Secret Key -> Base64 Encode -> Cookie.

## XOR Property: 
If Plaintext ^ Key = Ciphertext, then Plaintext ^ Ciphertext = Key.

By using the default JSON string and the encrypted cookie value from the browser, I was able to mathematically extract the hidden server-side key.

## Solution Steps

### 1. Key Extraction

I captured the default cookie from the site and ran a PHP script to XOR it against a known default JSON string:

Known Plaintext: {"showpassword":"no","bgcolor":"#ffffff"}

Captured Cookie: HmYkBwozJw4WNyAAFyB1VUcqOE1JZjUIBis7ABdmbU1GIjEJAyIxTRg%3D

Executed the script using an online PHP sandbox to calculate the XOR key:

```php
<?php

function xor_encrypt($in) {
    $key = json_encode(array( "showpassword"=>"no", "bgcolor"=>"#ffffff"));
    $text = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}
$cookie = "HmYkBwozJw4WNyAAFyB1VUcqOE1JZjUIBis7ABdmbU1GIjEJAyIxTRg%3D";
echo xor_encrypt(base64_decode($cookie));
?>
```
The extracted repeating key was: eDWoe (or the specific key you found).

### 2. Exploit Generation

Using the recovered key, I crafted a new PHP payload:
```php
<?php
function xor_encrypt($in) {
    $key = "eDWo"; #change for specific key you found
    $text = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}
echo base64_encode(xor_encrypt(json_encode(array( "showpassword"=>"yes", "bgcolor"=>"#ffffff"))))
?>
```
### 3. Execution

Opened Browser DevTools (Storage/Cookies).

Replaced the data cookie value with the generated exploit string.

Refreshed the page to reveal the password for Natas 12.
