# OverTheWire Natas — Level 0

## Objective

The goal of **Natas Level 0** is to find the password for **Natas Level 1**.

Natas is a web-based security challenge. Unlike Bandit, where we mainly work with the Linux terminal, Natas teaches us about **web security** and how websites work.


## Step 1: Open the Natas Level 0 Website

Open the Natas Level 0 website in your browser.

You will see a webpage with a message telling you that you need to find the password for the next level.

At first, the password is not visible on the normal webpage.


## Step 2: Inspect the Webpage

Right-click anywhere on the webpage and select:

```text
Inspect
```

You can also open Developer Tools using:

```text
F12
```

or:

```text
Ctrl + Shift + I
```

The exact shortcut may depend on your browser.


## Step 3: Look at the HTML Source

After opening Developer Tools, go to the **Elements** tab.

This tab shows the HTML code used to create the webpage.

Look through the HTML source carefully.

You should find a comment containing the password.

It will look similar to:

```html
<!-- The password for natas1 is ... -->
```

The password is hidden inside the HTML comment.


## Step 4: Understand HTML Comments

An HTML comment is written like this:

```html
<!-- This is a comment -->
```

Comments are not normally displayed on the webpage.

They are written for developers and can contain notes or other information.

In this challenge, the password is placed inside an HTML comment.

This means:

> The password is hidden from the normal webpage, but it can still be seen by inspecting the page source.


## Step 5: Get the Password

Copy the password from the HTML comment.

That password is used to log in to **Natas Level 1**.

The important thing is to understand the technique rather than simply copying the password:

```text
Open webpage
      ↓
Open Developer Tools
      ↓
Inspect HTML
      ↓
Find HTML comment
      ↓
Find the password
      ↓
Use it for Natas Level 1
```


## What We Learned

In this level, we learned:

1. Natas is a web-based security challenge.
2. Websites are built using HTML.
3. Developer Tools can be used to inspect a webpage.
4. HTML comments are not normally visible on the webpage.
5. Sensitive information hidden in HTML can still be discovered by inspecting the page source.

## Key Takeaway

**Never assume that information is secret just because it is not visible on the webpage.**

If sensitive information is included in the HTML sent to the browser, a user can usually inspect the page and find it.


## Tools Used

* Web browser
* Developer Tools
* HTML inspection

## Level 0 → Level 1

After finding the password, use it with the Natas Level 1 credentials to continue to the next challenge.
