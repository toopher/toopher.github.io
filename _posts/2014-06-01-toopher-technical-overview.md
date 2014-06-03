---
layout: post
category: 
tags: []
author: Seth
display_title: Toopher Technical Description
---
{% include JB/setup %}

If you're reading this, you're probably a techincal person. Hopefully,
you've read about [Toopher](https://www.toopher.com/) and understand the
basics. Now, we'll talk technical details.

## Toopher Technical Overview

Toopher is smart, simple multi-factor authentication. Our advantage is
our ability to automate requests based on the phone’s context such as
its location. You can tell Toopher to automatically allow similar
requests when you’re in the same location; for example, if you do all
your online banking from your laptop at home, Toopher can learn this and
automatically respond without prompting you for those logins. By using
the phone’s location awareness, we provide the same level of security
with far less hassle. The Toopher app provides a clean, unified user
experience where all accounts are collected in one place.

Toopher is secure from the ground up, built using proven technologies
such as TLS/SSL, OAuth 1.0a, and RFC 6238 (TOTP). These processes ensure
that data remains confidential on-the-wire and prevent standard attacks
that rely on replaying requests or tampering with data.

Below we present the high level flow of two-factor authentication, the
Toopher component architecture, and we dive deeply into pairing and
authenticating.

## Two-factor Authentication and Toopher

Multi-factor authentication adds additional security over standard
logins, which typically rely solely on a username and password.
Multi-factor refers to the three possible factors: knowledge,
possession, and inherence. Said another way, the factors are something
you know (username and password, security questions, etc.), something
you have (badge, key fob, phone), and something you are (biometrics like
retinas and fingerprints). Toopher, as a second factor, typically comes
after the standard something you know (username and password), providing
something you have (your Toopher-enabled device).

![Two factor authentication flowchart](http://i.imgur.com/MUP4dEZh.png)

#### Figure 1. The flow of two-factor authentication. Each authentication system is checked in series until a request is denied or passes all checks. 

Figure 1 shows the states in a generic two-factor authentication system.
In a standard login, users input their credentials and an authentication
system (e.g., RADIUS , LDAP/AD, or Kerberos) checks the validity. If the
credentials are incorrect, the login attempt fails. If the first factor
authentication is successful, we go into the second factor. In a
permissive system, users would be allowed to enroll if and when they’re
ready; however, in a stricter system you could also require that all
users enable Toopher. In the permissive system if the user has not
enabled the second factor, they only use the first system. In the strict
system, users would be asked to enroll on their first authentication. 

To enroll in Toopher, users simply need to pair their device with the
service. This is accomplished through the user’s device. We currently
support pairing using a unique phrase generated on the phone or a
Toopher-specific QR code. The user’s choices are saved and he/she is
ready to allow or deny requests the Toopher way. We provide more details
in the section titled The Toopher Two Step: Pairing and Authentication.

After enrollment, valid authentication attempts—those passing the first
factor—will be routed through Toopher. If the user allows the request,
log them in; if they do not, fail the authentication attempt.

## Toopher component architecture

The Toopher infrastructure is powerful and robust, consisting of three
main components: the API server, requesters, and authenticators (shown
in Figure 2 below). 

![Toopher Components](http://i.imgur.com/qYSn27fh.png)

#### Figure 2. The Toopher architecture consists of three main components: the API server, requesters, and authenticators. We also maintain a developer portal with additional documentation.

The brain of the operation is the API. Its main role is to shuffle
requests from requesters to the authenticators that handle them
(presenting request details to users if necessary). The API is hosted on
Google’s cloud computing infrastructure, which provides instant
scalability and robustness. Google’s platform is well-tested, secure,
fast, and fault tolerant. 

Requesters, or implementing services, invoke the Toopher API when they
want to authenticate an action. Requesters are identified with a unique
key and secret that they use to sign requests. Requesters should provide
a way for users to pair with the requester’s service, and then seek the
user’s approval for authentication and authorization. The Toopher Demo
site (https://demo.toopher.com) provides an example implementation.

Authenticators (the devices used by our end-users e.g.: an Android
device with the Toopher app installed) keep the API server informed of
the status of requests via a private API.  They register with the API
server, and are provisioned credentials for all future communication
with the API server. Smart phone apps exist for iOS and Android, with
BlackBerry, Windows Phone, and SMS-based solutions currently in
progress.

## The Toopher Two Step: Pairing and Authenticating

Pairing (Figure 3, below) ties a specific device and user to a
Toopher-enabled service or site. The process begins when the user
requests a pairing phrase on their device (commonly a smartphone or
tablet).  After the phrase is generated, the user inputs the phrase on
the site that initiated a Toopher pairing request. The pairing request
will propagate to the user’s device and present the user with the option
to allow or deny the pairing. In practice, the user generates a pairing
phrase on their phone, inputs it on the site through a web browser, then
receives a notification on their phone asking them to authorize the
pairing; they click allow and the user’s account is Toopher-enabled.

![Toopher pairing sequence diagram](http://i.imgur.com/92A1QVq.png) 

#### Figure 3. A sequence diagram for Toopher pairings.
 
Authentication requests (Figure 4, below) are much more common than
pairings—for each Toopher-enabled site you pair once and authenticate
many times. When asked to allow or deny a request, the user is shown the
action being taken (e.g., “login”), the organization that is seeking
permission, the username, and the terminal from which the request
originated (e.g., “Kitchen PC” or “my macbook.”). To provide these
pieces of information to the user, the authentication requires a pairing
ID, a terminal name, and an action. Ideally, a user will remember what
they have named their terminals and will be aware of any actions they’re
taking on a site, so the user can quickly deny a request if any of the
presented information is wrong.
 
![Toopher authenticating sequence
diagram](http://i.imgur.com/daaxxyn.png) 

#### Figure 4. A sequence diagram for Toopher authentication requests.

