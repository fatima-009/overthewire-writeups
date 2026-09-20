# OverTheWire Writeups

Solutions and notes for the [OverTheWire](https://overthewire.org/wargames/) wargames — a set of Linux and web-security challenges used to practice command-line skills, permissions, scripting, and basic exploitation.

Each writeup explains **what the level asks for, the commands used, and why they work** — not just the final answer. The goal was to actually understand each concept, not just pass the level.

## 🧠 Wargames Covered

### Bandit
Focuses on basic Linux command-line skills: navigating the filesystem, reading files, permissions, and simple scripting.

| Level | Topic |
| --- | --- |
| [0 → 1](./bandit/level-0-1.md) | Connecting via SSH, reading a file |
| [1 → 2](./bandit/level-1-2.md) | Handling filenames starting with `-` |

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

*(more levels added as they're solved)*

## 🛠️ Skills Practiced

- Linux command line (`ls`, `cat`, `grep`, `find`, `ssh`, permissions, etc.)
- Reading and reasoning about shell behavior
- Basic web security concepts (HTTP, cookies, source inspection)
- Problem-solving and documenting a process clearly

## 📁 Structure

```text
overthewire-writeups/
├── bandit/
│   ├── level-0-1.md
│   ├── level-1-2.md
│   └── ...
├── natas/
│   ├── level-1-2.md
│   ├── level-2-3.md
│   ├── level-3-4.md
│   ├── level-4-5.md
│   ├── level-5-6.md
│   └── ...
└── README.md
```

## ✍️ About

Written by [Fatima Basharat](https://github.com/fatima-009) while learning Linux fundamentals and web security alongside full-stack development.

> Note: Passwords for each level are intentionally omitted or partially masked in these writeups — the focus is on the method, not the answer.
