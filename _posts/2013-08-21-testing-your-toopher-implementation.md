---
layout: post
category: 
tags: []
author: Seth
---
{% include JB/setup %}

So, you've implemented Toopher and you're about to take over the world.  Not so fast! Below is a checklist of scenarios to test before you unleash the beast.

## Scenarios to Test
* Can you pair? Ensure `allow` and `deny` from the Toopher-enabled device yield proper results on the implementing service.

* Does pairing time out? If the user has not accepted the pairing after some time (say 90 seconds), we recommend you fail the pairing and offer the user the chance to try again.

* Can you unpair (effectively removing Toopher)? This is the opposite of pairing and should simply flip the `is Toopher enabled?` bit. While we can't imagine anyone not wanting to use Toopher, it could happen.

* Can you authenticate with and without Toopher enabled? After pairing, subsequent logins should trigger calls to Toopher. Ensure `allow` and `deny` from the Toopher-enabled device yield proper results on the implementing service.

* Does authenticating time out? If the user has not accepted the login after some time (say 90 seconds), we recommend you fail the authentication request and offer the user the chance to try again.  

* Can multiple people authenticate from the same computer? Implementers generally tie a user to a terminal with a cookie; if you've done so, ensure that the cookies are not user-specific. A good test is to create two accounts.  Log into the first account, then into the second. You should have to name the terminal for both accounts. Finally, log back in on the first user--you should not have to rename the terminal. One way to achieve this is to store a random string in the cookie then associate a user's terminal with the value of the cookie.

* Have you set up `terminal_name_extra` properly? Open a browser and log in to the site. When prompted, name the computer--remember this phrase. When prompted by the Toopher app, allow automation. Now, open a new browser or incognito window and log in to the site. You should be prompted to name the computer, so repeat the exact same name from the other browser. If the second browser's logins are automated, you are vulnerable to terminal name guessing and you need to utilize `terminal_name_extra` when calling the `authenticate` method.

* Can you recover your account if you are locked out of Toopher? If a user loses their phone or removes their pairing (on the Toopher-enabled device), they will not be able to login with Toopher. It is important to provide a way for users to reset Toopher without access to Toopher--for example, an email similar to password reset emails.

* Do you block when waiting for user responses? For site performance, you should poll for status changes (as opposed to blocking and waiting). Check this by authenticating in one browser/tab and navigating around the site from a second browser; if you're able to browse the site while an authentication is happening, you are not blocking. One way around this issue is to create two API endpoints, one to check the pairing status and one to check the authentication status. Then, use JavaScript (if this is a website) to poll the server until the request is complete or a timeout is reached.

---

Did we miss something? Does something not make sense? Leave a comment or email us and we'll try to help you out.

