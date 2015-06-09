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