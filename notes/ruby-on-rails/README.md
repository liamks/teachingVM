# Creating a Twitter Clone with Ruby on Rails

These instructions assume that ruby, rails and psql are already setup. Some of these instructions are also specific to the VM that they are running on.

## Additional Documentation

* [Active Record](http://guides.rubyonrails.org/active_record_basics.html)
* [Form Helpers](http://guides.rubyonrails.org/form_helpers.html)

## Step 1: Creating your app

```bash
rails new twitter-clone --database=postgresql
```

After the above is done go into the rails app: `cd twitter-clone`. *All subsequent commands, and file paths, are relative to the twitter-clone folder (in it)*.

## Step 2: Configuring your app

Edit config/database.yml. You'll have the add the username, password and host lines.

 ```yml
default: &default
  adapter: postgresql
  encoding: unicode
  # For details on connection pooling, see rails configuration guide
  # http://guides.rubyonrails.org/configuring.html#database-pooling
  pool: 5
  username: rails
  password: rails
  host: localhost
```

Then run:

```
rake db:setup
rake db:migrate
```

## Step 3: Test Your App works (and run the server)

```
bin/rails server -b 0.0.0.0
```
### Viewing your Ruby on Rails App

On your machine (not the VM) open a browser and go to `http://127.0.0.1:3001/`. You should see something that looks like:

![Rails Getting Started Page](http://guides.rubyonrails.org/images/getting_started/rails_welcome.png)

## Step 4: Replacing the default landing page

run: `rake routes`, this will list all the URLs associated with your app (there should be none at the point).

Now let's create a controller, and a view that will become our landing page.

```bash
rails generate controller Welcome index
```

That command will create a controller called `WelcomeController` with a single method `index`. It will also create a view called at `app/views/welcome/index.html.erb`. Additionally it will create a URL `welcome/index` that maps to the controller's `index` method. Our landing page will be very basic for now, it will just say welcome and display the current timestamp. First open `app/controllers/welcome_contoller.rb` and fill in the `index` method so it looks like:

```ruby
class WelcomeController < ApplicationController
  def index
    @date = DateTime.now
  end
end
```

`@date` is an instance method on the `WelcomeController`. Now let's update `app/views/welcome/index.html.erb` to show that date:

```erb
<h1>Welcome</h1>
Today's date is <%= @date %>
```

Now we need to tell our app that the `index` method in our `WelcomeController` should also map to the root URL (aka `/`). Open up `config/routes.rb` and uncomment the following line (one starting with root):

```ruby
# You can have the root of your site routed with "root"
root 'welcome#index'
```

When you reload your rails app (assuming your rails app is still running), you should see the new landing page with the current time.

## Step 5: Creating The User Model

Users are key component to just about any app, and especially Twitter. Let's use Rails' generate to create our model and migration file. The migration file is what Rails uses to actually create a table in the database.

```
rails generate model User name handle
```

The above will create a model called `User` and will create a migration file in `db/migrate`. The migration's file name should start with a timestamp and end with `_create_users.rb`.  The migration will create a table with 5 columns, two of which we defined explicitely: name and handle. The third is `id` which is done implicitly by rails. Finally the 4th and 5th are `created_at` and `updated_at` timestamps. 

To actually create the table run:

```
rake db:migrate
```

Before we proceed let's create a couple users using Rails' console ([a REPL](http://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)). To start the console run:

```
rails console
```
The following commands are done inside the console:

```ruby
julia = User.new name: "Julie Newmar", handle: "julie"
julia.save
```
The first line creates the user, in memory only, and the second line saves them to the database. We can do those two actions with a single method (`create`):

```ruby
frank = User.create name: "Frank Smith", handle: "franky"
```

To exit the console type `exit`. We've now created a User model and two users!

## Step 6: Creating the Tweet Model

Now that we have a user model, we need to create a Tweet model. That is each user should be able to create many tweets. This model should be able to encapsulate the tweet's content and the tweet's relationship to a user. First let's create the model.

```
rails generate model Tweet content:text
```

Open up the migration in `db/migrate` - the migration file should end with `_create_tweets.rb` and update it so it looks like: 

```ruby
class CreateTweets < ActiveRecord::Migration
  def change
    create_table :tweets do |t|
      t.text :content
      t.references :user, index: true
      t.timestamps null: false
    end
  end
end
```

`t.references` creates a column called `user_id` which associated each tweet with a single user. Now run:

```
rake db:migrate
```

That will create the `tweets` table in the database. Let's open up the console and create some tweets. To open the console type `rails console`. In the console:

```rails
t1 = Tweet.create content: "First tweet!", user_id: 1
t2 = Tweet.create content: "second tweetzzz", user_id: 2
Tweet.all
exit
```

The first two lines create two new tweets and associated them with user 1 and user 2. The third command returns all the tweets in the database.

Now we need to update both the User and Tweet model to let Rails know they are related - this will make our life easier.

Open `app/models/tweet.rb` and update it to look like:

```ruby
class Tweet < ActiveRecord::Base
  belongs_to :user
end
```

the `belongs_to` method tells Rails that the Tweet model is associated with the User model. Now open `app/models/user.rb` and update it to look like:

```ruby
class User < ActiveRecord::Base
  has_many :tweets
end
```

This is the other side of the relationship which tells Rails that each user might be associated with 0 or more tweets. Defining these relationships provides us with several convenience methods for fetching a user's tweets and creating new ones. Now let's explore what new syntatic sugar defining these relationships gives us. Open the console (`rails console`).

```ruby
julia = User.first
julia.tweets
```

By defining the relationship, we can call `tweets` on the instance of User (in this case julia) and get all of her tweets. Furthermore we can create tweets for julia much easier:

```ruby
julia.tweets.create content: "A tweet by julia"
```

`julia.tweets.create` will automatically add julia's `user_id` to the created tweet.

## Step 7: Creating the Tweet Controller

Now that we've a user model and a tweet model it's time to create a tweet controller so we can actually view tweets in a browser. Let's use one of Rails' generators to create our controller, views and routes. 

```bash
rails generate scaffold_controller Tweet
```

Now open `config/routes.rb` and add the line `resources :tweets` - this will create routes that will map to the actions in the TweetsController.

By calling `scaffold_controller`, instead of just `controller`, it will fill in the `TweetsController` with a set of default methods, and code, to create, read, update, and view tweets. If we just ran the command with `controller` it would create an empty controller. If we now run:

```
rake  routes
```

You should see 8 new routes that are mapped to "actions" (methods) in the `TweetsController`. You should see the following:

```
       Prefix Verb   URI Pattern                Controller#Action
welcome_index GET    /welcome/index(.:format)   welcome#index
         root GET    /                          welcome#index
       tweets GET    /tweets(.:format)          tweets#index
              POST   /tweets(.:format)          tweets#create
    new_tweet GET    /tweets/new(.:format)      tweets#new
   edit_tweet GET    /tweets/:id/edit(.:format) tweets#edit
        tweet GET    /tweets/:id(.:format)      tweets#show
              PATCH  /tweets/:id(.:format)      tweets#update
              PUT    /tweets/:id(.:format)      tweets#update
              DELETE /tweets/:id(.:format)      tweets#destroy
```

If you start the rails server (`rails server`) you can visit some of those new pages. Specifically you can go to `http://localhost:3001/tweets` to view all the tweets. If you visit the page you'll notice that you can't actually see the tweets, you'll have to make an update to this view. All of tweet views are in `app/views/tweets` - open `app/views/tweets/index.html.erb` and update the `<table>` to look like:

```erb
<table>
  <thead>
    <tr>
      <th>Tweet</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @tweets.each do |tweet| %>
      <tr>
        <td><%= tweet.content %></td>
        <td><%= link_to 'Show', tweet %></td>
        <td><%= link_to 'Edit', edit_tweet_path(tweet) %></td>
        <td><%= link_to 'Destroy', tweet, method: :delete, data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

By adding `tweet.content` we're telling the view to display each tweet's content. If you refresh `http://localhost:3001/tweets` you should now see the content of each tweet. Now click on a tweet - the same issue is present with this page, the contents of the tweet aren't showing. Let's update the view - open `app/views/tweets/show.html.erb` and update it to:

```erb
<p id="notice"><%= notice %></p>

<p>
  <%= @tweet.content %>
</p>

<%= link_to 'Edit', edit_tweet_path(@tweet) %> |
<%= link_to 'Back', tweets_path %>
```

If you reload the page it should now show the tweet's content. Now click `Edit` and you'll see a form without any inputs. We'll have to add those. If you open either `app/views/tweets/edit.html.erb` or `app/views/tweets/new.html.erb` you'll notice that they both include the following line:

```erb
<%= render 'form' %>
```

`'form'` is a partial - a partial is a way of creating a small resuable view that can be rendered into multiple views. By convention, the file name of the partial is prefixed with an underscore - `app/views/tweets/_form.html.erb`. By updating the `_form.html.erb` partial we should see updates to both the edit and new pages that include it. We'll go ahead and change it to:

```erb
<%= form_for(@tweet) do |f| %>
  <% if @tweet.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@tweet.errors.count, "error") %> prohibited this tweet from being saved:</h2>

      <ul>
      <% @tweet.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= f.label :content, "Tweet" %> <br>
    <%= f.text_field :content %>
  </div>
  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>
```

If you reload the edit page you should now see the content of the tweet in the text field. If you click save you should see the following error:

![ForbiddenAttributesError](/notes/ruby-on-rails/ForbiddenAttributesError.png?raw=true "ForbiddenAttributesError")

## Part 2 (Second Workshop)

### Step 1. 

To fix the `ForbiddenAttributersError` Open `app/controllers/tweets_controller.rb` and change line 72 to: `params.require(:tweet).permit(:content)`. This creates an explicit whitelist on what attributes of the `Tweet` model can be updated.

### Step 2. 

Restart your rails app - you should be able to create new tweets, and update existing tweets.
### Step 3. 

Adding user authentication with [Devise](https://github.com/plataformatec/devise). I highly recommend always using a well respected 3rd party library/gem/package for doing authentication. Not only is it a lot of work to build a fully functioning authentication system, but there are many things that you can do wrong that can lead to security issues. Only build your own for educational purposes, for anything you plan on using in a production setting use a 3rd party library - we will use Devise.

### Step 4. 

Add `gem 'devise'` to your Gemfile, in the root of your app

### Step 5 

Run `rails generate devise:install` to install Devise on your app.

1. In `config/environments/development.rb` add `config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }`
2. In `app/views/layouts/application.html.erb` add `<p class="notice"><%= notice %></p><p class="alert"><%= alert %></p>` to line 10.
3. run `rails g devise:views` to add Devise views to your app (e.g. login, logout, register, etc).

### Step 6. 

Run `rails generate devise User` to add devise to our User model (Open the new migration file that was created to see what's being added to the User model/table)

### Step 7. 

*before* running the migration you'll have to delete the existing users and their tweets. If you don't delete the tweets and users the above migration will fail as it includes some constraints that will be invalid in your existing data. Log into psql 

`psql --username=rails -d twitterclone_development` (password is rails) and run `delete from tweets;` and `delete from users;`

### Step 8. 

Now run `rake db:migrate`

### Step 9.

Congratulations, you've added Devise to your app! Users can now register, login, logout, reset their passwords and more. To check what URLs are associated with login, register, logout, etc run `rake routes`. Devise should have added:

```
                  Prefix Verb   URI Pattern                    Controller#Action
        new_user_session GET    /users/sign_in(.:format)       devise/sessions#new
            user_session POST   /users/sign_in(.:format)       devise/sessions#create
    destroy_user_session DELETE /users/sign_out(.:format)      devise/sessions#destroy
           user_password POST   /users/password(.:format)      devise/passwords#create
       new_user_password GET    /users/password/new(.:format)  devise/passwords#new
      edit_user_password GET    /users/password/edit(.:format) devise/passwords#edit
                         PATCH  /users/password(.:format)      devise/passwords#update
                         PUT    /users/password(.:format)      devise/passwords#update
cancel_user_registration GET    /users/cancel(.:format)        devise/registrations#cancel
       user_registration POST   /users(.:format)               devise/registrations#create
   new_user_registration GET    /users/sign_up(.:format)       devise/registrations#new
  edit_user_registration GET    /users/edit(.:format)          devise/registrations#edit
                         PATCH  /users(.:format)               devise/registrations#update
                         PUT    /users(.:format)               devise/registrations#update
                         DELETE /users(.:format)               devise/registrations#destroy
```

Please take note of the first column `Prefix`, you can use these to easily create links in your app using "link helpers" (examples in the next step).

### Step 10. Creating a top menu with login/logout & register links

In `app/views/layouts/` create a file called `_top_menu.html.erb`, this file is what Ruby on Rails calls a "partial", it will hold our menu. We will create a very basic menu (we will improve it in a later step). In `_top_menu.html.erb` add the following code:

```erb
<ul>
  <% if user_signed_in? %>
  
  <li>
    <%= link_to current_user.email, '/' %>
  </li>
  <li>
    <%= link_to "Sign Out", destroy_user_session_path, method: "DELETE" %>
  </li>

  <% else %>

  <li>
    <%= link_to "Sign In", new_user_session_path %>
  </li>
  <li>
    <%= link_to "Register", new_user_registration_path %>
  </li>

  <% end %>
</ul>
```

Note `link_to` which is a "link helper". You should recognize `new_user_session_path` as `new_user_session` is in the `Prefix` column. By combining a "Prefix" (e.g. `new_user_session`) with `_path` (e.g. `new_user_session_path) you get a method that returns the path for that route (useful for links).

Finally, add `<%= render 'layouts/top_menu' %>` to `app/views/layouts/application.html.erb` right after the line with the opening `<body>` tag. The `render` statement is what actually includes the partial into the main application's layout. Reload your app and you should see the links "Sign In" and "Register" at the top of the page. Go ahead and click register to create a new user.

### Step 11. Update the TweetsController

Now we need to update the TweetsController to only show tweets of the logged in user, and ensure that only a logged in user can create tweets. Add the following to `TweetsController` (`app/controllers/tweets_controllers.rb`) after line 2:

```ruby
  before_action :authenticate_user!, except: [:index, :show]
```

That will ensure that the user *has* to be authenticated before every action except index and show. Now lets show the user's tweets if they're looking at the index page. If they're not logged in we'll show everyone else's tweets. In the `TweetsController` update the `index` method (aka action) to look like:

```ruby
def index
  if user_signed_in?
    @tweets = current_user.tweets
  else
    @tweets = Tweet.all
  end
end
```

### Step 12. Add Bootstrap to project

We will be using the `bootstrap-sass` gem to ad Twitter's Bootstrap CSS library. Bootstrap gives us a collection of CSS that helps our app look great without needing to hire a designer! Add `gem 'bootstrap-sass', '~> 3.3.4'` to the `Gemfile` right before the line that has `gem 'sass-rails', '~> 5.0'`.

Now run `bundle install`.

Now rename `app/assets/stylesheets/application.css` to `app/assets/stylesheets/application.scss` (we've only changed the extension). Then open `app/assets/stylesheets/application.scss` and delete the contents of the file and replace them with:

```css
@import "bootstrap-sprockets";
@import "bootstrap";
@import "bootstrap/theme";
```

If you restart the server, and reload the page, you'll notice that the fonts and some of the styles have changed. We have successfully added [Bootstrap](http://getbootstrap.com/). Let's now proceed to adding bootstrap classes and making our clone look a little nice.

#### 1. Update The Top Menu

Start by opening `app/views/layouts/_top_menu.html.erb` and making updates so the file should look like the folllowing:

```erb
<header class="navbar navbar-inverse navbar-static-top">
  <div class="container">
    <div class="navbar-header">
      <%= link_to "Twitter Clone", root_path, class: "navbar-brand" %>
    </div>
    <nav>
      <ul class="nav navbar-nav navbar-right">
        <% if user_signed_in? %>
        
        <li>
          <%= link_to current_user.email, '/' %>
        </li>
        <li>
          <%= link_to "Sign Out", destroy_user_session_path, method: "DELETE" %>
        </li>

        <% else %>

        <li>
          <%= link_to "Sign In", new_user_session_path %>
        </li>
        <li>
          <%= link_to "Register", new_user_registration_path %>
        </li>

        <% end %>
      </ul>
    </nav>
  </div>
</header>
```

Again we're making heaving use of "link helpers" and we have some conditional logic for whether the use is signed in or not (`user_signed_in?`, a method provided by Devise).

#### 2. Update The Flash Message

Replace line 11 on `app/views/layouts/application.html.erb` with:

```erb
<%= render 'layouts/flash_messages' %>
```

Now we need to create the partial (resuable view) that we've just referenced. In `app/views/layouts/` add a file named `_flash_message.html.erb` and fill it with:

```erb
<div class="container">
  <% if notice %>
  <p class="alert alert-success"><%= notice %></p>
  <% end %>

  <% if alert %>
  <p class="alert alert-danger"><%= alert %></p>
  <% end %>
</div>
```

3. Update Login/register

We'd like to jazz up the login and registration page.

Add a `authentication.scss` to `app/assets/stylesheets/` and inside it add:

```css
.user-auth-box-outer{
  text-align: center;
}

.user-auth-box{
  width: 400px;
  text-align: left;
  margin: 10px auto;
  padding: 5px;
  background-color: #eee;
  border: 1px solid #ddd;
}
```

Now, import this new file into `application.scss`:

```css
@import 'authentication'
```

Finally, surround the existing content in `app/views/devise/sessions/new.html.erb` with:

```html
<div class="user-auth-box-outer">
  <div class="user-auth-box">
    <!-- EXISTING CONTENT -->
  </div>
</div>
```

### Step 13. Deploy to Heroku

Once you've created an account with Heroku, create a new app. In the deploy tab click on Github and authorize your account to be used by Heroku. Now you can deploy by simply clicking a button, or you can deploy everytime someone merges into master.




