# OverTheWire Bandit: Level 13 → Level 14

## Challenge Description

> The password for the next level is stored in `/etc/bandit_pass/bandit14` and
> can only be read by user `bandit14`. For this level, you don't get the next
> password, but you get a private SSH key that can be used to log into the next
> level.

> Note: `localhost` is a hostname that refers to the machine you are working on.

**Goal:** Use the provided private SSH key to log in as `bandit14`, then read
the password file that only `bandit14` can access.

**Connection:**

    ssh bandit13@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **SSH key-based authentication**: instead of a password, the client proves its
  identity with a private key whose matching public key is trusted by the server.
- **`ssh -i`**: selects the identity (private key) file to use for login.
- **`localhost`**: the machine you are currently on. Here, the Bandit server connects to itself.
- **File permissions**: only `bandit14` can read `/etc/bandit_pass/bandit14`,
  so we must become that user.

## Solution

### Step 1: Look around the home directory

```bash
    bandit13@bandit:~$ ls
    sshkey.private
```
The home directory contains a private key, which belongs to `bandit14`.

### Step 2: Confirm the file type (optional)
```bash 

    bandit13@bandit:~$ head -n 1 sshkey.private
    -----BEGIN RSA PRIVATE KEY-----
```

### Step 3: Log in as bandit14 using the key

Since we are already on the Bandit server, we connect to `localhost` on the
same SSH port (2220):

   ` bandit13@bandit:~$ ssh -i sshkey.private bandit14@localhost -p 2220`

On the first connection, SSH asks to confirm the host fingerprint:

```bash
    The authenticity of host '[localhost]:2220 ...' can't be established.
    Are you sure you want to continue connecting (yes/no)? yes
```
After typing `yes`, we are logged in as `bandit14`:

    bandit14@bandit:~$

### Step 4: Read the password file
```bash

    bandit14@bandit:~$ cat /etc/bandit_pass/bandit14
    <Natas14_password>
```
This works because we are now `bandit14`, the only user allowed to read this file.


## Alternative: Using the key from your own machine

You can also copy the key to your local computer and log in directly.
```bash

    # Copy the key (run on your local machine)
    scp -P 2220 bandit13@bandit.labs.overthewire.org:sshkey.private .

    # SSH requires strict permissions on private keys
    chmod 600 sshkey.private

    # Log in
    ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
```

## Troubleshooting

|                     Problem                          |                             Cause / Fix                                     |

| `Permissions 0644 for 'sshkey.private' are too open` | Run `chmod 600 sshkey.private`. SSH refuses keys readable by others.        |
| `Connection refused` on localhost                    | Make sure to add `-p 2220`, since the default port 22 is not used.          |
| Password prompt appears                              | The key was not accepted. Check the filename and the username (`bandit14`). |

## Key Takeaways

- A private key can replace a password for authentication. Anyone holding it can
  log in as that user, so private keys must be protected carefully.
- Always use `-p 2220` for Bandit, including when connecting to `localhost`.
- Private key files must have restrictive permissions (`600`) or SSH will refuse them.
- Access to a restricted file is gained by becoming the user who owns it.