---
title: dont-use-client-side Writeup PicoCTF
description: A picoCTF web challenge writeup about recovering a client-side checked password from JavaScript source code.
author: Koussay Dhifi
categories: [CTF, WebExploitation]
tags: [WebExploitation, PicoCTF, JavaScript, ClientSide]
pin: true
math: true
mermaid: true
---

## Introduction

This is another easy web CTF challenge from picoCTF titled [dont-use-client-side](https://learn.cylabacademy.org/library/66?category=1&page=1&difficulty=1&solved=true&event=1). It has the description: **Can you break into this super secure portal?** From the description, maybe it is an authentication bypass or something, so let's try to solve it.

## Recon

When we open the challenge, we get to see an eye-soring design. Aside from that, there is an input and a label mentioning `This is the secure login portal Enter valid credentials to proceed`, and then a verify button, as shown in the following image.

![Secure Login Portal](../assets/dont_use_client_side/Pasted%20image.png)

When we insert something like `123`, we get an alert pop-up saying that the password is incorrect, as shown in the following image.

![Incorrect Password Alert](../assets/dont_use_client_side/Pasted%20image%20(2).png)

Since it is an alert, maybe the verification happens in JavaScript or on the client side (see what I did there?), so let's check the source code by viewing the page source.

We first stumble upon something interesting, which is the verify function for the password.

```html
<script type="text/javascript">
  function verify() {
    checkpass = document.getElementById("pass").value;
    split = 4;
    if (checkpass.substring(0, split) == 'pico') {
      if (checkpass.substring(split*6, split*7) == 'eb02') {
        if (checkpass.substring(split, split*2) == 'CTF{') {
         if (checkpass.substring(split*4, split*5) == 'ts_p') {
          if (checkpass.substring(split*3, split*4) == 'lien') {
            if (checkpass.substring(split*5, split*6) == 'lz_2') {
              if (checkpass.substring(split*2, split*3) == 'no_c') {
                if (checkpass.substring(split*7, split*8) == 'b45}') {
                  alert("Password Verified")
                  }
                }
              }

            }
          }
        }
      }
    }
    else {
      alert("Incorrect password");
    }

  }
</script>
```

Basically, it checks the password in a weird way. Basically, the password is the flag, but the way of checking it makes it hard for attackers to know which part is which part of the password. Hard, not impossible.

So, to reorder the flag in the right order, we are going to solve this using a simple algorithm.

First, from the looks of it, the length of the flag is `4*8=32` because the ending of the string is `}` and we have `if (checkpass.substring(split*7, split*8) == 'b45}')`. So let's start by declaring a list with 32 indexes in Python.

```py
l = [aa for aa in range(32)]
```

Now, we are basically going to replace each substring with its corresponding part.

```py
split = 4
l[0: split] = 'pico'
l[split*6: split*7] = 'eb02'
l[split: split*2] = 'CTF{'
l[split*4: split*5] = 'ts_p'
l[split*3: split*4] = 'lien'
l[split*5: split*6] = 'lz_2'
l[split*2: split*3] = 'no_c'
l[split*7: split*8] = 'b45}'
```

Then, we basically join all of that in a single string.

```py
print(''.join(l))
```

After we print that, we get the full flag, and the whole script is the following:

```py
l = [aa for aa in range(32)]
split = 4

split = 4

l[0: split] = 'pico'
l[split*6: split*7] = 'eb02'
l[split: split*2] = 'CTF{'
l[split*4: split*5] = 'ts_p'
l[split*3: split*4] = 'lien'
l[split*5: split*6] = 'lz_2'
l[split*2: split*3] = 'no_c'
l[split*7: split*8] = 'b45}'

print(''.join(l))
```

We basically replaced each index with its corresponding one from the verify function, and we got the full flag.

## Conclusion

It was a nice challenge, more of an algorithmic solution rather than full exploitation, but it was nice to solve.

**NOTE: There is actually another way to solve this by sorting the right side of the substring. The first one is the first, and so on.**
