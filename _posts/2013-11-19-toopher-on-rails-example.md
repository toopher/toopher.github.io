---
layout: post
category: 
tags: []
author: Seth
display_title: Toopher on Rails - Augmenting the Ruby on Rails tutorial sample app
---

Over the weekend I added Toopher to a sample application from the [Ruby on Rails Tutorial](http://railstutorial.org/) by [Michael Hartl](http://michaelhartl.com/) (specifically, the [sample app](https://github.com/railstutorial/sample_app_2nd_ed)). The application is a basic social microposting site with a simple authentication system (under the covers it uses `has_secure_password`). I hope the lean application is easy to understand--I tried to write clean, idiomatic Ruby without too many frills or tricks.

You can see the code on [GitHub](https://github.com/smholloway/sample_app_2nd_ed_with_toopher/). See my [pull request](https://github.com/smholloway/sample_app_2nd_ed_with_toopher/pull/1/files), which added Toopher and made the changes necessary to deploy to Heroku. The main changes are in the `sessions_controller` (authenticating) and `users_controller` (pairing). To provide a better user experience we refactored several methods to return JSON and offload processing to JavaScript. The client-side JavaScript is in `toopher.js`.

If the diff is too abstract, I also threw the [example application up on Heroku](https://rails-sample-app-with-toopher.herokuapp.com/). If your goal is simply to see how Toopher works, you'll be better served by the [Toopher Demo](https://demo.toopher.com/)--it is awesome!

# Highlights of the integration
Below is a high level overview of the changes. Many of the changes to existing files are very small, but I've tried to succinctly describe the rationale. I'll skip over any UI additions because your implenentation will likely be very different; for example, you likely have a loading spinner or library. If you're not sure why something changed, feel free to ask via comment here or on the GitHub repo.

## Changes to existing files
* `Gemfile` -- add the Toopher gem
* `app/models/user.rb` -- add Toopher bits to the user
* `config/routes.rb` -- wire up the Toopher endpoints
* `app/views/sessions/new.html.erb` -- add a unique ID that we can reliably hook a JavaScript listener to
* `app/controllers/sessions_controller` -- augment the existing authentication logic with Toopher for 2FA
* `app/controllers/users_controller` -- logic for pairing and unpairing
* `app/views/users/edit.html.erb` -- add Toopher to the user edit page

## New code
* `app/models/toopher_terminal.rb` -- Toopher terminals represent unique devices with which users connect
* `app/controllers/toopher_terminals_controller` -- logic to create new Toopher terminals for a user
* `app/assets/javascripts/toopher.js` -- client-side JavaScript to poll until a user responds to pairing and authentication requests
* `app/views/users/_toopher.html.erb` -- the Toopher configuration frontend
* `db/migrate/2031118000000_add_toopher_to_users.rb` -- add Toopher bits to the user
* `db/migrate/20131118000001_create_toopher_terminals.rb` -- create the ToopherTerminals table

# More details about specific pieces
The "Toopher Two Step" is simple: it's pairing and authenticating.

## Updates to the User model
We will update the User model (`user.rb`) to include a `toopher_pairing_id` and a utility method to determine if Toopher is enabled for the user. Here's a highlight of the changes:

```ruby
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

```ruby
class AddToopherToUsers < ActiveRecord::Migration
  def change
    add_column :users, :toopher_pairing_id, :string
  end
end
```

With these changes in place, we can add in logic to pair/remove Toopher
and conditionally authenticate login requests with Toopher.

## Adding Toopher to the UI
In this example, we added Toopher to the bottom of the user settings page using a partial view template (`_toopher.html.erb`):

```rhtml
<h3>Toopher</h3>

<% if current_user and current_user.toopher_enabled? %>
  <p>
  Thanks for setting up Toopher.
  </p>
  <form action="toopher_delete_pairing" method="post" id="unpair">
    <%= token_tag(nil) %>
    <input class="btn btn-large btn-primary" type="submit" value="Remove Toopher">
  </form>
<% else %>
  <p>
  Set up Toopher by generating a pairing phrase on your mobile device and inputting the phrase below.
  </p>
  <form action="toopher_create_pairing" method="post" id="pair">
    <%= token_tag(nil) %>
    <input type="text" name="pairing_phrase" id="pairing_phrase" placeholder="Pairing phrase" />
    <input class="btn btn-large btn-primary" type="submit" value="Pair with Toopher">
  </form>
<% end %>
```

This hooks into the user settings page (`edit.html.erb`):

```rhtml
<%= render 'toopher' %>
```

Whenever a user navigates to their settings, they will see the option to add or remove Toopher from their account. As you see in the Toopher partial above, to add Toopher the app will `POST` to `toopher_create_pairing` with a pairing phrase; to remove Toopher the app will `POST` to `toopher_delete_pairing`. Let's look at the pairing methods.

## The pairing methods 
We implement pairing and removing Toopher as methods on the user, so we update our routes and our controller. First, the routes (`routes.rb`):

```ruby
resources :users do
  member do
    get :following, :followers
    post :toopher_create_pairing, :toopher_delete_pairing
  end
end
```

... and the controller (`users_controller.rb`):

```ruby
def toopher_create_pairing
  @user = User.find(params[:id])
  pairing_phrase = params[:pairing_phrase]

  begin
    toopher = ToopherAPI.new(ENV['TOOPHER_CONSUMER_KEY'], ENV['TOOPHER_CONSUMER_SECRET'])
  rescue
    return toopher_setup_error
  end

  if not session[:toopher_pairing_start]
    begin
      pairing = toopher.pair(pairing_phrase, @user.email)
    rescue => e
      puts $!, $@
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
  @user = User.find(params[:id])
  if @user.update_attribute(:toopher_pairing_id, "")
    sign_in @user
    toopher_pairing_disabled
  else
    toopher_pairing_disabled_failed
  end
end
```

With this, users can add and remove Toopher. 

During pairing, we store details about the request in the session. The `toopher_create_pairing` endpoint is polled by client-side JavaScript until the user accepts the pairing or the pairing times out (no response for 60 seconds, in this case). Note that we have a 90 second timeout in JavaScript in case the server doesn't work as expected.

## Authentication changes
Your standard login might look something like this:

```ruby
class SessionsController < ApplicationController
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

The basic Toopher logic hooks in after the `user.authenticate` call. Basically, if the first factor passes, move on to authenticating with the second factor, like this (`sessions_controller.rb`):

```ruby
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

```ruby
def toopher_auth(user=nil)
  begin
    toopher = ToopherAPI.new(ENV['TOOPHER_CONSUMER_KEY'], ENV['TOOPHER_CONSUMER_SECRET'])
  rescue
    return toopher_setup_error
  end

  terminal_name = user.toopher_terminals.where(:cookie_value => cookies[:toopher]).first.terminal_name rescue nil
  if terminal_name.nil?
    return name_terminal
  end

  if not session[:toopher_auth_start]
    begin
      auth_status = toopher.authenticate(user.toopher_pairing_id, terminal_name, 'login', { terminal_name_extra: cookies[:toopher] })
    rescue => e
      puts $!, $@
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

## Next steps

This is a basic example. In a full-fledged implementation you should spruce up the UI and match your site's design. 

We also recommend providing a method to remove Toopher; this can be self-service or performed by an administrator who flips a bit in the database. Self-service options include a security question and answer or email reset.

## Running the app locally

Getting the sample app running locally is similar to starting any Rails project: clone the repo, install your gems, and migrate your database. One additional step is to configure Toopher. In this example, the Toopher client is instantiated using API credentials stored in the environment (`toopher = ToopherAPI.new(ENV['TOOPHER_CONSUMER_KEY'], ENV['TOOPHER_CONSUMER_SECRET'])`), so you need to set `TOOPHER_CONSUMER_KEY` and `TOOPHER_CONSUMER_SECRET`. 

```sh
git clone git@github.com:smholloway/sample_app_2nd_ed_with_toopher.git
cp config/database.yml.example config/database.yml
bundle install
bundle exec rake db:migrate
export TOOPHER_CONSUMER_KEY=xxx
export TOOPHER_CONSUMER_SECRET=xxx
rails server
```

Questions or comments? Let us know. We aim to please!

