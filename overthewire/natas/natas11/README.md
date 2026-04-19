Natas Level 11 → Level 12
Description
This level demonstrates a Weak Cryptography vulnerability using a XOR cipher. The server stores user preferences (background color and a "show password" flag) in an encrypted cookie. Since the XOR cipher is symmetrical and the plaintext is partially known, the secret key can be recovered.

Vulnerability Analysis
The provided PHP source code revealed the following:

Data Structure: An array {"showpassword":"no", "bgcolor":"#ffffff"} is converted to JSON.

Encryption Flow: JSON -> XOR with Secret Key -> Base64 Encode -> Cookie.

XOR Property: If Plaintext ^ Key = Ciphertext, then Plaintext ^ Ciphertext = Key.

By using the default JSON string and the encrypted cookie value from the browser, I was able to mathematically extract the hidden server-side key.

Solution Steps

1. Key Extraction

I captured the default cookie from the site and ran a Python script to XOR it against the known default JSON string:

Known Plaintext: {"showpassword":"no","bgcolor":"#ffffff"}

Captured Cookie: mYkBwozJw4WNyAAFyB1VUcqOE1JZjUIBis7ABdmbU1GdGdfVXRnTRg=

The extracted repeating key was: kYDw (or the specific key you found).

2. Exploit Generation

Using the recovered key, I crafted a new JSON payload:
{"showpassword":"yes","bgcolor":"#ffffff"}

I then encrypted this new string using the same XOR logic and encoded it back to Base64.

3. Execution

Opened Browser DevTools (Storage/Cookies).

Replaced the data cookie value with the generated exploit string.

Refreshed the page to reveal the password for Natas 12.
