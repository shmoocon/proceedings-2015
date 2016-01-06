# Knock Knock: A Survey of iOS Authentication Methods

## Abstract

Of the millions of iOS applications available in the iTunes App Store, the vast majority are, essentially, "stand alone" applications, not using individual accounts on a network server. Those applications which do utilize network-based servers, however, process users' most sensitive information, including email, cloud-based storage, healthcare and banking applications. 

Understanding exactly how these applications handle account credentials is key to determining whether such applications are (generally) safe for end-users, or if people should shun highly-sensitive uses of iStuff in favor of web-based clients on the desktop.

To that end, I performed an informal survey of iOS applications, reviewing how they transmitted and stored account credentials. I identified common, and uncommon, methods in use by the surveyed applications, noting also where apps appeared to violate current industry best practices. The results of this survey were presented in detail at ShmooCon in January 2015, and this paper represents a short summary of my findings. 


## Introduction

Many iOS applications rely on Internet-based servers for storage and processing of user data. This data is almost always accessed using a userid and password. To get a rough idea for how well iOS apps protect these credentials, I undertook a short survey of iOS apps, in which I assessed the security of credential storage and transmission. 

### Methodology

To acquire a representative sampling of iOS applications, I simply looked at my own iPhone: Of over 200 applications, fewer than half featured network accounts. I dropped applications which utilized only local servers (for example, SSH clients) or third-party authentication (such as a game posting high scores to Twitter), and any app for which I didn't actually have an account. This left approximately 40 applications for review.

The bulk of the testing was performed using a jailbroken iPhone and Burp Suite Pro, a man-in-the-middle proxy that permitted interception and inspection of  network traffic. Testing required multiple passes through the list of apps, with a slightly different focus for each pass.
 
Exhaustive testing was not performed during the survey, as the time requirements would have been prohibitive. If an app was found to use security strong enough that a major time investment was required, that application was dropped from the review. A handful of applications were thus abandoned prior to testing, and only 2 apps (out of the final list of 40) had to be dropped partway through testing.

### Threats and Security Goals

This survey focused on two primary threat vectors: Interception of account information via a network-based attack, and extraction of those credentials directly from a stolen device. Extraction from a stolen device is reasonably straightforward: off-the-shelf tools can be used to browse an unlocked iOS filesystem over USB. An example scenario involves an unlocked phone, left in a coffee shop. The attacker notices the phone, quickly connects it to their computer, extracts files for the most sensitive apps on the phone, then "helpfully" brings the phone to store staff.

Interception of credentials in transit can primarily happen in two ways: an attacker (using a fake hotspot, or other network attacks) causes all device traffic to route through a proxy under their control. This generally requires some kind of TLS bug or attack, to fool the applications into believing they're communicating directly with the server, instead of the attacker's system. In the case of a lost and unlocked phone, however, the attacker can install a certificate, which causes the device to trust their proxy. In both cases, the attacker monitors application traffic to obtain usernames, session tokens, or passwords.

## Findings

Most applications seemed reasonably secure: "Not bad, but could be better." The common issues included insecure storage of credentials, passing of credentials in URL parameters, and reuse of actual passwords for session authentication.

### TLS Certificate Verification

For the first pass, I did not install the Burp Suite CA certificate, causing all traffic to appear untrusted. Of the 40 apps under review, one worked over TLS in spite of the untrusted certificate, and one didn't even use TLS at all. The remaining 38 applications provided a warning to the user, though in all but one case the warning messages provided were terribly inadequate and frequently misleading. Some examples:

* "Network connectivity may be poor"
* "Oops! Thanks for your patience while we get it fixed. Looks like something was left unplugged."
* "Oops! We’re currently experiencing a number of technical difficulties within our app. We are actively working to resolve these issues and return the app to full functionality as soon as possible. Thank you for your patience."

Only one application displayed a clear error message, explaining why it refused to connect: *"Cannot establish a secure sync connection. [...] To protect your password, data, and privacy, Overcast refuses to connect with reduced security."*

I then installed the Burp CA on my device, and started a second pass. Four applications continued to refuse connections, because the TLS connection didn't present the expected certificate. One of these four pinned apps was the same application with the helpful TLS error message, which, amusingly, was "just" a podcast player.

I defeated the certificate pinning for two applications, but, in the limited time available, could not do so for the other two apps, so the total count was reduced one final time to 38.


### Login and Session Authentication

Most applications sent the user's userid and password in plaintext (wrapped inside the encrypted TLS session), sometimes in HTTP headers but most frequently as POST data. A few apps obfuscated the credentials somehow (by Base-64 encoding it, or gzip-compressing the request data, for example), while another two appeared to use additional encryption. Two applications sent user credentials as URL parameters, risking logging of the userid and password by any system that sees the URL (for example, in a corporate HTTP proxy).

