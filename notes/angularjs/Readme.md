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
      .when('/r/:subreddit/comments/:id/:slug/', {
        templateUrl : 'templates/comments.html',
        controller : 'CommentsCtrl',
        controllerAs : 'cmnts'
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

Now that we have a basic understand of routes, controllers and views, it's time to learn above services! Services come in 3 varieties: providers, factories and services. All three produce singleton object and they're effectively the same (for the purpose of this workshop we'll use factories). In AngularJS, business logic and API calls are done with a service. We'll create a service called: `RedditService` that we'll use for interacting with Reddit's API.

Create a file called services.js, this is where we'll place our `RedditService`. Add a reference to the file in `index.html`, right before the reference to app:

```html
<script type="text/javascript" src="services.js"></script>
<script type="text/javascript" src="app.js"></script>
```

In `services.js` add the following:

```javascript
var services = angular.module('services', []);

services.factory('RedditService', ['$http', 
  function($http){
    function Reddit(){};

    Reddit.domain = 'http://www.reddit.com';

    Reddit.top = function(){
      var path = '/top.json';
      var url = Reddit.domain + path;

      return $http.get(url)
        .then(function(response){
          return response.data.data;
        });
    };

    Reddit.comments = function(sub, id, slug){

    };

    return Reddit;
  }
]);
```

In `app.js` we'll have to add `servies` as a dependency of our app. Update the following line in `app.js` to:

```javascript
var redditApp = angular.module('RedditApp', ['ngRoute', 'services']);
```

Now let's connect the service to the controller!

Update the `HomeCtrl` to:

```javascript
redditApp.controller('HomeCtrl', [ 'RedditService',
  function(RedditService){
    
    RedditService.top()
      .then(function(stories){
        this.stories = stories;
      }.bind(this));

  }
]);
```

Update `home.html` to:

```html
<h1>Top Stories</h1>

<div class="stories">
  <div class="story" ng-repeat="story in home.stories">
    <h2><a ng-href="{{ story.data.url }}" target="_blank">{{ story.data.title }}</a></h2>
    <p>
      {{ story.data.score }} votes | 
      <a href="#{{ story.data.permalink }}">{{ story.data.num_comments }} comments</a>
    </p>
  </div>
</div>
```

Now let's add a bit of CSS to our page. Add the following to `style.css`:

```css
body {
  font: normal small verdana,arial,helvetica,sans-serif;
}

a {
  text-decoration: none;
}

.story {
  margin: 12px 0;
}

.story p, h2 {
  margin: 0;
  padding: 0;
}
```

## Step 7 - Loading Comments

To get the comments page working we'll have to update the `RedditService`, `CommentsCtrl` and `comments.html`. First we'll update the service. In `services.js` add the following to the `RedditService`:

```javascript
Reddit.comments = function(params){
  var path = '/r/' + params.subreddit + 
             '/comments/' + params.id + 
             '/' + params.slug + '.json';

  var url = Reddit.domain + path;

  return $http.get(url)
    .then(function(response){
      return response.data[1].data.children;
    });
};
```

The above makes an AJAX request to the comments API and grabs just the comments from the response (comments are nested in `response`).

Now update the `CommentsCtrl` to use the the `comments` function that we just created:

```javascript
redditApp.controller('CommentsCtrl', ['$routeParams', 'RedditService',
  function($routeParams, RedditService){
    this.params = $routeParams;

    RedditService.comments($routeParams)
      .then(function(comments){
        this.comments = comments;
      }.bind(this));
  }
]);
```

Next, we'll update the `comments.html` view:

```html
<p>Comments</p>

<div class="comments">
  <div class="comment" ng-repeat="comment in cmnts.comments">
    {{ comment.data.author }}
    <p>{{ comment.data.body }}</p>
  </div>
</div>
```

Finally we'll add a bit of CSS to `style.css`

```css
.comment {
  border-bottom: 1px solid #ccc;
  padding: 4px 0;
}
```

## Step 8

The [Reddit API](http://www.reddit.com/dev/api) is pretty extensive, I'd encourage you to go further and add more functionality to your app.