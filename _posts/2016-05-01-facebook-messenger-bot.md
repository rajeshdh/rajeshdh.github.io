---
title:  "Creating a chatbot using Facebook messenger!"
header:
  teaser: "facebook.png"
categories: 
  - bots
tags:
  - bots
  - facebook-messenger-bot
  - eliza
  - node.js
  - heroku
  - eliza-bot
hashtags: bots, facebook_messenger_bot, nodejs
---
    
Few days ago facebook launched [messenger platform](https://developers.facebook.com/docs/messenger-platform) with support to develop [Bots for messenger.](https://developers.facebook.com/blog/post/2016/04/12/bots-for-messenger/) 

Bots for Messenger are for anyone who's trying to reach people on mobile - no matter how big or small your company or idea is, or what problem you're trying to solve. Whether you're building apps or experiences to share weather updates, confirm reservations at a hotel, or send receipts from a recent purchase, bots make it possible for you to be more personal, more proactive, and more streamlined in the way that you interact with people.

In this tutorial we will create a facebook messenger bot in node.js, that uses [eliza bot's](http://www.masswerk.at/elizabot/) functionality for conversations
and deploy it on [heroku.](https://www.heroku.com/)

## Setup Node Server
Lets start with creating a node server application, that we can host at heroku to make to accessible to facebook-messenger api.

+ Make sure you have [Node](https://nodejs.org/en/) installed. 
+ Create Project directory

```javascript

$ mkdir messengerBot
$ cd messengerBot/

```
+ Create a file, name it `package.json` and make sure it looks like following.

```javascript 

{
  "name": "messengerbot",
  "version": "1.0.0",
  "description": "A facebook messenger bot",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
  "author": "rajesh dhiman @paharihacker",
  "license": "ISC",
  "dependencies": {
    "body-parser": "^1.15.0",
    "express": "^4.13.4",
    "request": "^2.72.0"
  }
}

```
+ run `$ npm install` 
+ create a new file `index.js` with following code.

```javascript

var express = require('express');
var bodyParser = require('body-parser');
var request = require('request');
var app = express();


app.use(bodyParser.urlencoded({extended: false}));
app.use(bodyParser.json());
app.listen((process.env.PORT || 1337), function () {
  console.log('app listening on port 1337');
});

// Server default URL
app.get('/', function (req, res) {
    res.send('This is a Bot Server');
});

// Facebook Webhook
app.get('/webhook', function (req, res) {
    if (req.query['hub.verify_token'] === 'mybot_verify_token') {
        res.send(req.query['hub.challenge']);
    } else {
        res.send('Invalid verify token');
    }
});

```

First GET handler  is only there to make sure the server is up and running when you visit app’s web address.

The second GET handler `/webhook` is used by Facebook to verify our server app as designated to handle Messenger Bot integration.
`mybot_verify_token` will be used as a verify token for the app which we’ll use in a while.

Now we will deploy our application on heroku, so we can get it verified from Facebook api.

*Note: Advanced users can also use [ngork](https://ngrok.com/) with a self-signed SSL certificate to host your application*

## Setup Git

1. Create a [`.gitignore`](https://git-scm.com/docs/gitignore)  file that ignores [`node_modules/`.](https://github.com/github/gitignore/blob/master/Node.gitignore)
2. Initialize a git repository and commit your project changes. 

```bash
$ git init
$ git add .
$ git commit -m 'intial commit'
```

## Setup Heroku
1. Make sure you have a [heroku](https://www.heroku.com) account and [heroku toolbelt](https://toolbelt.heroku.com/) installed. 
2. Create your heroku app, give it a name (replace *myApp*) and push it to heroku server.

```bash

$ heroku apps:create myApp 
$ git push heroku master

```
If everything works fine, you can checkout your app running by using `heroku open` command.

## Setup Facebook App

For this integration, you will need the following:

**Facebook App:** The Facebook App contains the settings. 
This is where you will setup your webhook, retrieve your page access token and submit your app for approval.

**Facebook Page:** A Facebook Page will be used as the identity of your bot.
 When people chat with your bot, they will see the Page name and the Page profile pic.

**Webhook URL:** Facebook use secure callbacks in order to send you messaging events. 
These will be sent to your webhook.

We have already setup the **Webhook URL** its `https://yourapp.herokuapp.com/webhook` *(yourapp is your heroku app)*

To create a **Facebook page** and setup a **Facebook App** please refer to instructions from [facebook here.](https://developers.facebook.com/docs/messenger-platform/implementation#create_app_page)
`<YOUR_VERIFY_TOKEN>` is  `mybot_verify_token` or whatever you have added in index.js.
Copy your `PAGE_ACCESS_TOKEN` and come back when its done. 

### Save your `PAGE_ACCESS_TOKEN` in heroku's config variables. 

1. Open your heroku dashboard.
2. Click on your app.
3. Go to settings 
4. Add `PAGE_ACCESS_TOKEN` as `key` and your access token from facebok as `value` in `Config vars`.

## Sending & Receiving Messages 

In order to receive messages, we need to register `POST` handler that loops over messages. 
Lets create it along with a generic function to send messages at the bottom of index.js file:

```javascript

/**
 *  POST handler to send and receive messages
 */ 
app.post('/webhook', function (req, res) {
    var events = req.body.entry[0].messaging;
    for (i = 0; i < events.length; i++) {
        var event = events[i];
        if (event.message && event.message.text) {
          
            sendMessage(event.sender.id, {text: "Echo: " + event.message.text});
        }
    }
    res.sendStatus(200);
});

/**
 *  function to send messages
 */ 
function sendMessage(recipientId, message) {
    request({
        url: 'https://graph.facebook.com/v2.6/me/messages',
        qs: {access_token: process.env.PAGE_ACCESS_TOKEN},
        method: 'POST',
        json: {
            recipient: {id: recipientId},
            message: message,
        }
    }, function(error, response, body) {
        if (error) {
            console.log('Error sending message: ', error);
        } else if (response.body.error) {
            console.log('Error: ', response.body.error);
        }
    });
};


```

First handler goes over message objects found in messaging property (they can be batched in one webhook call) 
and if there’s a message text available, it sends it back using the `sendMessage` function. 
Now our application is ready to receive messages and `Echo` them back to sender. 
You can commit changes and push them to heroku to check the functionality. 

```bash
$ git add -A
$ git commit -m 'Added Echo functionality'
$ git push heroku master

```
Most of the parts of this tutorial were taken from [this](http://x-team.com/2016/04/how-to-get-started-with-facebook-messenger-bots/) tutorial. 
You can check it out to send rich messages. 

## Using Eliza bot

Now when our program can send and receive messages lets add functionality for conversations. 
We will be using elizabot.js for conversations. 

**ELIZA** is a natural language conversation program described by Joseph Weizenbaum in January 1966 [1].
It features the dialog between a human user and a computer program representing a mock Rogerian psychotherapist.
The original program was implemented on the IBM 7094 of the Project MAC time-sharing system at MIT and was written in MAD-SLIP.
You can check eliza demo [here.](http://www.cs.vu.nl/~eliens/demo/sample-js-eliza-bot.htm#ELIZA)

Download [elizadata.js](https://github.com/rajeshdh/facebook-messenger-bot/blob/master/elizadata.js) and [elizabot.js](https://github.com/rajeshdh/facebook-messenger-bot/blob/master/elizabot.js)
to your project. 

Add depedency to inclue Elizabot in your project and create a new elizabot object. 

```javascript

var ElizaBot = require('./elizabot');
var eliza = new ElizaBot();

```

Modify your `POST` webhook to get replies from eliza. Everything else will be handled by elizabot.

```javascript

app.post('/webhook', function (req, res) {
    var events = req.body.entry[0].messaging;
    for (i = 0; i < events.length; i++) {
        var event = events[i];
        if (event.message && event.message.text) {
            // get reply from eliza 
            var reply = eliza.transform(event.message.text);

            sendMessage(event.sender.id, {text: reply});
    
        } 
    }
    res.sendStatus(200);
});

```

Save and commit your project changes and push them to heroku. 

```bash
$ git add -A
$ git commit -m 'Added Elizabot for conversations'
$ git push heroku master

```
You can also download complete project from [Github](https://github.com/rajeshdh/facebook-messenger-bot) 
~~or checkout the live demo by sending a message at my [facebook page.](https://www.facebook.com/teamSanyatra/) Replies will be sent by elizabot.~~ 
You need to get your bot approved from facebook, only then it can work for all otherwise it will work for the tester accounts for that page only. 
