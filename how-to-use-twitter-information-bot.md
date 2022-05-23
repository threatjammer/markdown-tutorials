---
title: 'How to use the Threat Jammer Twitter information bot'
excerpt: 'This article explains how to use the Twitter information bot to share IP address assessment details in a conversation.'
coverImage: '/tutorialsimg/twitter-threatjammer-logo.png'
created: '2022-05-23'
updated: '2022-05-23'
readTime: 2
version: '1.0.0'
navigation:
  github: https://github.com/threatjammer/markdown-tutorials/blob/main/how-to-use-twitter-information-bot.md
  home: /tutorials/main
  previous: 
  next: 
authors:
  - name: Diego Parrilla
    link: 'https://github.com/orgs/threatjammer/people/diegoparrilla'
ogImage:
  url: ''
---

## Sharing Threat Jammer assessment results in Twitter

Twitter is an impressive social network that allows users to share information and interact with other users worldwide. It's an excellent tool to engage with other cyber security professionals and researchers and share information about the latest cyber security trends. But it's also a tool that can help discuss specific security threads with other users. If a user wants to share information about a specific IP address, users can use two different ways to do it. And they even need to be Threat Jammer users!

## Post a link of the Threat Jammer assessment results

Sharing the Threat Jammer assessment results on several social networks is possible by posting a link. That is the simplest way, and it's not unique to Twitter, but it's also available on other social networks:

- Facebook
- LinkedIn
- Telegram
- Whatsapp

![Threat Jammer Share Buttons](/tutorialsimg/share-buttons-toolbar.png)

Any button will open the social network selected and embed the link to the Threat Jammer assessment results in the post. For example, if the user wants to share the results of the IP address in Telegram the message embedded will look like the one in the picture below:

![Threat Jammer share in Telegram](/tutorialsimg/share-link-telegram.png)

Same will happen with other social networks.

## Ask the Twitter Threat Jammer bot to share the assessment results

The Twitter Threat Jammer bot allows users to share the Threat Jammer assessment results in a conversation. The bot does not require any special permissions to use it, and it's available to any user who has a Twitter account, no matter if they are Threat Jammer users. Also, it's not necessary to follow the bot account [```@threatjammerbot```](https://twitter.com/threatjammerbot).

![Threat Jammer bot Twitter account](/tutorialsimg/threatjammerbot-twitter.png)

If a user mentions the bot in a conversation, the bot will parse all the IP addresses in the thread, and it will reply with a link to the Threat Jammer assessment results. The IP address will be included in the link, and the link will be shared in the conversation. An example of a real conversation:

![Threat Jammer bot information](/tutorialsimg/threatjammerbot-info.png)

In this example, the [```@threatjammer```](https://twitter.com/threatjammer) (aren't you following us yet?) mentions the bot, and the bot replies with a link to the Threat Jammer assessment results of the IP address found in the conversation.

In this example, the bot will only show information about the IP address. It's also possible to ask the bot to ``ban`` the IP address and report it to the Threat Jammer team. If the IP address is found to be reported multiple times, then it will be banned for 24 hours. If the IP address is banned again, Threat Jammer will extend the time. 

To ban an IP address, the user must mention the bot in a conversation and add the word ```ban``` in the conversation as a reply. For example:

![Threat Jammer bot ban](/tutorialsimg/threatjammerbot-ban.png)

The bot will also ban it if the IP address is obfuscated with ```[.]``` (bracket-dot-bracket) characters.

## What's next?

In Threat Jammer, we believe that the best way to learn about cyber security is to share information easily and fast. The Twitter bot is only an example, but other platforms like Slack, Discord, Telegram, and others could be good candidates to share information. Do you want to create your own bot for them? It's a simple matter of creating a custom script that will parse the information and share it in the social network of your choice!

If you want to keep learning, [read our documentation](https://threatjammer.com/docs/index) and mainly [check out the possibilities of our API](https://dublin.api.threatjammer.com/docs).

*If you need help you can try first in our [community site](/community), or our [support services](/support)*
