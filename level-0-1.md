## OverTheWire: Natas — Level 0 to Level 1 Writeup

Natas is a beginner-friendly web security wargame from OverTheWire. Each level is a webpage where you need to find a password hidden somewhere (in the HTML source, cookies, headers, etc.) to move on to the next level. Here's how to solve the first two levels.

## Level 1

**Goal: Same idea as Level 0, but with a small twist — right-click is disabled.**

- Go to: http://natas1.natas.labs.overthewire.org
- Log in with:  
    Username: natas1
    Password: (the one you found in Level 0)

- This time, if you try to right-click, nothing happens — the page has JavaScript blocking the context menu. But this is only a cosmetic restriction; it doesn't actually stop you from viewing the source. You can bypass it in a few ways:

*Keyboard shortcut:* 
Press Ctrl+U (or Cmd+Option+U on Mac) directly — this works even if right-click is disabled.
*Browser menu:* 
Go to your browser's menu (⋮ or ☰) → "More Tools" → "View Source" (varies by browser).
*Developer Tools:* 
Press 'F12' or 'Ctrl+Shift+I' to open DevTools and look at the "Elements" tab.
*URL trick:* 
Type "view-source:http://natas1.natas.labs.overthewire.org" directly in the address bar.

- Just like before, look through the HTML for an embedded comment containing the password for natas2.

**Lesson learned:**
Disabling right-click is a weak, purely client-side "protection." It doesn't actually secure anything since the browser still downloads and renders the full HTML/JS — you can always access it through other means. Never rely on hiding the "view source" option as a real security measure.

*Quick Tips Going Forward*

- Always check: page source, cookies, HTTP response headers, and any linked JS/CSS files — passwords and hints are often hidden in these.
- Use browser DevTools (F12) heavily; it's your main tool throughout Natas.
- Keep a text file noting down each level's password as you go, since you'll need the previous level's password to log into the next one.

Good luck with Level 2 onward — the levels start introducing PHP source code review, cookies, and basic injection concepts soon after this!