# Natas 24 - 25 — OverTheWire Wargame Writeup

## Challenge
Natas25 has a `lang` parameter that filters out `../` to prevent path traversal, but the filter isn't recursive — so `....//` becomes `../` once the inner pattern is stripped, allowing traversal outside the intended directory.

## Vulnerability
The application logs each request (including the `User-Agent` header) to a file named:
```
/var/www/natas/natas25/logs/natas25_<PHPSESSID>.log
```
The `lang` parameter is later used in an `include()` call. Combining path traversal with this behavior means we can:
1. Make the server write attacker-controlled data (our `User-Agent`) into a log file.
2. Use the same `lang` parameter to traverse to and `include()` that log file, causing any PHP code inside it to execute — classic **log poisoning + Local File Inclusion (LFI)**.

## Exploitation Steps

**1. Identify the log file path** using your session cookie:
```
http://natas25.natas.labs.overthewire.org/?lang=....//....//....//....//....//var/www/natas/natas25/logs/natas25_<PHPSESSID>.log
```

**2. Intercept the request with Burp Suite** and send it to Repeater.

**3. Modify the `User-Agent` header** to inject PHP code:
```
User-Agent: <?php echo shell_exec("cat /etc/natas_webpass/natas26"); ?>
```

**4. Send the request twice.**
- The first request writes the payload into the log file (it may not execute correctly right away, since PHP includes the file as it's still being written).
- The second identical request writes a new log entry *and* includes the file again, this time the previously-saved PHP code is fully written to disk and executes cleanly via `include()`.

## Result
The response reveals the contents of `/etc/natas_webpass/natas26`, giving the password for the next level.

## Key Takeaways
- Naive blacklist filters (like stripping `../`) can be bypassed with patterns such as `....//`.
- Never trust or reflect user-controlled input (like `User-Agent`) into files that are later executed or included.
- Log poisoning is a powerful technique when LFI and attacker-controlled logging intersect.