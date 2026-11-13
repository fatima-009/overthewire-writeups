# Natas26 → Natas27 Writeup (OverTheWire)

**Category:** Web Security / SQL Whitespace Inconsistency
**Tools used:** Burp Suite, Browser
**Target:** `http://natas27.natas.labs.overthewire.org`

## Overview

This level presents a login/registration form backed by a MySQL `users` table:

```sql
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);
```

The PHP source exposes four functions:

- `checkCredentials()` — verifies username/password against the DB.
- `validUser()` — checks if a username already exists.
- `dumpData()` — fetches and prints a user's row (after `trim()`-ing the supplied username).
- `createUser()` — inserts a new user, but rejects any username that doesn't match its own `trim()`ed version (`Go away hacker`).

The request flow is:

1. If `username` exists → check credentials, and if correct, dump the user's data.
2. If `username` does not exist → create it with the supplied username/password.

## Vulnerability

`createUser`'s whitespace check happens *before* the 64-char truncation. So a username can carry trailing junk that defeats the check, then get silently cut off on insert — leaving a row that's effectively "real username + padding spaces." Later, `dumpData()` trims that padding away and collides with the *actual* `natas28` account, leaking its data.
 
**Payload username:** `natas28` + 57 spaces + `x` (65 chars total; the `x` gets truncated on insert).
 
## Exploitation Steps (Burp Suite)
 
1. **Capture & Repeater:** Submit the login form once, send the request to Repeater.

2. **Create the padded user:** Set `username=natas28` + 57 spaces + `x`, any `password`. Send. This passes `createUser`'s trim check but stores as `natas28` + 57 spaces (no `x`) after truncation.

3. **Log in as the padded user:** Resend with `username=natas28` + 57 spaces (no `x`) and the same password. `validUser`/`checkCredentials` match it directly, login succeeds.

4. **Trigger the leak:** Being logged in as the padded user, `dumpData()` trims the username to `natas28`, querying the *real* account instead. The response now contains the real `natas28` row.

5. **Read the response:** In Burp's Response tab, the dumped array includes `password => <natas28's password>` the credential for the next level.

## Root Cause
 
Inconsistent normalization: some functions trim, some don't, and the truncation-before-check ordering in `createUser` lets an attacker craft a username that's "clean" only after truncation. Fix: apply the same trim/escape/length rules everywhere `username` is used.