---
layout: post
title: "Web Security"
---

### Clickjacking

A clickjacked page tricks a user into performing undesired actions by clicking on concealed links. 
On a clickjacked page, the attackers load another page over the original page in a transparent layer to trick the user 
into taking actions, the outcomes of which will not be the same as the user expects. 
The unsuspecting users think that they are clicking visible buttons, 
while they are actually performing actions on the invisible page, clicking buttons of the page below the layer. 
The hidden page may be an authentic page; therefore, the attackers can trick users into performing actions 
which the users never intended.

Clickjacking can be prevented by adding into HTTP response headers of an HTML page the 
[frame-ancestors](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors) 
directive of the [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) 
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
When the victim signs in into the malicious website, the attacker steals her credentials.

Using two-factor authentication can prevent from stealing a victim's account, but can't prevent from stealing
victim's credentials.

### Man-in-the-middle Attacks (MITM)

In Man-in-the-middle attacks an attacker intercepts traffic between a web server and the victim's browser 
to capture data e.g. credentials, credit card information or replace the data with another, fraudulent.
Open WiFi networks are a typically means of executing this attack.

Use of VPN can create a secure and encrypted communication channel and protect from MITM.

To protect a website from MITM the site must be accessed using HTTPS.
Only the latest TLS protocol version must be enabled on web server of the site, because old versions could be insecure.
If a site is opened in the browser using HTTPS then all traffic between the browser and web server is encrypted 
and even if an attacker intercepts the traffic he isn't able to decrypt or alter it.

Enforcing browsers to use HTTPS over HTTP could be achieved by 
configuring a HTTP to HTTPS (301) redirect on a web server. This approach isn't secure, 
because the initial HTTP connection is still vulnerable to a MITM attack.

The more secure option is using of the 
[Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)
HTTP response header (often abbreviated as HSTS). The first time a site is accessed using HTTPS and its response 
contains the Strict-Transport-Security header, the browser records this information, 
so that future attempts to load the site using HTTP will automatically use HTTPS instead.

The Strict-Transport-Security header is ignored by the browser when a site is accessed using HTTP.
This is because an attacker may intercept HTTP connections and inject the header or remove it.

```shell
# In this example the browser will remember the HSTS policy for 1 year
$ curl --head https://foo.com

HTTP/1.1 200 OK
Strict-Transport-Security: max-age=31536000; includeSubdomains; preload
```

### Credentials-Guessing Attacks

https://datatracker.ietf.org/doc/html/rfc6749#section-10.10

### Open Redirectors

https://datatracker.ietf.org/doc/html/rfc6749#section-10.15

### Spoofing Attacks

### References / Further Reading

[Types of attacks](https://developer.mozilla.org/en-US/docs/Web/Security/Types_of_attacks) <br>
[Web security](https://developer.mozilla.org/en-US/docs/Web/Security) <br>
[Executing a Man-in-the-Middle Attack in just 15 Minutes](https://www.thesslstore.com/blog/man-in-the-middle-attack-2/) <br>
