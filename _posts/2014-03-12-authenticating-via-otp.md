---
layout: post
category: 
tags: []
title: Authenticating via OTP
author: Seth
---
{% include JB/setup %}

Toopher provides invisible authentication, but we understand the need
for a backup. When users cannot authenticate in the standard way, we 
recommend you provide an alternate method. The Toopher app allows users 
to enter a one time password (OTP), and the standard
`authentication` endpoint simply needs an extra parameters.

## API information

Endpoint: `v1/authentication_requests/initiate`

Optional Parameter: `otp`

Parameter Definition: The IETF standard RFC6238 one-time-password,
unique to each pairing, generated in the Toopher mobile app when the
user clicks on a pairing.  If submitted, the Toopher API will verify the
OTP, and immediately return the authentication result (i.e., pending will
be `False`, and granted will be `True` if the OTP is valid).  This feature
allows for fallback authentication in case the user's mobile device does
not have network access.

## Example using the toopher-python library

Below is a rough sketch of how you could authenticate using a
user-entered one time password.

    import toopher
    api = toopher.ToopherApi(key, secret)
    auth = api.authenticate(pairing.id, username, action, otp=123456)
    if auth.status == "granted":
        # success
    else:
        # failure

## So, where does this go?

Where should you put an OTP entry? We pride ourselves on being flexible, so you're free to do this however
you see fit. That being said, here are a few options: 

 * include a link beside your "Forgot password" link on the login page.
   Something like "Limited cell service?" or "Input an OTP instead"
would lead to a login with an OTP entry field.
 * in a modal dialog displaying the status of Toopher authentication.
   For example, after starting the login process, tell the user that
Toopher is contacting their phone and include a text entry box where
they can enter an OTP at anytime.
 * on a specialized login page that includes the OTP entry. This page
   would be shown after the user's initial authentication request times
out.

With any of these methods, your backend can check for the existence of the OTP and follow the appropriate path. Something like this:

    if otp:
        auth = api.authenticate(pairing.id, username, action, otp=otp)
    else:
        auth = api.authenticate(pairing.id, username, action)

If you've done it better, we would love to hear about it--please share.
