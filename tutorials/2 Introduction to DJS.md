# Introduction to DJS

This lesson will go over how to set up a discord bot and install discord.js, as well as guide you through making your first basic bot.

## Setup

Firstly, you need to create a Discord bot. If you already did or already know how, skip this entire section. Go to the developer portal at https://discord.com/developers. You may need to log in first.

Next, click "New Application" in the top-right corner. Pick a name for your application. This will be the default for your bot's username, but it doesn't have to be the same; you can choose your bot's username later. This creates an integration with Discord, which is not the same as a bot, as your application does not actually possess a bot account yet. Not every integration will have a bot; for example, you can create an integration to use Discord's OAuth to allow users to sign in to a service such as a website using their Discord account.

You can set a description which will be your bot's About Me section. You can also set tags, which only really matters if you plan to publicize your bot. The Application ID and Public Key are important pieces of information but you don't need to worry about them and can just ignore them for now. The interactions endpoint, terms of service, and privacy policy URLs are not important for making a bot. On the left sidebar, click "Bot".

Then, click the "Add Bot" button. This will create a bot account for your integration, meaning it now has a user account that will appear as a member when the integration is added to a server and can do things like read and send messages and close to anything else a regular user can do. You cannot reverse this action, but since this tutorial is about making a Discord bot, just go ahead and confirm. You can change your bot's username now if you want. Let's go over the settings first:

-   If you don't want other people to be able to add your bot to other servers, turn off "Public Bot". Note that even if you don't give anyone a URL, it is trivial to generate one for any given bot, as the ID used in bot invitation links is simply the application ID which is also the same as the bot's user account ID, which is publicly accessible.
-   Don't turn on the OAuth2 Code Grant requirement. It's not important for this tutorial.
-   If you want to see when users' presences (activity, status) change, turn on the Presence Intent. If not, it doesn't matter. Turning it off will save you some bandwidth if that matters to you, although you can also specify that at runtime.
-   If you want to be able to access your server's user events (like when people join or leave), turn on the Server Members Intent.
-   If you want to be able to get the content of each message, turn on the Message Content Intent. For making a prefix command bot, you will need to enable this otherwise your bot will not see or respond to commands.
-   The permissions checkboxes at the bottom don't actually do anything. It is just a tool to help you calculate the permissions bit, which we'll get to later.

## Adding your bot

