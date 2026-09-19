## OverTheWire Bandit — Level 0

## Objective:

The goal of Level 0 is to connect to the OverTheWire Bandit server using SSH (Secure Shell).

SSH allows us to connect to and work with a remote computer through the terminal.

## Step 1: Connect to the Bandit Server

Open your terminal and run:

"ssh bandit0@bandit.labs.overthewire.org -p 2220"

Let's understand this command:

- ssh → The command used to connect to a remote computer.
- bandit0 → The username we are using.
- bandit.labs.overthewire.org → The hostname of the Bandit server.
- -p 2220 → Tells SSH to use port 2220 instead of the default port 22.

## Step 2: Accept the Host Key

The first time you connect, SSH will show the server's fingerprint and ask:

**Are you sure you want to continue connecting (yes/no/[fingerprint])?**

Type yes and press Enter. This happens only on the first connection.

## Step 3: Enter the Password

When SSH asks for the password, enter:

bandit0

Note: When you type a password in the Linux terminal, nothing may appear on the screen. This is normal. Type the password and press Enter.

## Step 4: Successful Login

If the login is successful, you will be inside the Bandit Level 0 environment.

You may see a prompt similar to:

**bandit0@bandit:~$**

This means you are successfully connected to the Bandit server.

To disconnect at any time, run:

```bash
exit
```

## What We Learned

In this level, we learned:

- What SSH is.
- How to connect to a remote Linux machine.
- How to specify a username.
- How to connect using a specific port.
- How passwords work in the terminal.

## Command Summary

ssh bandit0@bandit.labs.overthewire.org -p 2220

- Username: bandit0
- Host: 	bandit.labs.overthewire.org
- Port:     2220    
- Password: bandit0