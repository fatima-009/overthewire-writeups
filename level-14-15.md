# OverTheWire Bandit: Level 14 → Level 15

## Challenge Description

> The password for the next level can be retrieved by submitting the password of
> the current level to **port 30000 on localhost**.

**Goal:** Send the current level's password (bandit14's password) to a service
listening on `localhost:30000`, which replies with the password for the next level.

**Connection:**

   ` ssh bandit14@bandit.labs.overthewire.org -p 2220`

(Log in as `bandit14` using either the password obtained from the previous
level, or the private key from Level 13 → 14.)

## Concepts Involved

- **Ports and network services**: a program can listen on a TCP port and respond to clients.
- **`localhost`**: the machine we are currently logged into.
- **`nc` (netcat)**: a simple tool that opens a TCP connection and lets you send
  and receive raw data.
- **Pipes (`|`)**: feed the output of one command into another.

## Solution

### Step 1: Get the current level's password

Since we are logged in as `bandit14`, we can read the password file directly:

    bandit14@bandit:~$ cat /etc/bandit_pass/bandit14
    <redacted>

### Step 2: Connect to port 30000 with netcat
```bash
    bandit14@bandit:~$ nc localhost 30000

The connection stays open and waits for input. Paste the password and press Enter:

    <bandit14 password>
    Correct!
    <Natas15_password>
```

The service replies with `Correct!` followed by the password for `bandit15`.

### Alternative: One-liner with a pipe

Instead of pasting the password manually, pipe it straight into netcat:
```bash

    bandit14@bandit:~$ cat /etc/bandit_pass/bandit14 | nc localhost 30000
    Correct!
    <Natas15_password>
``` 

### Optional: Check that the port is open
```bash

    bandit14@bandit:~$ nc -zv localhost 30000
    Connection to localhost 30000 port [tcp/*] succeeded!
```
## Result

    Password for bandit15: <******>

## Troubleshooting

|                        Problem                                      |                          Cause / Fix                                |

| Connection hangs with no response                                   | The service is waiting for the password. 
                                                                        Type or paste it and press Enter.                                   |
| Service replies `Wrong! Please enter the correct current password.` | Password typed incorrectly (extra space or missing character).
                                                                        Use the pipe method instead.                                        |
| `Connection refused`                                                | Check the port number (`30000`) and that you are using `localhost`. |

## Key Takeaways

- `nc` (netcat) is a flexible tool for talking to TCP services manually.
- Piping a file into `nc` avoids copy-paste mistakes.
- Services can authenticate users by a shared secret sent over a plain TCP
  connection. This is **not secure**, since the data travels unencrypted
  (the next level introduces SSL/TLS for this reason).
