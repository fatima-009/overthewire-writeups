# OverTheWire Bandit: Level 15 → Level 16

## Challenge Description

> The password for the next level can be retrieved by submitting the password of
> the current level to **port 30001 on localhost** using **SSL/TLS encryption**.
>
> Helpful note: Getting "DONE", "RENEGOTIATING" or "KEYUPDATE"? Read the
> "CONNECTED COMMANDS" section in the manpage.

**Goal:** Send the current level's password (bandit15's password) to the
TLS-enabled service on `localhost:30001` and read the reply.

**Connection:**

    ssh bandit15@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **SSL/TLS**: a protocol that encrypts network traffic. Unlike the previous
  level, a plain `nc` connection will not work because the server expects a TLS handshake.

- **`openssl s_client`**: a generic TLS client that can connect to a TLS server
  and exchange data.

- **Connected commands**: while `s_client` runs in interactive mode, lines
  beginning with certain letters (such as `R` or `k`) are treated as commands
  (renegotiate, key update) instead of data. The `-quiet` option avoids this.

- **Pipes (`|`)**: feed the output of one command into another.

## Solution

### Step 1: Why plain netcat fails
```bash
    bandit15@bandit:~$ nc localhost 30001
```

The service does not respond sensibly, because it expects a TLS handshake
rather than plain text.

### Step 2: Connect with `openssl s_client`
```bash

    bandit15@bandit:~$ openssl s_client -connect localhost:30001

This prints the certificate and session details, then waits for input. Paste the
current password and press Enter:

    <bandit15 password>
    Correct!
    <bandit16 password>
```

If your password starts with a letter such as `R` or `k`, `s_client` may
interpret it as a command and print `RENEGOTIATING` or `KEYUPDATE`. The `-quiet`
option fixes this (see below).

### Step 3: Cleaner one-liner with `-quiet`

`-quiet` hides the certificate and session output and also stops `s_client`
from treating input lines as commands.

```bash
    bandit15@bandit:~$ cat /etc/bandit_pass/bandit15 | openssl s_client -connect localhost:30001 -quiet
    Correct!
    <bandit16 password>
```

### Alternative: Using ncat with SSL

```bash
    bandit15@bandit:~$ cat /etc/bandit_pass/bandit15 | ncat --ssl localhost 30001
```
## Result

    Password for bandit16: <****>

## Key Takeaways

- Encrypted services need an encryption-aware client. `openssl s_client` plays
  the role that `nc` played for plain TCP.

- `-quiet` makes `s_client` behave like a plain pipe-friendly client.

- Piping the password file into the command avoids typing errors.

- TLS protects data in transit, but the secret itself (the password) is still
  just a shared string, so it must still be kept private.