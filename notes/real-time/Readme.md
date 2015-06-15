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

## Step 2 - 

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

Then, in the of `index.js` update to:
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

http.listen(3000, function(){
  console.log('listening on *:3000');
});
```
