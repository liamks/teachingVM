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
var PORT = process.env.PORT || 5000;

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

The 3rd line assigns either PORT from your OSs environment variables or 5000 if it doesn't exist on the environment.

The above maps the route (`'/'`) to an annonymous function (`function(req, res){}`). The annonymous function takes two parameters, `req` and `res`, both of which are passed in by express. `req` points to the request object that has all the information about the incoming request. `res` is the response object that has a number of functions that will help us respond to the request. In the above code we use the response's `send()` function to respond with a simple string (`'hello world'`). 

Now if we want to create an API for our app, we'll need to return JSON. Let's create an endpoint that returns JSON, instead of just a string. Before the `app.listen` line add the following:


```javascript
app.get('/api/', function(req, res){
  res.json({hello : 'World'});
});
```

After restarting your server, and hitting `/api/` you should get a JSON response.

## Step 3 - Creating our first router

Express is extremely minimalist, but it's design encourages creation of smaller routers that we can "mount", or attach to our main express app. Each router could in theory run on it's own, it's self contained and modular.

Create a folder called `routers` and in it create a file called `blogEntries.js` with the contents:

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
var PORT = process.env.PORT || 5000;

app.get('/', function(req, res){
  res.send('Hello World');
});

// Micro-apps
var blogEntries = require('./routers/blogEntries.js');

app.use('/api/blog-entries', blogEntries);


// Start server
app.listen(PORT, function(){
  console.log('server has started.');
});
```

We import the router using the `var blogs = require('./routers/blogEntries.js');` and finally we mount the `blogEntries` router to `/api/blog-entries`. In otherwords, any route in the `blogEntries` router will have a path that is prefixed with `/api/blog-entries`.

We can test this new API end point by going to `'http://127.0.0.1:5001/api/blog-entries/'` in our browser. However, entering a URL in your browser will only perform a `GET` request. As we implement other endpoints, that use other HTTP verbs (e.g. `POST`, `PUT, `DELETE`), we'll need more than just the browser's default `GET` requests. To accomplish this we'll download "Postman - REST client", a chrome plugin.

Try entering `'http://127.0.0.1:5001/api/blog-entries/'` in Postman, it should return nicely formatted JSON.

## Step 4 - Creating More Endpoints

In the last step we setup a single API endpoint in our `blogEntries` router, it returned a hard-coded list of blog entries. However, in our real app we'll be storing entries in a database (MongoDB).

* Showing a single blog entry (`GET /:id`)
* Creating a blog entry (`POST /`)
* Updating a blog entry (`PUT /:id`)
* Deleting a blog entry (`DELETE /:id`)

In brackets we have the HTTP verb, followed by the path. To implement the an endpoint that can accept `POST`ed data, we'll need to add another node module `'body-parser'`. To install `'body-parser'`:

```
npm install body-parser --save
```

The `--save` flag tells npm to add the `body-parser` dependency to your `package.json` file. To require the package in your `index.js` file, add the following (after line 1):

```javascript
var bodyParser = require('body-parser');
```

Now you have to tell express to use the `body-parser` (specifically to parse JSON), to do so add the following to `index.js` on line 6:

```javascript
app.use(bodyParser.json());
```

Now we can add the endpoint for creating a blog entry to `blogEntries.js` (before the last line):

```javascript
router.post('/', function(req, res){
  // Insert code to create a new blog post
  res.json(req.body);
});
```

Step 5 - Adding the remaining endpoints

Once you've added the remaining endpoints to `blogEntries.js` it should look something like:

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

router.get('/:id', function(req, res){
  // Insert code to fetch a single entry by id
  res.json({id : req.params.id});
});

router.post('/', function(req, res){
  // Insert code to create a new blog post
  res.json(req.body);
});

router.put('/:id', function(req, res){
  // Insert code to update a single entry by id
  res.json(req.body);
});

router.delete('/:id', function(req, res){
  // Insert code to delete a single entry
  res.json({id: req.params.id});
});

module.exports = router;
```

Of course they don't do much, except return their input, but we can fill them in after we've added MongoDB. `req.params.id` will take the `id` that was specified in the path (e.g. `'/:id'`) and `req.body` will be the content of your `POST` or `PUT`s request.

Step 6 - Adding MongoDB

At the moment our API has no way of storing data for the long term. To save data we'll use MongoDB and to use it in NodeJS we'll use the `Mongoose` package. To install Mongoose run:

```
npm install mongoose --save --no-bin-links
```

In the root of our app we'll create a new folder called `models` and in it we'll create a file called `entry.js`. `entry.js` will be used to store any functions that will access MongoDB. In `entry.js` enter:

```javascript
var mongoose = require('mongoose');
var uriString = process.env.MONGOLAB_URI ||
                process.env.MONGOHQ_URL ||
                'mongodb://localhost/blog';

