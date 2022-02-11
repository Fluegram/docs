# Handlers

Handlers is just like middlewares, but with support [Actions](../actions) and [Filters](../filters) features.

## Creating a handler

Creating a handler is very easy process.

For example, we need to log some information while processing a Message:

```csharp

public class LogHandler : HandlerBase<Message>
{
    private readonly ILogger _logger;

    public LogHandler(ILogger logger)
    {
        _logger = logger;
    }
    
    public override async Task HandleAsync(EntityContext<Message> entityContext, CancellationToken cancellationToken)
    {
        _logger.Info(
            $"Processing message with id {entityContext.Entity.MessageId} from user with id {entityContext.User!.Id}.");
    }
}

```

Here you can see a custom constructor with argument of type `ILogger` which means that instance of `ILogger` will be taken from components, so make sure you've registered it, otherwise this will result in an error when resolving this component.
