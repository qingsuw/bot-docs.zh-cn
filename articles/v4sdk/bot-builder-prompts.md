---
title: 使用对话提示收集用户输入 | Microsoft Docs
description: 了解如何在 Bot Framework SDK 中使用对话框库提示用户输入。
keywords: 提示, 用户输入, 对话框, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, 重新提示, 验证
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 68c01b0f12790393fe0ee7ae0bd28addf2d26ae7
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591118"
---
# <a name="gather-user-input-using-a-dialog-prompt"></a>使用对话提示收集用户输入

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

通过发布问题来收集信息是机器人与用户交互的主要方式之一。 使用对话库可以轻松提问和验证响应，以确保响应与特定的数据类型匹配或符合自定义的验证规则。 本主题详细介绍如何从瀑布对话创建和调用提示。

## <a name="prerequisites"></a>先决条件

- 本文中的代码基于 **DialogPromptBot** 示例。 需要获取 [C# 示例](https://aka.ms/dialog-prompt-cs)或 [JS 示例](https://aka.ms/dialog-prompt-js)的副本。
- 需要基本了解[对话库](bot-builder-concept-dialog.md)以及如何[管理聊天](bot-builder-dialog-manage-conversation-flow.md)。
- 用于测试的 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator)。

## <a name="using-prompts"></a>使用提示

仅当对话和提示位于同一对话集时，对话才能使用提示。 可以在某个对话的多个步骤中，以及在同一个对话集的多个对话中，使用同一个提示。 但是，在初始化时，需将自定义验证关联到提示。 若要对相同类型的提示使用不同的验证，需要创建提示类型的多个实例，每个实例具有自身的验证代码。

### <a name="define-a-state-property-accessor-for-the-dialog-state"></a>为对话状态定义状态属性访问器

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

本文中使用的“对话提示”示例提示用户输入预订信息。 为了管理聚会规模和日期，我们在 DialogPromptBot.cs 文件中为预订信息定义了一个内部类。

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Location { get; set; }

    public string Date { get; set; }
}
```

接下来，为预订数据添加一个状态属性访问器。

```csharp
public class DialogPromptBotAccessors
{
    public DialogPromptBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState
            ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; }
        = "DialogPromptBotAccessors.DialogState";
    public static string ReservationAccessorKey { get; }
        = "DialogPromptBotAccessors.Reservation";

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<DialogPromptBot.Reservation> ReservationAccessor { get; set; }

    public ConversationState ConversationState { get; }
}
```

在 Startup.cs 中，更新 `ConfigureServices` 方法以设置访问器。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    IStorage dataStore = new MemoryStorage();
    var conversationState = new ConversationState(dataStore);

    // Create and register state accesssors.
    services.AddSingleton<DialogPromptBotAccessors>(sp =>
    {
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new DialogPromptBotAccessors(conversationState)
        {
            DialogStateAccessor =
                conversationState.CreateProperty<DialogState>(
                    DialogPromptBotAccessors.DialogStateAccessorKey),
            ReservationAccessor =
                conversationState.CreateProperty<DialogPromptBot.Reservation>(
                    DialogPromptBotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });

    // ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

无需对 JavaScript 的 HTTP 服务代码进行更改，可将 index.js 文件保留原样。

在 bot.js 中，包含对话提示机器人所需的 `require` 语句。

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, ChoicePrompt, DialogTurnStatus }
    = require('botbuilder-dialogs');
```

添加状态属性访问器、对话和提示的标识符。

```javascript
// Define identifiers for our state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';

// Define identifiers for our dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const SIZE_RANGE_PROMPT = 'rangePrompt';
const LOCATION_PROMPT = 'locationPrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="create-a-dialog-set-and-prompts"></a>创建对话集和提示

一般情况下，应在初始化机器人时创建提示和对话并将其添加到对话集。 然后，对话集可以在机器人从用户接收输入时解析提示的 ID。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 `DialogPromptBot` 类中，定义对话、提示以及需在对话中跟踪的值的标识符。

```csharp
// Define identifiers for our dialogs and prompts.
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partySizePrompt";
private const string SizeRangePrompt = "sizeRangePrompt";
private const string LocationPrompt = "locationPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";

