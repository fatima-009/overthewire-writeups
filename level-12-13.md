# Natas Level 12 → Level 13 

**Goal:** Find the password for natas14. The server only accepts files that pass `exif_imagetype()`, but this function is vulnerable because it only checks the `first few bytes (magic bytes)` of a file to decide whether it's a valid image, it doesn't validate the rest of the content. This lets us prepend valid JPEG magic bytes to a PHP payload and bypass the check.

### Step 1: Set up Burp Suite

- Open Burp Suite → Proxy tab → make sure Intercept is ON.
- Route your browser through Burp's proxy, or use Burp's built-in browser.

### Step 2: Log in to the level

In the Burp browser:

http://natas13.natas.labs.overthewire.org
Username: natas13
Password: (from previous level)

### Step 3: Find the JPEG magic bytes

A quick search shows that JPEG files start with the magic bytes `FF D8 FF E0`. Create a file containing just these bytes:

"printf '\xff\xd8\xff\xe0' > natas13_magic` "

### Step 4: Write the malicious PHP payload

Create a second file, `natas13.php`, with PHP code that reads the next level's password file:

"
<?php
$file = file_get_contents('/etc/natas_webpass/natas14');
echo "\n" . $file;
?>
"

### Step 5: Concatenate both files

Combine the magic bytes and the PHP code into a single file:

```bash
cat natas13_magic natas13.php > natas13_2.php
```

The resulting file now starts with valid JPEG magic bytes (so it passes `exif_imagetype()`) while still containing executable PHP code.

### Step 6: Start Burp Suite and enable Intercept

Open Burp Suite, go to the Proxy tab, and turn Intercept ON. Connect your browser to Burp's proxy.

### Step 7: Upload the malicious file

On the natas13 page, browse to natas13_2.php and click Upload File. Burp will pause the request.

### Step 8: Inspect the Burp response/request

A new filename will have been auto-generated with a `.jpg` extension (from the hidden filename field in the form).

### Step 9: Edit the extension and forward

In Burp, change the filename's extension from `.jpg to .php`, then click Forward to send the request through (forward any remaining intercepted steps until the process completes).

### Step 10: Confirm the upload

The response confirms the file was uploaded successfully, with a link like:

The file <a href="upload/xxxxxxxxxx.php">upload/xxxxxxxxxx.php</a> has been uploaded

### Step 11: Open the uploaded file

Click the uploaded file's link (or visit it directly in the browser). Since the PHP code runs `file_get_contents()` on the password file, the password is printed immediately, no extra `?cmd= parameter` needed.

### Step 12: Get the password

The response body will contain:

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

*Lesson learned*

`exif_imagetype()` (and similar functions like `getimagesize()`) only validate a file's header/magic bytes, not its entire content. By prepending valid image magic bytes to a malicious payload, an attacker can pass this check while the rest of the file still executes as PHP once given an executable extension. Proper image upload validation should re-encode/re-process the image server-side (which strips non-image data), and uploaded files should be stored outside the web root or in a location where script execution is disabled.