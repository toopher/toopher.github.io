---
layout: post
category: 
tags: []
author: Seth
---

Last week [Toopher made it onto Reddit](http://www.reddit.com/r/netsec/comments/1kxn5f/toopher_a_simple_phonebased_twofactor/) organically, which was really cool! Below are responses to some of the issues raised. If you've got another question, please ask!

* *I don't understand automating based on location--can't I just log in when you're in a known location?* I've heard this concern from a couple brilliant techies. To explain why this isn't the case, let's examine the authentication screen: the user is presented with an action (e.g., "login"), a username (e.g., "seth@toopher.com"), and a terminal name (e.g., "my computer"). Behind the scenes, there's also a `terminal_name_extra` that should be unique for every device. Sites can also choose not to allow automation, although the automation improves the user experience. Automation only occurs if (1) the site allows automation, (2) the user has enabled automation, (3) the user is in the location where they enabled automated, (4) the credentials are correct, the terminal name exactly matches the known terminal name, the `terminal_name_extra` matches the automated version, and the action matches the automated action.

* *Location awareness defeats the purpose.* Your Toopher mobile app can use your location to automate responses in locations where you enable automation. Toopher does not know your location--it is stored on your phone and never transmitted. We care about privacy, a lot. Automation is super handy if you're logging in from your house because you have the security of a second factor without the hassle.

* *How does two-factor authentication fit in to my normal workflow?* In case people didn't know, I posted a two-factor authentication flowchart: 

![two-factor authentication flowchart](http://i.imgur.com/MUP4dEZ.png)

* *Toopher is not out-of-band.* Toopher is not part of the site--we are a separate service accessible over wifi or a cell network--thus we are out-of-band.

* *With standard 2FA tokens, I am not bothered when a criminal tries to login to my account.* I love that Toopher lets me know someone has my username and password. If my bank sends a push notification when I'm not trying to login, I will go change my credentials! And, if a person recycled passwords, they would know that other sites were vulnerable.

* *People aren't hacked--sites are hacked.* Companies are hacked then your credentials are used by the hackers or sold onto the black market. Identity theft and online fraud often starts this way. I would feel much better if every account was secured by two-factor auth like Toopher because it mitigates the risk associated with bad guys knowing my login information.

* *How is this better than or different from X?* Toopher is modern two-factor authentication. We provide additional information about each request that you're accepting. We're easier to use than standard 2FA; pressing `Allow` or `Deny` is simpler than inputting a 6-8 character number. And we have automation so you can be just as secure with far less hassle. We can dramatically reduce the number of two-factor auth apps and tokens: no more futzing with one token per account--the Toopher app stores all your accounts in one handy place.  

Here are a few screenshots to help make it more clear.

![Toopher pairing phrase](http://i.imgur.com/LnlyiGb.png)

![Approve the Toopher pairing](http://i.imgur.com/Fm8yqc6.png)

![Approve the Toopher authentication](http://i.imgur.com/MHlzawR.png)

![Automate the Toopher authentication](http://i.imgur.com/2NJ5O9N.png)

