---
title:  "Create a Facebook messenger bot"
header:
  teaser: "./images/facebook.png"
categories: 
  - bots
tags:
  - bots
  - facebook-messenger-bot
  - eliza
  - node.js
  - heroku
  - eliza-bot
---
    
Few days ago facebook launched [messenger platform](https://developers.facebook.com/docs/messenger-platform) with support to develop [Bots for messenger.](https://developers.facebook.com/blog/post/2016/04/12/bots-for-messenger/) 

Bots for Messenger are for anyone who's trying to reach people on mobile - no matter how big or small your company or idea is, or what problem you're trying to solve. Whether you're building apps or experiences to share weather updates, confirm reservations at a hotel, or send receipts from a recent purchase, bots make it possible for you to be more personal, more proactive, and more streamlined in the way that you interact with people.

In this tutorial we will create a facebook messenger bot in node.js, that uses [eliza bot's](http://www.masswerk.at/elizabot/) functionality for conversations
and deploy it on [heroku.](https://www.heroku.com/)

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
    if (req.query['hub.verify_token'] === 'testbot_verify_token') {
        res.send(req.query['hub.challenge']);
    } else {
        res.send('Invalid verify token');
    }
});

```

First GET handler  is only there to make sure the server is up and running when you visit app’s web address.

The second GET handler `/webhook` is used by Facebook to verify our server app as designated to handle Messenger Bot integration.
`testbot_verify_token` will be used as a verify token for the app which we’ll in a while.