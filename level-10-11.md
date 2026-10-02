# Natas Level 11 → Level 12

**Goal:** Find the password for natas12. This level stores user preferences (like background color) in an `XOR-"encrypted" cookie` and since you know the default plaintext, you can recover the XOR key and forge your own cookie.

### Step 1: Access the level

http://natas11.natas.labs.overthewire.org

Username: natas11
Password: (from level 10)

### Step 2: Read the page

The page lets you set a background color, and mentions it uses XOR encryption to store your settings in a cookie.

### Step 3: View the PHP source code

Go to:

http://natas11.natas.labs.overthewire.org/index-source.html

You'll see logic roughly like:

```php

<?
$defaultdata = array( "showpassword"=>"no", "bgcolor"=>"#ffffff" );

function xor_encrypt($in) {
    $key = '<censored>';
    $text = $in;
    $outText = '';
    for($i=0;$i<strlen($text);$i++) {
        $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }
    return $outText;
}

function loadData($def) {
    global $_COOKIE;
    $mydata = $def;
    if(array_key_exists("data", $_COOKIE)) {
        $tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
        if(is_array($tempdata) && array_key_exists("showpassword", $tempdata) && array_key_exists("bgcolor", $tempdata)) {
            $mydata = $tempdata;
        }
    }
    return $mydata;
}

function saveData($d) {
    setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
}
?>
```

*Key points:*

- The cookie stores base64-encoded, XOR-encrypted JSON.
- The XOR key is hidden, but the default (un-set) cookie value corresponds to known plaintext:  `{"showpassword":"no","bgcolor":"#ffffff"}` .
- Since XOR is reversible with known plaintext, you can recover the key: `key = ciphertext XOR known_plaintext`.

### Step 4: Get the default cookie value

Visit the page fresh (or clear cookies) and check the data cookie set by the server (via browser dev tools → Application → Cookies, or curl -v).

### Step 5: Recover the XOR key

Write a small script (Python) to XOR the base64-decoded cookie against the known JSON plaintext:


```python
import base64
import json

cookie_b64 = "PASTE_YOUR_COOKIE_VALUE_HERE"

cipher = base64.b64decode(cookie_b64)
known_plain = json.dumps({"showpassword": "no", "bgcolor": "#ffffff"}, separators=(',', ':'))
# Natas actually uses PHP's json_encode default, may include no spaces

key = bytearray()
for i in range(len(cipher)):
    key.append(cipher[i] ^ ord(known_plain[i % len(known_plain)]))

print(key)  # inspect repeating pattern to find the actual key length/value
```

Look for a short repeating pattern in the recovered key bytes that's your XOR key (usually a few characters).

### Step 6: Forge a malicious cookie

Once you have the key, encrypt a new JSON payload with `showpassword` set to `yes`:

```python
def xor_encrypt(data, key):
    return bytes([data[i] ^ key[i % len(key)] for i in range(len(data))])

payload = json.dumps({"showpassword": "yes", "bgcolor": "#ffffff"}, separators=(',', ':')).encode()
new_cipher = xor_encrypt(payload, key)
new_cookie = base64.b64encode(new_cipher).decode()
print(new_cookie)
```

### Step 7: Set the forged cookie and reload

Using browser dev tools (Application → Cookies), replace the data cookie value with your newly generated one, then reload the page.

**Alternatively, with curl:**

```bash
curl -u natas11:<password_from_level11> \
  -b "data=<your_forged_cookie>" \
  http://natas11.natas.labs.overthewire.org/
```

### Step 8: Get the password

With showpassword=yes decrypted successfully, the page will reveal:

The password for natas12 is XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

*Lesson learned*

XOR encryption with a static, reused key is trivially breakable via a known-plaintext attack, if an attacker can predict or observe any portion of the plaintext (like a default cookie value), they can recover the key and forge arbitrary encrypted data. Never use raw XOR for security-sensitive data; use authenticated encryption (e.g., AES-GCM) with unique keys/nonces, and never trust client-side-modifiable state for authorization decisions.