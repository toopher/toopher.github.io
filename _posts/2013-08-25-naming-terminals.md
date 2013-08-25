---
layout: post
category: 
tags: []
author: Evan
---
{% include JB/setup %}

We are commonly asked about how to track terminals. Implementers need
to provide a unique string as the `$terminalName` argument for each
device from which the user connects.  Some clients come up with these
strings automatically based on the device (e.g.: "Safari on iPhone",
"Chrome on Windows 8", etc.) or ask their users for a name when they log
in from a new device (e.g.: these names vary but will be something like
"My Mac" or "Evan's Desktop").

In any case you'll want to make sure that the terminal name is never the
same for requests from different end devices.  To make this easier we've
recently changed our library so that it allows you to specify a terminal
name "extra" that will ensure that terminals are treated as unique even
if they happen to have the same name.  To do this you'll need to update
the library (just grab the latest from
[github](https://github.com/toopher)) and modify the `authenticate` call
as seen below:

    $auth = $toopher->authenticate($pairingId, $terminalName, $actionName, array("terminal_name_extra" => $terminalExtra));

A common tactic is to store a random string as a secure cookie in the
browser and then use this cookie's value to determine a unique
terminalExtra.

Let us know if you hit any snags and we'll try to help.

