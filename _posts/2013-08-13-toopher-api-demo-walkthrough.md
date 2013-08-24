---
layout: post
category: 
tags: []
author: Seth
display_title: Toopher API Demo Walkthrough
---
{% include JB/setup %}

The Toopher API is really easy to use! But you don't have to take my
word for it--just install one of the [Toopher language libraries](https://github.com/toopher) from GitHub and
check it out. Each one contains a demo that lets you see the basics. The steps are shown below, which should be helpful for people integrating Toopher into their services.

If you're not looking to interact with Toopher programmatically, check
out the [Toopher demo site](https://demo.toopher.com/) for a quick
course in using Toopher.


## Notes
To proceed sign up for a Toopher account and download the mobile
application. The images shown are Toopher v1.0.1 on iOS 6.0.1--the app
looks better now :) Updated images coming soon.


## Create a new Requester on the Toopher web site

Navigate to the Toopher Dev site: https://dev.toopher.com/requesters/

![Toopher Requesters homescreen](/assets/images/ToopherRequestersBlank.png)

Press the "Create new requester" button in the top right.

![The new Requester page](/assets/images/ToopherCreateRequesterBlank.png)

Fill in your information and "Create requester"

![Fill in your information to create a new
Requester](/assets/images/ToopherCreateRequester.png)

You will be redirected to the Requesters page again. 

![Toopher Requesters after creating your first
Requester](/assets/images/ToopherRequesters.png)

---

## Insert your key and secret in the demo file

Edit the demo file and insert your Requester's key and secret.

---

## Create a new pairing phrase from the mobile app

Open the Toopher mobile app

![Toopher iOS Homescreen](/assets/images/Toopher_iOS_Homescreen.png)

Press the plus sign (+) in the top right corner of the app, which will
present a new pairing phrase.

![Toopher iOS Pairing Phrase](/assets/images/Toopher_iOS_Pairing_Phrase.png)

Take note of the pairing phrase.

---

## Run the demo

Open a terminal and run the demo. For example in Ruby:

```
ruby demo/toopher_demo.rb
```

Still in the command line, enter the pairing phrase from the mobile app
when prompted.

![Toopher demo in Ruby](/assets/images/ToopherRubyAPIPairingPhrase.png)

Now, the demo will query your app for permission.


### Accept the request on the mobile app

Press "Grant" on the phone. 

![Toopher iOS Pairing Request](/assets/images/Toopher_iOS_Pairing_Request.png)


### Verify that the pairing succeeded

Switch back to the running demo and you should see a successful response
(in this case, "paired successfully!").

![Toopher demo in Ruby](/assets/images/ToopherRubyAPIDemoPairingSuccess.png)


### Continue running the demo

All subsequent requests will trigger an authentication request on the
mobile app. 

![Toopher iOS Authentication
Request](/assets/images/Toopher_iOS_Authentication_Request.png)


## Next steps

This demonstrates the user and administrator actions when setting up
Toopher. From here, you can easily add two-factor authentication to your
sites!

