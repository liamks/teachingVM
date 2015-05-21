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