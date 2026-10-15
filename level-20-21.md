# OverTheWire Bandit: Level 20 → Level 21

## Challenge Description

> There is a **setuid binary** in the homedirectory that does the following: it
> makes a connection to localhost on the port you specify as a commandline
> argument. It then reads a line of text from the connection and compares it to
> the password in the previous level (bandit20). If the password is correct, it
> will transmit the password for the next level (bandit21).

> NOTE: Try connecting to your own network daemon to see if it works as you think.

**Goal:** Start your own network listener that sends the current password,
then make the setuid binary connect to it so it replies with the next password.

**Connection:**

    ssh bandit20@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **Setuid binary**: runs with the privileges of its owner (`bandit21`), so it
  is allowed to read the next level's password.

- **Client / server roles**: the binary (`suconnect`) acts as a **client**, so
  we must provide the **server** it connects to.

- **`nc -l` (netcat in listen mode)**: opens a port and waits for a connection.

- **Background jobs (`&`)**: let us run the listener and the binary in the same
  terminal session.

- **Pipes (`|`)**: send the password file's content into the listener.

## Solution

### Step 1: Look at the binary
```bash
    bandit20@bandit:~$ ls -l
    -rwsr-x--- 1 bandit21 bandit20 <size> <date> suconnect

    bandit20@bandit:~$ ./suconnect
    Usage: ./suconnect <portnumber>
```

    This program will connect to the given port on localhost using TCP. If it
    receives the correct password from the other side, this program will then
    transmit the password for the next level back to the other side.

The `s` in the permissions confirms the setuid bit. The binary connects to the
port we give it and compares what it receives with the bandit20 password.

### Step 2: Start a listener that sends the current password

Run `nc` in listen mode on a free port above 1024 (here `4444`) and put it in
the background with `&`. The password is piped in, so it will be sent to
whoever connects:
```bash

    bandit20@bandit:~$ cat /etc/bandit_pass/bandit20 | nc -l -p 4444 &
    [1] <pid>
```

### Step 3: Run the setuid binary against our listener
```bash

    bandit20@bandit:~$ ./suconnect 4444
    Read: <bandit20 password>
    Password matches, sending next password
```

The binary connected to our listener, read the password we sent, found it
correct, and sent back the password for `bandit21`.

### Step 4: Read the reply

Our listener receives the data from `suconnect` and prints it to the terminal:

    <bandit21 password>

### Alternative: Use two terminals

Instead of a background job, open two SSH sessions as `bandit20`:

    # Terminal 1: start the listener and send the current password
    bandit20@bandit:~$ cat /etc/bandit_pass/bandit20 | nc -l -p 4444

    # Terminal 2: run the setuid binary
    bandit20@bandit:~$ ./suconnect 4444

The next password appears in **Terminal 1**.

## Troubleshooting

               Problem                 |                                        Cause / Fix 

| `nc: Address already in use`         | Another user or process is using that port. Pick a different one (for example `4567`, `31337`). |
| `Could not connect` from `suconnect` | The listener is not running yet, or the port numbers do not match. Start `nc` first.            |
| `Password does not match`            | The listener sent the wrong data. Make sure you pipe `/etc/bandit_pass/bandit20`.               |
| No output appears                    | The listener's output may be hidden. Press Enter, or use the two-terminal method.               |
| Background job still running         | Check with `jobs` and close with `kill %1`.                                                     |