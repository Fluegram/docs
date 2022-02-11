# Sessions

This module adds one of most important and widely used features - **stateless requests**.

For example, you need to ask user to fill a simple form which consists of two steps:

- Asking for the first name.
- Asking for the last name.

And then save these values to the some property.

You might think that you need some states for this task, e.g some kind of a dictionary, etc., but sessions nullifies this need.

Example below shows how to implement above form:

```csharp

public class RequestNameMessageHandler : HandlerBase<Message>
{
    public override async Task HandleAsync(EntityContext<Message> entityContext, CancellationToken cancellationToken)
    {
        var session = entityContext.Session<EntityContext<Message>, Message>();

        string firstName = await RequestNameAsync("First");
        string lastName = await RequestNameAsync("Last");

        //Save firstName and lastName somewhere.

        string message = $"Your name {firstName} {lastName} was successfully saved. Thank you.";
        
        return entityContext.InvokeAsync<SendTextMessageParameters, Message>(_ => _.UseText(message), cancellationToken: token);
        
        
        async Task<string> RequestNameAsync(string kind)
        {
            var receivedMessage = await session.RequestAsync(entityContext, requestOptions => requestOptions
                .UseAction((state, context, token) =>
                {
                    string messageText = $"Please, send me your {kind} name:";
                    
                    return context.InvokeAsync<SendTextMessageParameters, Message>(_ => _.UseText(messageText), cancellationToken: token);
                })
                //.UseMatcher() // Use this method when you need to check incoming entity. By default equals to (...) => true.
                //.UseAttempts() // Use this method when you want to specify number of attempts for your request. By default, there will be infinite attempts.
            , cancellationToken);

            return receivedMessage!.Text!;
        }
    }
}

```
