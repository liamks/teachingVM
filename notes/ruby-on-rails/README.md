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