var EntrySchema = new mongoose.Schema({
  title : { type : String, required : true },
  body : String,
  createdAt : { type : Date, default : Date.now },
  updatedAt : { type : Date, default : Date.now },
  published : Boolean
});

var Entry;

function ModelActions(){
  mongoose.model('Entry', EntrySchema);
  Entry = mongoose.model('Entry');
  mongoose.connect(uriString);
  return ModelActions;
}

ModelActions.find = function(id, cb){
  Entry.findById(id).exec(cb);
};

ModelActions.create = function(entry, cb){
  var entry = new Entry(entry);

  entry.save(cb);
};

ModelActions.update = function(id, entry){
  Entry.findByIdAndUpdate(id, {$set : entry}, cb);
};

ModelActions.destroy = function(id){
  Entry.findByIdAndRemove(id, cb);
};

ModelActions.all = function(){
  Entry.find({}, cb);
};

module.exports = ModelActions();
```

In line 2 we're assigning a connection string, a string that mongoose will use to connect to MongoDB. That string is either the hard-coded string `'mongodb://localhost/blog'` that will be used in our VM, or the environment variables that we'll use when we deploy to Heroku.

After defining the connection string we define the schema for our blog entry document. It *has* to have a title (required), and it will have a `body`, `createdAt`, `updatedAt` and `published` attributes. For `createdAt` and `updatedAt` we've defined default values of the current time.

In the above only `.find()` and `.create()` are actually functional. In `Entry.findById(id).exec(cb);` you can see the separation between creating a query `.findById(id)` and executing it `.exec(cb)`. By separating the two parts you can chain together many query functions before finally executing the chain of queries.

## Step 7 - Connecting our model to the router

Towards the top of 'blogEntries.js' add:

```javascript
var Entry = require('../models/entry');
```

Then update the "show one entry" and the "create one entry" routes to the following:

```javascript
var express = require('express');
var router = express.Router();
var Entry = require('../models/entry');

router.get('/', function(req, res){
  Entry.all(function(err, entries){
    if (err){
      res.status(400).json(err);
    } else {
      res.json(entries);
    }
  });
});

router.get('/:id', function(req, res){
  // Insert code to fetch a single entry by id
  Entry.find(req.params.id, function(err, entry){
    if (err){
      res.status(400).json(err);
    } else {
      res.json(entry);
    }
  });
});

router.post('/', function(req, res){
  // Insert code to create a new blog post
  Entry.create(req.body, function(err, entry){
    if (err){
      res.status(400).json(err);
    } else {
      res.json(entry);
    }
  });
});

router.put('/:id', function(req, res){
  // Insert code to update a single entry by id
  var entry = req.body;
  entry.updatedAt = Date.now();

  Entry.update(req.params.id, entry, function(err, e){
    if (err){
      res.status(400).json(err);
    } else {
      res.json(e);
    }
  });
});

router.delete('/:id', function(req, res){
  // Insert code to delete a single entry
  Entry.delete(req.params.id, function(err, entry){
    if (err){
      res.status(400).json(err);
    } else {
      res.json(entry);
    }
  });
});

module.exports = router;
```

In the above code we've completed 5 routes that allow us to: 

1. view many entries (`GET /`)
2. view a single blog entry (`GET /:id`)
3. create an entry (`POST /`)
4. update an entry (`PUT /:id`)
5. destroy an entry (`DELETE /:id`)

You'll notice that each function on our model (`all`, `find`, `create`, `update`, `delete`) accepts a callback that takes two parameters: 1) the return value from the query and 2) an error. To handle a potential error we check if the error is defined and if it is we change the status code to 400 and return a JSON version of the error. Alternatively if their is no error we return the result of the query in JSON.

## Step 8 - Deploying to Heroku

In the root of your app create a file called `Procfile` and in it:

```
web: node index.js
```

The `Procfile` tells Heroku how to run our app.

1. Create a free Heroku account if you haven't already: https://signup.heroku.com/signup/dc
2. Download Heroku Toolbelt https://devcenter.heroku.com/articles/getting-started-with-nodejs#set-up
3. Run `heroku login`

Assuming `heroku login` worked, it should return `Authentication successful.`. I'm also assuming your app is a git repo. Next enter the following commands one at a time:

```
heroku create
heroku addons:create mongolab
git push heroku master
heroku open
```

`heroku create` adds the heroku origin to your git repo. `heroku addons:create mongolab` adds mongodb to our app. `git push heroku master` deploys our app and finally `heroku open` opens it in a browser.

## MEAN Workshop Part 2

## step 8 - Setting up our static assets app

We'll now need to setup our express app to serve an `index.html` file that will run our AngularJS app in a user's browser. To do so we'll need to make several additions to the `index.js` file:

After the line with `app.use(bodyParser.json());` add:

```javascript
app.use(express.static('public'));
```

This allows our app to serve static files (e.g. JavaScript, CSS, images) from a folder called `public` (we'll need to create that folder in the next step).

Then add the following (after the line with `app.use('/api/blog-entries', blogEntries);`):

```javascript
app.get('/*', function(req, res){
  res.sendFile(__dirname + '/views/index.html');
});
```

The above route will match any url that hasn't already been matched and it will serve the client an `index.html` file.

Now that we've made changes to `index.js` we'll need to create the folders and files that the above code expects. Go ahead and create a folder called `views` and in it create a file called `index.html` with the following content:

```html
<!DOCTYPE html>
<html>
<head>
  <title></title>
  <script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.0/angular.min.js"></script>
  <script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.0/angular-route.min.js"></script>

  <script type="text/javascript" src="/js/app.js"></script>

  <link rel="stylesheet" type="text/css" href="/css/main.css">
