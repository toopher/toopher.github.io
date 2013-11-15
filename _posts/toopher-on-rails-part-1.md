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

## Updates to the User model
We will update the User model (`user.rb`) to include a `toopher_pairing_id` and a utility method to determine if Toopher is enabled for the user. Here's a highlight of the changes:

    class User < ActiveRecord::Base
      attr_accessible :name, :email, :password, :password_confirmation, :toopher_pairing_id

      # ... removed for brevity ...

      def toopher_enabled?
        !toopher_pairing_id.blank?
      end

      # ... removed for brevity ...
      
    end

The `toopher_pairing_id` can be added with a migration like this:

    class AddToopherToUsers < ActiveRecord::Migration
      def change
        add_column :users, :toopher_pairing_id, :string
      end
    end

With these changes in place, we can add in logic to pair/remove Toopher
and conditionally authenticate login requests with Toopher.

## Adding Toopher to the UI
In this example, we will add Toopher to the bottom of the user settings page. We'll start by creating a partial view template (`_toopher.html.erb`):

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

This hooks into the user settings page (`edit.html.erb`):

    <% provide(:title, "Edit user") %>
    <h1>Update your profile</h1>

    <div class="row">
      <div class="span6 offset3">
        <h3>User Settings</h3>
        <%= form_for(@user) do |f| %>
          <%= render 'shared/error_messages', object: f.object %>

          <%= f.label :name %>
          <%= f.text_field :name %>

          <%= f.label :email %>
          <%= f.text_field :email %>

          <%= f.label :password %>
          <%= f.password_field :password %>

          <%= f.label :password_confirmation, "Confirm Password" %>
          <%= f.password_field :password_confirmation %>

          <%= f.submit "Save changes", class: "btn btn-large btn-primary" %>
        <% end %>

        <h3>Gravatar</h3>
        <%= gravatar_for @user %>
        <a href="http://gravatar.com/emails">change</a>
      </div>

      <div class="span6 offset3">
        <%= render 'toopher' %>
      </div>
    </div>

Now, whenever a user navigates to their settings, they will see the option to add or remove Toopher from their account. As you see in the Toopher partial above, to add Toopher the app will `POST` to `toopher_create_pairing` with a pairing phrase; to remove Toopher the app will `POST` to `toopher_delete_pairing`. Let's look at the pairing methods.

## The pairing methods 
We implement pairing and removing Toopher as methods on the user, so we update our routes and our controller. First, the routes:

    # routes.rb
    resources :users do
      member do
        get :following, :followers
      end

      post :toopher_create_pairing
      post :toopher_delete_pairing
    end

... and the controller (`users_controller.rb`):

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

With this, we can add and remove Toopher from a user. However, we are not done yet because we are blocking the server as we wait for the user to acknowledge the pairing. We'll discuss this more after looking at the first-pass on authentication updates.

## Authentication changes
Your standard login might look something like this:

    class SessionsController < ApplicationController
      def new
      end

      def create
        user = User.find_by_email(params[:session][:email].downcase)
        if user && user.authenticate(params[:session][:password])
          pass_login(user)
        else
          fail_login
        end
      end

      def destroy
        sign_out
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
    end

The basic Toopher logic hooks in after the `user.authenticate` call, like this:

    class SessionsController < ApplicationController
      def new
      end

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

      def name_terminal
        render 'new'
      end

      def clear_toopher_session_data
        session.delete(:toopher_auth_start)
        session.delete(:toopher_auth_id)
      end
    end

This implementation has a couple issues:

* It blocks the server, waiting for the `auth_status` to be acknowledged by the user or timeout
* it uses the user's reported `remote_ip`, which isn't very meaningful

The latter issue we will fix by letting the user name their terminal, which will be tracked in a cookie. We find "My old PC" more meaningful than "108.162.201.68" or "Chrome on Mac OS X", but the choice is yours.

The former we will fix by moving the timeout logic to client-side JavaScript where we will periodically poll the server for the request's status. Note: JavaScript polling is not the only way to solve this problem. Other languages bring their own concurrency primitives that can be leveraged; for example, similar code in Node.js would release control to the event loop and continue polling on the server.

With this, you will have a basic Toopher implementation working on your Rails site. In the next post we will discuss how to tidy things up.

Questions or comments? Let us know. We aim to please!
