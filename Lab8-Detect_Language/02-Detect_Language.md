# Lab 3 - Detect Language

In this lab we are going to integrate language detection ability of cognitive services into our bot.

## Lab 3.1: Retrieve your Cognitive Services url and keys

1. Open the [Azure Portal](https://portal.azure.com)

2. Navigate to your resource group, select the cognitive services resource that is generic (aka, it contains all end points).

3. Under **RESOURCE MANAGEMENT**, select the **Quick Start** tab and record the url and the key for the cognitive services resource

## Lab 3.2: Add language support to your bot

1. Please open the **PictureBot** solution from **C:\AllFiles\AI-100-Design-Implement-Azure-AISol-master\Lab8-Detect_Language\code\Finished\PictureBot.sln**

      >**Note:** please make sure we are selecting the solution from the finished folder

2. Right-click the project and select **Manage Nuget Packages**

3. Select **Browse**

4. Search for **Microsoft.Azure.CognitiveServices.Language.TextAnalytics**, select it then select **Install**, then select **I Accept**

5. Open the **Startup.cs** file, Check the following using statements:

```csharp
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics;
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
```

6. Check the following code to the **ConfigureServices** method:

```csharp
services.AddSingleton(sp =>
{
    string cogsBaseUrl = Configuration.GetSection("cogsBaseUrl")?.Value;
    string cogsKey = Configuration.GetSection("cogsKey")?.Value;

    var credentials = new ApiKeyServiceClientCredentials(cogsKey);
    TextAnalyticsClient client = new TextAnalyticsClient(credentials)
    {
        Endpoint = cogsBaseUrl
    };

    return client;
});
```

7. Open the **PictureBot.cs** file, check the following using statements:

```csharp
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics;
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics.Models;
```

8. check the following class variable:

```csharp
private TextAnalyticsClient _textAnalyticsClient;
```

9. Check whether the constructor includes the new TextAnalyticsClient:

```csharp
public PictureBot(PictureBotAccessors accessors, ILoggerFactory loggerFactory,LuisRecognizer recognizer, TextAnalyticsClient analyticsClient)
```

10. Inside the constructor, check the class variable is initialized:

```csharp
_textAnalyticsClient = analyticsClient;
```

11. Navigate to the **OnTurnAsync** method and find the following line of code:

```csharp
var utterance = turnContext.Activity.Text;
var state = await _accessors.PictureState.GetAsync(turnContext, () => new PictureState());
state.UtteranceList.Add(utterance);
await _accessors.ConversationState.SaveChangesAsync(turnContext);
```

12. Check the following line of code is present after it

```csharp
//Check the language
var result = _textAnalyticsClient.DetectLanguage(turnContext.Activity.Text, "us");

switch (result.DetectedLanguages[0].Name)
{
    case "English":
        break;
    default:
        //throw error
        await turnContext.SendActivityAsync($"I'm sorry, I can only understand English. [{result.DetectedLanguages[0].Name}]");
        return;
        break;
}
```

13. Open the **appsettings.json** file and ensure that your cognitive services settings are entered:

```csharp
"cogsBaseUrl": "",
"cogsKey" :  ""
```

`Note: For cogsBaseURL and cogsKey you will get in your azure environment`

14. Press **F5** to start your bot

15. Using the Bot Emulator, send in a few phrases and see what happens:

- Como Estes?
- Bon Jour!
- Привет
- Hello

## Going further

Since we have already introduced you to LUIS in previous labs, think about what changes you may need to make to support multiple languages using LUIS.  Some helpful articles:

- [Language and region support for LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-language-support)

## Resources

- [Example: Detect language with Text Analytics](https://docs.microsoft.com/en-us/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-language-detection)
- [Quickstart: Text analytics client library for .NET](https://docs.microsoft.com/en-us/azure/cognitive-services/text-analytics/quickstarts/csharp)
