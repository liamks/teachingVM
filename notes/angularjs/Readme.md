# Creating a Reddit app with AngularJS

TODO: High level explanation for value of AngularJS

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

Web applications are made up of dozens of URLs that each map to a controller and a view (for MVC architecture). AngularJS is no different! We want to tell AngularJS about the URLs that we intend to use and which controllers and views (templates) they map to. To do so update `app.js` by adding:

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