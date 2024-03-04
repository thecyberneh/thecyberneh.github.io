---
title: $6000 with Microsoft Hall of Fame | Microsoft Firewall Bypass | CRLF to XSS | Microsoft Bug Bounty
date: 2024-03-04 22:53:00 +0530
categories: [writeup, bugbounty]
tags: [bugbounty-writeup]     # TAG names should always be lowercase
---


Hello Hackers, Hope you are doing great. I am Neh Patel, also known as [THECYBERNEH](https://twitter.com/thecyberneh), a Security Researcher from India. Today, I am excited to share my experience of receiving my first 4-digit bounty from our favorite **#Microsoft** and achieving the dream of every bug hunter – the **#Microsoft Hall of Fame** for a P2 vulnerability;

* Priority: P2
* Severity: Important;

## Bug: CRLF to XSS and Further Exploitation

So I was trying for the last 2 or 3 weeks on Microsoft for valid bugs but only got some P4 and P5.My main methodology is to manually check all subdomains, so I tested all the subdomains and mostly all the functionalities on each subdomain.

After testing 50 to 70 subdomains, I got a kind of different subdomain ( as per my knowledge, this subdomain includes features for premium customers only ).I thought "let's focus on this subdomain because it has features that are mostly paid so those are less explored by other security teams.I mainly checked all features that are not paid ( do not require any premium for the test ) but did not get anything.

After some refreshment, I thought "let's start with non-functional testing or server-side testing". I started with the most popular "Host Header Injection" and some other non-functional testing.

After spending some time with the web applications, I was like "Let's test for the most ignored vulnerability **CRLF INJECTION**

### What is CRLF?
CRLF refers to the special character elements "CR: Carriage Return" and "LF: Line Feed." These elements are embedded in HTTP headers to signify an End of Line (EOL) marker.

So whenever the server gets CRLF, it assumes that it is the end of the line which is a feature, not a bug.


### What is CRLF Injection?
CRLF injection occurs when the web server directly renders those special characters without encoding them and passes them to response headers like Location, Set-Cookie, etc.

So basically, when the web application is vulnerable to CRLF injection, we can send those special characters as our payload and the server will parse that as end-of-line,

After that CRLF, if we send any texts, the server will set them as **Response headers**

set arbitrary headers like we can set a cookie in the victim’s browser, or we can set the “Location:” header to force the victim to redirect to malicious sites and many more.

### Back to the story…

With the intention of CRLF, I fired my first payload:

```bash
/%0D%0A%20Set-Cookie:whoami=thecyberneh
https://subDomain.microsoft.com/%0D%0A%20Set-Cookie:whoami=thecyberneh
```

However, I did not get any CRLF injection. The response was **400 Bad Request** and the HTML was **HTTP Error 400 The requested URL is invalid**

Here, I observed something different, mostly when CRLF is not possible or when the server is well protected, the web server should return **404 Not Found** because if the server is well protected and we enter CRLF payload, the server parses that payload as a part of the URL and in that case, the server actually parses **%0D%0A%20Set-Cookie:whoami=thecyberneh** as a path not like payload.

and if there is no path or directory like **%0D%0A%20Set-Cookie:whoami=thecyberneh**, the server should return **404 Not Found** instead of **400 Bad Request**.

so I thought that server is not well protected or the firewall is weak.

For confirmation, I tried random URLs like this

```
https://subDomain.microsoft.com/abcxyzabcxyz
```

and as per the previously expected behavior of the server, the server returned **“404 Not Found”** with the HTML response **We are sorry, the page you requested cannot be found. The URL may be misspelled or the page you're looking for is no longer available**

So as per the whole situation

> The server was returning **404 Not found** if the path is not there
>
> The Server was returning **400 Bad Request** when I tried with the payload

After this, I was sure that the server is not protected or the firewall is weak. So I have to bypass the firewall or fire the correct payload

I tried with some different payloads like:

```
%0D%0A%20Set-Cookie:whoami=thecyberneh
%20%0D%0ASet-Cookie:whoami=thecyberneh
%0A%20Set-Cookie:whoami=thecyberneh
%2F%2E%2E%0D%0ASet-Cookie:whoami=thecyberneh
```

After trying that payload, I was only getting **400 Bad Request** so I thought that I need something different because most firewall blocks normal and basic payloads.

I did little research and got some unique encoding called **GBK encoding**

## Understanding of Encoding and #FirewallBypass

```
Payload responsible for CRLF injection is :- 嘍嘊
```

so after getting this payload, I crafted a new payload and new URL for CRLF Injection and this time with **“Firewall Bypass”**

I also used **URL encoding** to avoid further errors like this

```
BeforeURL Encoding
Main Payload:- 嘍嘊
After URL Encoding
嘍 :- %E5%98%8D
嘊 :- %E5%98%8A
```

so after URL encoding, I crafted a new payload like this

```
https://subDomain.microsoft.com/%E5%98%8D%E5%98%8ASet-Cookie:crlfinjection=thecyberneh
```

I tried with this URL and Boom!!!

I got a response and the header was: **Set-Cookie: crlfinjection=thecyberneh**

<figure><img src="/assets/postImg/pid1/img1.webp" alt=""><figcaption></figcaption></figure>

But I know that I can do many things with this CRLF injection, so I did not report that and tried to chain with some other bugs.

### CRLF TO XSS with Firewall Bypass

As you know that there is a blank line at the end of the request and response header like this:- ( line 13 in request and line 20 in response )

<figure><img src="/assets/postImg/pid1/img2.webp" alt=""><figcaption></figcaption></figure>

So at that time, the server is sending a response like this:- ( by default, as functionality )

**Structure of response I got:-**

<figure><img src="/assets/postImg/pid1/img3.png" alt=""><figcaption></figcaption></figure>

**Real Image:-**

<figure><img src="/assets/postImg/pid1/img4.jpg" alt=""><figcaption></figcaption></figure>

So now if I want XSS, I have to insert a Javascript payload in the body section of the Response because Javascript can be render in body and for that, **I have to force the server to send a response like this**

<figure><img src="/assets/postImg/pid1/img5.png" alt=""><figcaption></figcaption></figure>

Here,

* **HEADERS** means we have to craft the payload which forces the server to send a blank line after our payload ends so that the headers after that payload will parse as garbage or just ignore them
* **PAYLOAD\_XSS** means our payload for XSS

I was in one of the difficult parts of this because I have to manually force the server to send a malicious response like this,

I inserted a single CRLF payload like “嘍嘊” ( with URL encoding [%E5%98%8D%E5%98%8A](https://subdomain.microsoft.com/%E5%98%8D%E5%98%8ASet-Cookie:crlfinjection=thecyberneh) ), but did not get the single blank line as expected,

So I inserted that payload 2 times and after that, I successfully forced the server to send a single blank line in response.

I immediately crafted a payload ( with firewall bypass ), to get XSS and HTML injection with it.

**Structure of response I want:-**

<figure><img src="/assets/postImg/pid1/img6.png" alt=""><figcaption></figcaption></figure>

**Real Image:-**

<figure><img src="/assets/postImg/pid1/img7.png" alt=""><figcaption></figcaption></figure>

I am sharing that payload with you.

### Understanding of new payload with Firewall bypass

I created the following payload to get XSS\


<figure><img src="/assets/postImg/pid1/img8.png" alt=""><figcaption></figcaption></figure>

Now I encoded the normal payload as follows to bypass the firewall

```
"<" as 嘼
">" as 嘾
```

<figure><img src="/assets/postImg/pid1/img9.png" alt=""><figcaption></figcaption></figure>

So my payload for CRLF to XSS was:-

Also, I performed URL encoding for avoiding errors and my final payload was:-

```html
ENCODING
"<" --> 嘼 --> %E5%98%BC
">" --> 嘾 --> %E5%98%BE

https://subDomain.microsoft.com/%E5%98%8D%E5%98%8ASet-Cookie:whoami=thecyberneh%E5%98%8D%E5%98%8A%E5%98%8D%E5%98%8A%E5%98%8D%E5%98%8A%E5%98%BCscript%E5%98%BEalert(1);%E5%98%BC/script%E5%98%BE
```

After inserting that payload, I got the most beautiful results

I am sharing some beautiful screenshots

<figure><img src="/assets/postImg/pid1/img10.png" alt=""><figcaption></figcaption></figure>

<figure><img src="/assets/postImg/pid1/img11.png" alt=""><figcaption></figcaption></figure>

I submitted this bug and after that, I create an automation tool with shell script which check CRLF on the list of subdomains,

After running that tool, I got another CRLF in another domain from Microsoft.

> Note:- I did not mention further exploitation of CRLF because it is already a long writeup and now it is very easy to chain with some more impactful bug

## Impact:- 

* Arbitrary Header injection
* Arbitrary Cookie Injection
* Javascript injection ( XSS )
* HTML Injection
* HTTP request splitting
* Bypassing Cookie-Based CSRF Protection
* Application DOS using overly long Cookies
* the attacker can set new headers

I reported both of them and got a response regarding bounty in 3 weeks.

---
-  **Priority: P2**
-  **Severity: Important**
--- 

And with that, I got a total of $6000 bounty and theMicrosoft Hall of Fame.

A Message for Readers: If you enjoyed this write-up, connect with me on [Twitter](https://twitter.com/thecyberneh), [LinkedIn](https://www.linkedin.com/in/thecyberneh), and [Instagram](https://www.instagram.com/thecyberneh/).

Let me know if I missed anything. Thanks for reading!

---
### Let’s Connect…

Twitter :- [https://twitter.com/thecyberneh](https://twitter.com/thecyberneh) \[ [thecyberneh](https://twitter.com/thecyberneh) ]

Linkedin :- [https://www.linkedin.com/in/thecyberneh](https://www.linkedin.com/in/thecyberneh) \[ [thecyberneh](https://www.linkedin.com/in/thecyberneh) ]

Instagram :- [https://www.instagram.com/thecyberneh/](https://www.instagram.com/thecyberneh/) \[ [thecyberneh](https://www.instagram.com/thecyberneh/) ]

Thanks for reading…