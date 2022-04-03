# noted

## Description: 
I made a nice web app that lets you take notes. I'm pretty sure I've followed all the best practices so its definitely secure right?

## Points: 
500

## Solution: 

Website allows to signup, login and create notes. The notes are private, available only to the logged author and does not have any unique URL identifying specific note.

The website also allows to send a report to an admin. This makes us quickly realise this is probably an XSS challenge.

We can see the source code. The reporting bot seems to create a new account with random credentials, add flag as a note and then leave the page to visit reported url.

The XSS can be easily put into the note content, but this is a self XSS as we cannot share the note content with the admin.

It's not easy to find an exploit until we realise there is another vulnerability apart from the XSS. Most of the pages got CSRFs tokens in all the forms, but the login is not protcted against CSRF.

Looking for possible solutions I've encountered this writeup which seems to exploit similar vulnerability: https://medium.com/@Ch3ckM4te/self-xss-to-account-takeover-72c89775cf8f

The idea for the exploit is to force admin to get logged into our account and get our XSS payload executed. The problem is that we then need to get their flag which at that point will not be on the account that they will be logged into.

The additional difficulty (which later I've found was not true) was that the bot is visiting the page without the internet access. This made me think I need to store the flag on my account instead of sending it to some sort of a webhook.

The trick I figured out is to open two windows. One with the admin logged in, and after a short while the other with an auto submitting form that will force admin to get logged on my account and execute XSS.

Clever trick allowed me to get handle to the admin tab from the XSS tab by naming windows I've opened and then using `window.open` again.

The XSS payload was:

```javascript
if(window.opener) {
        const flagWindow = window.open("", "flag");
        const flag = flagWindow.document.querySelector("p").innerHTML;
        setTimeout(() => {
            const csrf = document.forms[0][0].value;
            fetch("/new", {
                credentials: "include",
                method: "POST",
                body: `_csrf=${csrf}&title=flag&content=${flag}`,
                headers: {
                    "Content-Type": "application/x-www-form-urlencoded",
                },
            });
        }, 200)
    }
```

This code gets the flag window, queries the DOM to find the note with a flag and then posts it as a new note on my account.

Visiting the account back allowed me to get the flag.