# Filters

Filters is used when you need to check some information, before execute your [handler](../handlers) or [command](../commands).

## Creating a custom filter

Creating a custom filter is easy as creating your own action - you just need to create an attribute which implements `IFilter<TEntityContext, TEntity>` interface.

For example, below example shows how to create a filter, which checks that entity chat id equals to the specified chat id:

```csharp

public class ChatIdFilterAttribute<TEntityContext, TEntity> : Attribute, IFilter<TEntityContext, TEntity>
    where TEntityContext : IEntityContext<TEntity> where TEntity : class
{
    public ChatIdFilterAttribute(ChatId chatId)
    {
        ChatId = chatId;
    }

    public ChatId ChatId { get; }

    public Task<bool> MatchesAsync(TEntityContext entityContext, CancellationToken cancellationToken)
    {
        return Task.FromResult(entityContext.Chat!.Id == ChatId.Identifier);
    }
}

```

And then just add `ChatIdFilterAttribute` attribute to any handler or command you want.

