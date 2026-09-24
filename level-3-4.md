## Natas Level 4 → Level 5

*Goal: Find the password for natas5. This level checks the HTTP Referer header and only grants access if you appear to have come from an "internal" admin page.*

**Step 1: Access the level**

http://natas4.natas.labs.overthewire.org

Username: natas4
Password: (from level 3)

**Step 2: Read the page message**

The page tells you:

Access disallowed. You are visiting from "" while you are required to visit from "http://natas5.natas.labs.overthewire.org/"

This means the server is checking the Referer HTTP header, and expects it to be http://natas5.natas.labs.overthewire.org/.

**Step 3: Spoof the Referer header**

- Option A — using curl

```bash
curl -u natas4:<password_from_level4> \
  -e "http://natas5.natas.labs.overthewire.org/" \
  http://natas4.natas.labs.overthewire.org/ 
```

(-e sets the Referer header in curl)

- Option B — using a browser extension

Use an extension like "ModHeader" (Chrome/Firefox) to set:

Referer: http://natas5.natas.labs.overthewire.org/

Then reload the natas4 page.

- Option C — Burp Suite

Intercept the request and modify the Referer header manually before forwarding.

**Step 4: Get the password**

Once the correct Referer is sent, the page will respond with:

Access granted. The password for natas5 is XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

**Step 5: Log in to Level 5**

Username: natas5
Password: <password found above>
URL: http://natas5.natas.labs.overthewire.org

*Lesson learned*

The Referer header is fully controlled by the client and can be trivially spoofed. It should never be used as a security or authentication mechanism — it's meant for analytics/navigation context, not access control.