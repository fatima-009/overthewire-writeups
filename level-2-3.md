## OverTheWire Bandit: Level 2 → Level 3
## Objective

The goal of this level is to log in to the next level using the password stored in a file named --spaces in this filename--.

## Steps

First, I checked the files in the current directory:

```bash
ls
```

The output showed a file named:

```bash
--spaces in this filename--
```

Since the filename starts with --, simply using cat --spaces in this filename-- would cause the command to interpret the filename as an option.

To safely access a filename that starts with -, I used ./ before the filename:

```bash
cat ./--spaces\ in\ this\ filename--
```

Alternatively, the filename can be enclosed in quotes:

```bash
cat "./--spaces in this filename--"
```

The command displayed the password for the next level.

## Key Concept

This level demonstrates how to work with filenames that contain spaces and begin with hyphens.

Two useful techniques are:

Escape spaces using \

Use ./ to make it clear that the argument is a filename rather than a command-line option

*For example:*

cat ./--spaces\ in\ this\ filename--

## Password

The command outputs the password required to log in to Bandit Level 3.

Note: I have intentionally not included the actual password here so that the write-up remains suitable for a public GitHub repository.

*Takeaway*

This level taught me that special characters in filenames can affect how Linux commands interpret their arguments. Using ./ or properly quoting/escaping the filename allows the file to be accessed safely.