// Define keys for tracked values within the dialog.
private const string LocationKey = "location";
private const string PartySizeKey = "partySize";
```

在机器人的构造函数中，创建对话集、添加提示，然后添加预订对话。 在创建提示时包含自定义验证，稍后我们将实现验证函数。

```csharp
private readonly DialogSet _dialogSet;
private readonly DialogPromptBotAccessors _accessors;

// ...

// Initializes a new instance of the <see cref="DialogPromptBot"/> class.
public DialogPromptBot(DialogPromptBotAccessors accessors, ILoggerFactory loggerFactory)
{
    // ...

    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);

    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
    _dialogSet.Add(new NumberPrompt<int>(SizeRangePrompt, RangeValidatorAsync));
    _dialogSet.Add(new ChoicePrompt(LocationPrompt));
    _dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));

    // Define the steps of the waterfall dialog and add it to the set.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptForPartySizeAsync,
        PromptForLocationAsync,
        PromptForReservationDateAsync,
        AcknowledgeReservationAsync,
    };

    _dialogSet.Add(new WaterfallDialog(ReservationDialog, steps));
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在构造函数中创建状态访问器属性。
接下来，创建对话集并添加提示，包括自定义验证。
然后定义瀑布对话的步骤，并将该对话添加到该集。

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;

    // Create the dialog set and add the prompts, including custom validation.
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(SIZE_RANGE_PROMPT, this.rangeValidator));
    this.dialogSet.add(new ChoicePrompt(LOCATION_PROMPT));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));

    // Define the steps of the waterfall dialog and add it to the set.
    this.dialogSet.add(new WaterfallDialog(RESERVATION_DIALOG, [
        this.promptForPartySize.bind(this),
        this.promptForLocation.bind(this),
        this.promptForReservationDate.bind(this),
        this.acknowledgeReservation.bind(this),
    ]));
}
```

---

### <a name="implement-dialog-steps"></a>实现对话步骤

在机器人主文件中，实现瀑布对话的每个步骤。 添加提示后，在瀑布对话的某个步骤中调用它，并在下一个对话步骤中获取提示结果。 若要从瀑布步骤内部调用提示，请调用瀑布步骤上下文对象的 _prompt_ 方法。 第一个参数是要使用的提示的 ID，第二个参数包含提示的选项，例如，用于请求用户输入的文本。

这些方法介绍：

- 如何从瀑布步骤调用提示，包括如何传递提示选项。
- 如何使用 _validations_ 属性为自定义验证程序提供其他参数。
- 如何使用 _choices_ 属性为选项提示提供选项。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **DialogPromptBot.cs** 文件中，实现瀑布对话的步骤。

在这里，我们演示瀑布的头两个步骤：`PromptForPartySizeAsync` 和 `PromptForLocationAsync`。

```csharp
// First step of the main dialog: prompt for party size.
private async Task<DialogTurnResult> PromptForPartySizeAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        SizeRangePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
            Validations = new Range { Min = 3, Max = 8 },
        },
        cancellationToken);
}

// Second step of the main dialog: prompt for location.
private async Task<DialogTurnResult> PromptForLocationAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    // Record the party size information in the current dialog state.
    var size = (int)stepContext.Result;
    stepContext.Values[PartySizeKey] = size;

    // Prompt for the location.
    return await stepContext.PromptAsync(
        LocationPrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Please choose a location."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a location from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "Redmond", "Bellevue", "Seattle" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **bot.js** 文件中，实现瀑布对话的步骤。

在这里，我们演示瀑布的头两个步骤：`promptForPartySize` 和 `promptForLocation`。

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        SIZE_RANGE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?',
            validations: { min: 3, max: 8 },
        });
}

