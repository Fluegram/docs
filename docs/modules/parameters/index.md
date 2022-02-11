# Parameters

This module adds fluent builders for each request `Telegram.Bot`'s `ITelegramBotClient` has.

For example, you need to send a message, so you could do that:

```csharp

public class SendTextMessageHandler : HandlerBase<Message>
{
    public override async Task HandleAsync(EntityContext<Message> entityContext, CancellationToken cancellationToken)
    {
        string message = "Hello, world!";
        
        await entityContext.InvokeAsync<SendTextMessageParameters, Message>(messageParameters => messageParameters
            .UseText(message));
    }
}

```

You may have noticed that I did not specify the chat ID to which our message should be sent, but it's not necessary. Why? Let's take a look at `InvokeAsync<TParameters, T>` method:

```csharp

public static async Task<T> InvokeAsync<TParameters, T>(this IContext entityContext, Action<TParameters> configure, CancellationToken cancellationToken = default)
    where TParameters : IParameters<T>, new()
{
    TParameters parameters = new();

    if (parameters is IHasChatId hasChatId)
    {
        hasChatId.ChatId = entityContext.Chat!;
    }
    configure(parameters);

    return await parameters.InvokeAsync(entityContext.Client, cancellationToken);
}

```

Here you can see that we have condition which checks if parameters implements `IHasChatId` interface.

**And what does this all means?**

Every parameters class implements a bunch of interfaces which who a responsible for holding and setting various parameters. So our method will set target chat from entity context for **every** parameters class, which implements `IHasChatId`.

And this feature is available for **all** parameters classes and its' properties.