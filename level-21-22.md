# Natas Level 21-22 

**Challenge:** [OverTheWire: Natas — Level 22](http://natas22.natas.labs.overthewire.org)
**Category:** Web Exploitation / Improper Redirect Handling
**Tools used:** Burp Suite (Community Edition)

*(Credentials to reach this level were obtained via the Level 21 session-sharing trick — see the Natas21 writeup.)*


## 1. Challenge Overview

Logging into `natas22` with Basic Auth just shows an empty-looking page no visible way to reach admin content, no form, no obvious parameter to abuse. Unlike Level 21, there's no "experimenter" twin site here to lean on.

The clue lives in how the server **responds**, not in what the page displays.


## 2. Observing the Behavior in Burp

Watching **Proxy → HTTP History**, a plain `GET /` to natas22 comes back as **HTTP 302** a redirect with no visible body content in the browser (since the browser just follows it silently).

That 302 status is the whole hint: a redirect was issued from server-side PHP via `header("Location: ...")`. In many PHP apps, developers forget to call `exit()`/`die()` immediately after `header()` and when that happens, **the script keeps executing and keeps printing output**, even though the HTTP status code already says "go somewhere else." Browsers throw that body away because they immediately follow the `Location` header. Burp does not.


## 3. Exploitation Steps (Burp Suite)

### Step 1 — Capture a request in Proxy
Log in via Basic Auth and load the natas22 page once with the Proxy intercepting, so it lands in **HTTP History**.

### Step 2 — Send to Repeater
Send the request to Repeater to control it manually and inspect the *raw* response, rather than letting a browser auto-follow the redirect.

### Step 3 — Add the parameters that unlock the admin branch
Based on testing, the request that returns the flag in-body was:

```
GET /index.php?revelio&admin=1 HTTP/1.1
Host: natas22.natas.labs.overthewire.org
Authorization: Basic <natas22 credentials>
Cookie: PHPSESSID=<session id>
```

- `revelio` acts as a "reveal" flag the server checks for.
- `admin=1` mirrors the same pattern from Level 21, a client-supplied value that ends up influencing `$_SESSION['admin']` with no server-side authorization check.

### Step 4 — Send and read the raw response
Even though the **status code is still 302** (a redirect is issued), the response **body** visible in Burp's Pretty/Raw view, contains the full page HTML, including:

```html
<div id="content">
  You are an admin. The credentials for the next level are:<br>
  <pre>
  Username: natas23
  Password: <Natas23_password>
  </pre>
```

A normal browser would never show you this: it follows the `Location` redirect instantly and discards the body. Burp Repeater, by not auto-following redirects, exposes the "dead" content the developer never intended to be reachable.


## 4. Root Cause

Two stacked mistakes:

1. **Client-controlled privilege flag** 
An `admin` parameter (or equivalent session flag) can be influenced directly by the request, the same class of bug as Level 21.

2. **Missing `exit()`/`die()` after `header("Location: ...")`** 
PHP does not stop script execution just because a `Location` header was sent. Any code after the `header()` call still runs and can still emit output, which lands in the HTTP response body of the redirect response itself.

Individually, mistake #2 is often harmless (browsers won't render it). Combined with mistake #1, i.e., sensitive branching logic executing *after* the redirect line it becomes a full information disclosure: an attacker using a raw HTTP client (like Burp) sees content the developer assumed was unreachable.


## Lessons Learned

- A redirect is not an access control. Anything printed after `header("Location: ...")` without `exit()` is still delivered to the client it's just usually invisible because of how browsers behave.
- Proxy tools that don't auto-follow redirects (Burp, curl `-i` without `-L`) are essential for spotting this class of bug.
- This level pairs naturally with Level 21: both hinge on the gap between "what the developer assumed the client would do" and "what a raw HTTP client actually lets you do."