To add your bot to your server, click "OAuth2" on the left sidebar and then click "URL Generator". Here, you will want to select the "bot" and "applications.commands" scope (the latter is not needed if you don't want to implement slash commands). The rest of the scopes don't apply here.

Then, select which permissions you want your bot to have in the checkboxes beneath it. Once you are done, copy the generated URL and paste it into your browser. Choose your server and you should be good to go.

## Installing discord.js

So far, you've made a bot and added it to your server. However, currently your bot cannot do anything. For it to do anything, you will need to write the code to control the bot.

Go to your workspace (or create a new one). Now, run `npm install --save discord.js`. This tells the node package manager to install the discord.js module and also save the package name in `package.json` which will be important later (running `npm install` will automatically install all requirements from `package.json` which lets someone else download and run your code, or lets you reinstall everything easily if you need to upload it to a server host or another computer).

You should now have discord.js installed and can now proceed. If `npm` doesn't exist on your system, go back to the last tutorial and see if you followed the node installation steps correctly.

## Your First Bot

Create a new file (I will call it `main.js`). Let's go over the rough idea of how this will work. We will create a new `Client` object which represents the bot client and handles things like incoming events and performing actions. The client will be logged in via the bot token, which is a private string that you _must_ keep a secret. If someone gets access to your bot's token, they get full access to your bot. If this happens, you can always reset it.

In fact, right now, you don't even know your own token. Go back to the [developer portal](https://discord.com/developers/applications), click your application, click "Bot", and click "Reset Token". If you have 2FA, you will need to enter your 2FA token. This replaces your bot token with a new, random token. This means that any programs running with the old token are invalid and will no longer work anymore. Copy this token.

Once the client is logged in, you have the option to write event handlers, which run when a certain event (such as an incoming message) occur, or you can just have the bot take actions such as sending messages on its own, unprompted.

Let's first create a client. To begin, go into `package.json`, and add an entry saying `"type": "module"` (I will be using ES6-style imports; if you plan to use the Common JS `require` style imports, just do that the way you normally do and skip this part). Your file should roughly look like the following:

```json
{
    "type": "module",
    "dependencies": {
        "discord.js": "version number"
    }
}
```

Now, we can begin writing code. To create a client, we must first import the `Client` class (type) from the library. Start by writing `import { Client } from "discord.js";`. discord.js exports a type called Client. We could have also done `import discord from "discord.js"` - what this will do is assign the discord.js library to the variable named `discord` (it's quite similar to `let discord = await import("discord.js")` and the differences aren't significant for this tutorial).

Recall destructuring assignment from the last lesson - `import { Client } from "discord.js"` is similar to `let { Client } = await import("discord.js")`, which is basically `let Client = (await import("discord.js")).Client`.

Next, we will create an instance of the client. To do this, we define a variable called `client` (you can call it something else if you'd like) and initialize it to `new Client()`. `Client` is a class, meaning we create new copies of it using its _constructor_, which is a special function that dictates what happens when a new instance of a class is initialized.

```js
import { Client } from "discord.js";

const client = new Client();
```

`Client`'s constructor takes some options in the form of an object so change `new Client()` to `new Client({})`. Now let's go over what options there are. By the way, the discord.js documentation is extremely useful and there is too much to memorize anyway so you definitely want to refer to it frequently. The page for `Client` is https://discord.js.org/#/docs/discord.js/stable/class/Client and you can find any other class or feature you need.

There are a lot of options that you can specify, but only a few are relevant (or that I know how to use / understand enough to recommend changing) - `allowedMentions`, `partials`, `failIfNotExists`, `presence`, and `intents`.

`allowedMentions` refers to which mentions will actually be converted into pings. It is possible for a bot to send a user, role, or even everyone/here ping without actually sending the ping itself. Setting the client option will dictate the default for all future messages, although it can be overriden per message as well. For now we can just ignore this.

`partials` refers to which partial structures you want your bot to receive. Sometimes due to API limitations, discord.js cannot actually get certain properties of certain objects like channels, so they are only _partially complete_ (or rather, they are not in the bot's cache, so it would cost extra bandwidth and API calls to get this data, so the library doesn't do it automatically and leaves it up to you to decide). If you do not specify the partials, you will not receive these objects at all, which can cause some issues such as not being able to receive DMs unless the bot sent a DM first. The available options are `USER`, `CHANNEL` (only DM channels; guild channels are always available), `GUILD_MEMBER`, `MESSAGE`, `REACTION`, and `GUILD_SCHEDULED_EVENT`. For the purposes of this tutorial, turn on `CHANNEL`, `MESSAGE`, and `REACTION`, which allow you to receive DMs always, get all message events, and see reactions even if the message is too old to be cached. That is, change `new Client({})` to `new Client({ partials: ["CHANNEL", "MESSAGE", "REACTION"] })`.

`failIfNotExists` refers to whether the bot should fail (error) when trying to reply to a message that either doesn't exist or can't be replied to, or if it should just send it as a standard message instead. By default, it is true; if you want to set it to false, do `{ partials: [...], failIfNotExists: false }`.

`presence` is the status that your bot will have when it logs in; that is, the status, activity, etc. You can ignore this, but if you'd like something like "Listening to your DMs" as its status message (for a modmail bot, for example), you can do the following:

```js
new Client({
    partials: ["CHANNEL", "MESSAGE", "REACTION"],
    presence: {
        activities: [
            {
                type: "LISTENING",
                name: "your DMs",
            },
        ],
    },
});
```

The presence is an object of type [`PresenceData`](https://discord.js.org/#/docs/discord.js/stable/typedef/PresenceData). You can specify other things; for example, `status` can be one of `online`, `idle`, `invisible`, or `dnd` (do not disturb) to change the bot's status type. Ignore the `afk` and `shardId` fields. `activities` is a list of [`ActivitiesOptions`](https://discord.js.org/#/docs/discord.js/stable/typedef/ActivitiesOptions) which have a type, name, and optional URL. In this case, we only have one activity, which is of type `LISTENING`. This means it will show up as "**Listening to** (activity name)", so we set the name to `"your DMs"`.

Finally, but most importantly, `intents` tells the API which events to send to you. To avoid wasting bandwidth and computational power on both your end and Discord's end, Discord will not send you any events by default; you have to tell it which ones you request. You can find a list of intents on the official API documentation [here](https://discord.com/developers/docs/topics/gateway#list-of-intents), which lists the intent types and the event types associated with each. Some of these will be intuitive but don't worry about the ones that aren't.

There are two ways you can approach this. You can add in new intents every time you add a new event that you need to handle, or you can just start off by setting all intents to on. To set a specific list of intents, you can just do `intents: ["...", ...]`, where the intent names are listed [on the docs](https://discord.com/developers/docs/topics/gateway#list-of-intents). For example, to only receive incoming messages, you can do `intents: ["GUILD_MESSAGES"]`, which will send events to your bot when messages are sent, edited, deleted, or purged. To enable all intents, you can just do `intents: 131071`. The intents field can be a number, which is a _bitfield_, meaning the individual bits of the number (in this case the bits are just 16 1s in a row) correspond to one intent each, so 131071 turns on all 16 intents. Note that this will fail if you do not enable all privileged intents in the developer portal (we set this up earlier in this lesson), and it's good practice to only enable intents that you need; however, it's easier to just enable all and won't have too much of a negative impact on performance at a small scale.

Now, let's try adding our first event listener - the `ready` handler, which is called when the bot has gone online and is ready to begin processing. There is one important thing to keep in mind - this might not only occur once per time you execute the program.

There are two main ways to add an event listener to a client. `client.on("event name", () => {})` will be run every time a certain event occurs, and `client.once("event name", () => {})` will be run only once, the first time the event fires. If you need to do per-execution set-up work, make sure you do `client.once("ready", ...)` and not `client.on` as that could be called multiple times.

Let's use `.on` this time though for this test program it really does not matter:

```js
client.on("ready", async () => {});
```

Generally, your event handlers should be `async`, just so you can use `await` statements inside them to make things easier for yourself. Now, let's just put a simple debug message in the function:

```js
client.on("ready", async () => {
    console.log("bot is ready!");
});
```

Note that the lack of `await` statements here means the `async` does not technically make a difference. However, it is just more convenient to always include it in case you need to `await` later.

For our final step, we need to actually log in the bot. Once all of the event listeners are set up, the bot is ready to connect to the API. This is very simple - just append `client.login("(your token here)")`. Hardcoding the token here is generally a bad idea so you should move it to a safer way such as storing it in a file (if you are using git, remember to put it into your `.gitignore`) or using an environment variable (you can read a tutorial [here](https://www.javascripttutorial.net/nodejs-tutorial/nodejs-process-env/)). For testing, you can just paste the token directly (remember that you must keep it secret), or you can just set up your file / variables now for use later.

Then, go ahead and run the program (`node main.js` from within your workspace directory). After a while, you should notice two things happen. Firstly, your bot will go from offline to online in Discord. Secondly, after a brief delay, you should see "bot is ready!" appear in your console. You can just hit <kbd>Ctrl</kbd> + <kbd>C</kbd> to terminate this program, and after a while your bot will go offline in Discord again.

If this has all worked, congratulations! You've technically made your first bot in discord.js - granted, all it does is print a message locally when the bot is ready, but it's progress, and we can very quickly begin adding new features from here. Remember to refer to the [official documentation](https://discord.js.org/#/docs/discord.js/stable/general/welcome) frequently.

If this hasn't worked, you can refer to the below code and see if there are any issues in yours.

```js
import { Client } from "discord.js";

const client = new Client({
    partials: ["CHANNEL", "MESSAGE", "REACTION"], // optional
    presence: {
        // optional
        activities: [{ type: "LISTENING", name: "your DMs" }],
    },
    intents: 131071, // if you didn't select all privileged intents in the developer portal,
    // you will need to calculate this yourself or use a list of flags
});

client.on("ready", async () => {
    console.log("bot is ready!");
});

client.login("your token here");
```

## A Bot That Actually Does Something

So, we made our bot log in, but it doesn't really do anything yet. Let's make a simple ping command - when someone sends `!ping`, we will reply with `Pong!`. You can keep most of your current code. It is up to you if you keep your `ready` event listener; I like having it in so I can know when my bot is ready and if startup time is too long, but you can remove it unless you need it for setup later on.

To respond to messages, we need to watch for incoming messages. The event for that is `messageCreate` - you can find a full list of these events in the Events section of https://discord.js.org/#/docs/discord.js/stable/class/Client.

```js
client.on("messageCreate", async (message) => {});
```

Note that this time, the function we provide as the listener takes one argument, which is, of course, the message that we receive. This message isn't a string; it's a [`Message`](https://discord.js.org/#/docs/discord.js/stable/class/Message) object.

We first need to check if the message says `!ping` - to do that, we check `message.content`:

```js
client.on("messageCreate", async (message) => {
    if (message.content == "!ping") {
    }
});
```

To reply to this message, we have two options. If you want to just send it as a normal message in the channel, you can do `await message.channel.send("Pong!");`. If you want to reply to the message, it's actually even simpler - `await message.reply("Pong!");`. The reply will, by default, ping the author. If you want to remove that, you can set `allowedMentions` to `{ repliedUser: false }`, which tells Discord to not mention the user you are replying to. To do this, you will need to change the string to an object. Technically, `send(string)` is a shortcut for sending a [`MessagePayload`](https://discord.js.org/#/docs/discord.js/stable/class/MessagePayload). The string is the `content`, so we change our reply to this:

```js
client.on("messageCreate", async (message) => {
    if (message.content == "!ping") {
        await message.reply({
            content: "Pong!",
            allowedMentions: { repliedUser: false },
        });
    }
});
```

You can now run your bot. Confirm that sending `!ping` in a channel that the bot can see will have the bot respond with `Pong!` (this should work in DMs if you have the `CHANNEL` option specified under `partials`), and that sending other messages won't reply.

## Slash Commands

If you don't want to use slash commands for your bot, you can just skip to the end. This section will be the remainder of this lesson.

There are two parts to slash commands. Firstly, you need to register them with Discord so they will show up to the user. Then, you need to create an event handler for it.

Slash commands are just one of multiple types of application commands. As of the time of writing, there are three types - slash commands, message context menu commands, and user context menu commands. The latter two can be invoked by right-clicking a message/user and hovering over "Apps" - if there's nothing there, that just means that no bots in your server have any context menu commands available.

To create a slash command, we will use the [`ApplicationCommandManager`](https://discord.js.org/#/docs/discord.js/stable/class/ApplicationCommandManager). These managers belong to [`ClientApplication`](https://discord.js.org/#/docs/discord.js/stable/class/ClientApplication)s which in turn belong to [`Client`](https://discord.js.org/#/docs/discord.js/stable/class/Client)s. Thus, we obtain the command manager using `client.application.commands`.

There are four primary actions you can do with the manager - create, delete, or edit a command, or set the list of commands. Let's first take a look at the [`ApplicationCommandData`](https://discord.js.org/#/docs/discord.js/stable/typedef/ApplicationCommandData) type so we can understand what the components of a command are.

-   `name` - Every command has a name. For context menu commands, this name appears in the menu. For slash commands, this is the name of the command that you type in the message box. For example, the command `/modmail close` would have a name of `modmail`.
-   `nameLocalizations` - This is used to let commands show with different names in different client languages. You can look into this if you want, but I will not cover it in this tutorial.
-   `description` - The description that shows up in the slash command popup. This does not do anything for context menu commands.
-   `descriptionLocalizations` - Similarly, this lets commands show different descriptions by language, and I will not cover it.
-   `type` - The [type](https://discord.js.org/#/docs/discord.js/stable/typedef/ApplicationCommandType) of command. As of the time of writing, there are three: `CHAT_INPUT` (slash), `USER` (user context menu), and `MESSAGE` (message context menu).
-   `options` - a list of [`ApplicationCommandOptionData`](https://discord.js.org/#/docs/discord.js/stable/typedef/ApplicationCommandOptionData) (only applies to slash commands) which are the command parameters that the user can specify. We'll get into this eventually.
-   `defaultPermission` - whether or not the command is allowed to be used by default. By default, it's true, but if not, the command will need to be manually enabled through the (as of the time of writing, relatively) new command permission management system in the server settings. We'll leave this as the default and use a different way to check permissions.

Let's create a `/ping` command that will just reply with `Pong!`. To register this command, we need to create it via the API, but we also don't want to create it multiple times. Not to worry - by using `set`, it won't create commands that exist, it'll update commands that we've changed, and it will create commands as needed.

`ApplicationCommandManager#set` takes a list of commands as its first argument, but it also has a second optional argument - the guild ID. If you specify this, it will only create the commands within your server. If you only use your bot in one server, the only difference between server-specific and global is that people can use your commands in DMs if it's global. If you don't need this, I would recommend specifying your guild ID since updating global commands can take a long time to actually update for users.

So, let's create our slash command named `ping`.

```js
client.once("ready", async () => {
    await client.application.commands.set(
        [
            {
                type: "CHAT_INPUT",
                name: "ping",
                description: 'Replies with "Pong!"',
            },
        ],
        "GUILD ID"
    );
});
```

Note that the guild ID is a _string_ and not a _number_ - JavaScript numbers don't have enough precision at high enough values. Also note that `type: "CHAT_INPUT"` is actually not necessary because it is the default value.

If we run the bot now and wait for it to set the commands, if you go back into your server you should find a `/ping` command in the slash command menu now. However, if you use it, of course nothing will happen since we haven't specified how to handle it. So, how do we handle it? Rather than waiting for an incoming message, we wait for an incoming interaction. When a user uses a slash command, it creates an interaction and sends it to your bot specifically (this is why slash commands are better than message commands from the development side - all interactions your bot receives were intended for your bot and are not sent to anyone else, and you can reply to them much more easily and without using as much of your rate limit).

The event we listen to is [`interactionCreate`](https://discord.js.org/#/docs/discord.js/stable/class/Client?scrollTo=e-interactionCreate), which sends an [`Interaction`](https://discord.js.org/#/docs/discord.js/stable/class/Interaction) as its argument. Note that this will not just be slash commands - the two context menus are also interactions, as well as selection menus and buttons. Modals are also interactions but are not supported in the stable version of discord.js as of the time of writing, so you need to manually add websocket hooks to get modals (we will not go over that in this tutorial as it involves manually connecting to the API).

```js
client.on("interactionCreate", async (interaction) => {});
```

We now need two filters. Firstly, we need to check if this interaction is a slash command. Secondly, we need to check for its name.

```js
client.on("interactionCreate", async (interaction) => {
    if (interaction.isCommand()) {
        if (interaction.commandName == "ping") {
        }
    }
});
```

Note that we used `interaction.commandName` here - keep that in mind that for slash and context menu commands, you need `.commandName` and not `.name`.

Finally, we just need to reply. We need to briefly talk about replying to interactions. You can just send a message in the channel but the interaction will show as failed to the user. This is because interactions need to be explicitly replied to in order for the user to receive that it succeeded. When the user creates an interaction, your bot will have access to the response token for 15 minutes. It will have 3 seconds to reply to the interaction. If it doesn't do so, the user will just see something like "interaction failed" or "application does not respond" (which is what you should see if you try to use your bot's ping command right now).

If your command will take longer to compute, you can also _defer_ the response. You must do this within 3 seconds, but if you do so, you will be able to reply up until 15 minutes after the interaction was created (after which it will show as failed). This will leave a "(Bot Name) is thinking" message.

You can also follow up to an interaction with an additional reply and edit or delete the reply itself.

Note that you cannot reply multiple times to an interaction (you can send follow-ups, but you can only send exactly one _initial_ reply itself).

You can read the documentation for each of these methods [here](https://discord.js.org/#/docs/discord.js/stable/class/CommandInteraction), but we'll go over them as needed throughout the tutorial.

To reply here, we will simply call `await interaction.reply("Pong!");`. Add this line inside the inner `if` statement and restart your bot, then run `/ping`. You should get something like the following:

[![][1]][1]

Something else that you can do with interactions that you cannot with normal message commands is keep them _completely hidden_ from other users. Try replacing your reply line with the following: `await interaction.reply({ ephemeral: true, content: "Pong!" });`. You will see the following when you restart your bot and run the command again:

[![][2]][2]

Once you start having a lot of commands, I would recommend that you either extract your command logic to functions and call them from within the handler, or you can do something like the following:

```js
const commands = {
    ping: async (interaction) => await interaction.reply("Pong!"),
};

client.on("interactionCreate", async (interaction) => {
    if (interaction.isCommand()) {
        await commands[interaction.commandName](interaction);
    }
});
```

That way, you won't end up with a really long event handler containing potentially tens of if-else statements, which can rapidly get messy and hard to maintain.

Now, to delete this command, just remove the command from the list and run the bot (with something like `await client.application.commands.set([], "ID")`). This will clear all commands from the server that belonged to your bot.

---

Congratulations on your first Discord bot in DJS! Next time, we will make a simple boost message system that will detect when someone boosts your server and send a thank-you message.

    [1]: https://i.imgur.com/kwkThm8.png
    [2]: https://i.imgur.com/1IfjNAE.png
