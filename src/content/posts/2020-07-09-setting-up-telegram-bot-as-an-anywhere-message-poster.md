---
template: blog-post
title: Setting up telegram bot as an anywhere message poster
slug: telegram-bot
date: 2020-02-11 15:53
description: telegram-bot
---
**Use case**: Creating an endpoint that posts any text message to your telegram chat without any login

**Required :**

1.  **Telegram account**

Sometimes we are into a situation where we want to send a text to our mobile or laptop or vice versa. So usually we come up with solutions like sending messages to our dear ones from any messenger/IMs/email from the laptop and we access from mobile to copy that text. But basically it requires multiple steps to do that.

I personally faced issues when I want to copy some text from the office laptop to my mobile but all web IMs are blocked. So created a google document and shared publicly and used to add text in that document and later access that in mobile. To get it simpler is there any one-click way of sending a message and we can access it from anywhere.

So finally I came up with Telegram bots. This solution is not so great but can be helpful in some scenarios where we don't need to login to any account to send a message.

Telegram Bot API is so much vast and I just used a bit of it. So lets dive into this little utility process.

### Step 1: Setting up Telegram Bot from Bot Father

The first thing we have to do is set up a bot in our telegram account. To do that telegram provides a user handle that handles are operations regarding bot. So add user @BotFather.

### Step 2: Creating a bot

Click on /newbot link and follow the steps to create a bot with a name, About, Description, and BotPic.

### Step 3: Copying Bot token

Click on /mybots and go to the created bot details. Click on API Token and note down the token somewhere ., this token is the one where we use further to access our endpoint. Bot API token looks like as below

> 425896547:AACDERDDSewr34343GJAbtriFQA2lxuOnLByQun8Isw

### Step 4: Generating Endpoint with bot token

So telegram API to send message is

[https://api.telegram.org/bot{bot_token}/sendMessage?chat_id={chat id of the user where we want to send message}&text=](https://api.telegram.org/bot%7Bbot_token%7D/sendMessage?chat_id={chat%20id%20of%20the%20user%20where%20we%20want%20to%20send%20message}&text=)

#### To get to know about the chat id please follow below steps.

1.  Add above created bot in your Telegram account.
2.  Send a test message to the bot chat.
3.  Navigate to this URL. https://api.telegram.org/bot{bot_token}/getUpdates
4.  You will receive a response as below. Take id in that resposne.

> {"ok":true,"result":[{"update_id":1457854,"message":{"message_id":2174,"from":{"id":123456789,"is_bot":false,"first_name":"name","username":"user","language_code":"en"},"chat":{"id":123456789,"first_name":"/\\//\\\\/!/\\/","username":"user","type":"private"},"date":1587729104,"text":"Hi"}}]}

So going back to our API , we will insert chat id in the previous URL.
[https://api.telegram.org/bot{bot_token}/sendMessage?chat_id=123456789&text= {any text message}](https://api.telegram.org/bot%7Bbot_token%7D/sendMessage?chat_id=123456789&text=%20{any%20text%20message})

So from anywhere you just hit the URL and get the text given in the text field will be delivered to your telegram account.

So to remember the above URL is pretty hard as it contains token and all. So to simplify this what we do is generate a shorter URL from TinyURL.Suppose if my tiny url is www.tinyurl.com/mytelegrambot then we can easily remember the URL.

Apart from sending messages through URL we can also read messages that we sent in via same URL by changing a bit. [https://api.telegram.org/bot{bot_token}/getUpdates](https://api.telegram.org/bot%7Bbot_token%7D/getUpdates)

### Usual issues:

1.  We have to give compulsory text field value to a successful message post response.
2.  If the text contains spaces, URL request failed. So to allow spaces in the text we have to add another flag., parse_mode=HTML

I hope it helps as a utility to send a message to our Telegram account from anywhere without logging.