# Creating a Reddit app with AngularJS

TODO: High level explanation for value of AngularJS

## Angular's Parts

### Views

### Controllers

### Services

### Directives

## Step 1 - Create folders and files

Create a folder `reddit-app` and in the folder create 3 files:

1. index.html
2. app.js
3. style.css

## Step 2 - Add boiler plate code 

```html
<!DOCTYPE html>
<html>
<head>
  <title>Reddit App</title>

  <script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.0/angular.min.js"></script>
  <script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.0/angular-route.min.js"></script>

  <script type="text/javascript" src="app.js"></script>

  <link rel="stylesheet" type="text/css" href="style.css">
</head>
<body>

</body>
</html>
```

## Step 3 - Creating our Angular App

Update the line `<html>` line to `<html ng-app="RedditApp">` - this tells AngularJS that everything nested in this DOM element is part of an AngularJS app called "RedditApp".

Now we need to update the `app.js` file and add:

```
var redditApp = angular.module('RedditApp', ['ngRoute']);
```

The above line creates a module called 'RedditApp' that is dependent on the module 'ngRoute' (we'll cover ngRoute in the next step). If you reload the page you should see a blank page and no errors in the console.

## Step 4 - Mapping urls to specific controllers

Web applications are made up of dozens of URLs that each map to a controller and a view (for MVC architecture). AngularJS is no different! We want to tell AngularJS about the URLs that we intend to use, and which controllers and views (templates) they map to. To do so update `app.js` by adding:

```javascript
redditApp.config(['$routeProvider',
  function($routeProvider){
    $routeProvider
      .when('/', {
        templateUrl : 'templates/home.html',
        controller : 'HomeCtrl',
        controllerAs : 'home'
      })
      .when('/r/:subreddit/comments/:id/:slug', {
        templateUrl : 'templates/comments.html',
        controller : 'CommentsCtrl',
        controllerAs : 'comments'
      })
      .otherwise('/');
  }
]);
```

The above code defines two routes: `'/'` and `'/r/:subreddit/comments/:id/:slug'` that are mapped to `HomeCtrl` and `CommentsCtrl` respectively.  `'/r/:subreddit/comments/:id/:slug'` includes 3 parameters: `:subreddit`, `:id` and `:slug` - we will have access to those parameters in `CommentsCtrl`. `controllerAs` will be explained in a few steps. Now that we have the mapping defined, we need to create the controllers and views.

## Step 5 - Creating controllers and views

To keep things simple we'll create two basic controllers (both can be added to the end `app.js`):

```javascript
redditApp.controller('HomeCtrl', [ 
  function(){
    this.message = "Hello form the home controllers";
  }
]);
```

```javascript
redditApp.controller('CommentsCtrl', ['$routeParams',
  function($routeParams){
    this.params = $routeParams;
  }
]);
```

By using the `controllerAs` key in the routes above, anything we attached to `this` we have access to in the corresponding view. Now lets create two new views. In `reddit-app` create a folder called `templates` and create two new files `home.html` and `comments.html`.

In `home.html`:

```html
<h1>{{ home.message }}</h1>
```

In `comments.html`:

```html
<p>{{ comments.params }}</p>
```

Finally, we need to update the `index.html` file, after the `<body>` tag add the following:

```
  <a href="#/">Home</a>
  <a href="#/r/test/comments/1/a-test-article">A test article</a>
  <div ng-view></div>
```

The first two lines are links to our two different routes. The final line, `<div ng-view></div>` tells AngularJS where to render the views when a link is matched (e.g. the html from the template is inserted into the div). 

Unfortunately, if you try and load the app you'll get a bunch of errors in the console. When you load the `index.html` in your browser it will use the `file:` protocol, but AngularJS will use `http:` to load the templates (views) - this difference in protocols results in an error. To fix this issue we'll have to run `python -m SimpleHTTPServer 8000` in the `reddit-app` folder. In your browser go to `http://127.0.0.1:8000` and the app should now work!

## Step 6 - Loading our Reddis data

Now that we have a basic understand of routes, controllers and views, it's time to learn above services! Services come in 3 varieties: providers, factories and services. All three produce singleton object and they're effectively the same (for the purpose of this workshop we'll use factories). In AngularJS business logic, and API calls, are done with a service. 
