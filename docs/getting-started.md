# Getting Started

## Core installation

Via NuGet Package manager console:

```{.py3 title="NuGet Package Manager console"}

Install-Package Fluegram

```

## Creating and configuring bot

To create and configure bot, firstly you need to create Autofac's ContainerBuilder instance:

```csharp

using Autofac;

ContainerBuilder containerBuilder = new ContainerBuilder();

```

Then, configure bot by `RegisterFluegram` method:

```csharp

containerBuilder.RegisterFluegram(fluegramBot =>
{
    //Configure your bot here  
});

```

This method takes Action<IFluegramBotBuilder> parameter, so for comfort, you can create method and pass it to the constructor:

```csharp

containerBuilder.RegisterFluegram(ConfigureFluegramBot);

void ConfigureFluegramBot(IFluegramBotBuilder builder)
{
    
}

```

To configure a pipeline for specified Update type, use `IFluegramBotBuilder UseFor<TEntityContext, TEntity>(UpdateType updateType, Action<IPipelineBuilder<TEntityContext TEntity>> configurePipeline)` method, where:

- `TEntityContext` - Context, which holds default information about update - Entity (in that case - our Message), User and Chat, and Telegram Bot Api Client instance.
- `TEntity` - Entity type (in that case - Message type).

Framework core already has it's own implementation of `IEntityContext<TEntity>` interface - `EntityContext<TEntity>`, so if you doesn't want to extend default realization and won't add your own fields, just use built-in interface implementations that wrapped around of `EntityContext<TEntity>`,  (e.g. `HandlerBase<TEntity>` - from [Handlers](../modules/handlers) module or `CommandBase<TEntity>` - from [Commands](../modules/commands) module).

These types are used everywhere, so make sure you understand how and when they should be used.


Pipelines is using middleware-chaining for processing various types of entities, which means that they're flexible and easily configurable.


Below example shows example of configuring pipeline for `Message` type:

```csharp

void ConfigureFluegramBot(IFluegramBotBuilder builder)
{
    builder.UseFor<EntityContext<Message>, Message>(UpdateType.Message, ConfigureMessagePipeline);
}

void ConfigureMessagePipeline(IPipelineBuilder<EntityContext<Message>, Message> pipelineBuilder)
{
    //Configure your pipeline here
}

```

You could also create your own middlewares if you want to add your functionality:

```csharp

public class CustomMiddleware<TEntityContext, TEntity> : IMiddleware<TEntityContext, TEntity>
    where TEntityContext : IEntityContext<TEntity> where TEntity : class
{
    public async Task ProcessAsync(TEntityContext context, CancellationToken cancellationToken)
    {
        //...
    }
}

```

There're a few modules which uses they own middlewares:

- [Handlers](/modules/handlers)
- [Commands](/modules/commands)
- [Sessions](/modules/sessions)

### Configuring pipeline features

There's an overload for `IFluegramBotBuilder UseFor<TEntityContext, TEntity>` method which has `Action<IPipelineFeaturesConfigurator<TEntityContext, TEntity>> configureFeatures` parameter at the second place, which allows you to add specific features to the pipeline, like menus or other things:

```csharp

void ConfigureFluegramBot(IFluegramBotBuilder builder)
{
    builder.UseFor<EntityContext<Message>, Message>(UpdateType.Message, ConfigureMessagePipelineFeatures, ConfigureMessagePipeline);
}

void ConfigureMessagePipelineFeatures(
    IPipelineFeaturesConfigurator<EntityContext<Message>, Message> pipelineFeaturesConfigurator)
{
    // Configure pipeline features here
}



void ConfigureMessagePipeline(IPipelineBuilder<EntityContext<Message>, Message> pipelineBuilder)
{
    // Configure pipeline here
}

```






