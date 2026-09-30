# OverTheWire Writeups

Solutions and notes for the [OverTheWire](https://overthewire.org/wargames/) wargames, a set of Linux and web-security challenges used to practice command-line skills, permissions, scripting, and basic exploitation.

Each writeup explains **what the level asks for, the commands used, and why they work** not just the final answer. The goal was to actually understand each concept, not just pass the level.

## 🧠 Wargames Covered

### Bandit
Focuses on basic Linux command-line skills: navigating the filesystem, reading files, permissions, and simple scripting.

| Level | Topic |
| --- | --- |
| [0 → 1](./bandit/level-0-1.md) | Connecting via SSH, reading a file |
| [1 → 2](./bandit/level-1-2.md) | Handling filenames starting with `-` |
| [2 → 3](./bandit/level-2-3.md) | Handling filenames containing spaces |
| [3 → 4](./bandit/level-3-4.md) | Finding hidden (dotfile) files with `ls -la` |
| [4 → 5](./bandit/level-4-5.md) | Finding the only human-readable file with `file` |
| [5 → 6](./bandit/level-5-6.md) | Finding a file by size, permissions and owner with `find` |
| [6 → 7](./bandit/level-6-7.md) | Finding a file system-wide by owner, group and size |
| [7 → 8](./bandit/level-7-8.md) | Extracting a value next to a keyword with `grep` |
| [8 → 9](./bandit/level-8-9.md) | Finding the one non-duplicate line with `sort` \| `uniq -u` |
| [9 → 10](./bandit/level-9-10.md) | Extracting human-readable strings from a binary file with `strings`/`grep` |
| [10 → 11](./bandit/level-10-11.md) | Decoding Base64-encoded data |
| [11 → 12](./bandit/level-11-12.md) | Decoding a ROT13-obfuscated string with `tr` |

*(more levels added as they're solved)*

### Natas
Focuses on web application security: source code review, HTTP headers, cookies, and common web vulnerabilities.

| Level | Topic |
| --- | --- |
| [1 → 2](./natas/level-1-2.md) | Bypassing client-side right-click/devtools blocks via view-source |
| [2 → 3](./natas/level-2-3.md) | Exposed directory listing via a linked image path |
| [3 → 4](./natas/level-3-4.md) | Sensitive paths leaked through `robots.txt` |
| [4 → 5](./natas/level-4-5.md) | Spoofing the `Referer` header |
| [5 → 6](./natas/level-5-6.md) | Bypassing a client-controlled `loggedin` cookie |
| [6 → 7](./natas/level-6-7.md) | Sensitive PHP `.inc` include file exposed and directly accessible |
| [7 → 8](./natas/level-7-8.md) | Local File Inclusion (LFI) via an unsanitized `page` GET parameter |
| [8 → 9](./natas/level-8-9.md) | Recovering a secret from reversed hex + Base64 obfuscation in PHP source |
| [9 → 10](./natas/level-9-10.md) | OS command injection via unsanitized input passed to `passthru()`/`grep` |
| [10 → 11](./natas/level-10-11.md) | Command injection with a character blacklist, bypassed |
| [11 → 12](./natas/level-11-12.md) | Forging an XOR-encoded auth cookie |
| [12 → 13](./natas/level-12-13.md) | Bypassing a file-upload extension check to upload a PHP webshell |
| [13 → 14](./natas/level-13-14.md) | Bypassing file-upload content/magic-byte validation |
| [14 → 15](./natas/level-14-15.md) | SQL injection login bypass by commenting out the query |
| [15 → 16](./natas/level-15-16.md) | Blind SQL injection to extract a password character by character |

*(more levels added as they're solved)*

## 🛠️ Skills Practiced

- Linux command line (`ls`, `cat`, `grep`, `find`, `ssh`, permissions, etc.)
- Reading and reasoning about shell behavior
- Basic web security concepts (HTTP, cookies, source inspection)
- Web app exploitation with Burp Suite (SQL injection, encryption oracles, filehandle tricks, command injection)
- Problem-solving and documenting a process clearly

## 📁 Structure

```text
overthewire-writeups/
├── bandit/
│   ├── level-0-1.md
│   ├── level-1-2.md
│   ├── level-2-3.md
│   ├── level-3-4.md
│   ├── level-4-5.md
│   ├── level-5-6.md
│   ├── level-6-7.md
│   ├── level-7-8.md
│   ├── level-8-9.md
│   ├── level-9-10.md
│   ├── level-10-11.md
│   ├── level-11-12.md
│   └── ...
├── natas/
│   ├── level-1-2.md
│   ├── level-2-3.md
│   ├── level-3-4.md
│   ├── level-4-5.md
│   ├── level-5-6.md
│   ├── level-6-7.md
│   ├── level-7-8.md
│   ├── level-8-9.md
│   ├── level-9-10.md
│   ├── level-10-11.md
│   ├── level-11-12.md
│   ├── level-12-13.md
│   ├── level-13-14.md
│   ├── level-14-15.md
│   ├── level-15-16.md
│   └── ...
└── README.md
```
## ✍️ About

Written by [Fatima Basharat](https://github.com/fatima-009) while learning Linux fundamentals and web security alongside full-stack development.

> Note: Passwords for each level are intentionally omitted or partially masked in these writeups — the focus is on the method, not the answer.
