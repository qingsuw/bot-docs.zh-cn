---
title: 从 botbuilder-samples 存储库部署机器人 | Microsoft Docs
description: 将机器人部署到 Azure 云。
keywords: 部署机器人, azure 部署, 发布机器人, az 部署机器人, visual studio 部署机器人, msbot 发布, msbot 克隆
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3ca8ac4bfe14ed20f11a0ab26d8102ac21e60e2b
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/04/2019
ms.locfileid: "55711951"
---
# <a name="deploy-bots-from-botbuilder-samples-repo"></a>从 botbuilder-samples 存储库部署机器人

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

本文介绍如何将 [botbuilder-samples](https://github.com/Microsoft/BotBuilder-Samples) 存储库中的 C# 和 JavaScript 示例部署到 Azure。

部署示例机器人的说明不同于[部署可以通过所有已在 Azure 中预配的资源创建的机器人](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-deploy-az-cli?view=azure-bot-service-4.0&tabs=csharp)的说明。

> [!IMPORTANT]
> 将机器人从 [botbuilder-samples](https://github.com/Microsoft/BotBuilder-Samples) 存储库部署到 Azure 时，会预配 Azure 服务并需要支付服务使用费。
> [计费和成本管理](https://docs.microsoft.com/en-us/azure/billing/)一文可帮助你了解 Azure 计费方式、如何监视使用量与费用，以及如何管理帐户和订阅。

在执行相关步骤之前最好是先通读本文一次，完全了解在部署机器人时涉及到的方方面面。

## <a name="prerequisites"></a>先决条件

- 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/)。
- 安装 [.NET Core SDK](https://dotnet.microsoft.com/download) >= v2.2。 使用 `dotnet --version` 查看所使用的版本。
- 安装最新版本的 [Azure CLI 工具](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)。 使用 `az --version` 查看所使用的版本。
- 安装最新版本的 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) 工具。
  - 如果克隆操作包括 LUIS 或 Dispatch 资源，则需要 [LUIS CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#installation)。
  - 如果克隆操作包括 QnA Maker 资源，则需要 [QnA Maker CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#as-a-cli)。
- 安装 [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started)。
- 安装并配置 [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29)。
- 了解 [.bot](v4sdk/bot-file-basics.md) 文件。

使用 msbot 4.3.2 及更高版本时，需要 Azure CLI 2.0.54 或更高版本。 如果安装了 botservice 扩展，请使用此命令将其删除。

```cmd
az extension remove --name botservice
```

### <a name="c"></a>C\#

 `msbot clone services` 不将代码文件上传到 Azure，只上传 DLL 和一些其他的文件。 在运行此命令之前，需编译示例。

### <a name="service-names"></a>服务名称

除了部署机器人，`msbot clone services` 命令还会将所有关联的服务预配到所选订阅。

如果订阅中已存在这些名称-服务组合，则该命令会生成一个错误。你需要删除部分部署，然后重新开始。 需删除的部署包括 LUIS 应用程序、QnA Maker 知识库以及 Dispatch 模型。

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>使用 az cli 部署 JavaScript 和 C# 机器人

打开命令提示符以登录到 Azure 门户。

```cmd
az login
```

此时会打开一个用于登录的浏览器窗口。

### <a name="set-the-subscription"></a>设置订阅

使用以下命令设置订阅：

```cmd
az account set --subscription "<azure-subscription>"
```

如果你不确定要使用哪个订阅来部署机器人，可以使用 `az account list` 命令查看帐户的 `subscriptions` 列表。

导航到 bot 文件夹。

```cmd
cd <local-bot-folder>
```

### <a name="clone-the-sample"></a>克隆示例

**Azure 订阅帐户**：在继续之前，请先根据用于登录到 Azure 的电子邮件帐户类型阅读适用的说明。

**创建服务**：`msbot clone services` 命令会为机器人创建 Azure 服务。

1. 它会列出要创建的服务，并提示你确认该操作，然后才会继续。 如果你拒绝，该命令会退出，不创建任何服务。
1. 它会要求你通过 Azure 进行身份验证，然后继续操作。

**LUIS 服务**：如果机器人使用 LUIS 或 Dispatch，则需在 `msbot clone services` 命令中包括 LUIS 创作密钥。

#### <a name="msa-email-account"></a>MSA 电子邮件帐户

如果使用 [MSA](https://en.wikipedia.org/wiki/Microsoft_account) 电子邮件帐户，则需要创建要在 `msbot clone services` 命令中使用的 appId 和 appSecret。

- 转到[应用程序注册门户](https://apps.dev.microsoft.com/)。 单击“添加应用”以注册应用程序，创建**应用程序 ID**，然后单击“生成新密码”。
> 注意 - 如果生成的密码包含字符“|”，Azure 将拒绝此密码。 若要解决此问题，请生成另一个新密码。
- 保存刚刚生成的应用程序 ID 和新密码，以便可以在 `msbot clone services` 命令中使用这些信息。
- 若要部署机器人，请使用机器人适用的命令。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --proj-file "<your.csproj>" --name "<bot-name>" --appId "xxxxxxxx" --appSecret "xxxxxxx" --verbose --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>"   --code-dir . --name "<bot-name>" --appId "xxxxxxxx" --appSecret "xxxxxxx" --verbose --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

#### <a name="business-or-school-account"></a>企业或学校帐户

如果使用企业或学校提供的电子邮件帐户登录到 Azure，则无需创建应用程序 ID 和密码。 若要部署机器人，请使用机器人适用的命令。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --proj-file "<your-project-file>" --name "<bot-name>" --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --code-dir . --name "<bot-name>" --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

#### <a name="save-the-secret-used-to-decrypt-bot-file"></a>保存用于解密 .bot 文件的机密

必须注意，部署过程会创建新的 .bot 文件并使用机密将其加密。 在部署机器人期间，命令行中会显示以下消息，要求保存 .bot 文件机密。

`The secret used to decrypt myAzBot.bot is:`
`hT6U1SIPQeXlebNgmhHYxcdseXWBZlmIc815PpK0WWA=`

`NOTE: This secret is not recoverable and you should save it in a safe place according to best security practices.
Copy this secret and use it to open the <file.bot> the first time.`

请保存 .bot 文件机密供稍后使用。 在 Azure 门户中，我们将使用通过 botFileSecret 加密的新 .bot 文件。 如果以后需要更改机器人文件名或机密，请转到门户中的“应用服务设置”->“应用程序设置”部分。 请注意，在 appsettings.json 或 .env 文件中，将使用最新创建的机器人文件更新机器人文件名。  

### <a name="test-your-bot"></a>测试机器人

在仿真器中，使用生产终结点来测试应用。 若要在本地进行测试，请确保机器人在本地计算机上运行。

### <a name="to-update-your-bot-code-in-azure"></a>更新 Azure 中的机器人代码

切勿使用 `msbot clone services` 命令更新 Azure 中的机器人代码。 必须按如下所示使用 `az bot publish` 命令：

```cmd
az bot publish --name "<your-azure-bot-name>" --proj-name "<your-proj-name>" --resource-group "<azure-resource-group>" --code-dir "<folder>" --verbose --version v4
```

| 参数        | 说明 |
|----------------  |-------------|
| `name`      | 首次将机器人部署到 Azure 时使用的名称。|
| `proj-name` | 对于 C#，请使用需要发布的启动项目文件名称（不带 .csproj）。 例如：`EnterpriseBot`。 对于 Node.js，请使用机器人的主入口点。 例如，`index.js`。 |
| `resource-group` | `msbot clone services` 命令使用的 Azure 资源组。|
| `code-dir`  | 指向本地 bot 文件夹。|

## <a name="additional-resources"></a>其他资源

部署机器人时，通常会在 Azure 门户中创建以下资源：

| 资源      | 说明 |
|----------------|-------------|
| Web 应用机器人 | 部署到 Azure 应用服务的 Azure 机器人服务机器人。|
| [应用服务](https://docs.microsoft.com/en-us/azure/app-service/)| 用于生成和托管 Web 应用程序。|
| [应用服务计划](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| 为要运行的 Web 应用定义一组计算资源。|
| [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview)| 提供用于收集和分析遥测数据的工具。|
| [存储帐户](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction)| 提供高度可用、安全、持久、可缩放且冗余的云存储。|

若要查看有关 `az bot` 命令的文档，请参阅[参考](https://docs.microsoft.com/en-us/cli/azure/bot?view=azure-cli-latest)主题。

如果你不熟悉 Azure 资源组，请参阅此[术语](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#terminology)主题。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [设置持续部署](bot-service-build-continuous-deployment.md)
