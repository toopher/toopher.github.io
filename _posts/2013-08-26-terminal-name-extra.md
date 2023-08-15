---
layout: post
category: 
tags: []
author: Seth
---

We recently mentioned `terminal_name_extra` in our post about [testing your Toopher implementation](https://devblog.toopher.com/2013/08/21/testing-your-toopher-implementation/) and in our quick advice about [naming terminals](https://devblog.toopher.com/2013/08/25/naming-terminals/). In this post we'll provide a little more information as well as code snippets for each language.

## What does it mean?
So, what does it mean? It's not a [double rainbow](http://www.youtube.com/watch?v=OQSNhk5ICTI)--it's just a helpful parameter to differentiate between terminals with the same name. Terminal names are guessable: people are likely to choose phrases like "my computer" or "macbook", so `terminal_name_extra` stops the bad guys from guessing your terminal name and submitting requests as you. The `authenticate` method does not require `terminal_name_extra` but we **strongly** suggest using it. 

What should you use for the value? As with all things programming, there are a variety of options; however, an easy way to populate `terminal_name_extra` is to use the `id` of the user's `terminal`.

Here's an example `terminal` model in Ruby on Rails:

    class ToopherTerminal
      belongs_to :user
      field :id, type: String
      field :cookie_value, type: String
      field :display_name, type: String
    end

Here, a user *has many* `terminal`s that posess an `id` (assigned by the database), `cookie_value` (whatever value is in the Toopher cookie), and `display_name` (the user-entered description). Using this scheme you would lookup the `terminal` by the `cookie_value` and input `id` as the `terminal_name_extra` parameter in your `authenticate` call. Here's some sample code using the above Rails model.

    terminal = user.toopher_terminals.where(cookie_value: cookies[:toopher_terminal]).first
    auth = toopher.authenticate(user.toopher_pairing_id, terminal.display_name, 
                                'login', { terminal_name_extra: terminal.id })

## How does it work in my language?
Below is the current function defintion and an example usage for each language library.

### PHP
[PHP library](https://github.com/toopher/toopher-php). The `authenticate` [function](https://github.com/toopher/toopher-php/blob/master/lib/toopher_api.php) for version 1.0.0:

    public function authenticate($pairingId, $terminalName, $actionName = '', $extras = array())

Example usage:

    $auth = $toopher->authenticate($pairingId, $terminalName, 
                                   $actionName, array("terminal_name_extra" => $terminalExtra));

### Ruby
[Ruby library](https://github.com/toopher/toopher-ruby). The `authenticate` [function definition](https://github.com/toopher/toopher-ruby/blob/master/lib/toopher_api.rb) for version 1.0.5:

    def authenticate(pairing_id, terminal_name = '', action_name = '', options = {})

Example usage:

    auth = toopher.authenticate(user.toopher_pairing_id, terminal.display_name, 
                                action, { terminal_name_extra: terminal._id })

### Python
[Python library](https://github.com/toopher/toopher-python). The `authenticate` [function definition](https://github.com/toopher/toopher-python/blob/master/toopher/__init__.py) for version 1.0.4:

    def authenticate(self, pairing_id, terminal_name, action_name=None, **kwargs):

Example usage:

    auth = toopher.authenticate(pairing_id, terminal_name, 
                                action, terminal_name_extra=terminal_extra)

### Java
[Java library](https://github.com/toopher/toopher-ruby). The `authenticate` [function definition](https://github.com/toopher/toopher-java/blob/master/src/com/toopher/ToopherAPI.java) for version 1.0.0:

    public AuthenticationStatus authenticate(String pairingId, String terminalName,
                                             String actionName, Map<String, String> extras) 

Example usage:

    Map<String, String> extras = new HashMap<String, String>() { 
      { put("terminal_name_extra", terminalNameExtra); } };
    AuthenticationStatus auth = toopher.authenticate(pairingId, terminalName, action, extras);


### .NET (dotnet)
[.NET library](https://github.com/toopher/toopher-dotnet). The `Authenticate` [function definition](https://github.com/toopher/toopher-dotnet/blob/master/ToopherDotNet/ToopherDotNet.cs) for version 1.0.0:

    public AuthenticationStatus Authenticate (string pairingId, string terminalName, 
                                              string actionName = null, 
                                              Dictionary<string, string> extras = null)

Example usage:

    var extras = new Dictionary<string, string> { { "terminal_name_extra", terminalNameExtra } };
    var auth = toopher.Authenticate (pairingId, terminalName, actionName, extras);

---

So, now that you know a bit about `terminal_name_extra`, we hope you'll use it in every Toopher implementation. If you run into any problems, just let us know.

