---
page_type: sample
products:
- office-365
languages:
- csharp
title: Microsoft Teams C# Helloworld Sample
description: Microsoft Teams "Hello world" application for .NET/C#
extensions:
  contentType: samples
  platforms:
  - CSS
  createdDate: 10/16/2017 10:02:21 PM
---

# Readme

This is a quick and dirty demonstration of resource specific consent (RSC).
The implementation is very inefficient – effectively neither tokens or messages are cached, and any UI update requires refreshing the page.

Key files:

- Microsoft.Teams.Samples.HelloWorld.Web\Controllers\HomeController.cs has the majority of the app logic.
- Microsoft.Teams.Samples.HelloWorld.Web\Views\Home\First.cshtml has the main UI
- Microsoft.Teams.Samples.HelloWorld.Web\Manifest\manifest.json shows how to do the manifest for RSC
- Microsoft.Teams.Samples.HelloWorld.Web\Views\Home\Auth.cshtml and  Microsoft.Teams.Samples.HelloWorld.Web\Views\Home\AuthDone.cshtml and Microsoft.Teams.Samples.HelloWorld.Web\Scripts\teamsapp.js get the user delegated token for the tab.
- AAD-appregistrations has info on how to set up your AAD app registrations. This sample is currently set up with the bot using the same app registration as the RSC graph app, I'm not sure I would do that again – bot framework registrations are set up to use AAD v2 tokens, and that creates a problem where AAD won't show a consent prompt when it's needed unless you explicitly pass in prompt=consent. Currently I'm granting consent for the user token in the admin portal, which is not how you'd want to ship a real multitenant app.

## Installation
I haven't had a chance to write up setup instructions for this app, but it's the same basic steps as copying anyone's Teams app that uses Graph:
- register your own Graph/AAD appid
- set the app up for implicit flow auth, w/ endpoint yourdomain/authdone
- add the appropriate user delegated permissions to that app – I believe it's just User.Read
- Update the teams app manifest with that graph appid, and change the tab URLs to point to wherever your hosting your app. (ngrok presumably, since Azure hosting doesn't work with webhooks yet)
- if you want to use sideloading, make sure the graph app registration is owned by the tenant you are going to sideload into.

## Demo script

git clone https://github.com/nkramer/graphbot.git 

Q&A Tracker is a tab that uses resource-specific consent to read messages and get notified of new messages (Webhooks). It also installs a bot but you should ignore that for now. It should not be used in the production tenant until some additional security work is done in the app.

For reasons I haven't figured out Webhooks aren't working in the Azure-deployed version, so you'll need to run from ngrok.

Q&A Tracker comes in several flavors:
- (Azure) -- this is the main one you should use. 
- (Azure, NoRSC) -- uses the Q+A Tracker graph app w/ user delegated permissions
- (ngrok) -- runs off my local machine :)

You can find all of these versions in the tenant app catalog.

All versions of the app require you to establish your identity by logging in with user delegated permissions.  For the RSC version, its an app that has only the openid permission. It's not an especially clever implementation of tab auth -- it's not even silent sign-on, much less SSO. But there's no reason other than time in the day that we couldn't add those. 

Graph app ids:
- RSC uses "Q+A Tracker" with application permissions, and does authn with "HelloBot" user delegated.
- NoRSC uses the "Q+A Tracker" graph app w/ user delegated permissions.

Both appids are registered in the TEST_TEST_IP_TEST tenant, owned by nkramer & billbl.

Webhook functionality is fragile:
- You can only set up one subscription at a time per tenant! So don't run this app in two separate channels within 10 minutes of each other.
- Subscriptions only last for 10 minutes, to help mitigate the above.
- If the subscription set up fails for some reason, it will fail silently so you can at least demo reading messages. 
- When a new message comes in, it will reload the Q&A tab – even if the tab was not in the same channel as the message. 
- The non-RSC version doesn't support webhooks, because we don't have user delegated webhooks.

Demo setup:
- https://teams.microsoft.com , launch the webclient.
- Log in to @teamsip.onmicrosoft.com. Any account will do.
- switch to R1.5 to avoid showing unrelated UI changes we aren't ready to talk about.
- Create a new team so there's not too much clutter in the general channel.
- Add some questions. Q&A Tracker will only look at root messages (not replies), it only looks at the most recent 30 messages (including control messages), and it only looks at messages that contain a question mark.
- Like some of those questions -- questions will be ranked by reaction count.
- Create a second window, logged into the same account or a different account that's a member of the team you just created
- For the best demo, install your app to the LOB catalog rather than sideload on stage.
- If you have to sideload, you can do it from the Team | Manage Team | Apps | Upload a custom app screen. 
- For maximum sanity, uninstall your app before you install a new version.

Demo steps:
- Click + Pin a tab button. Pick "Q&A Tracker (Azure)". Search in tenant app catalog (Built for TEST_TEST_IP_TEST) if necessary.
- See app details V3. Scroll to bottom of dialog, talk about the new RSC permissions.
- Click Add.
- Uncheck "Post to the channel about this tab" to reduce the clutter in your channel.
- Click Save.
- If tab errors out, refresh – we've have some live sites today due to a CSA deployment.
- Mark one or two questions as answered so the app looks more real
- go to your other window, and type in a new question. (Must be a root message, must contain a question mark)
- Wait ~5 seconds for it to show up in Q&A Tracker. (Most of this is webhook – despite Q&A Tracker rerunning from scratch everything on both client and server with every new message, the webhook is the slow part.)

Good demo questions:
- Are we open for the holidays?
- Is the business growing?
- How do we beat the competition?
- Are we moving offices?
- Are we done yet?
