---
title: Reflected XSS in a JavaScript URL with Some Characters Blocked
description: A PortSwigger XSS lab writeup about reflected XSS in a JavaScript URL with some characters blocked.
author: Koussay Dhifi
categories: [Vulnerabilities, WebExploitation]
tags: [WebExploitation, XSS, PortSwigger, Labs]
pin: true
math: true
mermaid: true
---

## Introduction

This is the 28th lab in PortSwigger's XSS series. It consists of a reflected XSS in a JS URL with some characters blocked to prevent such an attack. The lab is titled [Reflected XSS in a JavaScript URL with some characters blocked](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-url-some-characters-blocked). Our objective is to trigger an alert with the string `1337` contained somewhere in the alert message.

## Recon

We have the usual website, but this time it is without a search bar, as shown in the following image.

![Blog Page](../assets/xss/lab28/Pasted%20image.png)

Let's start by specifying where the reflected XSS is exactly located.

When we click on a specific post, the URL parameters change to `/post?postId=4`, supposedly because we clicked on the fourth post.

![Post URL Parameter](../assets/xss/lab28/Pasted%20image%20(2).png)

When we check the source code to see whether this `postId=4` is reflected or not, we indeed find it in the following code snippet:

```html
<a href="javascript:fetch('/analytics', {method:'post',body:'/post%3fpostId%3d4'}).finally(_ => window.location = '/')">Back to Blog</a>
```

The role of this is that when we click the Back to Blog button, a POST request is sent to the analytics endpoint for that specific post.

Now all we have to do is prepare an injection payload so we can run the `alert` function.

## Payload and Exploit

The payload here is one of the hardest, if not the hardest, payloads you'll ever find in XSS. At some point in hard XSS problems, you are just leveraging weird JavaScript behavior and syntax, since it is known for weird behaviors. Before we get into the payload, we need to explain some things in JavaScript.

### Throw Errors

In JavaScript, as in any other programming language, there is error handling using the `throw` statement to raise an error when the developer wants. For example, let's say you are creating a calculator, and someone divides by 0. You would write this code snippet:

```js
// Division of x/y

if (Operation == '/') {
  if (y == 0) {
    throw 'You cannot divide by 0';
  }
}
```

This will raise an uncaught error with the string you entered.

The default behavior of the `throw` statement in JS is basically printing the error within the console.

So, if we want to change this default behavior, we use the JavaScript hook `window.onerror` by assigning a function to change that behavior.

For example, if we use the following code snippet:

```js
window.onerror=alert;

throw 1337;
```

Basically, an alert like the following image will be shown, since we changed the default behavior of error handling in JS by assigning the `alert` function to the `onerror` hook.

![Alert from onerror](../assets/xss/lab28/Pasted%20image%20(3).png)

And if we play with some JavaScript, we can write that code snippet in just one line like the following:

```js
throw onerror=alert, 1337;
```

This will give us exactly the same previous behavior.

This is a great and known payload for obfuscating XSS, especially if we do not have access to parentheses, like in this lab where `(` and `]` are being ignored.

### Function Arguments in JavaScript

In JavaScript, let's say that we define the following function:

```js
function myFunc(a,b) {
  return a+b;
}
```

It is a trivial, simple function used for addition, so when we call `myFunc(1,3)`, it returns 4.

Let's try to manipulate the calling and call `myFunc(1,3,4)`. It will return the same thing, which is 4.

If we add more arguments like `myFunc(1,3,4,5,6,7,8,9, ... +inf)`, the rest of the added arguments will all be ignored, and the function will just return 4.

It is valid JavaScript and weird behavior that we can leverage. Basically, any JavaScript function has this behavior, even the `fetch` function, so if we make `fetch(URL,method:,body:,2,3,4,5,6,9)`, it will still be valid JavaScript and run perfectly. You can see where I'm taking this if you were paying attention.

Another thing about weird function behavior is the following code snippet:

```js
let e = 5;
myFunc(2,3,e=8);
console.log(e); // This will print out 8.
```

So, when we assign values to global variables in function parameters, they actually change, and the function runs perfectly.

### toString

In JavaScript, there is a method called `toString`. Either you call it explicitly, or it is implicitly called when you try to mess with JavaScript a little bit.

Let's say you do the following operation:

```js
window+'';
```

Here, the `toString` method is called implicitly. The interesting thing here is that we can override `toString` behavior like what we did with the `onerror` hook. The only difference is that `toString` is not a hook, so in terminology, what we are doing is overriding. `toString` is not a hook, although it is being overridden entirely, not replacing a behavior like a hook.

So if we do:

```js
toString = alert;
window+'';
```

An alert will be called here, as shown in the following picture.

![Alert from toString](../assets/xss/lab28/Pasted%20image%20(4).png)

### Building the Final Payload

Now we have all the pieces of the puzzle, and we can build this complex payload. But first, there are some things that you need to understand about the behavior of the page.

- Parentheses, if you test them, are ignored and not even shown.
- Brackets are the same: ignored.
- Every major dangerous character is URL-encoded, like `<`, `>`, `"`, and `'`.

So, if we are going to insert this payload, we are going to encode it in URL format so the browser can decode it, and then it can be run.

To build the payload, follow these steps:

1. Add `&` because the backend makes sure whether the `postId` you entered exists or not. By adding `&`, we are escaping this validation, since it is the separator between URL parameters.

2. We are in a JavaScript string inside an object, so add `'` to close the string and `}` to close the object. We now have `&'}`.

3. Add a comma so we can add another argument to the `fetch` function. As we mentioned before, it will be executed perfectly, and it is normal JavaScript. What we have now is `&'},`.

4. Declare an arrow function that has the name `x` (or any name you want), and inside that arrow function, put the `throw` statement like the trick we've seen before. Since space is not accepted, we can replace it with `/**/` to obfuscate it like this: `x = x => {throw/**/onerror=alert,1337}`. We put the `throw` inside an arrow function since it is a statement and not a function, so it will not return anything. This arrow function is necessary so it will execute, Inshallah. Now the payload is:

```text
&'},x=x=>{throw/**/onerror=alert,1337}
```

5. We need to call this function, `x`, and by doing this, we are going to override the `toString` method as we mentioned before by adding another argument like this: `&'},x=x=>{throw/**/onerror=alert,1337},toString=x`.

6. Now, to call `x` and alert the message we want, we need to do an event that calls `toString` implicitly, like `window+''`, so we add it as an argument: `&'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+''`.

7. The last step is to not forget the `'}` that are still in the string. We did not replace them; they were just delayed, and we are entering input behind them. So we add another argument to `fetch`, which is `{x:'`, and now the final payload is:

```text
&'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:'
```

8. The last step is to encode this payload in URL format, except for `&`, and it will give us the following result:

```text
%27%7D%2Cx%3Dx%3D%3E%7Bthrow%2F%2A%2A%2Fonerror%3Dalert%2C1337%7D%2CtoString%3Dx%2Cwindow%2B%27%27%2C%7Bx%3A%27
```

Add it to `?postId=4`.

When we click on the `a` tag, the trigger to get back to the page is done, the alert is shown, and the lab is solved.

![Solved Lab](../assets/xss/lab28/Pasted%20image%20(5).png)

## Conclusion

That was by far the hardest XSS payload I've seen so far. I never found the solution on my own. I read the payload, searched for an explanation, and decided to teach myself by writing this blog. For further information and better understanding, there is a video by [z3nsh3ll](https://www.youtube.com/watch?v=bCpBD--GCtQ&t=2s). You can check it out; it goes like I did by explaining the payload piece by piece.
