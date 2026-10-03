# OverTheWire Bandit: Level 12 → Level 13

## Challenge Description

> The password for the next level is stored in the file `data.txt`, which is a
> hexdump of a file that has been repeatedly compressed. For this level it may
> be useful to create a directory under `/tmp` in which you can work.

**Goal:** Reverse the hexdump and decompress the file layer by layer until the
plain-text password appears.

**Connection:**

    ssh bandit12@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **Hexdump reversal (`xxd -r`)**: converts a hex dump back into the original binary file.

- **`file` command**: detects the real file type from its content, not its extension.

- **Compression formats**: gzip (`gunzip`), bzip2 (`bunzip2`) and tar (`tar -xf`).

## Solution

### Step 1: Look at the file
```bash

    bandit12@bandit:~$ ls
    data.txt
    bandit12@bandit:~$ head data.txt
```

The file contains a hexdump (offsets followed by hex bytes), which confirms the
hint from the challenge.

### Step 2: Create a working directory and copy the file

The home directory is read-only, so we work in `/tmp`.

```bash

    bandit12@bandit:~$ mkdir /tmp/random_dir
    bandit12@bandit:~$ cd /tmp/random_dir
    bandit12@bandit:/tmp/random_dir$ cp ~/data.txt .
    bandit12@bandit:/tmp/random_dir$ mv data.txt data
    bandit12@bandit:/tmp/random_dir$ ls
    data
```
> Tip: on a shared server, choose a hard-to-guess directory name, or use
> `cd $(mktemp -d)`, which creates and enters a unique directory.

### Step 3: Reverse the hexdump
```bash 

    bandit12@bandit:/tmp/random_dir$ xxd -r data > binary
    bandit12@bandit:/tmp/random_dir$ file binary
    binary: gzip compressed data, was "data2.bin", ...
``` 
### Step 4: Decompress layer by layer

At each layer: run `file`, then use the matching tool.

**Layer 1: gzip**
```bash

    bandit12@bandit:/tmp/random_dir$ mv binary binary.gz
    bandit12@bandit:/tmp/random_dir$ gunzip binary.gz
    bandit12@bandit:/tmp/random_dir$ file binary

    binary: bzip2 compressed data, block size = 900k
``` 

**Layer 2: bzip2**
```bash

    bandit12@bandit:/tmp/random_dir$ bunzip2 binary

    bunzip2: Can't guess original name for binary -- using binary.out
    bandit12@bandit:/tmp/random_dir$ file binary.out
    binary.out: gzip compressed data, was "data4.bin", ...
```
`bunzip2` expects a `.bz2` extension. Without it, it still works but writes the
result to `binary.out`. Renaming the file to `binary.bz2` first avoids the warning.

**Layer 3: gzip again**

```bash
    bandit12@bandit:/tmp/random_dir$ mv binary.out binary.gz
    bandit12@bandit:/tmp/random_dir$ gunzip binary.gz
    bandit12@bandit:/tmp/random_dir$ file binary
    binary: POSIX tar archive (GNU)
 ``` 

**Layer 4: tar**
```bash

    bandit12@bandit:/tmp/random_dir$ tar -xf binary
    bandit12@bandit:/tmp/random_dir$ ls
    <extracted file> binary data
```
### Step 5: Read the password

Once `file` reports ASCII text:
```bash

    bandit12@bandit:/tmp/random_dir$ file <final file>
    <final file>: ASCII text
    bandit12@bandit:/tmp/random_dir$ cat <final file>
    The password is <natas_13_password>
```

## Key Takeaways

- Never trust file extensions. Use `file` to identify the real format at every step.
- The workflow is a loop: **identify → rename → decompress**.
- `gunzip` needs a `.gz` extension and `bunzip2` needs `.bz2`, otherwise they
  complain or create `.out` files.
- Delete intermediate files you no longer need to keep the directory readable.
