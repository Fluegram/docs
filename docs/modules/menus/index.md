# Menus

This feature becomes handy if you planning to make your bot with some kind of settings menu, when user could change his language, some king of bot-scoped username, and other things.

Here's an example of creating a complex menu which could be used for changing bot language for the user:


```csharp

public class SettingsMenu : MenuBase
{
    public SettingsMenu() : base("settings", _ => "Settings Menu", _ => "Settings")
    {
    }
}

public class SettingsEditMenu : MenuBase
{
    public SettingsEditMenu() : base("edit", _ => "Settings Edit Menu", _ => "Edit")
    {
    }
}

public class SettingsEditLanguageMenu : MenuBase, IMenuWithAction<EntityContext<CallbackQuery>>
{
    public SettingsEditLanguageMenu() : base("language", _ => "Language Change Menu", _ => "Language")
    {
    }

    public async Task InvokeAsync(EntityContext<CallbackQuery> entityContext, CancellationToken cancellationToken)
    {
        //Here will be your logic for requesting user input and changing language.
    }
}

```

And let's add these menus to the [pipeline features](../../getting-started.md#configuring-pipeline-features):

```csharp

void ConfigureMessagePipelineFeatures(
    IPipelineFeaturesConfigurator<EntityContext<CallbackQuery>, CallbackQuery> pipelineFeaturesConfigurator)
{
    pipelineFeaturesConfigurator.UseMenus(_ => _
        .Use<SettingsMenu>(settingsMenuConfigurator => settingsMenuConfigurator
            .Use<SettingsEditMenu>(settingsEditMenuConfigurator => settingsEditMenuConfigurator
                .Use<SettingsEditLanguageMenu>())));
}


```

Please note, that `text` and `name` parameters is `Func`s, which means they could be localized depending on the user used language.

And then, in any other handler or command you could just send this menu by using `SendMenuAsync` extension method. For example:

```csharp

public class MenuSendHandler : HandlerBase<Message>
{
    public override async Task HandleAsync(EntityContext<Message> entityContext, CancellationToken cancellationToken)
    {
        await entityContext.SendMenuAsync<SettingsMenu>(cancellationToken).ConfigureAwait(false);
    }
}

```