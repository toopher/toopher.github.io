---
layout: post
category: 
tags: []
author: Seth
display_title: Technical Terms Defined, Part I
---
{% include JB/setup %}

![Most security books read like dusty, arcane tomes](http://i.imgur.com/lboC8so.jpg)

Between marketing speak and security jargon, it can be really hard to keep things straight. Sometimes it feels like reading some dusty, arcane tome. Below are a few terms that I've had to brush up on recently--hopefully this post clarifies the meaning.

 - [authentication](http://en.wikipedia.org/wiki/Authentication) - Verifying that a user is who they say they are--that is, confirming their identity
 - [authorization](http://en.wikipedia.org/wiki/Authorization) - Verifying the user is allowed to access the content (think of roles like `admin` versus `user`)
 - [two-factor authentication](http://en.wikipedia.org/wiki/Two-factor_authentication) - Also known as 2FA or more broadly as multi-factor authentication. The "factors" are something you have, something you are, and something you know. Passwords are something you know. You are your fingerprint, retina scan, or image. What about something you have? Those are things like key fobs or badges. Most modern 2FA systems build on your password (something you know) with something you have; there are also systems leveraging something you are, but these are less common--perhaps because they require specialized hardware. Note that a password (something you know) plus security question (something you know) is not considered multi-factor because both inputs are something you know.
 - [out-of-band](http://en.wikipedia.org/wiki/Out-of-band#Authentication) - An outside provider that is not part of the site.
 - [OTP](http://en.wikipedia.org/wiki/One_time_password) - One time passwords, which might be implemented as a time-based one time password (TOTP). At this point, texting is probaby the most popular mode of delivering one time passwords.
 - [location-awareness](http://en.wikipedia.org/wiki/Location_awareness) - A subset of [context awareness](http://en.wikipedia.org/wiki/Context_awareness).  Toopher uses your phone's GPS to determine your location. You can choose to automate subsequent responses at that location.
 - [nonce](http://en.wikipedia.org/wiki/Cryptographic_nonce) - An arbitrary number that is only used once. Nonces are often used to ensure that messages are sent only once--the database tracks nonces and invalidates them once they're used.
 - [OAuth](http://en.wikipedia.org/wiki/OAuth) - Open authentication. OAuth allows a user to access a resource without providing a username and password. Instead, the identity is verified through credentials like a user's key and secret. [OAuth has a lot of moving parts](http://tools.ietf.org/html/rfc5849), but the general idea is that you combine all user input plus a nonce and timestamp, urlencode and sort the parameters, then hash the parameters with a secret key to create a unique string (technically, a hash message authentication code [HMAC](http://en.wikipedia.org/wiki/HMAC)); this string is sent with each message. The server then follows the same process and checks that the client and server hashes match. If someone tweaked a parameter, the server hash would not match the user's and an OAuth error would be thrown. Another error could arise if someone hashes with the wrong secret key.
 - [OATH](http://en.wikipedia.org/wiki/Initiative_For_Open_Authentication) - Essentially a reference implementation for open authentication.  The two-factor offerings from compaines including Google, Dropbox, and Linode fall under this umbrella.

With that out of the way, I hope the following statement makes more sense:

_Toopher is usable, out-of-band two-factor authentication that uses location-awareness to automate critical actions like logins (authentication), balance transfers, payments, and more (authorization). We also support OATH style OTPs, and we sign all requests with OAuth._

