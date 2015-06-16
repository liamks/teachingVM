# Download

# Install

# Walkthrough of existing code


# Creating our real-time chat app

## Step 1

```
npm install socket.io --save --no-bin-links
```

On line 4 of `index.js` add:

```javascript
var io = require('socket.io')(http);
```

On line 11 of 'index.js' add:

```javascript
io.on('connection', function(socket){
  console.log('a user connected');
});
```

On line 8 of `index.html` add:
```html
<script type="text/javascript" src="/socket.io/socket.io.js"></script>
```

Add `'btford.socket-io'` as a dependency to your app, and update `app.js` to look like:
```
var app = angular.module('realTimeChat', ['btford.socket-io']);

app.factory('socket', function(socketFactory){
  return socketFactory();
});

app.controller('ChatCtrl', function($scope, socket){
  $scope.time = new Date();
});
```

## Step 2 - Showing connected users

```
npm install underscore --save
```

Add underscore to the top of the requirements of `index.js`:

```
var _ = require('underscore');
```

On line 13 of `index.js` add:
```javascript
var users = {};
var sockets = {}

function broadcastUsers(){
  _.each(sockets, function(socket){
    socket.emit('users', {
      users : Object.keys(users)
    });
  });
}
```

Then update `io.on('connect',` in `index.js` to:
```javascript
io.on('connection', function(socket){
  var name;
  socket.on('join', function(msg){
    name = msg.name;
    users[name] = name;
    sockets[socket.id] = socket;
    broadcastUsers();
  });

  socket.on('disconnect', function(){
    delete users[name];
    delete sockets[socket.id];
    broadcastUsers();
  });
});
```

In the above code, within the block of code that run once a client connects ("on connection"), we've added to event handlers, one for the `'join'` event and one for the `'disconnect'` event. The `'join'` event is something that we'll need to emit/trigger on the client side. The `'disconnect'` event is emitted by Socket.IO automatically when the client's connection ends.

In the `'join'` event we're:

1. assigning their name to the `name` variable.
2. adding their name to the object containing all of the names.
3. adding their socket to the object containing all of the sockets.
4. Finally, broadcasting the array of names to all connected users.

In the `'disconnect'` event we're:

1. removing the user from the object containing all of the users names'.
2. removing the socket from the object containing all of sockets.
3. broadcasting the array of names (minus the person who just disconnected) to all connected users.

Now that we have the server-side portion, we need to udpate the client-side.

In `app.js` add the following: 
```javascript
app.factory('chat', function(socket){
  var factory = {};
  factory.users = [];

  factory.getUsersName = function(){
    factory.name = window.localStorage.getItem('name');

    if(!factory.name){
      factory.name = window.prompt("Please Enter Your Name: ");
      window.localStorage.setItem('name', factory.name);
    }

    return factory.name;
  };

  factory.joinRoom = function(){
    socket.emit('join', {
      name : factory.name
    });
  };

  factory.initialize = function(){
    socket.on('users', function(msg){
      factory.users = msg.users;
    });

    factory.getUsersName();
    factory.joinRoom();

    return factory.name;
  };

  return factory;
});
```

### `factory.getUsersName()`

`getUsersName` first checks localStorage, the browser's persistant data store, for `name`, if it doesn't exist it will prompt the user to enter the name (and store the name in localStorage). By using localStorage, when the page is refreshed, the user won't have to enter their name again.

### `factory.joinRoom()`

`joinRoom()` triggers/emits the 'join' event to the server and sends the user's name.

### `factory.initialize()`

`initialize()` adds an event handler to receive the `'users'` event (e.g. the list of users) from the server and stores those users in `factory.users`. Next, it get's the user's name, and calls `joinRoom`. Finally it return's the user's name.

Next we'll need to update the `ChatCtrl` to use the `chat` service that we just created.

Finally, update `ChatCtrl` in app.js to: 
```javascript
app.controller('ChatCtrl', function($scope, chat){
  $scope.usersName = chat.initialize();
  
  $scope.$watch(function(){
    return chat.users;
  }, function(users){
    $scope.users = chat.users;
  });
});
```

First we call `chat.initialize()` which gets the user's name, and joins the chat room. Then we setup a `$watch` statement. The first callback (`function(){ return chat.users; })` is called on every digest cycle. If the value that the callback returns is different from the value it returned on the previous digest cycle, it will call the second callback. The second callback assigns the users to `$scope.users` which in turn will result in the view being updated. Now we'll need to update the template for the list of users.

Update `public/templates/sidebar.html` to:
```html
<div class="sidebar">
  <ul class="nav nav-sidebar">
    <li ng-repeat="user in users">
      <i class="glyphicon glyphicon-user"></i> {{user}}
    </li>
  </ul>
</div>
```

The above will render the array of users and place a nice icon next to each one.