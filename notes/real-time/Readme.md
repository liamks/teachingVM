# Creating a Real-Time Chat App

## Download

Download the [Starter Code](https://github.com/DCSIL-summer-course/real-time-chat/archive/0.1.zip) and unzip it to a folder that is shared with your VM.

## Install 

From the VM, in the `real-time-chat` folder:

```
npm install  --no-bin-links
```

That will install the starter code's dependencies.

# Walkthrough of existing code

The code consists of:

1. NodeJS/ExpressJS app `index.js`
2. An `index.html` (`view/index.html`) file that includes our js files
3. AngularJS app `public/js/app.js`
4. AngularJS templates `public/templates/`

## Step 1 - Installing Socket.IO

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

## Step 3 - Chat

First we'll update the `chat` service with a dummy function that we can fill in later:

```javascript
factory.sendMessage = function(msg){
  console.log(msg);
};
```

Eventually the above function will be used for sending a message to the server. Right now, it will just print the message to the console.`

Then update the `ChatCtrl` and add: 
```javascript
$scope.message = { text : '' };

$scope.sendMessage = function(){
  chat.sendMessage($scope.message.text);
  $scope.message.text = '';
};
```

We create an object, with a text property and assign it to `$scope.message`, so we can use it in the view. `sendMessage` will take the current message and call `chat.sendMessage` and finally set the text back to `''`.

Next update `public/templates/main.html` (lines 4 - 7) and change them to:
```html
<div class="input-conversation">
  <input type="text" ng-model="message.text">
  <button class="btn btn-primary" ng-click="sendMessage()">Send</button>
</div>
```

`ng-model="message.text"` sets up the two way binding of `message.text` between the view and the controller. If the value is changed in the `<input>` it will also be changed in the controller, and vice versa. `ng-click="sendMessage()"` adds a click handler to the send button - specifically, when the user clicks on the button `sendMessage()` will be called.

At this point we need to send the message to the server!

Update the `chat` service's `sendMessage` function to:
```javascript
factory.sendMessage = function(msg){
  socket.emit('send-message', {
    message : msg
  });
};
```

The above will send/emit the `'send-message'` event to the server with the data: `{ message : msg }`.

Next we need to add an event handler on the server to listen for the `'send-message'` event. In `index.js` in the `io.on('connection', function(socket){` block:

```javascript
socket.on('send-message', function(msg){
  console.log(msg.message);
});
```

At this point clients can send messages to the server, now we need to get the server to send the message to each connected client.

In `index.js` update the `'send-message'` handler to:
```javascript
socket.on('send-message', function(msg){
  msg.name = name;
  broadcastNewMessage(msg);
});
```

This adds the user's name to their message, then calls `broadcastNewMessage(msg)`, which we'll add to `index.js` (before the `io.on()` block):

```javascript
function broadcastNewMessage(msg){
  _.each(sockets, function(socket){
    socket.emit('new-message', msg);
  });
}
```

The server can now receive messages and send them to each connected client. Next, we need to make sure that each client can receive the message.

In `chat` service, towards the top add:
```javascript
  factory.messages = [];
```

This will store the messages. Then, in the `initialize()` function, in `chat` service add:

```javascript
socket.on('new-message', function(msg){
  factory.messages.unshift(msg);
});
```

This will listen for `'new-message'` that are sent from the server to the client. We'll add each message to the beginning of the `factory.messages` array. 

Next we need to update the `ChatCtrl` to listen for changes to the `factory.messages` array. Add the following to `ChatCtrl`:

```javascript
$scope.$watch(function(){
  return chat.messages;
}, function(messages){
  $scope.messages = messages;
});
```

The first callback returns `chat.messages`, if the value returned differs from the value returned by the previous digest cycle, it will call the second callback, which will update `$scope.messages`, which in turn will update our view (once we make a few changes to the view). Now we'll update the view (`public/templates/main.html`:

```html
<div class="conversation">
 <ul>
  <li ng-repeat="message in messages">
    <strong>{{message.name}}</strong>: {{message.message}}
  </li>
 </ul>
</div>
<div class="input-conversation">
  <input type="text" ng-model="message.text">
  <button class="btn btn-primary" ng-click="sendMessage()">Send</button>
</div>
```

If you restart your server, and reload your page, you should be able to chat with another user (open up a new incognito tab, or a different browser).