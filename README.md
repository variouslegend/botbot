# discord-giveawaybot

[![Build Status](https://travis-ci.org/shukriadams/discord-giveawaybot.svg?branch=master)](https://travis-ci.org/shukriadams/discord-giveawaybot)

A Discord bot that manages automated game giveaways on Discord channels. Built-in integration for Steam titles, but can
handle anything connected to a URL. Heavily inspired by https://github.com/jagrosh/GiveawayBot, differs from the
original with :

- bot commands are in private message, allowing for surprise giveaways, direct messaging of game keys to winners,
  detailed data queries and other quiet admin functions
- queuing of future giveaways
- anti-greed features automatically preventing a winner from entering another giveaway for a while
- better Steam integration

A demo version can be seen on Discord: https://discord.gg/gMEGQBj (Bot is limited to non-admin functions, I can't
auto-assign admin rights to users. I'm not active on this Discord channel, it's for demo purposes, if you need help or
found a bug use Github).

## Requirements

- Docker (recommended host)

or

- NodeJS 7 or greater if you plan on mounting the bot directly

## Create your bot on Discord first

- go to https://discordapp.com/developers/applications/me
- click on "new app"
- follow the instructions and create your app - you need to add only a name
- after creating your app scroll down the app page and click on "create a bot user", this converts your app to a bot
  (a good thing)
- on the bot's config page, copy the bot's client id, you'll need this later. Also click on "click to reveal token",
  copy this too for the next step.

## Host your bot

There are several ways to fetch the bot's code. Regardless of which you use, you need to

1. Create a root folder for your bot, this will contain static config and/or files. This is where you'll put docker-compose.yml, or package.json
   etc from the root of this project.

   mkdir myBot

2. In the root folder create a work folder where the bot writes its own volatile files.

    mkdir myBot/discord-giveawaybot

3. In the work folder, create a settings file. The bot will write to this file too.

    touch myBot/discord-giveawaybot/settings.json

4. In the root of this Github project you'll find exampleSettings.json, copy its contents to your settings file, and
replace "ADD YOUR BOT TOKEN HERE" with the Discord bot token you copied in "Create your bot on Discord first" above. Remember to use the token,
not the client id.

Getting the bot code ...

### 1) From Docker image

This is the recommended method because it's easiest to set up and keep up after that. Create a docker-compose.yml file in
your bot root folder and add the following to it

        version: "2"
        services:
          node:
            container_name: discordgiveawaybot
            image: shukriadams/discord-giveawaybot:latest
            restart: unless-stopped
            command: npm start
            volumes:
            - ./discord-giveawaybot/:/usr/giveawaybot/discord-giveawaybot/:rw

In the root folder run

        docker-compose up -d

These settings can of course be tweaked to suite your host setup, only npm start and the volume map are required. All
volatile content is in ./discord-giveawaybot, back this up if desired, your container is disposable. For reference, the
container image is at https://hub.docker.com/r/shukriadams/discord-giveawaybot.

### 2) From NPM

Install

        npm install discord-giveawaybot --save
    
Run
        npm start

### 3) From source

Clone this repo, then run
    
        npm install
        npm start

**Keep-alive**

If you're not hosting with Docker, you need to restart the bot process when it unexpectedly exits. [pm2]
(http://pm2.keymetrics.io/) is an excellent option, but you can use whatever you're most comfortable with, just as long
as you handle exits, because the bot _will_ exit periodically.

## Add your bot to your Discord server

- back on your app's Discord config page (from the first section above), use the bot client id you copied and paste it
into this url, replacing YOURCLIENTID

      https://discordapp.com/oauth2/authorize?&client_id=YOURCLIENTID&scope=bot&permissions=0

- then navigate to that url in a browser. You'll be able to select which of your Discord servers you want to add it to.
After doing this you should see your bot as a user on your server.
- Your bot needs to know which channel you'll be broadcasting giveaways in. Go to the channel you want to use and write
"@BOTNAME channel" where BOTNAME is whatever name you gave your bot.
- That's it, you're set to go.

## Known issues

The bot can handle a maximum of 100 participants per giveaway. Anyone above that 100 will be ignored - this is a
limitation in Discord's API, and will be fixed when Discord fixes their API. As a workaround, a giveaway will
automatically end when it reaches 100 participants.

## Additional config

By default, only admins can create and manage giveaways. If you want to delegate giveaway responsibilities to non-admins

- go to your Discord server settings and select "roles"
- on the roles page, add a role called "Giveaways". If you don't want to use this name, create any role you want, and
add that name to settings.json (requires bot restart)
- Assign the role "Giveaways" (or whatever you called it) to users who'll run giveaways.

## Commands

One of the major differences between this bot and jagrosh's giveawaybot is that this bot uses direct communication -
you don't talk to it in public chat.

### brackets

Price brackets let you limit how often users can win a game within a price range. If you register a bracket of $0-100,
and a user wins a game that costs $50, that user will automatically be prevented, for seven days, from entering another
giveaway for any game costing between 0 and 100 USD. Brackets are optional - you can register none, one, or as many as
you like.

    brackets -b 0-20-50-100

sets 3 brackets, 1-20, 20-50 and 50-100. If a game costs 20 USD, it falls in the first bracket that it fits in, 0-20 in
this case. You can start and brackets at any price range. For example

    brackets -b 20-30

sets one bracket, and will catch only games that fall in its range, and a user will be allowed unlimited entry in games
below 20USD or above 30USD.

Prices are always in USD.

For a list of current brackets, use

    brackets

### cancel

Admins or the creator of a giveaway can cancel that giveaway if it hasn't started yet, or is in progress.

### channel

This is the only bot command done in public chat. It registers the channel from which it is sent as the channel in which
giveaways will be broadcast.

### help

Gets a list of commands from the bot.

### list

Lists all ongoing giveaways.

    list all

Shows ongoing and complete and cancelled giveaways.

Additionally, admins and users with giveaway rolls will be able to see pending giveaways.

### me

Tell a user if they're on cooldown for a given bracket if they recently won a game in that bracket.

### queue

Creates a giveaway to be started at some time in the future. Requires admin or giveaway roll. A game activation code
can optionally be added to this command - the winner will receive this code in a private message.

### reroll

An admin or user with giveaway role can reroll a winner on a finished giveaway if they so wish. Note this obviously
doesn't have much meaning if the activation code is attached to the giveaway, as the previous winner will already have
received the code, so use common sense.

### rules

Simple rules text can be found using

    rules

To set rules text, admin pemission is required. Use

    rules Your text here ...

### start

Immediately starts a giveaway.

### status

This command is currently disabled.

## Other

The bot automatically cleans out completed/cancelled competitions after 14 days.

Get participate emoji characters at http://emojipedia.org

## Development

The bot is basically two processes

- a message handler that receives message instructions from Discord users and responds to them immediately.
- a daemon which ticks at an interval, and which carries out instructions that are not directly driven by incoming user
  messages

All other files are helpers for the above. Other stuff :

- Giveaway data is persisted with a local Loki.js store, in /discord-giveawaybot/__store
- Errors are logged out with Winston in /discord-giveawaybot/__logs
- The daemon uses node-cron for its timer.

If you use Vagrant, the included vagrant script will start an Ubuntu VM ready to run the bot (for development or
testing).

    cd /vagrant
    vagrant up
    vagrant ssh

Then in the VM run

    yarn --no-bin-links (flag needed only if your host machine is Windows)
    node start (or npm start)

If you want to run the bot directly on your host system without yarn

    npm install
    node start (or npm start)

## Tests

    npm test

or if you want to test with a debugger (Webstorm, VSCode etc), point your debugger to /tests/test.js and

    cd /tests
    node test
