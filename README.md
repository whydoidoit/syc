Syc
===

Sync a variable on the client and the server using Socket.io and Object.observe.

It works under a simple principle: All data bound to the variable in question is identical between the server, and the clients.

## Setting up syc

Syc uses socket.io to transmit information from client to server. You will need to set up socket.io independently of syc.

    var syc = require('syc');
    var io = require('socket.io').listen(80);

    io.sockets.on('connection', function (socket) {
        syc.connect(socket);
    });

## Using Syc

### Binding a variable (Server side)

    syc.sync(variable)

This is the most basic way to use syc. Any changes to `variable` on the server will be reflected on a variable by the same name on every client. Changes on the client will reflect on the server (and therefore to every other client).

This is accomplished by tracking the variable with Object.observe (or use a dirty-checking polyfill if Object.observe is not available. More on that in the Polyfill section).

**WARNING**: This is intended for prototyping a project. Syncing a variable without verifying it first is not recommended for public-facing or sites with untrusted clients. If any client maliciously modifies the variable, the changes will be reflected in the server as well as every other client.

    syc.serve(variable)
    
This is a safe way of allowing clients to receive changes to a variable. Changes on the server side will be sent to each client. If the client's version of the variable is modified, it will revert to match the server side's version immediately

### Juggling Users (Server Side)

    io.sockets.on('connection', function (socket) {
        user = syc.connect(socket);
    });

`syc.connect` returns a syc user object. If you want to bind a variable only to this user, use:

    user.sync(variable)

This is a 'safer' way to sync since the variable is not shared with other clients. You are allowed to sync the same variable to different users, but then a client may change the variable and the other users will see the change. You can also serve directly to a user.

    user.serve(variable)

Other common methods users contain: 

`user.socket()` This returns the socket.io socket bound to the user.
`user.synced` This returns an array of variables with two-way binding to the user
`user.serviced` Much like above, but listing just the one way bindings.
`user.group(name)` Adding the user to a group by name.

### Handling groups (Server)

You can create a group to add users into:

    misfits = syn.group('misfits')

And then add users by

    misfits.add(user)

Now you can sync a variable to everybody in the group, including anybody who joins the group.

    mistfits.sync(variable) 
    

    misfits.serve(variable)
    
`misfits.users` If you ever need to reference which users are assigned to a group.

`syc.groups` is a list of groups, by name.


### Watchers (Client + Server)

Often you want to do something the moment the variable changes.

    gerrymander = "I dare say!";
    
    bouffon = syc.sync(gerrymander);
    
    bouffon.watch(callback);
    
Now, if the value is modified by any of the clients the callback will be triggered. 


If you're working on the client, you can access bound variables via the syc.variables method.

    <script src="sync.js">
      syc = new syc();
      bouffon = syc.variables['buffon'];
      bouffon.watch(callback);
    </script>


Sometimes, buffon will not be available on the client at the time of asking. You can plan ahead instead:

    function watch (variable) { 
      variable.watch(callback);
    }
    
    syc.monitor('bouffon', watch);


You can see all the watchers on your variable by asking the sync object.

    bouffon.watchers()

### Verification (Server Side)

Watchers on the server allow you to respond to a variable change after a client has modified and synced it with the server. If you want to catch a change before it is accepted by the server and duplicated across clients, you can verify it:

    function check (variable, failure) {
      if (typeof variable === 'string') {
        return variable;
      } else { 
        failure();
      }
    }
    
    bouffon = syc.sync(variable)
    bouffon.verify(check)
   
If it turns out the client had modified the variable in an inapropriate way, you can call `failure` function, which will cause the client to reset his value to match the server.

    function check2 (variable, failure) {
      if (typeof variable === 'string') {
        return variable.toUpperCase();
      } else {
        return "SYC";
      }
    }
    
Using this technique, you can also cause the variable to be replaced with a different value if the client has damaged the variable in some way.
