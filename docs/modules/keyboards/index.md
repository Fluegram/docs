# Keyboard

Another handy feature that adds builders both for `InlineKeyboardMarkup` and `ReplyKeyboardMarkup`.

For example, there is how to create an inline keyboard using `InlineKeyboardMarkupBuilder`:

```csharp

InlineKeyboardMarkupBuilder inlineKeyboardMarkupBuilder = new();

        inlineKeyboardMarkupBuilder.UseRow(rowBuilder => rowBuilder
            .Use(new InlineKeyboardButton("button text"))
            .UseCallbackData("text", "callback:data")
        );

InlineKeyboardMarkup markup = inlineKeyboardMarkupBuilder.Build();

```

`InlineKeyboardMarkupRowBuilder` has the same static factory methods that `InlineKeyboardButton` has, which simplifies creating custom buttons.

Also, `ReplyKeyboardMarkupBuilder` has static factory methods from `KeyboardButton` class.