# Natas Level 5 → 6

**Goal:** Find the password for natas7. This level exposes its PHP source code, revealing that a "secret" value is read from an included file — and that file path is guessable/accessible.

# Step 1: Access the level

http://natas6.natas.labs.overthewire.org

Username: natas6
Password: (from level 5)

# Step 2: Read the page

The page shows a form asking you to input a "secret" value to gain access.

# Step 3: View the PHP source code

Natas conveniently provides a link on the page to view the source (labeled something like "View sourcecode"), or you can browse directly to:

http://natas6.natas.labs.overthewire.org/index-source.html

Inside, you'll see something like:

```php
<?
include "includes/secret.inc";

if(array_key_exists("submit", $_POST)) {
    if($secret == $_POST['secret']) {
        print "Access granted. The password for natas7 is <censored>";
    } else {
        print "Wrong secret";
    }
}
?>
```

This tells you the actual secret value is defined inside includes/secret.inc.

## Step 4: View the included file directly

Since it's just a normal file path, browse to:

http://natas6.natas.labs.overthewire.org/includes/secret.inc

You'll see something like:

```php
<?
$secret = "FOEIUWGHFEEUHOFUOIU";
?>
```

(the actual value will be some random string)

# Step 5: Submit the secret

Go back to the main natas6 page, enter that secret value into the form, and submit it.

# Step 6: Get the password

The page will respond with:

Access granted. The password for natas7 is XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

*Lesson learned*

Including server-side logic files (like .inc files) inside the web root means anyone can request them directly and read their raw contents, since the server doesn't always know to treat .inc as executable PHP. Sensitive configuration or secret values should never live in web-accessible files — they belong outside the document root or in properly protected config storage.