# OverTheWire Bandit — Level 1-2

## Objective

The goal of Level 1 is to find the password for the next level.

The password is stored in a file named:

```text
-
```

This file is located in the current directory.

## Step 1: Check the Files

After logging into Bandit Level 1, run:

```bash
ls
```

The `ls` command lists the files and directories in the current directory.

You should see:

```text
-
```

This means there is a file named `-`.

## Step 2: Understand the Problem

At first, we might try:

```bash
cat -
```

The `cat` command is normally used to display the contents of a file.

However, `-` has a special meaning in many Linux commands. It can be interpreted as **standard input** instead of a filename.

Therefore, `cat -` may not read the file we want.

## Step 3: Read the File Correctly

We can tell Linux that `-` is a file in the current directory by adding `./` before the filename:

```bash
cat ./-
```

### What does `./` mean?

`./` means:

> **The current directory**

So:

```text
./-
```

means:

> The file named `-` in the current directory.


## Step 4: Get the Password

Run:

```bash
cat ./-
```

The command will display the password stored inside the file.

The password shown by your terminal is the password you need for **Bandit Level 2**.


## Why Does `cat ./-` Work?

Linux commands often treat a filename beginning with `-` as an option.

For example, a command might interpret:

```text
- 
```

as a special argument rather than as a filename.

By writing:

```text
./-
```

we provide a path to the file instead of simply giving the filename.

This removes the ambiguity.


## Important Commands

### List files

```bash
ls
```

Used to see files and directories.

### Read a file

```bash
cat filename
```

Used to display the contents of a file.

### Read a file named `-`

```bash
cat ./-
```

The `./` tells Linux that `-` is the filename located in the current directory.


## What We Learned

In this level, we learned:

1. How to list files using `ls`.
2. How to read files using `cat`.
3. What `./` means in Linux.
4. Why filenames beginning with `-` can cause problems.
5. How to explicitly specify a file using its path.
