# Creating a blog with the MEAN Stack

* M - MongoDB
* E - ExpressJS
* A - AngularJS
* N - NodeJS

## Assumptions

All terminal commands are assumed to be entered within the `mean-blog` folder on your VM.

## Step 1 - Getting Started

Create a folder called `mean-blog` in the folder that is shared with your Virtual Machine (VM). 

Now, will need to create a `package.json` file to allow use to install node packages. Fortunately NPM has a wizard that will create the `package.json` file for us. In your VM navigate to the folder that you've just created and enter:

```
npm init
``` 

NPM will ask you a series of questions, to accept the default responses just hit enter. Finally, they'll show you a complete version of your `package.json` file, type `yes` to accept it.

As part of the `package.json` file it asked for the app's entry point (default is `index.js`) - let's go ahead and create that file (`index.js`) and fill it with a "hello world":

```js
console.log('Hello World');
```

In your VM, in the `mean-blog` folder, enter `node index.js` into the console - this will run your node app! Congratulations, you've created a NodeJS app. 

## Step 2 - Creating the ExpressJS App

In the terminal:

```
npm install express --save --no-bin-links
```

This install's express and the `--save` flag adds express as a dependency to your `package.json` file while `--no-bin-links` tells npm not to use symbolic links which don't work in shared folders on VM.

Now replace the contents of `index.js` with:

```javascript
var express = require('express');
var app = express();
var PORT = 5000;

app.get('/', function(req, res){
  res.send('Hello World');
});

app.listen(PORT, function(){
  console.log('server has started.');
});
```

The first line loads the express package, the second line creates the express app. In the 3rd line we define the port that our express server will run on. 

```javascript
app.get('/', function(req, res){
  res.send('Hello World');
});
```

The above maps the route (`'/'`) to an annonymous function (`function(req, res){}`). The annonymous function takes two parameters, `req` and `res`, both of which are passed in by express. `req` points to the request object that has all the information about the incoming request. `res` is the response object that has a number of functions that will help us respond to the request. In the above code we use the response's `send()` function to respond with a simple string (`'hello world'`). 

Now if we want to create an API for our app, we'll need to return JSON. Let's create an endpoint that returns JSON, instead of just a string. Before the `app.listen` line add the following:


```javascript
app.get('/api/', function(req, res){
  res.json({hello : 'World'});
});
```

After restarting your server, and hitting `/api/` you should get a JSON response.

## Step 3 - Creating our first micro-app

Express is extremely minimalist, but it's design encourages creation of smaller express apps (micro-apps) that we can "mount", or attach to our main express app. Each micro-app could in theory run on it's own, it's self contained and modular.

Create a folder called `microapps` and in it create a file called `blogs.js` with the contents:

```javascript
var express = require('express');
var router = express.Router();

router.get('/', function(req, res){
  var blogEntries = [
    {id: 1, title: 'First entry'},
    {id: 2, title: 'About bicycles'},
    {id: 3, title: 'React vs Angular?'}
  ];

  res.json({entries: blogEntries});
});

module.exports = router;
```

Now that we created a our first API endpoint - which returns a list of blog entries. The very last line is how we export code in Node. Now the router can be used by code in other files. Let's "mount" it to our main node app. Update `index.js` to look like:

```javascript
var express = require('express');
var app = express();
var PORT = 5000;

app.get('/', function(req, res){
  res.send('Hello World');
});

// Micro-apps
var blogs = require('./microapps/blogs.js');

app.use('/api/blogs', blogs);


// Start server
app.listen(PORT, function(){
  console.log('server has started.');
});
```

We import the router using the `var blogs = require('./microapps/blogs.js');` and finally we mount the `blogs` app to `/api/blogs`. In otherwords, any route in blogs will have a path that is prefixed with `/api/blogs`.