</head>
<body>
  Hello World
</body>
</html>
```

This is the start of our AngularJS app. We're include AngularJS, as well as Angular's router. Finally we're including a `app.js`, and `main.css`.

Now create a folder called `public` and put two folders in it: `css` and `js`. In `css` create a file called `main.css` and in `js` create a file called `app.js`.

## Step 9 - Initializing our AngularJS app.

In `index.html` update the `<html>` tag to: `<html ng-app="meanBlog">`. This informs AngularJS that our Angular app will be inside of the `<html>` DOM element. Now update the `<body>` tag to:

```html
<body ng-init="msg='World'">
  Hello {{msg}}
</body>
```

The above code will allow us to test if we've created our Angualr app correctly. In the `ng-init` we're creating a variable called `msg` and assigning the string `'World'` to it. Finally we reference the `msg` variable by using Angular's interpolation (the curly brackets).

Now we need to initialize our app in `app.js` add:

```javascript
var app = angular.module('meanBlog', []);
```

If you restart your express server and go to http://127.0.0.1:5001 you should see "Hello World" if everything is working correctly.

To get a feel for some of the things we can do without writing any JavaScript let's make some changes to the `<body>` tag again and update it to:

```html
<body ng-init="counter=0;">
  <button ng-click="counter=counter+1;">Increment counter</button>
  <div>count is at: {{counter}}</div>
</body>
```

The above code initializes a `counter` variable to 0. `ng-click` will run the code `counter=counter+1` everytime the button is clicked. Finally AngularJS will update the view to show the latest value of the `counter` variable. Refresh your browser and give it a try.

Let's do one more example before we move on:

```html
<body ng-init="shouldShow=false;">
  <button ng-click="shouldShow=!shouldShow">Toggle Hidden Hello</button>
  <div ng-show="shouldShow">Hidden Hello</div>
</body>
```

The above code initializes a variable called `shouldShow` with a boolean value of false. When the button is clicked `ng-click` will run `shouldShow=!shouldShow` which will toggle `shouldShow` from false to true. `ng-show` will only show the current DOM element that is on if `shouldShow` is true. Reload the page and try it out.

`ng-init`, `ng-click` and `ng-show` are all examples of what AngularJS calls directives. According to Angular documentation directives are a way of "teching html to do new things". The creators of Angular noticed that there are a certain set of actions that just about every app needs to do do the DOM (e.g. hide and show elements, responds to clicks and so on). Directives make this easy to do.

## Step 9 - The controller

We will now create our first controller, it will be used to show a list of blog entries. In `app.js` add the following:

```html
app.controller('EntriesIndexCtrl', 
function($scope){

  $scope.entries = [
    {title : 'First entry', body : 'This is my first blog post'},
    {title : 'Second', body: 'A classic example of another....'},
    {title : 'Oldy', body: 'How could we not forget writing...'}
  ];
  
});
```

This creates a controller called `EntriesIndexCtrl` and in it we assign an array of 3 blog entries to a variable called `$scope.entries`. `$scope` is special - anything on the `$scope` can be accessed in view/template. We'll update `<body>` again and replace it with:

```html
<body ng-controller="EntriesIndexCtrl">
  <div>{{entries}}</div>
