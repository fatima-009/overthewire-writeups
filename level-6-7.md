# Natas Level 6 → 7

**Goal:** Find the password for natas8. This level demonstrates a classic Local File Inclusion (LFI) vulnerability through a page URL parameter.

# Step 1: Access the level

http://natas7.natas.labs.overthewire.org

Username: natas7
Password: (from level 5)

# Step 2: Observe the URL structure

The page has navigation links like "Home" and "About", and the URL looks like:

http://natas7.natas.labs.overthewire.org/index.php?page=home

This tells you the server is including a file based on the page parameter likely something like:

```php
include($_GET['page'] . ".php");
``` 

# Step 3: View the source code (if available)

This usually confirms the include logic, showing something like:

```php
<?
    if( ! isset( $_GET['page'] ) || ! is_string( $_GET['page'] ) ) {
        $page = "home";
    } else {
        $page = $_GET['page'];
    }
    ...
    include($page . ".php");
?>
```

Since there's no filtering on .. or absolute paths, this is exploitable.

# Step 4: Exploit the LFI to read local files

Try reading the /etc/passwd file (a classic proof of LFI), or more specifically, try to read the natas8 password file directly. Passwords for natas levels are typically stored at:

/etc/natas_webpass/natas8

Craft the URL:

http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8

# Step 5: Get the password

If successful, the password file's contents will be included directly into the page output:

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

*Lesson learned*

Passing user input directly into an include() (or similar file-loading function) without validation lets an attacker read arbitrary files on the server or even achieve remote code execution if they can control file content (e.g., via log poisoning or file upload). User input used in file paths must always be validated against a strict allow-list, never trusted directly.