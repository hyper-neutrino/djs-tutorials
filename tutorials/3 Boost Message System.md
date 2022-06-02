# Boost Message Tutorial

This lesson will go over how to create a simple bot feature that will send a message when someone boosts the server with a thank-you message and anything else you might want to add.

## Detecting Boosts

When a user goes from not boosting the server to boosting the server, they will fire off a `memberUpdate` event. You must have the `GUILD_MEMBERS` intent enabled to detect this event.

[`GuildMember`](https://discord.js.org/#/docs/discord.js/stable/class/GuildMember) objects contain `premiumSince` and `premiumSinceTimestamp` fields, which are the time since which the user has been boosting the server as a `Date` and as a timestamp (number), respectively. You can use either because we don't actually care about the value, we just care about whether or not it is defined. If it is defined, the user is a booster, and if not, they are not.

Therefore, we need to detect member update events and check for when someone starts boosting. The `memberUpdate` event will send two objects, the first representing the member and their properties before whatever update occurred, and the second representing them after.

Firstly, let's set up the event handler.

```js
client.on("memberUpdate", async (before, after) => {});
```

Now, we detect if the user has just started boosting. This occurs when `before.premiumSince` wasn't defined (the user is not a booster) but `after.premiumSince` is (the user is now a booster). Note that if the user adds multiple boosts, this method will only detect them beginning to boost, which is generally what you want as you don't want your bot to send the same message multiple times if the user adds multiple boosts to the server.

```js
client.on("memberUpdate", async (before, after) => {
    if (before.premiumSince === undefined && after.premiumSince !== undefined) {
    }
});
```

You can reverse `before` and `after` to detect when a user stops boosting / their boosts run out.

## Sending The Message

Now, it's just a matter of sending a custom message. Note that in previous examples, we were able to just reply to the message. However, there is no message here; in fact, there is not even a channel. We will need to define which channel to post to. There are two options here - firstly, you can use the system channel for the server, which you can obtain with [`guild.systemChannel`](https://discord.js.org/#/docs/discord.js/stable/class/Guild?scrollTo=systemChannel). To get the guild object, we can just do `before.guild` or `after.guild`, whichever you prefer, as member objects contain a reference to the guild that the member is in.

The second option is to set up a channel ourselves. You could make a command to change the channel but that is a bit complex as then you would need to either have the program dynamically modify its configuration file or store bot settings within a database, which is a bit beyond the scope of this lesson, although we will need to get into it eventually. Instead, we are just going to hardcode it (or you could put it in a configuration file as a static value and be able to edit the channel by altering that file).

Let's explore the first method first. Inside the `if` block, we can just put this line: `await after.guild.systemChannel.send("Thanks for the boost!");`. This is a terrible message, but we'll leave it as a placeholder for now.

The second method requires us to fetch a channel from the ID. To begin, copy the channel ID of the channel you want to post these messages to. Then, you can add the following line: `const channel = await after.guild.channels.fetch("ID goes here");`. `after.guild.channels` is a [`GuildChannelManager`](https://discord.js.org/#/docs/discord.js/stable/class/GuildChannelManager) which contains utilities to work with channels in a server. [`GuildChannelManager#fetch`](https://discord.js.org/#/docs/discord.js/stable/class/GuildChannelManager?scrollTo=fetch) is a method that accepts an optional ID and an optional options object, which we will ignore. If you don't give it an ID, it will just return a [`Collection`](https://discord.js.org/#/docs/collection/main/class/Collection) of all channels; otherwise, it will find the channel by that ID.

Once we've fetched the channel, we can just add `await channel.send("Thanks for the boost!");` after it.

You can now run the bot and test it by boosting your server, which is unfortunately the only way to actually test that this works.

## Improving The Message

You probably don't want to just send a plaintext message and want to make a fancy embed. You can take a look at [`MessageOptions`](https://discord.js.org/#/docs/discord.js/stable/typedef/MessageOptions) (which extends [`BaseMessageOptions`](https://discord.js.org/#/docs/discord.js/stable/typedef/BaseMessageOptions)) on the documentation to figure out how to construct the message yourself. You can also use something like Discohook and create the message and then click the "JSON Data Editor" button in the bottom left and copy-paste the JSON data into your program. Let's use a basic example:

[![][1]][1]

Firstly, we need to obtain the message content. How do we mention a user? You may know that the format is `<@ID>`, so we could do `"<@" + after.id + ">"`. Using string interpolation, we could also do `` `<@${after.id}>` ``. How this works is that when we denote a string using backticks, it is a template string, and then we can insert code inside `${ ... }` which will be evaluated when that string is evaluated, obtaining a dynamic result. In this case, we just insert `after.id` into the `<@ ... >`. However, discord.js has an even simpler method: `after.toString()` will simply return the string that is needed to mention the user.

Thus, we can start with the following:

```js
await channel.send({
    content: after.toString(),
});
```

Now, let's work on the embed. The message options object takes a list of embeds in the property name `embeds`, so let's start by creating one embed and giving it a title:

```js
await channel.send({
    content: after.toString(),
    embeds: [
        {
            title: "Thanks for the boost!",
        },
    ],
});
```

But, how do we add the user's tag? You can simply use [`User#tag`](https://discord.js.org/#/docs/discord.js/stable/class/User?scrollTo=tag). Be careful though - our current object is a `GuildMember` and not a `User` object. `GuildMember` doesn't have the `tag` property for some reason, but it has the `user` property which is the underlying user object for the user on their own without considering guild membership, so we can do `after.user.tag`. Therefore, our title becomes `"Thanks for the boost, " + after.user.tag + "!"`, or `` `Thanks for the boost, ${after.user.tag}!` ``.

Next, for the embed description, we have three things we need to insert - the guild name, the number of boosts, and the user's mention again. The first is easy - `after.guild.name`. The second is also actually easy - `after.guild.premiumSubscriptionCount`. Note that this is the number of boosts, not the number of boosters. To get the latter, you need to filter through all of the members yourself. The last one we already did. I will use string interpolation throughout this tutorial because I like it better. Thus, it will look something like this:

```js
await channel.send({
    content: after.toString(),
    embeds: [
        {
            title: `Thanks for the boost, ${after.user.tag}!`,
            description: `**${after.guild.name}** now has **${after.guild.premiumSubscriptionCount}** boosts!\n\nThank you for supporting the server, ${after}!`,
        },
    ],
});
```

`\\n` is a newline within a string. If you want the literal characters backslash followed by an `n`, put another backslash before the backslash to _escape_ it (`\\\\`), making it evaluate as a literal backslash character.

Note that I took a shortcut here - rather than doing `${after.toString()}`, I did `${after}`. The result of a sub-expression within string interpolation is converted into a string using `.toString()`.

(There is one small bug with this; if there is only one boost, it will say "1 boosts", which you can fix if you'd like.)

Finally, the last element we're missing is the embed color. You can just include the `color` field in the embed; for example, `color: "RED"`, or `color: "ff0000"` to use a HEX code. You can even use an array of RGB values like `color: [255, 0, 0]`. You can find a full list of existing color strings on the [`ColorResolvable`](https://discord.js.org/#/docs/discord.js/stable/typedef/ColorResolvable) documentation page.

## Bonus Feature

Let's also include the user's profile avatar in the boost message as a thumbnail. To do that, we specify the `thumbnail` property of the embed which is a [`MessageEmbedThumbnail`](https://discord.js.org/#/docs/discord.js/stable/typedef/MessageEmbedThumbnail) object. We can specify things like the width and height as well as the proxy URL but we can ignore all of those and just set only the URL itself:

```js
thumbnail: {
    url: after.displayAvatarURL({ dynamic: true });
}
```

`GuildMember#avatarURL` also exists but returns undefined if the member doesn't have a server-specific profile avatar. Using `displayAvatarURL` will make it default to the user's global avatar, and if that doesn't exist, their blank Discord logo avatar.

The `dynamic` option makes the function return an animated version. If you turn that off, the result will always be a static image, even if the user has an animated avatar.

## Conclusion

Your code may now look something like this:

```js
client.on("memberUpdate", async (before, after) => {
    if (before.premiumSince === undefined && after.premiumSince !== undefined) {
        const channel = await after.guild.channels.fetch("ID");
        await channel.send({
            content: after.toString(),
            embeds: [
                {
                    title: `Thanks for the boost, ${after.user.tag}!`,
                    description: `**${after.guild.name}** now has $${after.guild.premiumSubscriptionCount}** boosts!\n\nThank you for supporting the server, ${after}!`,
                    color: "RED",
                    thumbnail: {
                        url: member.displayAvatarURL({ dynamic: true }),
                    },
                },
            ],
        });
    }
});
```

---

Of course, you should put in your own data and decorations, and you can look into things like modifying the footer as well or adding a large image at the bottom, but you should now have your first discord.js bot feature, a simple boost message system. Next time, we'll make a QOTD system that allows trusted users (probably staff) to insert questions into a question bank that will be posted daily.

    [1]: https://i.imgur.com/apBPBwc.png
