# Natas Level 23 — Writeup

**Challenge:** [OverTheWire: Natas Level 23](http://natas23.natas.labs.overthewire.org)
**Category:** Web Exploitation / PHP Type Juggling & Loose Comparison
**Tools used:** Just a browser, no Burp Suite or interception needed for this level


## 1. Challenge Overview

Natas Level 23 exposes its own PHP source directly on the page:

```php
<?php
    if(array_key_exists("passwd",$_REQUEST)){
        if(strstr($_REQUEST["passwd"],"iloveyou") && ($_REQUEST["passwd"] > 10 )){
            echo "<br>The credentials for the next level are:<br>";
            echo "<pre>Username: natas24 Password: <censored></pre>";
        }
        else{
            echo "<br>Wrong!<br>";
        }
    }
    // morla / 10111
?>
```

Unlike the earlier levels, this one doesn't require intercepting or modifying traffic at all. The entire challenge can be solved by typing the right value into the URL bar. The interesting part is entirely in *understanding the PHP*, not in manipulating the HTTP request.

Two conditions must both evaluate to `true`:

1. `strstr($_REQUEST["passwd"], "iloveyou")`: the input must **contain** the substring `"iloveyou"` somewhere in it.
2. `$_REQUEST["passwd"] > 10`: the input, compared as if it were a number, must come out greater than 10.

At first glance these look mutually exclusive how can a value be "a number greater than 10" and also "a string containing a word"? PHP's type juggling makes both possible at once.


## 2. Understanding the Conditions

### a) `strstr()` is a substring search, not an equality check
`strstr($haystack, $needle)` returns the **portion of the string starting from the first occurrence** of `$needle`  or `false` if it isn't found. Any string that has `"iloveyou"` in it anywhere returns a non-empty (truthy) result, whether that substring is at the start, middle, or end of the input.

### b) Comparing a string to an integer triggers type juggling
The second condition compares a **string** to the **integer** `10` using `>`. In PHP, when you compare a string to a number this way, PHP converts the string's **leading numeric characters** into a number for the comparison and ignores the rest. So a value like:

```
100iloveyou
```

gets read as numerically `100` for the purpose of `> 10`, while the string still literally contains `"iloveyou"` for `strstr()` to find.

If you're unsure how a PHP function or comparison behaves, the reliable way to check is to test it directly — either in a local PHP shell (`php -a`) or an online PHP sandbox rather than assuming based on intuition. Testing this pair of conditions directly shows that any string starting with a number above 10, followed by any other characters, satisfies the numeric check; and any string containing `"iloveyou"` anywhere satisfies the `strstr()` check.


## 3. Solution

Since the password needs to:
1. Start with a number greater than 10, and
2. Contain the substring `"iloveyou"` somewhere in it,

a value like:

```
passwd=100iloveyou
```

satisfies both conditions in a single string.

### Steps
1. Log into `natas23` using the credentials carried over from Level 22.
2. In the browser's address bar, append the query parameter directly:

   ```
   http://natas23.natas.labs.overthewire.org/?passwd=100iloveyou
   ```

3. Load the page. The response now shows:

   ```
   The credentials for the next level are:
   Username: natas24
   Password: <Natas24-password>
   ```
