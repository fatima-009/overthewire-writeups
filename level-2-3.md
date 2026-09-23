## Natas Level 2 → Level 3

*Goal: Find the password for natas4. This level teaches that robots.txt can leak hidden paths even if a page looks empty.*

**Step 1: Access the level**

http://natas3.natas.labs.overthewire.org

Username: natas3
Password: (from level 2)

**Step 2: Inspect the page**

Again, the page appears to say "There is nothing on this page" — but this time, checking the plain HTML source doesn't reveal anything obvious like an image path.

**Step 3: Check robots.txt**

Since search engine crawler rules are a classic place developers accidentally disclose hidden paths, check:

*http://natas3.natas.labs.overthewire.org/robots.txt*

You'll find something like:

User-agent: *
Disallow: /s3cr3t/

**Step 4: Browse the disallowed directory**

*http://natas3.natas.labs.overthewire.org/s3cr3t/*

Directory listing is enabled here too, and you'll see a file:

users.txt

**Step 5: Read users.txt**

*http://natas3.natas.labs.overthewire.org/s3cr3t/users.txt*

Inside you'll find:

natas4:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

**Step 6: Log in to Level 4**

Username: natas4
Password: <password found above>
URL: http://natas4.natas.labs.overthewire.org

*Lesson learned*

robots.txt is meant to tell search engines what not to index — but it's publicly readable by anyone, including attackers. Listing sensitive paths in it (instead of properly restricting access server-side) is "security through obscurity" and doesn't actually protect anything.