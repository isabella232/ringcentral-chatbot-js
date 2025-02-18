# RingCentral chatbot framework for JavaScript

## Philosophy

- Let developers focus on business logic instead of RingCentral/Glip Platform implementation details


## Supported features

- Bot token management
- Bot WebHook management
- Remove & re-add bot
- Deploy to AWS Lambda
- Update bot name and avatar
- Bot join & leave group


## Quick Start tutorials

- If you want to create a Glip chatbot and deploy it to your own server
    - [read this tutorial](https://github.com/tylerlong/glip-ping-chatbot/tree/express)
- If you want to create a Glip chatbot and deploy it to AWS Lambda
    - [read this tutorial](https://github.com/tylerlong/glip-ping-chatbot/tree/lambda)

We recommend you to finish reading/watching at least one of the tutorials above before reading on.


## Create a RingCentral app

Click the link below to create a RingCentral app quickly:

[Create Bot App](https://developer.ringcentral.com/new-app?name=Sample+Bot+App&desc=A+sample+app+created+for+the+javascript+chatbot+framework&public=true&type=ServerBot&permissions=ReadAccounts,EditExtensions,SubscriptionWebhook,Glip&redirectUri=https://<chatbot-server>/bot/oauth)

Do remember to change redirect uri from `https://<chatbot-server>/bot/oauth` to the real uri.
If you don't have a public uri, leave that field blank.


## Setup database

The first time you setup the bot, the database is empty, you need to create tables.

There is an easy way:

```
curl -X PUT -u admin:password https://<bot-server>/admin/setup-database
```

## Migrate database

If you update the SDK from an early version, it might not work because the database schema has changed.

In such case, you can migrate the database:

```
curl -X PUT -u admin:password https://<bot-server>/admin/migrate-database
```
This command will smartly check your database and add new fields/tables if necessary.


## Maintain

"Maintain" is useful in the following cases:

- If for reason bot server changed, you need to re-setup WebHooks
- You bot server was down for quite a while and your WebHooks have been blacklisted
- There is orphan data in database

You can "maintain" to resolve the issues above:

```
curl -X PUT -u admin:password https://<bot-server>/admin/maintain
```

It is recommended that you create a cron job to run this command daily.


## Diagnostic

Sometimes there are issues and you don't know what happened. This framework provides interfaces for diagnostic:

```
curl -X GET -u admin:password https://<bot-server>/admin/dump-database
curl -X GET -u admin:password https://<bot-server>/admin/list-subscriptions
```


## Update token

```
curl -X PUT -u admin:password https://<bot-server>/admin/update-token?id=<bot-extension-id>&token=access-token'
```


## Hidden commands

The following commands are considered "hidden" or "easter eggs":

- `__rename__ <newName>`: rename bot to `newName`
- `__updateToken__ <access_token>`: update bot access token to `access_token`
- A message with text `__setAvatar__` and an attached image file: set bot avatar to the attached image


## Advanced topics

- [Skills](./skills.md)
- [Integrate with third party services](./third-party-services.md)


## About graduation

In order to graduate a RingCentral app, you need to invoke each API endpoint used in your app at least 5 times. 

In order to to meet this requirement quickly, you can invoke API endpoints explicitly in code, for example:

```
await bot.rc.get('/restapi/v1.0/account/~/extension/~')
```


## Advanced Demo bots

- [Glip Crontab Chatbot](https://github.com/tylerlong/glip-crontab-chatbot)
    - Allow Glip users to create cron jobs to send notifications
    - [Try it](https://www.ringcentral.com/apps/glip-crontab-chatbot)
    - By reading its source code, you will get how to handle messages from users and how to reply.
- [Glip RC Assistant Chatbot](https://github.com/tylerlong/rc-assistant)
    - Allow Glip users to query/edit their RingCentral data via Glip.
    - [Try it](https://www.ringcentral.com/apps/glip-rc-assistant-chatbot)
    - It is a demo for integrating Glip chatbot with RingCentral platform.
- [Glip Google Drive Chatbot](https://github.com/tylerlong/glip-google-drive-chatbot)
    - Allow Glip users to monitor their Google Drive events.
    - [Try it](https://www.ringcentral.com/apps/glip-google-drive-chatbot)
    - It is a demo for integrating Glip chatbot with a third party service (Google Drive)
    - It is also a demo for chatbot skills


## Notes

- AWS Lambda connects AWS RDS issue
    - If you create an RDS manually and let it create a new security group for you.
        - By default, the security group only allows inbound traffic from your current laptop's public IP address
        - AWS Lambda by default cannot access the newly created AWS RDS
        - We need to update security group to allow inbound traffic from `0.0.0.0/0`
- AWS Lambda and `await` issue
    - AWS Lambda is stateless and it won't keep a background runner
    - So if your code is async, DO remember to `await`. Otherwise that code will not be executed before Lambda invocation terminates
    - I forgot to `await`, a weird phenomenon was: whenever I issued a command, I always got reply of previous command
- AWS Lambda global code dead-lock issue
    - Do NOT invoke lambda functions in global code outside of handlers
    - I tried once and it seemed to case dead-lock. Lots of requests which causes 429 error and database connection exhausted
        - In global code I did `axio.put('https://<bot-server>/admin/maintain')` and it caused dead-lock


## Todo

- send api to change presence, let bot user to be online
