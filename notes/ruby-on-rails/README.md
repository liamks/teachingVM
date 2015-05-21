# Creating a Twitter Clone with Ruby on Rails

These instructions assume that ruby, rails and psql are already setup. Some of these instructions are also specific to the VM that they are running on.

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

Before we proceed let's create a couple users using Rails' console (a REPL)[http://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop]. To start the console run:

```
rails console
```
The following commands are done inside the console:

```
julie = User.new name: "Julie Newmar", handle: "julie"
julia.save
```
The first line creates the user, in memory only, and the second line saves them to the database. We can do those two actions with a single method (`create`):

```
frank = User.create name: "Frank Smith", handle: "franky"
```

To exit the console type `exit`. We've now created a User model and two users!