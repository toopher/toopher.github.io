---
layout: post
category: 
tags: []
author: Seth
display_title: Toopher on Rails - Part 1
---
{% include JB/setup %}

Over the weekend I added Toopher to a sample application from the [Ruby on Rails Tutorial](http://railstutorial.org/) by [Michael Hartl](http://michaelhartl.com/) (specifically, the [sample app](https://github.com/mhartl/sample_app)). The application is a basic social microposting site with a simple authentication system (under the covers it uses `has_secure_password`). I hope the lean application is easy to understand--I tried to write clean, idiomatic Ruby without too many frills or tricks.

You can see the code on [GitHub](https://github.com/smholloway/sample_app_2nd_ed_with_toopher/). The main changes are in the `sessions_controller` and `users_controller`.

The application is [running on Heroku](https://rails-sample-app-with-toopher.herokuapp.com/), but I suggest you visit the [Toopher Demo](https://demo.toopher.com/) if your goal is to see how Toopher works.

# Highlights of the integration

Let's walk through some of the changes made to integrate Toopher.

## Grab the Toopher API gem
First things first. Add the [Toopher API gem](http://rubygems.org/gems/toopher_api) to your `Gemfile`:

``` ruby
gem 'toopher_api', '~> 1.0.5'
```

After intalling the gem (`bundle install`), you will have access to [Toopher's Ruby language
library](https://github.com/toopher/toopher-ruby), which simplifies calls to the API.

## Updates to the User model
We will update the User model (`user.rb`) to include a `toopher_pairing_id` and a utility method to determine if Toopher is enabled for the user. Here's a highlight of the changes:

``` ruby
class User < ActiveRecord::Base
  # ... removed for brevity ...

  attr_accessible :toopher_pairing_id

  def toopher_enabled?
    !toopher_pairing_id.blank?
  end

  # ... removed for brevity ...
end
```

The `toopher_pairing_id` can be added with a migration like this:

``` ruby
class AddToopherToUsers < ActiveRecord::Migration
  def change
    add_column :users, :toopher_pairing_id, :string
  end
end
```

With these changes in place, we can add in logic to pair/remove Toopher
and conditionally authenticate login requests with Toopher.

## Adding Toopher to the UI
In this example, we will add Toopher to the bottom of the user settings page. We'll start by creating a partial view template (`_toopher.html.erb`):

``` rhtml
<h3>Toopher</h3>

<% if current_user and current_user.toopher_enabled? %>
  <p>
  Thanks for setting up Toopher.
  </p>
  <form action="toopher_delete_pairing" method="post" id="unpair">
    <input class="btn btn-large btn-primary" type="submit" value="Remove Toopher">
  </form>
<% else %>
  <p>
  Please set up Toopher
  </p>
  <form action="toopher_create_pairing" method="post" id="pair">
    <input type="text" name="pairing_phrase" placeholder="Pairing phrase" />
    <input class="btn btn-large btn-primary" type="submit" value="Pair with Toopher">
  </form>
<% end %>
```

This hooks into the user settings page (`edit.html.erb`):

``` rhtml
<%= render 'toopher' %>
```

Now, whenever a user navigates to their settings, they will see the option to add or remove Toopher from their account. As you see in the Toopher partial above, to add Toopher the app will `POST` to `toopher_create_pairing` with a pairing phrase; to remove Toopher the app will `POST` to `toopher_delete_pairing`. Let's look at the pairing methods.

## The pairing methods 
We implement pairing and removing Toopher as methods on the user, so we update our routes and our controller. First, the routes (`routes.rb`):

``` ruby
resources :users do
  # ... removed for brevity ...

  post :toopher_create_pairing
  post :toopher_delete_pairing
end
```

... and the controller (`users_controller.rb`):

``` ruby
  def toopher_create_pairing
    @user = User.find(params[:user_id])
    pairing_phrase = params[:pairing_phrase]
    toopher = ToopherAPI.new(ENV['TOOPHER_CONSUMER_KEY'], ENV['TOOPHER_CONSUMER_SECRET']) rescue nil

    if not session[:toopher_pairing_start]
      begin
        pairing = toopher.pair(pairing_phrase, @user.email)
      rescue
        return toopher_bad_pairing_phrase
      end
      session[:toopher_pairing_start] = Time.now
      session[:toopher_pairing_id] = pairing.id
    else
      pairing = toopher.get_pairing_status(session[:toopher_pairing_id])
    end

    if Time.now - session[:toopher_pairing_start] > 60
      return pairing_timeout
    end

    if pairing and pairing.enabled
      if @user.update_attribute(:toopher_pairing_id, session[:toopher_pairing_id])
        sign_in @user
        return toopher_pairing_enabled
      end
    end

    render :json => {:pairing_id => session[:toopher_pairing_id]}
  end

  def toopher_delete_pairing
    @user = User.find(params[:user_id])
    if @user.update_attribute(:toopher_pairing_id, "")
      sign_in @user
      return toopher_pairing_disabled
    else
      render 'edit'
    end
  end
```

During pairing, we store details about the request in the session. The same endpoint is polled by client-side JavaScript until the user accepts the pairing or the pairing times out (no response for 60 seconds, in this case).

With this, we can add and remove Toopher from a user. 

## Authentication changes
Your standard login might look something like this:

``` ruby
class SessionsController < ApplicationController
  # ... removed for brevity ...
  def create
    user = User.find_by_email(params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      pass_login(user)
    else
      fail_login
    end
  end
  # ... removed for brevity ...
end
```

The basic Toopher logic hooks in after the `user.authenticate` call, like this:

``` ruby
class SessionsController < ApplicationController
  def create
    user = User.find_by_email(params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      if user.toopher_enabled?
        toopher_auth(user)
      else
        pass_login(user)
      end
    else
      fail_login
    end
  end
  # ... removed for brevity ...
end
```

The `toopher_auth` method will initiate a Toopher authentication request if a request is not pending, storing information about the request in the session. As with the pairing endpoint, the authentication endpoint is polled by client-side JavaScript until the user replies or the request times out (60 seconds here).

Currently, the Toopher mobile app shows the user four pieces of information: 1) the site being accessed, 2) who initiated the request (typically a username or email address), 3) the action that triggered the request, and 4) the computer that originated the request. The site name comes from the Toopher API credentials. The username is provided by the implementer, as is the action. Terminal names are also provided by the implementer, but we suggest that the user names the terminal. A request coming from a meaningful, personally named terminal (like "downstairs PC") is easily differentiated from a request coming from an unknown or generically named terminal. 

``` ruby
def toopher_auth(user=nil)
  toopher = ToopherAPI.new(ENV['TOOPHER_CONSUMER_KEY'], ENV['TOOPHER_CONSUMER_SECRET']) rescue nil

  terminal_name = user.toopher_terminals.where(:cookie_value => cookies[:toopher]).first.terminal_name rescue nil
  if terminal_name.nil?
    return name_terminal
  end

  if not session[:toopher_auth_start]
    begin
      auth_status = toopher.authenticate(user.toopher_pairing_id, terminal_name, 'login', { terminal_name_extra: cookies[:toopher] })
    rescue
      return fail_login
    end
    session[:toopher_auth_start] = Time.now
    session[:toopher_auth_id] = auth_status.id
  else
    auth_status = toopher.get_authentication_status(session[:toopher_auth_id])
  end

  if (Time.now - session[:toopher_auth_start] > 60)
    return toopher_timeout
  end

  if !auth_status.pending
    if auth_status.granted
      return pass_login(user)
    else
      return toopher_deny
    end
  end

  render :json => { :pairing_id => session[:toopher_pairing_id] }
  return
end
```

With this, you will have a basic Toopher implementation working on your Rails site. 

Questions or comments? Let us know. We aim to please!

