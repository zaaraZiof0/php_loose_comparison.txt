##  Initial Access — Foothold as `www-data`

### Vulnerability Summary
The target API uses **loose PHP equality (`==`)** instead of strict comparison (`===`).  
In PHP, any string starting with `0e` followed by digits is interpreted as **scientific notation**, which evaluates to `0`.

Example:

```php
if ($userToken == $storedToken) {
    auth_success();
}
```

A value such as:
```
0e123456
```

is interpreted as:
```
0 == 0
```


Therefore, authentication is bypassed.

    Tested on PHP 8.3.27 – still vulnerable under == comparisons due to type juggling behavior.


### Objective

- Exploit PHP type juggling to:
- Bypass API authentication
- Access restricted data
- Leak MD5 hashes
- Crack credentials
- Spray creds into Cacti
- Achieve RCE and foothold as www-data


## Fuzzing for Magic Hash Tokens

A custom PHP loose-comparison wordlist was uploaded and fuzzed:
```
ffuf -c \
-u "http://monitorsfour.htb/user?token=FUZZ" \
-w php_loose_comparison.txt \
-fw 4
```

Output:
```
0e1234                  [Status: 200, Size: 1465, Words: 14, Lines: 1]
```

A 200 OK response confirms successful authentication bypass.


## Result

Authentication bypass successful → Access obtained as www-data.
This grants access to sensitive data used later for:
- MD5 hash extraction
- Cracking
- Credential spray against Cacti
- Remote Code Execution (RCE)
- Foothold established
