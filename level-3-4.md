## OverTheWire Bandit: Level 3 → Level 4

## Objective
The goal of this level is to find the password for the next level. The password is stored in a hidden file in the inhere directory.

## Steps

- First, I checked the files in the current directory:

```bash
ls
```
- The output showed a directory named:

inhere

I moved into the inhere directory:

```bash
cd inhere
```
- Then, I used the ls command to check the files:

```bash
ls
```
Nothing was displayed because the password is stored in a hidden file.

- To display hidden files, I used the -a option with ls:

```bash
ls -la
```

This showed a hidden file named:

.hidden

- I used the cat command to read the file:

```bash
cat .hidden
```

The command displayed the password for the next level.

## Key Concept

- This level demonstrates how to find and access hidden files in Linux.

- The important command used in this level is:

```bash
ls -la
```

- The -a option tells ls to show all files, including hidden files.

- Hidden files in Linux usually start with a ., such as:

.hidden

## Password

- The command outputs the password required to log in to Bandit Level 4.

- The actual password is intentionally not included in this public write-up.
