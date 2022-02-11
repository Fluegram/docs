# Commands

**Commands** module supports creating and configuring command, as well as adding subcommands.

## Creating a command

### Simple commands

This example shows creating a simple Ping-Pong command:

```csharp

public class PingPongCommand : CommandBase<Message>
{
    public PingPongCommand() : base("ping")
    {
    }

    public override async Task ProcessAsync(EntityContext<Message> entityContext, CancellationToken cancellationToken)
    {
        await entityContext.Client.SendTextMessageAsync(entityContext.Chat, "Pong!");
    }
}

```

And then, registration of this command will look like that:

```csharp

void ConfigureMessagePipeline(IPipelineBuilder<EntityContext<Message>, Message> pipelineBuilder)
{
    pipelineBuilder.UseCommands(commandsConfigurator => commandsConfigurator
        .Use<PingPongCommand>()
    );
}


```

### Complex commands with subcommands.

Let's assume, that you have these two commands:

```csharp

public class RootCommand : CommandBase<Message>
{
    public RootCommand() : base("root")
    {
    }

    public override async Task ProcessAsync(EntityContext<Message> entityContext, CancellationToken cancellationToken)
    {
        string message = "Hello from root!";

        await entityContext.Client.SendTextMessageAsync(entityContext.Chat!, message, cancellationToken: cancellationToken);
    }
}

public class RootSubCommand : CommandBase<Message>
{
    public RootSubCommand() : base("sub")
    {
    }

    public override async Task ProcessAsync(EntityContext<Message> entityContext, CancellationToken cancellationToken)
    {
        string message = "Hello from subcommand of root command!";

        await entityContext.Client.SendTextMessageAsync(entityContext.Chat!, message, cancellationToken: cancellationToken);
    }
}

```

And you want to add `RootSubCommand` to the `RootCommand` as subcommand. These could be easily done by using overload of `Use<TCommand>()` method:

```csharp

void ConfigureMessagePipeline(IPipelineBuilder<EntityContext<Message>, Message> pipelineBuilder)
{
    pipelineBuilder.UseCommands(commandsConfigurator => commandsConfigurator
        .Use<RootCommand>(rootCommandConfigurator => rootCommandConfigurator
            .Use<RootSubCommand>()
        )
    );
}

```

Congratulations! You're done!

### Commands with custom arguments

Another module feature, is that you could rid of manual arguments parsing.

Let's assume, that you have command, called `random`, which takes two arguments - `minValue` and `maxValue`.

Sure, you may would think about manual arguments parsing via `int.TryParse` and other ways, but Fluegram provides built-in feature for parsing command arguments into the class.

For example, there's an example how to use that:

```csharp

public class RandomArguments
{
    [CommandArgument]
    public int MinValue { get; set; }
    
    [CommandArgument]
    public int MaxValue { get; set; }
}

public class RandomCommand : CommandBase<Message, RandomArguments>
{
    public RandomCommand() : base("random")
    {
    }

    public override async Task ProcessAsync(EntityContext<Message> entityContext, RandomArguments arguments, CancellationToken cancellationToken)
    {
        int randomValue = RandomNumberGenerator.GetInt32(arguments.MinValue, arguments.MaxValue);

        await entityContext.Client.SendTextMessageAsync(entityContext.Chat!,
            $"Random value from {arguments.MinValue} to {arguments.MaxValue} is {randomValue}.", cancellationToken: cancellationToken);
    }

    public override async Task ProcessInvalidArgumentsAsync(EntityContext<Message> entityContext, IEnumerable<ICommandArgumentParseError> argumentsParseErrors,
        CancellationToken cancellationToken)
    {
        StringBuilder sb = new StringBuilder();

        sb.AppendLine("One of arguments you've entered is invalid. Or one of arguments was missing:");

        int index = 1;
        foreach (var error in argumentsParseErrors)
        {
            if (error.Exception is { Message: {} message })
            {
                sb.Append($"{index++}. {error.Argument.Name} - {message}");
            }
            else
            {
                sb.AppendLine($"{index++}. {error.Argument.Name}.");
            }
        }

        await entityContext.Client.SendTextMessageAsync(entityContext.Chat!, sb.ToString(), cancellationToken: cancellationToken);
    }
}

```

In above example we have class `RandomArguments` which stands for our parsed arguments.

Please note, that arguments parsing order will be the same, as order od properties that you've defined in your arguments class. So make sure, you have defined your arguments in the right order.

For example, we have input string `random 1 10`, which means that out `RandomArguments` instance will look like that:

- `MinValue` will equal to `1`.
- `MaxValue` will equal to `10`.

But, if we will swap our property declarations, our instance will look like that:

- `MinValue` will equal to `10`.
- `MaxValue` will equal to `1`.


And also note, that the following prerequisites must be met:

- Each argument property should be marked with `[CommandArgument]` attribute.
- Each argument property should have public `get` and `set` accessors.


Let's return to our example above. There we have two methods:

- `ProcessAsync` - will be executed only if arguments was parsed successfully.
- `ProcessInvalidArgumentsAsync` - will be executed if at least one argument wasn't successfully parsed. Note that this method has `IEnumerable<ICommandArgumentParseError> argumentsParseErrors` parameter, which holds information about arguments, which wasn't parsed successfully.

