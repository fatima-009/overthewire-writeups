# Natas Level 23 → 24

**Wargame:** [OverTheWire — Natas](http://overthewire.org/wargames/natas/)
**Category:** Web Exploitation / PHP Type Juggling

## Challenge Overview

The page checks a submitted `passwd` value against the real password using PHP's `strcmp()` function, and only reveals the password if the comparison indicates a match.

### How `strcmp()` Normally Behaves

`strcmp(string1, string2)`:

- Returns a value **< 0** if `string1` is less than `string2`
- Returns a value **> 0** if `string1` is greater than `string2`
- Returns **0** if the two strings are equal

If the application leaked any of this feedback to us (e.g., different responses for "too high" vs. "too low"), we could brute-force the password character by character. Unfortunately, that side channel isn't available here, the page only tells us whether we passed or failed, not which direction we were wrong in.

## Finding the Bypass

Searching for ways to defeat `strcmp()` checks led to a well-known trick, also seen in the [CSAW CTF "Web 200" challenge](https://blog.0daylabs.com/2015/09/21/csaw-web-200-write-up/): if you can make PHP treat the comparison value as `NULL` instead of a string, the check can be bypassed entirely, because in PHP, `NULL` is loosely equal to `0`.

The way to force this is by supplying an **array** instead of a string for the parameter `strcmp()` receives. Since `strcmp()` expects two strings, passing an array as an argument causes PHP to fail internally and return `NULL` (with a warning) rather than a normal integer result. That `NULL` result then gets compared loosely against `0`, and `NULL == 0` evaluates to `true` letting us through without knowing the real password at all.

Crucially, the `natas24` source code doesn't validate that the input is actually a string before passing it to `strcmp()`, which is what makes this bypass possible.

## Exploitation

Normally, submitting a value through the form updates the URL like this, with `passwd` set to a plain string:

```
http://natas24.natas.labs.overthewire.org/?passwd=test
```

Instead, if we submit `passwd` as an **array**, done simply by adding `[]` after the parameter name. PHP will parse it as an array, `strcmp()` will choke on it and return `NULL`, and the `NULL == 0` check will pass:

```
http://natas24.natas.labs.overthewire.org/?passwd[]=test
```

Opening this URL in the browser (logging in with the `natas24` credentials when prompted) reveals the password for **Natas Level 25** without ever needing to know or guess the actual password string.