Generally, the server then returned some form of session token to the app. Some were "dynamic" tokens (such as OAuth 1.0, which includes a signed nonce unique to each request), but most were static, repeated verbatim each time. Two apps didn't use tokens, but sent the password with every single request. Session credentials were also passed via a variety of methods: HTTP headers, cookies, the request body, and URL parameters.

### Storage

To assess how securely the applications stored user credentials, I first dumped and inspected the device keychain. Fewer than half the apps tested (17 of 38) stored session tokens in the keychain, while only 4 stored passwords there.

I then extracted the local "sandbox" for each application, reviewing data stored on the device filesystem. Many more credentials were found here, including userids, tokens, and passwords.

### Final Two Passes

I made two final passes -- once, several days after initial testing, to observe "renewed" authentication after a period of disuse, and the other, with the CA certificate removed, to ensure that the apps continued to react accordingly to the (now) untrusted connection. No significant differences from the previously observed behaviors were noted.

## Statistics and Findings Summary
The following tables provide detailed statistics for the various features and techniques surveyed.

### Network Security

| TLS certificate handling               | #  |
|----------------------------------------|:--:|
| Refused to connect via bad MITM cert   | 34 |
| Accepted bad MITM cert without warning | 1  |
| Connected using HTTP (no TLS at all)   | 1  |
| Refused to use trusted cert (pinned)   | 4  |

### Login Authentication

| Initial login credentials sent via:     | #  |
|-----------------------------------------|:--:|
| Plaintext POST parameters               | 25 |
| Obfuscated (but decipherable) POST data | 1  |
| Possibly encrypted POST data            | 2  | 
| URL parameters                          | 2  | 
| HTTP headers                            | 7  |

### Session Authentication

| Continuing session authentication via: | #  | Credentials sent via:    | #  |
|----------------------------------------|:--:|--------------------------|:--:|
| Resending userid and password          | 2  | URL parameters           | 9  |
| Dynamic tokens (OAuth 1.0, PKI, etc.)  | 7  | POST body                | 5  |
| Fixed tokens                           | 28 | HTTP headers             | 24 |


### Local Credential Storage

| Location             | Userid | Password | Token |
|----------------------|:------:|:--------:|:-----:|
| Keychain             | 11     | 5        | 14    |
| Preferences file     | 14     | 1        | 5     |
| Files in /Documents  | 11     | 0        | 4     |
| Other /Library files | 3      | 0        | 4     |
| Cookie file          | 3      | 1        | 5     |
| HTTP cache           | 9      | 2        | 9     |
 
It's worth noting that twice as many tokens were insecurely stored as were properly saved in the keychain (27 vs 14), and that the number of passwords stored in the keychain and on the filesystem were nearly equal (5 and 4).

## Conclusion

Generally, the applications were not bad, but could definitely have done a better job of protecting sensitive user credentials. Of 38 applications which completed review, 12 had only minor issues, and 6 had at least one major issue. Every application had at least one identified problem.

Although applying blanket severity ratings across such a wide range of apps is somewhat arbitrary, it can help to illustrate the range, frequency, and scope of the observed issues:

| Severity | Issue                                  | #  |
|----------|----------------------------------------|:--:|
| Low      | Userid stored insecurely               | 27 |
| Low      | Did not use certificate pinning        | 36 |
| Medium   | Credentials passed as URL parameters   | 9  |
| Medium   | Tokens stored insecurely               | 21 |
| Medium   | Password sent with every request       | 2  | 
| High     | Did not use TLS or ignored cert errors | 2  |
| High     | Password stored insecurely             | 4  |

[*I rated lack of certificate pinning as "Low" because it still has not been accepted as a standard practice, though I recommend its use wherever feasible.*]

Most of these issues can be easily addressed through three simple changes:

* Use TLS certificate pinning
* Only store credentials in the keychain (for sensitive applications, ensure this includes the userid as well)
* Take steps to avoid leakage of credentials to HTTP caches and cookie files

This survey only scratched the surface. Forty applications is far from a significant sampling. However, I do think that it provides a useful snapshot: These applications reside, today, on my personal device, are reasonably popular brands and commonly-used categories of applications, and receive regular use. The applications, generally, handle login credentials well, with a few bad (and a few exceptionally good) outliers. And the problems seen during this survey appear to match up with those of other applications I've tested as part of my employment. 

It is therefore hoped that this survey may provide insight into secure  credential management for iOS applications; that it may help developers to improve the security of their applications; and thus, reduce the risk to consumers' data in general.

## References

#### Metadata

Tags: ios, mobile applications, authentication, best practices

**Primary Author Name**: David Schuetz  
**Primary Author Affiliation**: Intrepidus Group  
**Primary Author Email**: david@dasnet.org