</body>
```

The above code says that anything in the `<body>` will be controlled by the `EntriesIndexCtrl` and will have access to anything on the controller's `$scope`. In the `<div>` we can reference `entries` since it's on the `$scope` variable. Reload the page and you should see an array of objects - not verry pleasing to the eye! Let's update the `<body>` again with:

```javascript
<body ng-controller="EntriesIndexCtrl">
  <div class="blog-entry" ng-repeat="entry in entries">
    <h2>{{entry.title}}</h2>
    <p>{{entry.body}}</p>
  </div>
</body>
```

`ng-repeat` is a nother directive that will 1) render the DOM element it is on once for every element in the array it's given and 2) allow us to access elements in the array (e.g. `entry.title`).

## Step 10 - Multiple Views & Controllers

Currently our app only has a single controller and we have no way of going from one view in the app, to another. To do so, we'll have to use Angular's router to map specific URLs to specific controller/view pairs. The first step is updating the `<body>` one last time to:

```html
<body>
  <div ng-view></div>
</body>
```

Angular's router will, based on the URL, find a specific view and place it inside of `<div ng-view></div>` and associate a specific controller with that view.

Let's setup the mapping for two views. Update `apps.js` (just the first line) to:

```html
var app = angular.module('meanBlog', ['ngRoute']);

app.config(function($routeProvider){
  $routeProvider
    .when('/', {
      templateUrl : 'templates/home.html',
      controller : 'EntriesIndexCtrl'
    })
    .when('/entry/:id', {
      templateUrl : 'templates/entry.html',
      controller : 'EntriesShowCtrl'
    })
    .otherwise('/');
});
```

In the first line we declare that our app is dependent on `ngRoute`. Then we call `.config` on our app to configure our app's routes. The `$routeProvider` as a function `.when()` that maps a url (`'/'`) with a template and controller. 

Now in the `public` folder create a new folder called `templates` and create two files in it called: `home.html` and `entry.html`

In `home.html`:
```html
<div class="blog-entry" ng-repeat="entry in entries">
  <h2>{{entry.title}}</h2>
  <p>{{entry.body}}</p>
</div>
```

In `entry.html`:
```html
{{entry}}
```
Now that we've made both of the templates that we specified in `app.config()` we'll need to create the `EntriesShowCtrl` (the `EntriesIndexCtrl` already exists). Add the following to `app.js`

```javascript
app.controller('EntriesShowCtrl', 
function($scope){
  $scope.entry = {
    title : 'An entry',
    body : 'How could we not forget writing...'
  };
});
```

Reload the app and you should still see a list of blog entries. If you go to `http://127.0.0.1:5001/#/entry/43` you should see the hard-coded entry.

Looking back at the `.config()` block you'll notice that `id` is a param in the url (`'/entry/:id'`). To get access to that param in your `EntriesShowCtrl` we can do the following:

```javascript
app.controller('EntriesShowCtrl', 
function($scope, $routeParams){
  $scope.routeParams = $routeParams;
  $scope.entry = {
    title : 'An entry',
    body : 'How could we not forget writing...'
  };
});
```

`$routeParams` is simply an object where the keys are the params defined in the `config` block and the values come from the actual url.

## Step 10 - Creating a Form

In `app.js`, in the `.config()` block, add another route:

```javascript
.when('/entry/create', {
  templateUrl : 'templates/entry-form.html',
  controller : 'EntriesCreateCtrl'
})
```

When the url is `/entry/create` AngularJS will use the template `entry-form.html` with the controller `EntriesCreateCtrl`.Now in `app.js` we'll create the controller `EntriesCreateCtrl`:

```javascript
app.controller('EntriesCreateCtrl',
function($scope){
  $scope.entry = {
    title : '',
    body: ''
  }

  $scope.save = function(entry){
    console.log(entry);
  };
});
```

In the above code we create a new empty entry and we create a `.save()` function. At the moment `.save()` just logs the `entry` to the console, but at a later step save will actually initiate an API request to save the entry.

In `templates` create a new template `entry-form.html` and inside of it:

```html
<h1>Create a blog entry</h1>
<form ng-submit="save(entry)">
  <label>Title:<br>
    <input type="text" name="title" ng-model="entry.title">
  </label>

  <br>
  <label>Content:<br>
    <textarea ng-model="entry.body"></textarea>
  </label>

  <input type="submit" value="Save">
</form>

<h2>preview:</h2>
<h4>{{entry.title}}</h4>
<p>{{entry.body}}</p>
```

`ng-submit` says: when the form is submitted, call the `.save()` function, in `EntriesCreateCtrl` and pass in the entry. `ng-model` binds a specific property of the entry to a form input (e.g. input & textarea). To show off Angular's 2-way binding we create a simple preview that will show progress live.

