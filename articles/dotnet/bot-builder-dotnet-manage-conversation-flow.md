---
title: 使用对话管理聊天流 | Microsoft Docs
description: 了解如何使用对话框和 Bot Framework SDK for .NET 为聊天建模和管理聊天流。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: cfb849474c23c62a666e013c700b755519c1868a
ms.sourcegitcommit: c6ce4c42fc56ce1e12b45358d2c747fb77eb74e2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/22/2019
ms.locfileid: "54453802"
---
# <a name="manage-conversation-flow-with-dialogs"></a>使用对话框管理会话流

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-manage-conversation-flow.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-manage-conversation-flow.md)

[!INCLUDE [Dialog flow example](../includes/snippet-dotnet-manage-conversation-flow-intro.md)]

本文介绍如何使用[对话框](bot-builder-dotnet-dialogs.md)和 Bot Framework SDK for .NET 为此聊天流建模。 

## <a name="invoke-the-root-dialog"></a>调用根对话

首先，机器人控制器调用“根对话”。 以下示例显示如何将基本 HTTP POST 调用连接到控制器，然后调用根对话。 

```cs
public class MessagesController : ApiController
{
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {
            // Redirect to the root dialog.
            await Conversation.SendAsync(activity, () => new RootDialog()); 
            ...
    }
}
```

## <a name="invoke-the-new-order-dialog"></a>调用“新建订单”对话

接下来，根对话调用“新建订单”对话。 

```cs
[Serializable]
public class RootDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        // Root dialog initiates and waits for the next message from the user. 
        // When a message arrives, call MessageReceivedAsync.
        context.Wait(this.MessageReceivedAsync); 
    }

    public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        var message = await result; // We've got a message!
        if (message.Text.ToLower().Contains("order"))
        {
            // User said 'order', so invoke the New Order Dialog and wait for it to finish.
            // Then, call ResumeAfterNewOrderDialog.
            await context.Forward(new NewOrderDialog(), this.ResumeAfterNewOrderDialog, message, CancellationToken.None);
        }
        // User typed something else; for simplicity, ignore this input and wait for the next message.
        context.Wait(this.MessageReceivedAsync);
    }

    private async Task ResumeAfterNewOrderDialog(IDialogContext context, IAwaitable<string> result)
    {
        // Store the value that NewOrderDialog returned. 
        // (At this point, new order dialog has finished and returned some value to use within the root dialog.)
        var resultFromNewOrder = await result;

        await context.PostAsync($"New order dialog just told me this: {resultFromNewOrder}");

        // Again, wait for the next message from the user.
        context.Wait(this.MessageReceivedAsync);
    }
}
```

## <a id="dialog-lifecycle"></a> 对话生命周期

调用对话时，该对话会控制聊天流。 每条新消息均会由该对话处理，直到该对话关闭或重定向到其他对话。 

在 C# 中，可以使用 `context.Wait()` 指定在下次用户发送消息时调用的回调。 若要关闭对话并将其从堆栈中删除（从而将用户发送回堆栈中的前一个对话），请使用 `context.Done()`。 必须使用 `context.Wait()`、`context.Fail()`、`context.Done()` 或某个重定向指令（如 `context.Forward()` 或 `context.Call()`）结束每个对话方法。 未使用其中一项结束的对话方法将导致错误（因为框架不知道下次用户发送消息时要执行的操作）。

## <a name="passing-state-between-dialogs"></a>在对话之间传递状态

虽然可以将状态存储在机器人状态中，但也可以通过重载对话类构造函数在不同对话之间传递数据。

```cs
[Serializable]
public class AgeDialog : IDialog<int>
{
    private string name;

    public AgeDialog(string name)
    {
        this.name = name;
    }
}
 ```

调用对话代码，从用户传入名称值。

```cs
private async Task NameDialogResumeAfter(IDialogContext context, IAwaitable<string> result)
{
    try
    {
        this.name = await result;
        context.Call(new AgeDialog(this.name), this.AgeDialogResumeAfter);
    }
    catch (TooManyAttemptsException)
    {
        await context.PostAsync("I'm sorry, I'm having issues understanding you. Let's try again.");
        await this.SendWelcomeMessageAsync(context);
    }
}
```

## <a name="sample-code"></a>代码示例 

有关演示如何使用 Bot Framework SDK for .NET 中的对话框管理聊天的完整示例，请参阅 GitHub 中的[基本的多对话框示例](https://aka.ms/v3cs-MultiDialog-Sample)。

## <a name="additional-resources"></a>其他资源

- [对话框](bot-builder-dotnet-dialogs.md)
- [设计和控制聊天流](../bot-service-design-conversation-flow.md)
- [基本多对话示例 (GitHub)](https://aka.ms/v3cs-MultiDialog-Sample)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Framework SDK for .NET 参考</a>
