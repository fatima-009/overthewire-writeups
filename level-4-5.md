## OverTheWire Bandit: Level 4 → Level 5

## Objective

The goal of this level is to find the password for the next level.

The password is stored in the only human-readable file in the inhere directory.

## Steps

- First, I checked the files in the current directory:

```bash
ls
```

The output showed a directory named:

inhere

- I moved into the inhere directory:

```bash
cd inhere
```

- Then, I listed the files:

```bash
ls
```

There were several files in the directory.

- To identify which file contains human-readable text, I used the file command:

file ./*

The output showed that most files were data or non-readable files, while one file was identified as ASCII text.

I then used the cat command to read that file:

```bash
cat ./-file07
```

The command displayed the password for the next level.

## Key Concept

- This level demonstrates how to identify the type of files using the file command.

- The following command checks the type of every file in the current directory:

file ./*

- The file command is useful when we need to determine whether a file contains readable text, binary data, or another type of content.
