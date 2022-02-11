# Actions

**Actions** module is used when you need to invoke some pre-processing or post-processing actions like [sending Typing action](https://core.telegram.org/bots/api#sendchataction) to the chat or Choose Sticker when you mean to send a sticker when processing handler or command.

**Please note**, that currently, actions are only supported when using **Attributes**.

## Creating your own action

To create action, simply derive your attribute from `I(Pre/Post)ProcessingAction<TEntityContext, TEntity>` like that:

```csharp

public class SendChatActionAttribute<TEntityContext, TEntity> : Attribute, IPreProcessingAction<TEntityContext, TEntity>
    where TEntityContext : IEntityContext<TEntity> where TEntity : class
{
    public SendChatActionAttribute(ChatAction action)
    {
        Action = action;
    }

    public ChatAction Action { get; }
    
    public async Task InvokeAsync(TEntityContext entityContext, CancellationToken cancellationToken)
    {
        await entityContext.Client.SendChatActionAsync(entityContext.Chat!, Action, cancellationToken)
            .ConfigureAwait(false);
    }
}

```

And then you can add that attribute, for example, to your handler:

```csharp

[SendChatAction<Message>(ChatAction.Typing)]
public class EchoMessageHandler : HandlerBase<Message>
{
    public override async Task HandleAsync(EntityContext<Message> entityContext, CancellationToken cancellationToken)
    {
        await entityContext.Client.SendTextMessageAsync(
            entityContext.Chat!, 
            entityContext.Entity.Text!, 
            cancellationToken: cancellationToken
        )
        .ConfigureAwait(false);
    }
}

```

And here, before invoking your `EchoMessageHandler`, chat action **Typing** will be send to the chat.