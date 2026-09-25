## Natas Level 4 → Level 5

*Goal: Find the password for natas6. This level checks a cookie value to decide if you're "logged in" — and cookies, like headers, are fully client-controlled.*

**Step 1: Access the level**

http://natas5.natas.labs.overthewire.org

Username: natas5
Password: (from level 4)

**Step 2: Read the page message**

The page says:

Access disallowed. You are not logged in

**Step 3: Inspect the cookies**

Check the cookies sent by the site (via browser dev tools → Application/Storage → Cookies, or curl headers). You'll find:

loggedin=0

**Step 4: Change the cookie value**

The fix is simple — set loggedin to 1.

- Option A — using curl

```bash
curl -u natas5:<password_from_level5> \
  -b "loggedin=1" \
  http://natas5.natas.labs.overthewire.org/
```

- Option B — using browser dev tools

. Open Developer Tools (F12 or browser menu → More Tools → Developer Tools)
. Go to the Application (Chrome) or Storage (Firefox) tab
. Find Cookies → select the natas5 domain
. Edit the loggedin cookie value from 0 to 1
. Reload the page

**Step 5: Get the password**

Once loggedin=1 is sent, the page will respond with:

Access granted. The password for natas6 is XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

**Step 6: Log in to Level 6**
Username: natas6
Password: <password found above>
URL: http://natas6.natas.labs.overthewire.org

*Lesson learned*

Cookies are stored and sent by the client, meaning the user has full control over their values. Using a cookie like loggedin=0/1 as the sole authentication check is insecure — session state and authorization must be validated and tracked server-side (e.g., via signed/encrypted session tokens), never trusted as-is from client input.