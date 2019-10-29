# Keep your mobile app up to date with SignalR

![Title Image](https://mallibone.com/posts/files/d383a562-90f5-4368-8e07-4f97335e1d93.jpg)

Whether writing a mobile app with Xamarin Forms or native Xamarin.iOS/Xamarin.Android sometimes the requirements demand that your app updates as quickly as possible when something changes on the server. This might be a chat application, stock ticker or any monitoring app showing live data to a user of a process currently running in the backend.

IIIIMAGE

Let\'s say we wanted to create a simple chat app. If we would go down the path of a traditional Create Read Update Delete (CRUD) app over HTTP, we might choose to poll the server every few seconds or so to read the latest value. While this approach delivers the results, it comes with some drawbacks. To name a few: making requests even though nothing has changed, climbing the leader board on the battery usage category plus your server gets hammered with requests only to tell them: \"Sorry no news yet\...\" - so if not poll lets push. And this is where WebSockets come in.

The only problem with WebSockets is that the implementation in .Net is close to the metal. This results in an additional implementation effort having to be done by the developers. Luckily for .Net developers, there is SignalR which comes with all the boilerplate code you want around WebSockets. A web developer will also tell you about fall back options backed into SingalR. As a Xamarin developer, you will most probably never use those features. But the chances are good that you will be delighted by the ease of handling connections, channels or writing to specific connected clients.

SignalR is around for quite a while; it has been ported over to [.Net Core](https://docs.microsoft.com/en-us/aspnet/core/signalr/introduction?view=aspnetcore-3.0?WT.mc_id=DT-MVP-5002881) and is .Net Standard compatible. This is excellent news since we can add the SignalR client directly into our Xamarin Forms app - no platform-specific/wrapper code required. But before we can start implementing our client, we will first have to create a SignalR enabled backend.

> If you are new to SignalR, be aware that there is quite a big difference between SignalR and SignalR Core. If you are writing a new app today using ASP.NET Core or Azure Functions. You will want to use [SignalR Core](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Core/) or else you will go on a nasty error hunt ending with your palm slapping against your forehead 🤦‍♂️

# The Server

When implementing the backend, we can choose between two options. Either we implement SignalR using ASP.NET Core, or we decide to go with Azures [SignalR Core Service](https://docs.microsoft.com/en-us/azure/azure-signalr/signalr-overview/?WT.mc_id=Azure-MVP-5002881). The later can be integrated into ASP.NET Core or an Azure Function app. The Azure option also comes with scaling capabilities - in other words, you get up to 1 Million simultaneous connections using SignalR out of the box.

For more detail on how to set up the Azure Functions and SignalR combo, you will find instructions in the official [documentation](https://docs.microsoft.com/en-us/azure/azure-signalr/signalr-quickstart-azure-functions-csharp?WT.mc_id=Azure-MVP-5002881).

For our simple chat application, we will want a way for our clients to send and receive messages. To achieve this, we will have to create a SignalR Hub that provides a method for sending messages:

```c#
[FunctionName("SendMessage")]
public static Task SendMessage(
    [HttpTrigger(AuthorizationLevel.Anonymous, "post")]string message,
    [SignalR(HubName = "SignalRDemo")]IAsyncCollector<SignalRMessage> signalRMessages)
{
    return signalRMessages.AddAsync(
        new SignalRMessage
        {
            Target = "NewMessage",
            Arguments = new[] { message }
        });
}
```

On invocation, the method will send the message to all connected clients. Which brings us to our next point. Every chat participant will first have to connect to the hub so that messages can be received. So let\'s implement that registration method:

```c#
[FunctionName("Negotiate")]
public static SignalRConnectionInfo Negotiate(
[HttpTrigger(AuthorizationLevel.Anonymous)]HttpRequest req,
[SignalRConnectionInfo(HubName = "SignalRDemo")] SignalRConnectionInfo connectionInfo)
{
    // connectionInfo contains an access key token with a name identifier claim set to the authenticated user
    return connectionInfo;
}
```

The naming of the method is a convention from SignalR. In other words, you must name the method `Negotiate`, or your code will not work. No, I do not want to elaborate on how I found this one out the hard way 😉

With the function and SignalR Service in place, we can now turn our focus to the client.

# The mobile client

On the mobile client, we want to be able to receive messages and type responses to the group. Our simple app will have to live with the limitation of only receiving messages while being connected. At least for the moment. But here is the chat running in all of its glory.

![Show chat in action](https://mallibone.com/posts/files/1eb601bc-2256-4cfb-a647-86f27674dc67.gif)

Now let\'s have a look at the `ChatService` which connects us to the backend and receives messages:

```c#
public async Task Connect()
{
    if (_connection.State == HubConnectionState.Connected) return;

    _connection.On<string>("NewMessage", (messageString) =>
    {
        var message = JsonConvert.DeserializeObject<Message>(messageString);
        _newMessage.OnNext(message);
        Debug.WriteLine(messageString);
    });

    await _connection.StartAsync();
}
```

Note that we register the receiver method before we connect to the backend. This way, we start receiving updates as soon as being connected to the SignalR Service. Now when implementing a receiver method, you must ensure that the type signature matches the method we defined earlier on the server. If the types or the name do not match, you will never receive any messages.

Since reading is only half the fun, let\'s implement the send message:

```c#
public async Task Send(Message message)
{
    var serializedPayload = JsonConvert.SerializeObject(message);

    var response = await _httpClient.PostAsync("https://gnabbersignalr.azurewebsites.net/api/SendMessage", new StringContent(serializedPayload));
    Debug.WriteLine(await response.Content.ReadAsStringAsync());
}
```

And if you ever had enough from the stream of messages but want to give your eyes the joy of staring at a bare app here is the disconnect method:

```c#
public async Task Disconnect()
{
    await _connection.DisposeAsync();

    _connection = new HubConnectionBuilder()
        .WithUrl(backendUrl)
        .Build();
}
```

I hope you could see that using SignalR it is a breeze to implement a bidirectional communication layer to your server. Which will allow your (mobile) clients to send and receive data in near real-time. Another side effect of using SignalR is that you could easily extend the app with a web client. Since your favourite JavaScript framework will allow you to use the [SignalR client](https://docs.microsoft.com/en-us/aspnet/core/signalr/javascript-client?view=aspnetcore-3.0). If you are ready to get started with SignalR, be sure to check out the [docs](https://docs.microsoft.com/en-us/aspnet/core/signalr/introduction?view=aspnetcore-3.0&viewFallbackFrom=aspnetcore-3.0%3FWT.mc_id%3DDT-MVP-5002881).

You can find the entire sample, including all the UI code on [GitHub](https://github.com/mallibone/XamarinSignalR).

This blog is part of the [October Xamarin Challenge](https://github.com/claudiosanchez/OctoberXamarinChallenge). So be sure to check out the other posts for more best practices when writing Xamarin apps.

HTH
