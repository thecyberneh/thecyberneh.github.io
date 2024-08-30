---
title: Breaking Barriers: Unmasking the Easy Password Validation Bypass in Security Key Registration | How a Dumb Frontend Led to 750 $ Bounty
date: 2024-08-30 13:32:00 +0530
categories: [writeup, bugbounty]
tags: [bugbounty-writeup]     # TAG names should always be lowercase
---

<figure><img src="/assets/postImg/pid2/img01.png" alt=""><figcaption></figcaption></figure>

## Bug: CRLF to XSS and Further Exploitation

Hello Hackers, Hope you are doing grate. I am Neh Patel also known as [THECYBERNEH](https://twitter.com/thecyberneh), I am a Security Researcher from India. Today, i want to discuss one of my easy finding on a public program from Bugcrowd.

In case you missed my previous blog post, I shared the incredible journey of how I successfully secured substantial bounties by uncovering high-impact vulnerabilities in Microsoft’s systems. If you haven’t had a chance to read it yet, you can catch up on the details right here :

[$6000 with Microsoft Hall of Fame | Microsoft Firewall Bypass | CRLF to XSS | Microsoft Bug Bounty](https://thecyberneh.github.io/posts/MicrosoftBugbounty/)

## Finding the Vulnerability
The program has only 1 single web page in scope and there were very few functionalities. I started by checking all the features for related vulnerabilities.
My main focus area was security section ( like all other security researchers ) because there were few functionalities like enabling 2FA, Password change, change the email address and add security key ( Physical Security Key ).

I almost checked all features of that section but no luck because a lot of bugs were already reported on that single web page and mostly all security researchers target that only.

After some testing, i thought about testing the feature which was allowing user to add “Physical Security Key” as 2FA.

<figure><img src="/assets/postImg/pid2/img02.png" alt=""><figcaption></figcaption></figure>

## How Hardware Security Key works ?

A hardware security key is a physical device, like a USB stick, that helps keep your online accounts safe. Here’s how it works:

1. When you want to log in to a secure website or service, you plug in the key or tap it on your device.

2. The key has a special code on it that only it knows. This code is like a secret handshake.

3. The website asks the key to prove it’s the real deal by sending a random challenge.

4. The key uses its secret code to solve the challenge and sends the solution back.

5. The website checks the solution using the known code of the key. If it matches, you get access.

6. This makes it very hard for hackers to break in, even if they know your password.

7. Hardware keys are super secure and protect you from phishing scams.

8. You can have a backup plan in case you lose your key.

## Exploiting The Feature

Now in this website, when you try to add security key, it asks for your current password before adding the security key to prevent unauthorized user to add security key.

If you enter wrong password, server returns response as ```401 Unauthorized``` with response body ```{"success":false}```

<figure><img src="/assets/postImg/pid2/img03.png" alt=""><figcaption></figcaption></figure>

Now after checking this, first thing in my mind was response manipulation, so i tried changing response from ```401 Unauthorized``` and ```{"success":false}``` to ```200 OK``` and ```{"success":true}``` but it did not work in first try, but i was confused because there were no specific session cookie which indicates that the response from the server accepted the password or not.

After spending some time with that function and checking javascript from the page source, there were few observations in my mind :

1. Server was continuously validating the password entered by user and if password is wrong, server was returning status code as ```401 Unauthorized``` and response body as ```{"success":false}```

2. Excluding these 2 response from server, there were no security security mechanism from which, frontend can determine if the response is really from server or someone changed it in-between.

3. In a secure password validation, server should return a specific cookie ( unique every time when user enters correct password ) so that the frontend can understand that the response is from server ( because attacker can change status code and response body but attacker can not put that unique cookie )

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> The final and the most important observation, if the frontend not receives the response from the server within 7 second, it shows "Incorrect Password, please try again".
{: .prompt-tip }
<!-- markdownlint-restore -->

So now i was sure that i need to change response and return it to the frontend within 7 second

I copied the response status code as `200 OK` and response body as `{"success":true}` , changed within 7 second and boom!!! Now i was able to register or remove 2FA keys without knowing real password

I reported the bug on bugcrowd and they triaged it

- Priority: P3

- Bounty : $750

( Even i was surprised 😂 )

## A Message for Readers:-

If you like this write-up, you can connect with me on Twitter where i used to post exploits of new CVES , private nuclei templates and other things.

Also ,i used to post about new CVEs on Instagram and Linkedin.

Let me know if I missed anything

## Let’s Connect…

Twitter :- [https://twitter.com/thecyberneh](https://twitter.com/thecyberneh) \[ [thecyberneh](https://twitter.com/thecyberneh) ]

Linkedin :- [https://www.linkedin.com/in/thecyberneh](https://www.linkedin.com/in/thecyberneh) \[ [thecyberneh](https://www.linkedin.com/in/thecyberneh) ]

Instagram :- [https://www.instagram.com/thecyberneh/](https://www.instagram.com/thecyberneh/) \[ [thecyberneh](https://www.instagram.com/thecyberneh/) ]

Thanks for reading…


