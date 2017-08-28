# Intermediator Bot Sample #

A bot build on [Microsoft Bot Framework](https://dev.botframework.com/) that
routes messages between two users on different channels. This is a C# sample -
if you're looking to do this with Node, see
[this sample](https://github.com/palindromed/Bot-HandOff).

A possible use case for this type of a bot would be a customer service scenario
where the bot relays the messages between a customer and a customer service agent.

See also [Chatbots as Middlemen blog post](http://tomipaananen.azurewebsites.net/?p=1851)
related to this sample.

## Development branch notes ##

Clone [the Bot Message Routing project}(https://github.com/tompaana/bot-message-routing/)
to the same folder where you have this project (same root folder for both
project). The reference to the Bot Message Routing project is already added to.
this project's solution.

**Note:** The documentation in this README file is **not** up-to-date!

## Getting started ##

To test the bot, publish it in
[Microsoft Bot Framework portal](https://dev.botframework.com) and connect it to
the channels of your choice. If you are new to bots, please familiarize yourself
first with the basics [here](https://dev.botframework.com/).
[Microsoft Bot Framework Emulator](https://docs.botframework.com/en-us/tools/bot-framework-emulator/)
is a great tool for testing and debugging. To communicate with a remotely hosted
bot, you should use [ngrok](https://ngrok.com/) tunneling software:

1. In emulator open **App Settings**
2. Make sure ngrok path is set:
    ![Setting ngrok path in emulator settings](Documentation/Screenshots/SettingNgrokPathInEmulatorSettings.png?raw=true)
3. See the emulator log to verify the path was set correctly:
    ![ngrok info in emulator log](Documentation/Screenshots/NgrokLocalhostPortInEmulatorLog.png?raw=true)
4. Set the bot end point in emulator (`https://<bot URL>/api/messages`)
5. Set **Microsoft App ID** and **Microsoft App Password**
    * **Note:** It is easy to forget to add the app ID and password to the
      `Web.config` file. Make sure you have the values set before you publish
      the bot:
    ![Setting app ID and password in Web.config](Documentation/Screenshots/AppSettingsInWebConfig.png?raw=true)
6. Click **CONNECT** to start a new conversation
    ![Setting end point, app ID and password in emulator](Documentation/Screenshots/SettingBotEndpointAppIdAndPasswordInEmulator.png?raw=true)

See also: [Microsoft Bot Framework Emulator wiki](https://github.com/microsoft/botframework-emulator/wiki/Getting-Started)

### Scenario 1: Channel <-> channel ###

This scenario utilizes an aggregation concept (see the terminology table in this
document). One or more channels act as aggregated channels where the customer
requests (for human assistance) are sent. The conversation owners (e.g. customer
service agents) then accept or reject the requests.

Once you have published the bot, go to the channel you want to receive the
requests and issue the following command to the bot (given that you haven't
changed the default bot command handler or the command itself):

```
@BOT_NAME watch
```
    
In case mentions are not supported, you can also use the command keyword:

```
command watch
```
    
Now all the requests from another channels are forwarded to this channel.
See the default flow below:

| Emulator with ngrok | Slack |
| ------------------- | ----- |
| | ![Initialization](Documentation/Screenshots/Initialization.png?raw=true) |
| ![Request sent](/Documentation/Screenshots/RequestSent.png?raw=true) | ![Request accepted](/Documentation/Screenshots/RequestAccepted.png?raw=true) |
| | ![Direct messaging channel created](/Documentation/Screenshots/DirectMessagingChannelCreated.png?raw=true) |
| ![Conversation in emulator](/Documentation/Screenshots/ConversationInEmulator.png?raw=true) | ![Conversation in Slack](/Documentation/Screenshots/ConversationInSlack.png?raw=true) |

### Scenario 2: Channel <-> call center (agent UI) ###

In this scenario the conversation owners (e.g. customer service agents) access
the bot via the webchat component, [Agent UI](https://github.com/billba/agent),
implemented by [Bill Barnes](https://github.com/billba). Each customer request
(for human assistance) automatically opens a new chat window in the agent UI.

| Emulator | Agent UI |
| -------- | -------- |
| ![Emulator](Documentation/Screenshots/ConversationInEmulatorWithAgentUI.png?raw=true) | ![Agent UI](Documentation/Screenshots/AgentUI.png?raw=true) |

To set this up, follow these steps:

0. Make sure you have [Node.js](https://nodejs.org) installed
1. Clone or download [the Agent UI repository](https://github.com/billba/agent)
2. Inside `index.ts`, update the line below with your bot's endpoint:

    `fetch("http://YOUR_BOT_ENDPOINT/api/agent/1")`
    
    Example: `fetch("http://intermediatorbotsample.azurewebsites.net/api/agent/1")`

3. Inside `index.ts`, update the line below with your bot secret key

    ```js
    iframe.src = 'botchat?s=YOUR_DIRECTLINE_SECRET_ID';
    ```
    
    * The bot secret key can be found in your bot's profile in
      [the portal](https://dev.botframework.com/bots)
    * Click on the **Edit** button next to the **Direct Line** channel to locate the secret key
    
    ![Edit button in the portal](Documentation/Screenshots/RetrievingBotSecret1.png?raw=true)
    
    * If your **Configure Direct Line** page is blank, create a new site by
      clicking **Add new site** and two bot secret keys will be generated for you:
    
    ![Bot secret on Configure Direct Line page](Documentation/Screenshots/RetrievingBotSecret2.png?raw=true)

4. Run `npm install` to get the npm packages 

    * You only need to run this command once, unless you add other node packages to the project
    * If you see error **TS2300**, run `npm install typescript@2.0.10`

5. Run `npm run build` to build the app 

    * You need to run this every time you make changes to the code before you start the application

6. Run `npm run start` to start the app
7. Go to http://localhost:8080 to see the Agent UI

Make sure that the value of `RejectPendingRequestIfNoAggregationChannel` key in
[Web.config](/IntermediatorBotSample/Web.config) is `false`:

```xml
<add key="<add key="RejectPendingRequestIfNoAggregationChannel" value="false" />" value="false" />
```

Otherwise the agent UI will not receive the requests, but they are automatically rejected
(if no aggregation channel is set).

## Implementation ##

TODO

### App Settings ###

A number of App Settings are available in the web.config of this sample which can be used to tailor the experience.

**PermittedAgentChannels** If you wish to only allow conversation owners (i.e. customer service agenst) to use a 
specific channel or channels, you can specify a comma seperated list of channel IDs here.  This will prevent agent 
commands from being used on other channels and prevent users from accidentally or deliberately calling such commands.
e.g. to allow agents to use the emulator and Skype channels you would use 

```
<add key="PermittedAgentChannels" value="emulator,skype" />
```

**RejectPendingRequestIfNoAggregationChannel** This setting, which is set to true by default, will cause the LocalRoutingDataManager 
to return the NoAgentsAvailable result when no agents are watching for incoming requests. You can then send an appropriate response to 
let the user know no agents are available within the implementation of IMessageRouterResultHandler. If this is set to false, then the
LocalRoutingDataManager will process and add the users request to the pending requests list and return the EngagementInitiated result instead. 

### Taking the code into use ###

The most convenient place to use the aforementioned classes is in the
**[MessagesController](/IntermediatorBotSample/Controllers/MessagesController.cs)**
class - you can first call the methods in `MessageRouterManager` and, for
instance, if no action is taken by the manager, you can forward the `Activity`
to a `Dialog`:

```cs
public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
{
    if (activity.Type == ActivityTypes.Message)
    {
        MessageRouterManager messageRouterManager = WebApiConfig.MessageRouterManager;
        IMessageRouterResultHandler messageRouterResultHandler = WebApiConfig.MessageRouterResultHandler;

        messageRouterManager.MakeSurePartiesAreTracked(activity);

        // First check for commands (both from back channel and the ones directly typed)
        MessageRouterResult messageRouterResult = WebApiConfig.BackChannelMessageHandler.HandleBackChannelMessage(activity);

        if (messageRouterResult.Type != MessageRouterResultType.EngagementAdded
            && await WebApiConfig.CommandMessageHandler.HandleCommandAsync(activity) == false)
        {
            // No command detected

            // Let the message router manager instance handle the activity
            messageRouterResult = await messageRouterManager.HandleActivityAsync(activity, false);

            if (messageRouterResult.Type == MessageRouterResultType.NoActionTaken)
            {
                // No action was taken by the message router manager. This means that the user
                // is not engaged in a 1:1 conversation with a human (e.g. customer service
                // agent) yet.
                //
                // You can, for example, check if the user (customer) needs human assistance
                // here or forward the activity to a dialog. You could also do the check in
                // the dialog too...
                //
                // Here's an example:
                if (!string.IsNullOrEmpty(activity.Text) && activity.Text.ToLower().Contains(CommandInitiateEngagement))
                {
                    messageRouterResult = messageRouterManager.InitiateEngagement(activity);
                }
                else
                {
                    await Conversation.SendAsync(activity, () => new RootDialog());
                }
            }
        }

        // Handle the result, if required
        await messageRouterResultHandler.HandleResultAsync(messageRouterResult);
    }
    else
    {
        await HandleSystemMessageAsync(activity);
    }

    var response = Request.CreateResponse(HttpStatusCode.OK);
    return response;
}
```
