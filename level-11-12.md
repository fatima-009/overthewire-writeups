# OverTheWire Bandit: Level 11 → Level 12

## Challenge Description

> The password for the next level is stored in the file `data.txt`, where all
> lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions.

**Goal:** Decode the contents of `data.txt` to retrieve the password for `bandit12`.

**Connection:**

    ssh bandit11@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **ROT13 cipher**: a simple letter-substitution cipher (a special case of the
  Caesar cipher) that replaces each letter with the one 13 positions after it
  in the alphabet. Since the English alphabet has 26 letters, applying ROT13
  twice returns the original text, so the same operation both encodes and decodes.

- **`tr` command**: a Unix utility that translates or deletes characters from
  standard input.

- **Pipes (`|`)**: send the output of one command as the input of another.

## Solution

### Step 1: Log in and inspect the file
```bash

    bandit11@bandit:~$ ls
    data.txt

    bandit11@bandit:~$ cat data.txt
    Gur cnffjbeq vf <rotated text>
```

The output is readable-looking text, but the letters are shifted, which matches
the `ROT13` hint from the challenge.

### Step 2: Decode with `tr`

We map each letter in `A-Z` / `a-z` to the letter 13 positions ahead, wrapping around:

```bash
    bandit11@bandit:~$ cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
    The password is <Natas12_password>
```

**How it works:**

|   Part         |                            Meaning                                                    |

| `A-Za-z`       | The source set: all uppercase, then all lowercase letters                             |
| `N-ZA-Mn-za-m` | The target set: the alphabet shifted by 13 (N→Z then A→M, and the same for lowercase) |

`tr` replaces each character from the first set with the character at the same position in the second set. 
For example, `G` becomes `T`, `u` becomes `h`, `r` becomes `e`, which turns `Gur` into `The`.
   

## Key Takeaways

- ROT13 is **not encryption**. It provides obfuscation only and is trivially reversible.
- `tr` is a quick and powerful tool for character-level transformations.
- ROT13 is symmetric: running the same command on the output returns the original text.