async promptForLocation(stepContext) {
    // Record the party size information in the current dialog state.
    stepContext.values.size = stepContext.result;

    // Prompt for location.
    return await stepContext.prompt(LOCATION_PROMPT, {
        prompt: 'Please choose a location.',
        retryPrompt: 'Sorry, please choose a location from the list.',
        choices: ['Redmond', 'Bellevue', 'Seattle'],
    });
}
```

---

_prompt_ 方法的第二个参数采用提示选项对象，该对象包含以下属性。

| 属性 | 说明 |
| :--- | :--- |
| _prompt_ | 发送给用户的、以请求输入的初始活动。 |
| _retry prompt_ | 未验证用户的第一次输入时发送给用户的活动。 |
| _choices_ | 供用户选择的选项列表，与选项提示配合使用。 |
| _validations_ | 其他与自定义验证程序配合使用的参数。 |

一般而言，prompt 和 retry prompt 属性属于活动，不过，在不同的编程语言中，这些属性的处理方式有一定的差异。

始终应该指定要向用户发送的初始提示活动。

当用户输入采用的是提示无法分析的格式（例如，用于数字提示的“tomorrow”），或者不符合验证条件，因而无法验证时，指定 retry prompt 很有用。 在这种情况下，如果未提供 retry prompt，则提示将使用初始提示活动来重新提示用户输入。

对于选项提示，应始终提供可用选项的列表。

## <a name="custom-validation"></a>自定义验证

可以在将值返回到**瀑布**对话框的下一个步骤之前验证提示响应。 验证程序函数包含提示验证程序上下文参数，并返回一个布尔值用于指示输入是否通过了验证。

提示验证程序上下文包含以下属性：

| 属性 | 说明 |
| :--- | :--- |
| _上下文_ | 机器人的当前轮次上下文。 |
| _Recognized_ | 提示识别器结果，包含识别器处理的有关用户输入的信息。 |
| 选项 | 包含提示选项，这些选项已在启动提示所需的调用中提供。 |

提示识别器结果包含以下属性：

| 属性 | 说明 |
| :--- | :--- |
| 成功 | 指示识别器是否能够分析输入。 |
| _值_ | 识别器的返回值。 如果必要，验证代码可以修改此值。 |

### <a name="implement-validation-code"></a>实现验证代码

在初始化时，请在机器人的构造函数中将自定义验证关联到提示。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// ...
_dialogSet.Add(new NumberPrompt<int>(SizeRangePrompt, RangeValidatorAsync));
// ...
_dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));
// ...
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// ...
this.dialogSet.add(new NumberPrompt(SIZE_RANGE_PROMPT, this.rangeValidator));
// ...
this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));
// ...
```

---

**聚会规模验证程序**

