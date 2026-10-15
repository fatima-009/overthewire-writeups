# OverTheWire Writeups

Solutions and notes for the [OverTheWire](https://overthewire.org/wargames/) wargames, a collection of Linux and web-security challenges used to practice command-line skills, permissions, scripting, source-code analysis, and basic exploitation.

Each writeup explains **what the level asks for, the commands or techniques used, and why they work** not just the final answer. The goal is to understand the underlying concepts while learning practical cybersecurity skills.

## 🧠 Wargames Covered

### Bandit

Focuses on Linux fundamentals, file handling, permissions, SSH, text processing, compression, networking, and basic scripting.

| Level                              | Topic                                                               |
| ---------------------------------- | ------------------------------------------------------------------- |
| [0 → 1](./bandit/level-0-1.md)     | Connecting via SSH and reading a file                               |
| [1 → 2](./bandit/level-1-2.md)     | Handling filenames starting with `-`                                |
| [2 → 3](./bandit/level-2-3.md)     | Handling filenames containing spaces                                |
| [3 → 4](./bandit/level-3-4.md)     | Finding hidden files with `ls -la`                                  |
| [4 → 5](./bandit/level-4-5.md)     | Identifying human-readable files with `file`                        |
| [5 → 6](./bandit/level-5-6.md)     | Finding files by size, permissions, and other properties            |
| [6 → 7](./bandit/level-6-7.md)     | Searching the filesystem by owner, group, and size                  |
| [7 → 8](./bandit/level-7-8.md)     | Extracting a value next to a keyword with `grep`                    |
| [8 → 9](./bandit/level-8-9.md)     | Finding a unique line with `sort` and `uniq -u`                     |
| [9 → 10](./bandit/level-9-10.md)   | Extracting readable strings from binary data                        |
| [10 → 11](./bandit/level-10-11.md) | Decoding Base64-encoded data                                        |
| [11 → 12](./bandit/level-11-12.md) | Decoding ROT13-obfuscated text with `tr`                            |
| [12 → 13](./bandit/level-12-13.md) | Reversing a hexdump and extracting compressed data                  |
| [13 → 14](./bandit/level-13-14.md) | Using an SSH private key to authenticate                            |
| [14 → 15](./bandit/level-14-15.md) | Sending a password to a local port with `nc`                        |
| [15 → 16](./bandit/level-15-16.md) | Connecting securely to a service using OpenSSL                      |
| [16 → 17](./bandit/level-16-17.md) | Identifying a listening port and retrieving an SSH key              |
| [17 → 18](./bandit/level-17-18.md) | Comparing files with `diff`                                         |
| [18 → 19](./bandit/level-18-19.md) | Working around a restricted login shell                             |
| [19 → 20](./bandit/level-19-20.md) | Using a setuid binary to execute a command with elevated privileges |
| [20 → 21](./bandit/level-20-21.md) | Communicating with a local network service using `nc`               |

### Natas

Focuses on web application security, HTTP requests, cookies, authentication, source-code analysis, file inclusion, command injection, and SQL injection.

| Level                             | Topic                                                            |
| --------------------------------- | ---------------------------------------------------------------- |
| [0 → 1](./natas/level-0-1.md)     | Inspecting HTML source code to discover a password in a comment  |
| [1 → 2](./natas/level-1-2.md)     | Bypassing client-side right-click restrictions using page source |
| [2 → 3](./natas/level-2-3.md)     | Discovering exposed files through an image path                  |
| [3 → 4](./natas/level-3-4.md)     | Discovering sensitive paths through `robots.txt`                 |
| [4 → 5](./natas/level-4-5.md)     | Understanding and spoofing the HTTP `Referer` header             |
| [5 → 6](./natas/level-5-6.md)     | Manipulating a client-controlled authentication cookie           |
| [6 → 7](./natas/level-6-7.md)     | Finding an exposed PHP include file                              |
| [7 → 8](./natas/level-7-8.md)     | Exploiting Local File Inclusion (LFI)                            |
| [8 → 9](./natas/level-8-9.md)     | Reversing Base64, hexadecimal, and string transformations        |
| [9 → 10](./natas/level-9-10.md)   | Understanding OS command injection through unsanitized input     |
| [10 → 11](./natas/level-10-11.md) | Exploring command-injection filters and character restrictions   |
| [11 → 12](./natas/level-11-12.md) | Analyzing an XOR-encoded authentication cookie                   |
| [12 → 13](./natas/level-12-13.md) | Testing file-upload extension validation                         |
| [13 → 14](./natas/level-13-14.md) | Examining file-upload content and magic-byte validation          |
| [14 → 15](./natas/level-14-15.md) | Understanding SQL injection in a login form                      |
| [15 → 16](./natas/level-15-16.md) | Extracting information using Boolean-based blind SQL injection   |
| [16 → 17](./natas/level-16-17.md) | Investigating filtered command injection and blind output        |
| [17 → 18](./natas/level-17-18.md) | Using timing differences to investigate blind SQL injection      |
| [18 → 19](./natas/level-18-19.md) | Analyzing session IDs and session behavior                       |
| [19 → 20](./natas/level-19-20.md) | Investigating session identifiers and cookie encoding            |
| [20 → 21](./natas/level-20-21.md) | Exploring session manipulation and server-side session handling  |

*Note: The Natas topics for later levels are brief descriptions. Verify them against the actual challenges before publishing detailed solutions.*

## 🛠️ Skills Practiced

* **Linux command line:** `ls`, `cat`, `grep`, `find`, `sort`, `uniq`, `strings`, and other utilities.
* **File handling:** Hidden files, unusual filenames, permissions, file types, compression, and hexdumps.
* **Networking:** SSH, Netcat (`nc`), OpenSSL, and local network services.
* **Shell behavior:** Input parsing, restricted shells, command execution, and basic scripting.
* **Web security:** HTTP headers, cookies, authentication, source inspection, and session management.
* **Web vulnerabilities:** Local File Inclusion (LFI), command injection, file-upload validation issues, and SQL injection.
* **Security testing:** Analyzing validation weaknesses, investigating application behavior, and documenting reproducible findings.
* **Technical writing:** Explaining objectives, methodology, commands, output, and lessons learned.

## 📁 Repository Structure

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
│   ├── level-12-13.md
│   ├── level-13-14.md
│   ├── level-14-15.md
│   ├── level-15-16.md
│   ├── level-16-17.md
│   ├── level-17-18.md
│   ├── level-18-19.md
│   ├── level-19-20.md
│   ├── level-20-21.md
│   └── ...
├── natas/
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
│   ├── level-12-13.md
│   ├── level-13-14.md
│   ├── level-14-15.md
│   ├── level-15-16.md
│   ├── level-16-17.md
│   ├── level-17-18.md
│   ├── level-18-19.md
│   ├── level-19-20.md
│   ├── level-20-21.md
│   └── ...
└── README.md
```

## ✍️ About

Written by [Fatima Basharat](https://github.com/fatima-009) while learning Linux fundamentals and web security alongside full-stack development.

> **Note:** Passwords for each level are intentionally omitted or partially masked. The focus is on understanding the methodology rather than simply copying answers.

These writeups are intended for educational use with authorized practice environments such as OverTheWire. Each solution should explain the underlying vulnerability or Linux concept and include enough context to reproduce the result responsibly.
