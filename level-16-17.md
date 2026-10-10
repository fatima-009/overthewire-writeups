# OverTheWire Bandit: Level 16 → Level 17

## Challenge Description

> The credentials for the next level can be retrieved by submitting the password
> of the current level to a **port on localhost in the range 31000 to 32000**.
> First find out which of these ports have a server listening on them. Then find
> out which of those speak SSL/TLS and which don't. There is only 1 server that
> will give the next credentials, the others will simply send back to you
> whatever you send to it.

**Goal:** Scan the port range, find the one TLS service that accepts the current
password, and use the credentials it returns to log in as `bandit17`.

**Connection:**

    ssh bandit16@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **Port scanning with `nmap`**: discovers which TCP ports are open and,
  with `-sV`, tries to identify the service behind each one.

- **SSL/TLS services**: only some of the open ports speak TLS, so we need to
  tell them apart from plain-text services.

- **`openssl s_client`**: TLS client used to talk to the encrypted services (as in Level 15 → 16).

- **Echo servers**: some ports simply return whatever is sent to them, which
  makes them look "working" even though they are not the target.

- **SSH private keys**: the correct service returns a private key instead of a password.

## Solution

### Step 1: Scan the port range
```bash
    bandit16@bandit:~$ nmap -p 31000-32000 localhost

Example output (your ports will differ):

    PORT      STATE SERVICE
    <port1>/tcp open  unknown
    <port2>/tcp open  unknown
    <port3>/tcp open  unknown
    <port4>/tcp open  unknown
    <port5>/tcp open  unknown
```

Only a handful of ports in the range are open.

### Step 2: Identify which ports speak SSL/TLS

Run service detection against the open ports:

   ` bandit16@bandit:~$ nmap -sV -p 31000-32000 localhost`

Ports reported as `ssl/unknown` speak TLS. Ports reported as plain `echo` or
`unknown` do not. This narrows the candidates to a small list.

### Step 3: Try the password on each TLS port

Send the current password (bandit16's password) to each TLS candidate:
```bash
    bandit16@bandit:~$ cat /etc/bandit_pass/bandit16 | openssl s_client -connect localhost:<port> -quiet

- Some ports simply **echo the password back**, which means they are not the target.
- Exactly one port replies with `Correct!` followed by an **RSA private key**:

      Correct!
      -----BEGIN RSA PRIVATE KEY-----
      <redacted>
      -----END RSA PRIVATE KEY-----
```

> The `-quiet` option prevents `KEYUPDATE` / `RENEGOTIATING` issues if the
> password starts with `R` or `k`.

### Step 4: Save the key and set safe permissions

The home directory is read-only, so work in a temporary directory:

```bash

    bandit16@bandit:~$ cd $(mktemp -d)
    bandit16@bandit:/tmp/tmp.XXXXXXXXXX$ nano sshkey.private

Paste the whole key (from `-----BEGIN RSA PRIVATE KEY-----` to
`-----END RSA PRIVATE KEY-----`), save, and then restrict permissions, because
SSH refuses keys that other users can read:

    bandit16@bandit:/tmp/tmp.XXXXXXXXXX$ chmod 600 sshkey.private
```

### Step 5: Log in as bandit17
```bash

    bandit16@bandit:/tmp/tmp.XXXXXXXXXX$ ssh -i sshkey.private bandit17@localhost -p 2220
```

### Step 6: Read the password
```bash

    bandit17@bandit:~$ cat /etc/bandit_pass/bandit17
    <Natas17_password>
```

## Key Takeaways

- Combine tools: `nmap` to discover services, `openssl s_client` to talk to TLS ones.

- Not every open port is useful. Echo services are decoys, so always verify the response.

- A service may return a **private key** instead of a password. Save it carefully
  and set permissions to `600` before using it with `ssh -i`.

- Keep scans narrow (only the stated range) and run them only against `localhost`.