我们会限制能够进行预订的聚会的大小。 有效范围通过 _validations_ 属性来定义，该属性已用于调用聚会大小提示。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the party size is appropriate to make a reservation.
private async Task<bool> RangeValidatorAsync(
    PromptValidatorContext<int> promptContext,
    CancellationToken cancellationToken)
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the number of people in your party.",
            cancellationToken: cancellationToken);
        return false;
    }

    // Check whether the party size is appropriate.
    var size = promptContext.Recognized.Value;
    var validRange = promptContext.Options.Validations as Range;
    if (size < validRange.Min || size > validRange.Max)
    {
        await promptContext.Context.SendActivitiesAsync(
            new Activity[]
            {
                MessageFactory.Text($"Sorry, we can only take reservations for parties " +
                    $"of {validRange.Min} to {validRange.Max}."),
                promptContext.Options.RetryPrompt,
            },
            cancellationToken: cancellationToken);
        return false;
    }

    return true;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async rangeValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the number of people in your party.");
        return false;
    }
    else if (promptContext.recognized.value % 1 != 0) {
        await promptContext.context.sendActivity(
            "I'm sorry, I don't understand fractional people.");
        return false;
    }

    // Check whether the party size is appropriate.
    var size = promptContext.recognized.value;
    if (size < promptContext.options.validations.min
        || size > promptContext.options.validations.max) {
        await promptContext.context.sendActivity(
            'Sorry, we can only take reservations for parties of '
            + `${promptContext.options.validations.min} to `
            + `${promptContext.options.validations.max}.`);
        await promptContext.context.sendActivity(promptContext.options.retryPrompt);
        return false;
    }

    return true;
}
```

---

**日期时间验证**

在预订日期验证程序中，将预订限制为从当前时间算起的一个小时或以上。 我们将保留与条件匹配的第一个解析，并清除剩余的解析。

以下验证代码并不详尽，它最适合用于分析成日期和时间的输入。 此代码演示了一些用于验证日期时间提示的选项，你的实现将取决于尝试从用户收集的信息。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the reservation date is appropriate.
private async Task<bool> DateValidatorAsync(
    PromptValidatorContext<IList<DateTimeResolution>> promptContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.",
            cancellationToken: cancellationToken);

        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    var earliest = DateTime.Now.AddHours(1.0);
    var value = promptContext.Recognized.Value.FirstOrDefault(v =>
        DateTime.TryParse(v.Value ?? v.Start, out var time) && DateTime.Compare(earliest, time) <= 0);

    if (value != null)
    {
        promptContext.Recognized.Value.Clear();
        promptContext.Recognized.Value.Add(value);
        return true;
    }

    await promptContext.Context.SendActivityAsync(
            "I'm sorry, we can't take reservations earlier than an hour from now.",
            cancellationToken: cancellationToken);

    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async dateValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.");
        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    const earliest = Date.now() + (60 * 60 * 1000);
    let value = null;
    promptContext.recognized.value.forEach(candidate => {
        // TODO: update validation to account for time vs date vs date-time vs range.
        const time = new Date(candidate.value || candidate.start);
        if (earliest < time.getTime()) {
            value = candidate;
        }
    });
    if (value) {
        promptContext.recognized.value = [value];
        return true;
    }

    await promptContext.context.sendActivity(
        "I'm sorry, we can't take reservations earlier than an hour from now.");
    return false;
}
```

---

日期时间提示返回与用户输入匹配的可能日期时间解析列表或数组。 例如，9:00 可能表示 9 AM 或 9 PM，而 Sunday 也可能有歧义。 此外，日期时间解析可以表示日期、时间、日期时间或范围。 日期时间提示使用 [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) 分析用户输入。

## <a name="update-the-turn-handler"></a>更新轮次处理程序

更新机器人的轮次处理程序，以启动对话，并在对话完成时接受对话的返回值。 此处假设用户正在与机器人交互，该机器人具有活动的瀑布对话，并且该对话中的下一个步骤使用某个提示。

当用户向机器人发送消息时，会执行以下操作：

1. 机器人检索状态信息。
1. 机器人创建对话上下文
    - 如果没有活动对话，机器人会启动对话，除非用户已进行了预订。
    - 如果有活动对话，机器人会继续该对话。 如果对话结束，预订详细信息会记录到状态缓存中。
1. 机器人会保存对状态所做的更改。

当对话中的某个步骤调用步骤上下文的 _prompt_ 方法时：

1. 将会创建提示的新实例并将其置于对话堆栈上，然后启动它。 （主对话在继续之前会等待提示结束。）
1. 提示会向用户发送活动，要求用户提供输入。

将输入发送到提示时：

1. 提示会尝试根据提示类型（例如数字提示、选项提示，等等）处理输入。
1. 如果提示包含自定义验证，则运行自定义验证代码。
1. 如果输入通过了所有验证，则提示结束，返回已处理的输入；否则，提示会再次重新自行启动。

**处理提示结果**

如何处理提示结果取决于从用户请求信息的原因。 选项包括：

