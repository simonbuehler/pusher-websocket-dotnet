# Pusher Channels .NET Client library

This is a .NET library for interacting with the Channels WebSocket API.

Register at [pusher.com/channels](https://pusher.com/channels) and use the application credentials within your app as shown below.

More general documentation can be found on the [Official Channels Documentation](https://pusher.com/docs/channels).

For integrating **Pusher Channels** with **Unity** follow the instructions at <https://github.com/pusher/pusher-websocket-unity>

## Supported platforms

* .NET Standard 1.3
* .NET Standard 2.0
* .NET 4.5
* .NET 4.7.2
* Unity 2018 and greater via [pusher-websocket-unity](https://github.com/pusher/pusher-websocket-unity)

## Installation

### NuGet Package

```
Install-Package PusherClient
```

## Usage

See [the example app](https://github.com/pusher/pusher-websocket-dotnet/tree/master/ExampleApplication) for full details.

### Connect

#### Event based
```cs
_pusher = new Pusher("YOUR_APP_KEY");
_pusher.ConnectionStateChanged += _pusher_ConnectionStateChanged;
_pusher.Error += _pusher_Error;
try
{
    await _pusher.ConnectAsync().ConfigureAwait(false);
    // _pusher.State will be ConnectionState.Connected
}
catch(Exception error)
{
    // Handle error
}
```

where `_pusher_ConnectionStateChanged` and `_pusher_Error` are custom event handlers such as

```cs
static void _pusher_ConnectionStateChanged(object sender, ConnectionState state)
{
    Console.WriteLine("Connection state: " + state.ToString());
}

static void _pusher_Error(object sender, PusherException error)
{
    Console.WriteLine("Pusher Channels Error: " + error.ToString());
}
```

In the case of the async version, state and error changes will continue to be published via events, but the initial connection state will be returned from the `ConnectAsync()` method.

### Authenticated Connect
If you have an authentication endpoint for private or presence channels:

#### Event based
```cs
_pusher = new Pusher("YOUR_APP_KEY", new PusherOptions(){
    Authorizer = new HttpAuthorizer("YOUR_ENDPOINT")
});
_pusher.ConnectionStateChanged += _pusher_ConnectionStateChanged;
_pusher.Error += _pusher_Error;
try
{
    await _pusher.ConnectAsync().ConfigureAwait(false);
    // _pusher.State will be ConnectionState.Connected
}
catch(Exception error)
{
    // Handle error
}
```

### Non Default Cluster
If you are on a non default cluster (e.g. eu):

#### Event based
```cs
_pusher = new Pusher("YOUR_APP_KEY", new PusherOptions(){
    Cluster = "eu"
});
_pusher.ConnectionStateChanged += _pusher_ConnectionStateChanged;
_pusher.Error += _pusher_Error;
await _pusher.ConnectAsync().ConfigureAwait(false);
```

### Connection States
You can access the current connection state using `Pusher.State` and bind to state changes using `Pusher.ConnectionStateChanged`.

#### Possible States
* Uninitialized: Initial state; no event is emitted in this state.
* Connecting: The channel client is attempting to connect.
* Connected: The channel connection is open and authenticated with your app. Channel subscriptions can now be made.
* Disconnecting: The channel client is attempting to disconnect.
* Disconnected: The channel connection is disconnected and all subscriptions have stopped.
* WaitingToReconnect: Once connected the channel client will automatically attempt to reconnect if there is a connection failure.

First call to `Pusher.ConnectAsync()` succeeds:
`Uninitialized -> Connecting -> Connected`. Auto reconnect is enabled.

First call to `Pusher.ConnectAsync()` fails:
`Uninitialized -> Connecting`. Auto reconnect is disabled.
`Pusher.ConnectAsync()` will throw an exception and a `Pusher.Error` may be emitted depending on the failure.

Call to `Pusher.DisconnectAsync()` succeeds:
`Connected -> Disconnecting -> Disconnected`. Auto reconnect is disabled.

Call to `Pusher.DisconnectAsync()` fails:
`Connected -> Disconnecting`.
`Pusher.DisconnectAsync()` will throw an exception.

#### Auto reconnect
After entering a state of `Connected`, auto reconnect is enabled. If the connection is dropped the channel client will attempt to re-establish the connection.

 Here is a possible scenario played out after a connection failure.
* Connection dropped
* State transition: `Connected -> Disconnected`
* State transition: `Disconnected -> WaitingToReconnect`
* Pause for 1 second and attempt to reconnect
* State transition: `WaitingToReconnect -> Connecting`
* Connection attempt fails, network still down
* State transition: `Connecting -> Disconnected`
* State transition: `Disconnected -> WaitingToReconnect`
* Pause for 2 seconds (max 10 seconds) and attempt to reconnect
* State transition: `WaitingToReconnect -> Connecting`
* Connection attempt succeeds
* State transition: `Connecting -> Connected`

### Subscribe to a public or private channel

#### Event based
```cs
_myChannel = _pusher.Subscribe("my-channel");
_myChannel.Subscribed += _myChannel_Subscribed;
```
where `_myChannel_Subscribed` is a custom event handler such as

```cs
static void _myChannel_Subscribed(object sender)
{
    Console.WriteLine("Subscribed!");
}
```

#### Asynchronous
```cs
Channel _myChannel = await _pusher.SubscribeAsync("my-channel");
```

### Bind to an event

```cs
_myChannel.Bind("my-event", (dynamic data) =>
{
    Console.WriteLine(data.message);
});
```

### Subscribe to a presence channel

#### Event based
```cs
_presenceChannel = (PresenceChannel)_pusher.Subscribe("presence-channel");
_presenceChannel.Subscribed += _presenceChannel_Subscribed;
_presenceChannel.MemberAdded += _presenceChannel_MemberAdded;
_presenceChannel.MemberRemoved += _presenceChannel_MemberRemoved;
```

Where `_presenceChannel_Subscribed`, `_presenceChannel_MemberAdded`, and `_presenceChannel_MemberRemoved` are custom event handlers such as

```cs
static void _presenceChannel_MemberAdded(object sender, KeyValuePair<string, dynamic> member)
{
    Console.WriteLine((string)member.Value.name.Value + " has joined");
    ListMembers();
}

static void _presenceChannel_MemberRemoved(object sender)
{
    ListMembers();
}
```

#### Asynchronous

```cs
_presenceChannel = await (PresenceChannel)_pusher.SubscribeAsync("presence-channel");
_presenceChannel.Subscribed += _presenceChannel_Subscribed;
_presenceChannel.MemberAdded += _presenceChannel_MemberAdded;
_presenceChannel.MemberRemoved += _presenceChannel_MemberRemoved;
```

### Unbind

Remove a specific callback:

```cs
_myChannel.Unbind("my-event", callback);
```

Remove all callbacks for a specific event:

```cs
_myChannel.Unbind("my-event");
```

Remove all bindings on the channel:

```cs
_myChannel.UnbindAll();
```


## Developer Notes

The Pusher application settings are now loaded from a JSON config file stored in the root of the source tree and named `AppConfig.test.json`. Make a copy of `./AppConfig.sample.json` and name it `AppConfig.test.json`. Modify the contents of `AppConfig.test.json` with your test application settings. All tests should pass. The AuthHost and ExampleApplication should also run without any start-up errors.

### Publish to NuGet

You should be familiar with [creating an publishing NuGet packages](http://docs.nuget.org/docs/creating-packages/creating-and-publishing-a-package).

From the `pusher-dotnet-client` directory:

1. Update `PusherClient/PusherClient.csproj` and  `PusherClient/Pusher.cs` with new version number etc.
2. Specify the correct path to `msbuild.exe` in `package.cmd` and run it.
3. Run `nuget push PusherClient.{VERSION}.nupkg`

## License

This code is free to use under the terms of the MIT license.
