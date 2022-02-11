# Widgets

This module adds feature of stateful widgets.

For example this is the counter widget implementation example:

```csharp

public class CounterState : IWidgetState<CounterState>
{
    public int Counter { get; set; }
}

public class CounterWidget : WidgetBase<Message, CounterState>
{
    private Message _sentMessage;
    
    public override async Task OnInitializedAsync(EntityContext<Message> entityContext, CancellationToken cancellationToken)
    {
        _sentMessage =
            await entityContext.InvokeAsync<SendTextMessageParameters, Message>(_ => _.UseText("Initiaizing counter widget"), cancellationToken: cancellationToken);
    }

    public override async Task OnUpdatedAsync(EntityContext<Message> entityContext, CounterState state, CancellationToken cancellationToken)
    {
        await entityContext.InvokeAsync<EditMessageTextParameters, Message>(_ => _.UseText($"Counter: {state.Counter}"), cancellationToken: cancellationToken);
    }
}

```

These widget updates message once it's state was changed.

Let's see how to configure and use this widget.

Firstly, we need to register this widget as a [feature](../../getting-started.md#configuring-pipeline-features) to our pipeline:

```csharp

void ConfigureMessagePipelineFeatures(
    IPipelineFeaturesConfigurator<EntityContext<Message>, Message> pipelineFeaturesConfigurator)
{
    pipelineFeaturesConfigurator.UseWidgets(_ => _
        .Use<CounterWidget, CounterState>()
    );
}

```

Then, lets create a command, which will be creating our widget and modifying it's state:

```csharp

public class CounterCommand : CommandBase<Message>
{
    private readonly IWidgetFactory<CounterWidget, EntityContext<Message>, Message, CounterState> _widgetFactory;

    public CounterCommand(IWidgetFactory<CounterWidget, EntityContext<Message>, Message, CounterState> widgetFactory) : base("counter")
    {
        _widgetFactory = widgetFactory;
    }

    public override async Task ProcessAsync(EntityContext<Message> entityContext, CancellationToken cancellationToken)
    {
        IWidgetController<EntityContext<Message>,Message,CounterState> widgetController 
            = await _widgetFactory.CreateAsync(entityContext, cancellationToken);

        while (true)
        {
            await Task.Delay(1000, cancellationToken);

            widgetController.State.Counter++;

            await widgetController.NotifyStateChangedAsync(cancellationToken);
        }
    }
}

```

