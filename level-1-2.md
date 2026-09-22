## Natas Level 1 → Level 2

*Goal: Find the password for natas3. The page literally just says "There is nothing on this page", which is your clue to look deeper than the rendered HTML.*

## Step 1: Access the level

http://natas2.natas.labs.overthewire.org

Username: natas2
Password: (from level 1)

## Step 2: View the page source

Even though the page looks empty, check the source (view-source: or Ctrl+U). You'll notice an <img> tag pointing to:

html
<img src="files/pixel.png">

## Step 3: Browse the files directory

Since there's a files/ folder referenced, browse to it directly:

*http://natas2.natas.labs.overthewire.org/files/*

This directory listing is open, and you'll see more than just pixel.png — there's also a file called:

*users.txt*

## Step 4: Read users.txt

Open it:

http://natas2.natas.labs.overthewire.org/files/users.txt

Inside, you'll find a list of usernames and passwords, including one like:

natas3:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

## Step 5: Log in to Level 3

Username: natas3
Password: <password found above>
URL: http://natas3.natas.labs.overthewire.org

*Lesson learned*

Referencing static assets (like images) from a subdirectory can accidentally expose that entire directory if directory listing is enabled and permissions aren't locked down. Always check linked paths and directory indexes — sensitive files are often left sitting in plainly guessable locations.