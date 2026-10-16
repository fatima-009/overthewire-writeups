# Natas Level 20 - 21 

**Challenge:** [OverTheWire: Natas — Level 21](http://natas21.natas.labs.overthewire.org)
**Category:** Web Exploitation / Session Management
**Tools used:** Burp Suite (Community Edition), Browser DevTools


## 1. Challenge Overview

Natas Level 21 introduces a second, **colocated** website:
`natas21-experimenter.natas.labs.overthewire.org`.

The banner on both pages explicitly states:

> *"Note: this website is colocated with `http://natas21.natas.labs.overthewire.org`"*

The experimenter site is a harmless-looking form that lets you customize a demo box (`align`, `fontsize`, `bgcolor`) and displays a debug dump of the current PHP `$_SESSION` array. The real `natas21` page, on the other hand, only shows the password **if `$_SESSION['admin'] == 1`** and gives you no way to set that value directly.

The vulnerability lies in how PHP sessions are stored and shared between the two "colocated" applications.


## 2. Recon

Loading `natas21-experimenter/index.php?debug` shows the live session contents after every request:

```
Array
(
    [align]    => center
    [fontsize] => 100%
    [bgcolor]  => yellow
    [admin]    => 0
    [submit]   => Update
)
```

Reviewing the experimenter's logic (paraphrased from behavior), it works roughly like this:

```php
session_start();

if (array_key_exists("align", $_REQUEST))    { $_SESSION["align"]    = $_REQUEST["align"]; }
if (array_key_exists("fontsize", $_REQUEST)) { $_SESSION["fontsize"] = $_REQUEST["fontsize"]; }
if (array_key_exists("bgcolor", $_REQUEST))  { $_SESSION["bgcolor"]  = $_REQUEST["bgcolor"]; }

$_SESSION["admin"] = 0;
if (array_key_exists("admin", $_REQUEST)) {
    $_SESSION["admin"] = $_REQUEST["admin"];
}
```

Key insight: `admin` is **first reset to 0**, but then **overwritten by whatever you send in the request** if the `admin` parameter is present. So by simply adding `admin=1` to the POST body, the experimenter app will happily store `admin => 1` in its own session.

The real `natas21` page never lets you set `admin` at all, it only *reads* it. So the goal is to get a session where `admin == 1` recognized by `natas21`, without ever being able to set it there directly.

**The bridge:** both applications are colocated on the same server and share the same PHP session storage backend (session files on disk, keyed by `PHPSESSID`). The only thing separating them in the browser is that each site's cookie is scoped to a different hostname. If you can get the *same* `PHPSESSID` value recognized by both hosts, they will read/write the **same underlying session file**.


## 3. Exploitation Steps (Burp Suite)

### Step 1 — Capture the experimenter request
Browse to `natas21-experimenter`, submit the form once with Burp's Proxy intercepting, and send that request to **Repeater**.

### Step 2 — Force `admin=1`
In the Repeater request, edit the POST body to include the extra parameter:

```
align=center&fontsize=100%25&bgcolor=yellow&admin=1&submit=Update
```

### Step 3 — Send and confirm
Send the request. The rendered response (with `?debug` in the URL) confirms:

```
[admin] => 1
```

Note the `PHPSESSID` cookie value used for this request — for example:

```
PHPSESSID=9m92oa9a0****************
```

This is now the ID of a session file, sitting on the shared server, that contains `admin => 1`.

### Step 4 — Capture a request to the real natas21 site
In a second Repeater tab, load a request to:

```
http://natas21.natas.labs.overthewire.org/
```

### Step 5 — Swap the session cookie
Replace that request's `Cookie` header value with the **exact** `PHPSESSID` captured from the experimenter site in Step 3:

```
Cookie: PHPSESSID=9m92oa9a0****************
```

### Step 6 — Send
Because `natas21` only *checks* `$_SESSION['admin']` and never resets it, and because it's reading the very same session file on disk, the page now renders the admin content — including the password for the next level.


## 4. Root Cause

- Two applications on the same host were designed to **share a session storage backend** while relying on the **hostname-scoped cookie** as their only isolation boundary.
- Session identifiers were treated as authorization tokens without validating *which application* actually issued or is entitled to trust them.
- One app allowed a client-controlled value (`admin`) to be written directly into session state with no server-side authorization check.

This is a practical demonstration of **session fixation / session sharing across trust boundaries** — a real-world risk whenever multiple applications (e.g., microservices, staging vs. prod, subdomains) share a session store without properly namespacing or re-validating session data per application.


## 5. Mitigation

- Never share a session storage backend across applications with different trust levels.
- If shared storage is unavoidable, namespace session keys per application (e.g., `app_a:admin` vs `app_b:admin`).
- Never let a client set privileged flags (like `admin`) directly via request parameters — derive them from authenticated, server-side state only.
- Regenerate/rotate session IDs on privilege changes (`session_regenerate_id()`), and bind sessions to additional context (IP, user-agent) where appropriate.


## 6. Result

Password for `natas22` was retrieved by reading the rendered response after Step 6.

*(Redacted here — replace with your own credential before publishing if you choose to include it.)*


## 7. Lessons Learned

- Session management bugs are often invisible from a single application's code, they emerge from **how systems are wired together**.
- Burp Suite's Repeater is ideal for this kind of exploit: it lets you hold two independent, editable requests side by side and control exactly when each is sent.
- Always ask: *"What does this application trust, and where does that trust actually come from?"*