- 使用信息来控制对话流（例如，在用户对确认或选项提示做出响应时）。
- 在对话的状态中缓存信息（例如，在瀑布步骤上下文的 _values_ 属性中设置一个值），然后在对话结束时返回收集的信息。
- 将信息保存到机器人状态。 这需要将对话设计为有权访问机器人的状态属性访问器。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(
    ITurnContext turnContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    switch (turnContext.Activity.Type)
    {
        // On a message from the user:
        case ActivityTypes.Message:

            // Get the current reservation info from state.
            var reservation = await _accessors.ReservationAccessor.GetAsync(
                turnContext,
                () => null,
                cancellationToken);

            // Generate a dialog context for our dialog set.
            var dc = await _dialogSet.CreateContextAsync(turnContext, cancellationToken);

            if (dc.ActiveDialog is null)
            {
                // If there is no active dialog, check whether we have a reservation yet.
                if (reservation is null)
                {
                    // If not, start the dialog.
                    await dc.BeginDialogAsync(ReservationDialog, null, cancellationToken);
                }
                else
                {
                    // Otherwise, send a status message.
                    await turnContext.SendActivityAsync(
                        $"We'll see you on {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }
            else
            {
                // Continue the dialog.
                var dialogTurnResult = await dc.ContinueDialogAsync(cancellationToken);

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.Status is DialogTurnStatus.Complete)
                {
                    reservation = (Reservation)dialogTurnResult.Result;
                    await _accessors.ReservationAccessor.SetAsync(
                        turnContext,
                        reservation,
                        cancellationToken);

                    // Send a confirmation message to the user.
                    await turnContext.SendActivityAsync(
                        $"Your party of {reservation.Size} is confirmed for " +
                        $"{reservation.Date} in {reservation.Location}.",
                        cancellationToken: cancellationToken);
                }
            }

            // Save the updated dialog state into the conversation state.
            await _accessors.ConversationState.SaveChangesAsync(
                turnContext, false, cancellationToken);
            break;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    switch (turnContext.activity.type) {
        case ActivityTypes.Message:
            // Get the current reservation info from state.
            const reservation = await this.reservationAccessor.get(turnContext, null);

            // Generate a dialog context for our dialog set.
            const dc = await this.dialogSet.createContext(turnContext);

            if (!dc.activeDialog) {
                // If there is no active dialog, check whether we have a reservation yet.
                if (!reservation) {
                    // If not, start the dialog.
                    await dc.beginDialog(RESERVATION_DIALOG);
                }
                else {
                    // Otherwise, send a status message.
                    await turnContext.sendActivity(
                        `We'll see you on ${reservation.date}.`);
                }
            }
            else {
                // Continue the dialog.
                const dialogTurnResult = await dc.continueDialog();

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.status === DialogTurnStatus.complete) {
                    await this.reservationAccessor.set(
                        turnContext,
                        dialogTurnResult.result);

                    // Send a confirmation message to the user.
                    await turnContext.sendActivity(
                        `Your party of ${dialogTurnResult.result.size} is ` +
                        `confirmed for ${dialogTurnResult.result.date} in ` +
                        `${dialogTurnResult.result.location}.`);
                }
            }

            // Save the updated dialog state into the conversation state.
            await this.conversationState.saveChanges(turnContext, false);
            break;
        case ActivityTypes.EndOfConversation:
        case ActivityTypes.DeleteUserData:
            break;
        default:
            break;
    }
}
```

---

可以使用类似技术来验证任何提示类型的提示响应。

## <a name="test-your-bot"></a>测试机器人

1. 在计算机本地运行示例。 如需说明，请参阅适用于 [C#](https://aka.ms/dialog-prompt-cs) 或 [JS](https://aka.ms/dialog-prompt-js) 的自述文件。
2. 启动仿真器，并按如下所示发送消息来测试机器人。

![测试对话提示示例](~/media/emulator-v4/test-dialog-prompt.png)

## <a name="additional-resources"></a>其他资源

若要直接从轮次处理程序调用提示，请参阅 [C#](https://aka.ms/cs-prompt-validation-sample) 或 [JS](https://aka.ms/js-prompt-validation-sample) _prompt-validations_ 示例。

对话库还包含用于获取 OAuth 令牌的 OAuth 提示，代表用户访问另一应用程序时需要该令牌。 有关身份验证的详细信息，请参阅如何[将身份验证添加到机器人](bot-builder-authentication.md)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用分支和循环创建高级聊天流](bot-builder-dialog-manage-complex-conversation-flow.md)
