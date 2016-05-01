---
title:  "Create a Facebook messenger bot"
header:
  teaser: "./images/facebook.png"
categories: 
  - bots
tags:
  - bots, facebook-messenger-bot, eliza, node.js. heroku, eliza-bot
---
    
Few days ago facebook launched [messenger platform](https://developers.facebook.com/docs/messenger-platform) with support to develop [Bots for messenger.](https://developers.facebook.com/blog/post/2016/04/12/bots-for-messenger/) 

Bots for Messenger are for anyone who's trying to reach people on mobile - no matter how big or small your company or idea is, or what problem you're trying to solve. Whether you're building apps or experiences to share weather updates, confirm reservations at a hotel, or send receipts from a recent purchase, bots make it possible for you to be more personal, more proactive, and more streamlined in the way that you interact with people.

In this tutorial we will create a facebook messenger bot in node.js, that uses [eliza bot's](http://www.masswerk.at/elizabot/) functionality for conversations
and deploy it on [heroku.](https://www.heroku.com/)

Lets start with creating a node server application, that we can host at heroku to make to accessible to facebook-messenger api.

1. Make sure you have [Node](https://nodejs.org/en/) installed. 
2. Create Project directory

```javascript

$ mkdir messengerBot
$ cd messengerBot/

```