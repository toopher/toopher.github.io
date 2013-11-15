---
layout: post
category: 
tags: []
author: Seth
display_title: Toopher on Rails - Part 1
---
{% include JB/setup %}

Integrating Toopher is simple! Here, we will provide an overview of how to integrate with Ruby on Rails. We will start with the basics, then incrementally improve the implementation. In later posts we will show an entire example application. 

Note: We are building off of Michael Hartl's popular [Rails Tutorial](http://ruby.railstutorial.org/) (specifically, the [sample app](https://github.com/mhartl/sample_app)), which walks the user through creating a web app like Twitter.

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
  attr_accessible :name, :email, :password, :password_confirmation, :toopher_pairing_id

  # ... removed for brevity ...

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

``` erb
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

``` erb
<%= render 'toopher' %>
```

Now, whenever a user navigates to their settings, they will see the option to add or remove Toopher from their account. As you see in the Toopher partial above, to add Toopher the app will `POST` to `toopher_create_pairing` with a pairing phrase; to remove Toopher the app will `POST` to `toopher_delete_pairing`. Let's look at the pairing methods.

## The pairing methods 
We implement pairing and removing Toopher as methods on the user, so we update our routes and our controller. First, the routes (`routes.rb`):

``` ruby
resources :users do
  member do
    get :following, :followers
  end

  post :toopher_create_pairing
  post :toopher_delete_pairing
end
```

... and the controller (`users_controller.rb`):

``` ruby
def toopher_create_pairing
  pairing_phrase = params[:pairing_phrase]
  toopher = ToopherAPI.new(ENV['TOOPHER_CONSUMER_KEY'], ENV['TOOPHER_CONSUMER_SECRET']) rescue nil
  @user = User.find(params[:user_id])
  pairing = toopher.pair(pairing_phrase, @user.email)
  start_time = Time.now

  while !pairing.enabled and (Time.now - start_time <= 60)
    pairing = toopher.get_pairing_status(pairing.id)
    sleep(1)
  end

  if pairing.enabled
    if @user.update_attribute(:toopher_pairing_id, pairing.id)
      flash[:success] = "Toopher now active for this account."
      sign_in @user
      redirect_to @user
    else
      render 'edit'
    end
  else
    render 'edit'
  end
end

def toopher_delete_pairing
  @user = User.find(params[:user_id])
  if @user.update_attribute(:toopher_pairing_id, "")
    flash[:success] = "Toopher removed from this account."
    sign_in @user
    redirect_to @user
  else
    render 'edit'
  end
end
```

With this, we can add and remove Toopher from a user. However, we are not done yet because we are blocking the server as we wait for the user to acknowledge the pairing. We'll discuss this more after looking at the first-pass on authentication updates.

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

  def toopher_auth(user=nil)
    toopher = ToopherAPI.new(ENV['TOOPHER_CONSUMER_KEY'], ENV['TOOPHER_CONSUMER_SECRET']) rescue nil

    if not session[:toopher_auth_start] # we have a request pending
      auth_status = toopher.authenticate(user.toopher_pairing_id, request.remote_ip)
      session[:toopher_auth_start] = Time.now
      session[:toopher_auth_id] = auth_status.id
    else
      auth_status = toopher.get_authentication_status(session[:toopher_auth_id])
    end

    while auth_status.pending and (Time.now - session[:toopher_auth_start] <= 60)
      auth_status = toopher.get_authentication_status(session[:toopher_auth_id])
      sleep(1)
    end

    if auth_status.granted
      pass_login(user)
    else
      fail_login
    end
  end

  def destroy
    sign_out
    clear_toopher_session_data
    redirect_to root_url
  end

  def pass_login(user)
    clear_toopher_session_data
    sign_in user
    redirect_back_or user
  end

  def fail_login
    clear_toopher_session_data
    flash.now[:error] = 'Invalid email/password combination'
    render 'new'
  end

  def clear_toopher_session_data
    session.delete(:toopher_auth_start)
    session.delete(:toopher_auth_id)
  end
end
```

This implementation has a couple issues:

* It blocks the server, waiting for the `auth_status` to be acknowledged by the user or timeout
* it uses the user's reported `remote_ip`, which isn't very meaningful

The latter issue we will fix by letting the user name their terminal, which will be tracked in a cookie. We find "My work laptop" more meaningful than "108.162.201.68" or "Chrome on Mac OS X", but the choice is yours.

The former we will fix by moving the timeout logic to client-side JavaScript where we will periodically poll the server for the request's status. Note: JavaScript polling is not the only way to solve this problem. For example, you could use something like [Socket.io](http://socket.io/) to effectively keep the socket open until the server responds; you could implement an authentication pending page that automatically forwards the user on after authentication; or you could create a callback that triggers the login. 

With this, you will have a basic Toopher implementation working on your Rails site. In the next post we'll show how to convert the this minimum viable Toopher implementation into an easy-to-use, non-blocking user delight.

Questions or comments? Let us know. We aim to please!

