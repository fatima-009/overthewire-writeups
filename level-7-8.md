# Natas Level 7 → Level 8

**Goal:**

Find the password for natas9. This level "encodes" a secret using a chain of reversible transformations, but since the code is visible, you can just reverse the logic manually.

# Step 1: Access the level

http://natas8.natas.labs.overthewire.org

Username: natas8
Password: (from level 6)

# Step 2: Read the page

Like level 6, there's a form asking for a "secret" value.

# Step 3: View the PHP source code

In the source code you'll see something like:

```php
<?
$encodedSecret = "3d3d516343746d4d6d6c315669563362";

function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}

if(array_key_exists("submit", $_POST)) {
    if(encodeSecret($_POST['secret']) == $encodedSecret) {
        print "Access granted. The password for natas9 is <censored>";
    } else {
        print "Wrong secret";
    }
}
?>
```

So the real secret goes through three transformations:

- base64_encode($secret)
- strrev(...) — reverse the string
- bin2hex(...) — convert to hex

To recover the original secret, you must reverse these steps in reverse order:

- Convert hex → binary/string (hex2bin)
- Reverse the string again (strrev)
- Base64-decode it (base64_decode)

# Step 4: Reverse it manually

Using PHP (via CLI):

```bash
php -r '$s = "3d3d516343746d4d6d6c315669563362"; echo base64_decode(strrev(hex2bin($s)));'
```

Using Python:

```python
import base64
encoded = "3d3d516343746d4d6d6c315669563362"
step1 = bytes.fromhex(encoded).decode()   # hex2bin
step2 = step1[::-1]                       # strrev
step3 = base64.b64decode(step2).decode()  # base64_decode
print(step3)
```

(Note: use the actual $encodedSecret value shown on your instance, it's randomized per session.)

# Step 5: Submit the secret

Take the decoded value from Step 4 and submit it in the form on the natas8 page.

# Step 6: Get the password

Access granted. The password for natas9 is XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

*Lesson learned*

Obfuscation (encoding, reversing, hex conversion) is not encryption — it provides no real security since anyone with access to the algorithm (which was visible in the source) can trivially reverse it. Never rely on "secret" transformation logic to protect sensitive values; use proper cryptographic hashing/salting for anything meant to stay hidden.