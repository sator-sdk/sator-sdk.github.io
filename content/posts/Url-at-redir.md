+++
title = "Unveiling URL Redirect Schema Abuse: The Stealthy Exploits of the @ Sign"
date = "2023-12-20"
+++

# Unveiling URL Redirect Schema Abuse: The Stealthy Exploits of the @ Sign

## Introduction

URL redirect schema abuse is a prevalent and evolving cyber threat that exploits the intricacies of web protocols to deceive users and manipulate their online experiences. One increasingly popular technique involves the misuse of the `@` sign within URLs to redirect users to unintended destinations, and also the use of additional techniques that expand the redirection trick. This article aims to shed light on this deceptive practice, examining its mechanics, potential consequences, and strategies for mitigation.
What I'm going to describe here it's also very well described from two major articles by [Mandiant](https://www.mandiant.com/resources/blog/url-obfuscation-schema-abuse) and [RedCanary](https://redcanary.com/blog/google-zip-domains/).

## Understanding URL Redirects

URL redirects are essential for web functionality, allowing websites to seamlessly navigate users from one page to another. Commonly used HTTP status codes like `301` and `302` signify permanent and temporary redirects, respectively. However, cybercriminals exploit these redirects to mislead users, often with malicious intent.

## The @ Sign as a Deceptive Element

The `@` sign, a ubiquitous symbol in email addresses, has found a new role in the hands of cyber attackers. By incorporating the @ sign into a URL, malicious actors can create redirects that appear legitimate at first glance. For instance, a URL like `hxxp://www[.]example[.]com@malicious-site[.]com` might seem harmless, but the presence of the `@` sign can introduce an element of misdirection.

The key concepts of this technique are two:

* The usage of an `@` sign to obscure the destination server
* The usage of unusual hostname formats to obscure the destination IP address

Lets start by understanding the [URL schema](https://www.rfc-editor.org/rfc/rfc1738) and so to know why the `@` is a deceptive element.

## URL schema

The basic structure for all URLs:

```bash
scheme//user:password@host:port/url-path
```

So the format for an HTTP URL follows the structure of:

```bash
hxxp://host:port/path?searchpart
```

The RFC states that "No user name or password is allowed”, the user name is defined as the text prior to the `@` sign. When a browser interprets a URL with the username section populated (anything before the "@” sign), it discards it, and sends the request to the server following the `@` sign, and it could be a FQDN or even the direct IP of the server, with the path to the required resource specified if needed. The destination server or "hostname" used here to achieve the redirection is part of the trick to trigger that result.

In the URL from our example the `www[.]example[.]com` section of the URL is considered a `username`, and this can be tweaked as needed to better trick victims to click a link in spear phishing campaigns. 

It could be replaced with the target email address domain and become much more believable due to the fact that by now most browser has been updated and when the simple redirection is triggered the browser pops-up an alert that state what is going to happen and dispaly the so called "**username**" and "**password**" that are part of the link, so if the link would be crafted in a deceiving way could still be effecting in tricking the user to click and bypass the security measure. We will see later that not all types of redirection are actually responsible of triggering that allert.

### Brake the schema

Important part that will be helpful when inspecting those crafted links and to better understrand why a redirect happen, is the concept that before the `@` sing if some `/` are present the syntax will broke, meaning that if a legit path is present before the @ sign the whole redirect is invalidated. Due to this in many exmaples you will see simple links that only contain a legit site domain or subdoamin but without a specifed path. Although it depends from what we want to achieve there are some workarounds.

### Unicode deception

One of those is a visual trick using `U+2044` (`⁄`) and `U+2215` (`∕`) in a URL to make the user think they are clicking a normal link, but the browser does not interpret those unicode slashes as actual slashes, and so the link syntax is not broken and thus bypassed:

```bash
# this is an example of a unicode link that will redirect to malicious-site[.]com
hxxp://www[.]example[.]com∕path∕you∕want@malicious-site[.]com

# this not
hxxp://www[.]example[.]com/path/you/want@malicious-site[.]com
```
This will trick the browser into reading the unicode fake slash char and output them in [punycode](https://www.punycoder.com/) most of the time.

## Hostname Formats

While the traditional dotted-quad notation (e.g., **192.168.1.1**) is the most familiar representation, IP addresses can also be expressed in decimal, hexadecimal, and octal forms.

### Dotted-Quad

The standard and widely recognized way of representing IP addresses is in dotted-quad notation, where four decimal numbers separated by dots range from **0** to **255**. For example, the IPv4 address `192.168.1.1` is a common representation used in everyday networking scenarios.

### Decimal

In decimal notation, each segment of the IP address is expressed as a decimal number. For instance, the IPv4 address `192.168.1.1` can be written as `3232235777` in decimal form. This notation provides an alternative perspective on the numerical values that make up an IP address.

### Hexadecimal

Hexadecimal notation uses **base-16** numbering, allowing each segment of the IP address to be represented by a combination of numbers and letters (0-9 and A-F). For example, the IPv4 address `192.168.1.1` can be expressed as `0xC0A80101` in hexadecimal. Hexadecimal notation is often favored in programming and networking configurations.

### Octal

Octal notation, based on the **base-8** numbering system, is less commonly used but provides another alternative representation for IP addresses. Using octal, the IPv4 address `192.168.1.1` becomes `030052000401` in octal form.

### Just give me an address

The important part here is that no matter how the IP is represented, the protocol is able to read each of those format and use them. So in example the browser will redirect you to the right location of the server even if you type the IP in octal form of the requested server.
The notation could also be putted in the dotted-quad format like `192.168.1.119` in hex `0xc0.0xa8.0x1.0x77`, or in octal `0o300.0o250.0o1.0o167`

All of this different notation could also be mixed toghether and the borwser will automatically calcualte each of them like `0o300.168.1.0o167` and here is where the interresting part comes in paly because in some situations the browsers will completly bypass the saftey checks.

> Note: The OCTAL representation is the one taht actually trigger the bypass, so a mixed notation address with at least one octal part will do the trick


## .zip Domains

This is a brief digression on why the `.zip` domains are very crucial to these kind of attacks and why you should be very careful.
Achieve a sthealth download via the use of `.zip` domains, trick the user to download a .zip file from a legit site but pointing the crafted link to a malicious `.zip` domain that holds the malware and cause the user to not raise suspicion upon a download of zip file.

The fact that with a particular `.zip` domain and specifically crafted subdomains the user could be tricked to download a not suspicious zipped file, the owner of a domain that mimic the name of a software package an archive file or whatever you may think that could be useful to make the user be interested in that file may be a perfect SpearPhishing vector to trick that user in clik and retrieve those files.

```bash
# this will trigger a download for a malicious file 
# and the user will be less suspicious then a .zip download from nowhere 
# if he is expecting that download of file.zip to come from a legit site
hxxp://www[.]example[.]com∕path∕to∕resource∕file.zip@malicious-site[.]zip/archive/file.zip
# To achieve a almost clean result a solutiuon could be to write a legit path to the file from
# first .zip domain and then short that link with another owned .zip domain to mask the path
```


## Craft those links

So when we actually want to try them out in our red team or phishing engagements to be able to replicate them the obvious way is to craft those link by yourself but ther are some handy tools out there that could help you with that. I developed a Python script that automate all that procedure, and i used it occasionally during assessments, the core of the translation of an IPv4 structure to the other notation formats was inspired from a tool by Vincent Yiu [IPFuscator](https://github.com/vysecurity/IPFuscator) that its a bit outdated by now and still in python2, so i decided to give it a modern look and add some interesing features to ease of use and craft redirection link in seconds.

## [URL@](https://github.com/sator-sdk/URL-at) link to github repo

### DISCLAIMER

> The script is for demonstration purposes only! The misuse of that script for illicit purposes is at your own risk, should be used only in legitimate penetration test assignments with explicit permission.

### Description

Is a comamnd line utility tool that automatically generation of those kind of links in multiple ways, by providing some command line arguments:

* `-i`: Redirect Destination IP to perform mutation on

Or if preferred the IP could be directly retrieved from DNS records of an existing server with `-d`

* `-s`: Domain name URL that act as legit domain

* `-p`: Redirect Destination URL path (optional) is the ending path of the file on final server

* `-sp`: URL arguments after the TLD of the "-s" specified domain to perform encoding on. This field could also be included in the `-s` argument but will only be encoded if passed via `-sp` not `-s` even though the unicode char substituion is performed in all the passed fields except from `-p` that should NOT be modified with fake slash encoding

* `-r`: Generate random URL schema. Obfuscating links inside code that should appear not pointing to a searchable domain. I have observed inside some malware samples only the goal of trick the scanning software as long as they can to avoid detection of hardcoded link that will possibly lead to a CnC infrastructure discovery. The format is actually inspired from source code of AgentTesla sample that store links in that format to avoid detection. Keep in mind that it doesn't have to be considered an obfuscation method the one that is used it only provide a less enthropy 64 char long link padded with zeroes and following some paricualar random rules when generating the letters in order to provide less enthropy to the file it will be embeeded in.

* `-f`: Replace slashes in schema or schema-path with unicode char U+2215

* `-e`: Encode the final URL - the argument of the `-sp` parameter is base64 encoded while the rest is url encoded

<p align="center">
<img src="/posts/urlathelp.png">
</p>
<p align="center">
Help menu
</p>

### Base64 Example

In that example you could craft base64 encoded links that actually even without the `/` set to the unicode `2215` value will redirect the page.

> As mentioned above is important to use when possible the mixed creafted dotted-quad notation in order to perform a flawless redirect.

In the script only the path of the legit domain (the User-Info path) is encoded to mimic some known behaviour of popular url rewrite mechanisms that encode only some data in that part and not in others, actually to base64 encode only specific values of some parameters would be even better for tricking an expert user, and i suggest to do it manually, or maybe i will add a functionality in the script, but for now a obfuscation of the whole path parameter is fine.
Also i hardcoded some values that improve the whole mechainism and are needed in some situations, like the `%20` after the TLD of the legit domain or the `%7F` before the `@`, but keep in mind that you would prefer to swap those values or even add as many as you want for each link you could use in order to not increment fingerprinting.

```bash
# fictional domain and parameters
python urlat.py -i 192.168.1.119 -s totally.legit.domain.tld -sp '/api/v1/signin/ident?klmer=G1927123976%3A893513460&redir=https%3A%2F%2Fanother.domain&Loing&dobui=b253cnZqbmRzd3JvdmJ3cm92amJvd3dka2piZHZvZWlqY2V3b2lubm9rbmN3d29u' -e
```

<p align="center">
<img src="/posts/urlatlinks.png">
</p>
<p align="center">
Example of base64 encoded link
</p>

Its not explained extensivley here but of course one of those link shortned taking some precautions will lead to a clean solution if needed, even if in some cases i notice that a well palnned link creation could be more efficent. In the github repo you could find some command examples and instrucions.

## Remediation and Detection

This two popular RegEx solutions will fail to find URLs using these techniques:

```bash
#
/https?:\/\/[^\/]*@[^\/]*\.zip$/
#
https?:\/\/(www\.)?{1,256}\.[a-zA-Z0-9()]{1,6}\b(*)
```

Network traffic analysis won’t show this technique in use. When a browser receives a request to go to a URL using this syntax, it automatically translates it to a valid destination before issuing the request. Therefore, by analyzing network traffic, you wouldn’t see an obfuscated URL.

Using file-based analysis like YARA or AV/EDR can reveal tools using URL schema obfuscation, as can process execution logs. If a program executes something like Powershell’s Invoke-WebRequest module pointing to an obfuscated URL, the obfuscated URL will be shown in the logs. As for detecting it in files.

[YARA Rules from MANDIANT](https://www.mandiant.com/resources/blog/url-obfuscation-schema-abuse)


## Conclusion

URL redirect schema abuse, particularly the use of the @ sign, represents a stealthy and potentially damaging form of cyber threat. As online security continues to be a major concern, it is imperative for users and organizations alike to stay vigilant, adopt best practices, and employ advanced security measures to protect against these evolving tactics employed by malicious actors. The ability to express IP addresses in decimal, hexadecimal, and octal forms adds a layer of versatility to the understanding and utilization of these essential components in networking. While dotted-quad notation remains the standard for human readability, the alternate representations play crucial roles in programming, networking configurations, and digital communication.