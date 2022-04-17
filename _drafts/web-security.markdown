---
layout: post
title: "Web Security"
---

### (Clickjacking)[https://en.wikipedia.org/wiki/Clickjacking]

A clickjacked page tricks a user into performing undesired actions by clicking on concealed links. 
On a clickjacked page, the attackers load another page over the original page in a transparent layer to trick the user 
into taking actions, the outcomes of which will not be the same as the user expects. 
The unsuspecting users think that they are clicking visible buttons, 
while they are actually performing actions on the invisible page, clicking buttons of the page below the layer. 
The hidden page may be an authentic page; therefore, the attackers can trick users into performing actions 
which the users never intended.

Clickjacking can be prevented by adding into response headers of an HTML page the 
(frame-ancestors)[https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors] 
directive of the (Content-Security-Policy)[https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy] 
header. The directive specifies valid parents that may embed the page using iframe, object, etc.

Examples

```text
Content-Security-Policy: frame-ancestors 'none';

Content-Security-Policy: frame-ancestors 'self' https://www.example.org;
```

### Cross-site Scripting (XSS)

Cross-site scripting (XSS) is a type of security vulnerability when an attacker injects malicious JavaScript code into
a web page viewed by other users. 

To prevent XSS attacks all data inserted into an HTML page should be encoded.
HTML content inserted into a page from an untrusted source should be sanitized.
The (Content-Security-Policy)[https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy] 
response header could be used to control resources the user agent is allowed to load for a given page.
Adding this header can prevent sending data and loading scripts from malicious servers.  

### Cross-Site Request Forgery (CSRF or XSRF)

https://datatracker.ietf.org/doc/html/rfc6749#section-10.12

### Phishing Attacks

Phishing is a type of social engineering attack where an attacker lures a victim into a malicious website under 
the attacker control. That website looks similar to some real website e.g. a victim's bank website.
The attacker can lure victims into the malicious website in different ways, 
e.g. the attacker can email the victim making him think the email is a legitimate email from the victim's bank.
When the victim signs in into the malicious website, the attacker steals his credentials.

Using two-factor authentication can prevent from stealing a victim's account, but can't prevent from stealing
victim's credentials.

### man-in-the-middle attacks

[Strict-Transport-Security header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security) 
https://datatracker.ietf.org/doc/html/rfc6749#section-10.9

### Credentials-Guessing Attacks

https://datatracker.ietf.org/doc/html/rfc6749#section-10.10

### Open Redirectors

https://datatracker.ietf.org/doc/html/rfc6749#section-10.15

### References / Further Reading

(Types of attacks)[https://developer.mozilla.org/en-US/docs/Web/Security/Types_of_attacks] <br>
(Web security)[https://developer.mozilla.org/en-US/docs/Web/Security] <br>
