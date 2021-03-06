---
title: 调试机器人 | Microsoft Docs
description: 了解如何使用机器人服务调试机器人。
author: v-ducvo
ms.author: v-ducvo
keywords: Bot Framework SDK, 调试机器人, 测试机器人, 机器人模拟器, 模拟器
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 2/26/2019
ms.openlocfilehash: 1e806358ffdba90848f0c8124c8315fd4e2dec76
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224835"
---
# <a name="debug-a-bot"></a>调试机器人

本文介绍如何使用 Visual Studio 或 Visual Studio Code 和 Bot Framework Emulator 等集成开发环境 (IDE) 调试机器人。 虽然可以使用这些方法在本地调试任何机器人，但本文使用了在快速入门中创建的 [C#](~/dotnet/bot-builder-dotnet-sdk-quickstart.md) 和 [JS](~/javascript/bot-builder-javascript-quickstart.md) 机器人。

## <a name="prerequisites"></a>先决条件 
- 下载并安装 [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started)。
- 下载并安装 [Visual Studio Code](https://code.visualstudio.com) 或 [Visual Studio](https://www.visualstudio.com/downloads)（Community Edition 或更高版本）。

### <a name="debug-a-javascript-bot-using-command-line-and-emulator"></a>使用命令行和模拟器调试 JavaScript 机器人

若要使用命令行运行 JavaScript 机器人，并使用模拟器测试机器人，请执行以下步骤：
1. 在命令行中，将目录更改为机器人项目目录。
1. 运行命令 node app.js，启动机器人。
1. 启动模拟器并连接到机器人的终结点（例如： **http://localhost:3978/api/messages** ）。 如果这是首次运行机器人，请单击“文件”>“新建机器人”，然后按照屏幕上的说明操作。 否则，单击“文件”>“打开机器人”打开现有机器人。 由于此机器人正在你的计算机上本地运行，可以将“MSA 应用 ID”和“MSA 应用密码”字段留空。 有关详细信息，请参阅[使用模拟器调试](bot-service-debug-emulator.md)。
1. 在模拟器中向机器人发送一条消息（例如：发送消息“Hi”）。 
1. 使用仿真器窗口右侧的“检查器”和“日志”面板调试机器人。 例如，单击任何消息气泡（例如，下面屏幕截图中的“Hi”消息气泡），将在“检查器”面板中显示该消息的详细信息。 由于消息是在模拟器和机器人之间交换，因此，可以使用它来查看请求和响应。 另外，还可以在“日志”窗格中单击任何链接的文本，在“检查器”窗格中查看详细信息。


   ![仿真器上的“检查器”窗格](~/media/bot-service-debug-bot/emulator_inspector.png)

### <a name="debug-a-javascript-bot-using-breakpoints-in-visual-studio-code"></a>在 Visual Studio Code 中使用断点调试 JavaScript 机器人

在 Visual Studio Code 中，可以设置断点并以调试模式运行机器人，逐步执行代码。 若要在 VS Code 中设置断点，请执行以下操作：

1. 启动 VS Code 并打开机器人项目文件夹。
2. 在菜单栏中单击“调试”，然后单击“开始调试”。 如果系统提示你选择一个运行时引擎来运行代码，请选择“Node.js”。 此时，机器人在本地运行。 
<!--
   > [!NOTE]
   > If you get the "Value cannot be null" error, check to make sure your **Table Storage** setting is valid.
   > The **EchoBot** is default to using **Table Storage**. To use Table Storage in your bot, you need the table *name* and *key*. If you do not have a Table Storage instance ready, you can create one or for testing purposes, you can comment out the code that uses **TableBotDataStore** and uncomment the line of code that uses **InMemoryDataStore**. The **InMemoryDataStore** is intended for testing and prototyping only.
-->
3. 根据需要设置断点。 在 VS Code 中，可以将鼠标悬停在行号左边的列上方设置断点。 将出现一个小红点。 单击该点即可设置断点。 再次单击该点可移除断点。

   ![在 VS Code 中设置断点](~/media/bot-service-debug-bot/breakpoint-set.png)

4. 启动 Bot Framework Emulator 并连接到机器人，如以上部分所述。 
5. 在模拟器中向机器人发送一条消息（例如：发送消息“Hi”）。 执行操作会在你放置断点所在的行停止。

   ![在 VS Code 中调试](~/media/bot-service-debug-bot/breakpoint-caught.png)

### <a name="debug-a-c-bot-using-breakpoints-in-visual-studio"></a>在 Visual Studio 中使用断点调试 C# 机器人

在 Visual Studio (VS) 中，可以设置断点并以调试模式运行机器人，逐步执行代码。 若要在 VS 中设置断点，请执行以下操作：

1. 导航到你的机器人文件夹并打开 .sln 文件。 这将在 VS 中打开解决方案。
2. 在菜单栏中单击“生成”，并单击“生成解决方案”。
3. 在**解决方案资源管理器**中，单击“EchoWithCounterBot.cs”。 此文件根据需要定义你的主要机器人 logic.Set 断点。 在 VS 中，可以将鼠标悬停在行号左边的列上方设置断点。 将出现一个小红点。 单击该点即可设置断点。 再次单击该点可移除断点。
5. 在菜单栏中单击“调试”，然后单击“开始调试”。 此时，机器人在本地运行。 

<!--
   > [!NOTE]
   > If you get the "Value cannot be null" error, check to make sure your **Table Storage** setting is valid.
   > The **EchoBot** is default to using **Table Storage**. To use Table Storage in your bot, you need the table *name* and *key*. If you do not have a Table Storage instance ready, you can create one or for testing purposes, you can comment out the code that uses **TableBotDataStore** and uncomment the line of code that uses **InMemoryDataStore**. The **InMemoryDataStore** is intended for testing and prototyping only.
-->

   ![在 VS 中设置断点](~/media/bot-service-debug-bot/breakpoint-set-vs.png)

7. 启动 Bot Framework Emulator 并连接到机器人，如以上部分所述。 
8. 在模拟器中向机器人发送一条消息（例如：发送消息“Hi”）。 执行操作会在你放置断点所在的行停止。

   ![在 VS 中调试](~/media/bot-service-debug-bot/breakpoint-caught-vs.png)

::: moniker range="azure-bot-service-3.0" 

## <a id="debug-csharp-serverless"></a> 调试消耗计划 C\# 函数机器人

与典型 C\# 应用程序相比，机器人服务中的消耗计划无服务器 C\# 环境与 Node.js 有更多的共同点，因为它需要运行时主机，与节点引擎非常相似。 在 Azure 中，运行时是云中托管环境的一部分，但必须在桌面上本地复制该环境。 

### <a name="prerequisites"></a>先决条件

在调试消耗计划 C# 机器人之前，必须完成以下任务。

- （从 Azure）下载机器人源代码，如[设置持续开发](bot-service-continuous-deployment.md)中所述。
- 下载并安装 [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started)。
- 安装 <a href="https://www.npmjs.com/package/azure-functions-cli" target="_blank">Azure Functions CLI</a>。
- 安装 <a href="https://github.com/dotnet/cli" target="_blank">DotNet CLI</a>。
  
如果你希望能够在 Visual Studio 2017 中使用断点来调试代码，还必须完成以下任务。
  
- 下载并安装 <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017</a>（Community Edition 或更高版本）。
- 下载并安装<a href="https://visualstudiogallery.msdn.microsoft.com/e6bf6a3d-7411-4494-8a1e-28c1a8c4ce99" target="_blank">命令任务运行程序 Visual Studio 扩展</a>。

> [!NOTE]
> 目前不支持 Visual Studio Code。

### <a name="debug-a-consumption-plan-c-functions-bot-using-the-emulator"></a>使用模拟器调试消耗计划 C# 函数机器人

在本地调试机器人的最简单方法是启动机器人，然后从 Bot Framework Emulator 连接到它。 
首先，打开命令提示符并导航到其中 project.json 文件位于存储库中的那个文件夹。 然后，运行命令 `dotnet restore`，还原在机器人中引用的各种包。

> [!NOTE]
> Visual Studio 2017 更改 Visual Studio 处理依赖关系的方式。 Visual Studio 2015 使用 project.json 处理依赖项，而在 Visual Studio 中加载时，Visual Studio 2017 使用 .csproj 模型。 如果使用的是 Visual Studio 2017，请<a href="https://aka.ms/bf-debug-project">下载此 .csproj 文件</a>到存储库中的 /messages 文件夹，然后再运行 `dotnet restore` 命令。

![命令提示符](~/media/bot-service-debug-bot/csharp-azureservice-debug-envconfig.png)

接下来，运行 `debughost.cmd` 来加载和启动机器人。 

![命令提示符运行 debughost.cmd](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughost.png)

此时，机器人在本地运行。 在控制台窗口中，复制 debughost 侦听所在的终结点（本示例中为 `http://localhost:3978`）。 然后，启动 Bot Framework Emulator 并将该终结点粘贴到模拟器的地址栏。 在本示例中，还必须将 `/api/messages` 附加到终结点。 由于不需要具备对本地调试的安全性，可以将“Microsoft 应用 ID”和“Microsoft 应用 密码”字段留空。 单击“连接”，使用指定的终结点建立到机器人的连接。

![配置模拟器](~/media/bot-service-debug-bot/mac-azureservice-emulator-config.png)

将模拟器连接到机器人后，在位于模拟器窗口底部（即，左下角显示的“键入消息...”）的文本框中键入一些文本，向机器人发送一条消息。 使用位于模拟器窗口右侧的“日志”和“监测器”面板，可以查看请求和响应，因为消息是在模拟器和机器人之间进行交换。

![通过模拟器测试](~/media/bot-service-debug-bot/mac-azureservice-debug-emulator.png)

另外，还可以在控制台窗口中查看日志详细信息。

![控制台窗口](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughostlogging.png)

::: moniker-end

## <a name="additional-resources"></a>其他资源

- 请参阅[排查常见问题](bot-service-troubleshoot-bot-configuration.md)和该部分中的其他疑难解答文章。
- 请参阅如何[使用 ngrok 在本地调试任何通道](https://blog.botframework.com/2017/10/19/debug-channel-locally-using-ngrok/)博客文章。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用脚本文件调试机器人](v4sdk/bot-builder-debug-transcript.md)。
