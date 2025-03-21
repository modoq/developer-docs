---
description: Getting Started in creating your first ever Rocket.Chat App
---

# Creating, Updating & Testing an App

Now, that you've understood the basic concepts of the Apps Engine and installed the CLI, you can create an extremely basic RC App and test it out to understand things initially. To get started, just recall the commands inside the Apps Engine CLI

### `rc-apps create`

The development tools provide a command to quickly scaffold a new Rocket.Chat App, simply run `rc-apps create` and a new folder will be created inside the current working directory with a basic App which does nothing but will compile and be packaged in the `dist` folder.

### App description

The app description file, named `app.json`, contains basic information about the app. You can check the [app-schema.json](https://github.com/RocketChat/Rocket.Chat.Apps-engine/blob/master/src/definition/app-schema.json) file for all the detailed information and fields allowed in the app description file, the basic structure is similar to this:

```
{
    "id": "5cb9a329-0613-4d39-b20f-cc2cc9175df5",
    "name": "App Name",
    "nameSlug": "app-name",
    "version": "0.0.1",
    "requiredApiVersion": "^1.4.0",
    "description": "App which provides something very useful for Rocket.Chat users.",
    "author": {
        "name": "Author Name <author@email.com>",
        "support": "Support Url or Email"
    },
    "classFile": "main.ts",
    "iconFile": "beautiful-app-icon.jpg"
}
```

### Extending the App class

The basic creation of an App is based on extending the `App` class from the Rocket.Chat Apps _definition_ library. Your class also has to implement the constructor and optionally the `initialize` function, for more details on those check the [App definition documentation](https://rocketchat.github.io/Rocket.Chat.Apps-engine/classes/app.html).

## Start Developing

In this example, we already have our main file called `LiftoffApp.ts` that was generated when we first created our project:

```javascript
import {
    IAppAccessors,
    ILogger,
} from '@rocket.chat/apps-engine/definition/accessors';
import { App } from '@rocket.chat/apps-engine/definition/App';
import { IAppInfo } from '@rocket.chat/apps-engine/definition/metadata';

export class LiftoffApp extends App {
    constructor(info: IAppInfo, logger: ILogger, accessors: IAppAccessors) {
        super(info, logger, accessors);
    }
}
```

Now let's add some functionality to it

### Adding a Slashcommand

A Slashcommand is a way to call the app installed in Rocket.Chat. Your app can have multiple slashcommands and subcommands. In our example, we will add the `liftoff` slashcommand and it will be called like this by the user inside the chat:

```
/liftoff
```

First, we create a new directory called `commands` in our project's root and create a file called `liftoff.ts`. You can copy the file's content:

```javascript
import {ISlashCommand, SlashCommandContext} from '@rocket.chat/apps-engine/definition/slashcommands';
import {IModify, IRead} from '@rocket.chat/apps-engine/definition/accessors';
import {App} from '@rocket.chat/apps-engine/definition/App';

export class LiftoffCommand implements ISlashCommand {
    public command = 'liftoff';
    public i18nDescription = 'Tells the user if it is time to liftoff';
    public i18nParamsExample = '';
    public providesPreview = false;

    constructor(private readonly app: App) {}

    public async executor(context: SlashCommandContext, read: IRead, modify: IModify): Promise<void> {
        const message = 'Time to lift off!';

        const messageStructure = await modify.getCreator().startMessage();
        const sender = context.getSender(); // the user calling the slashcommand
        const room = context.getRoom(); // the current room

        messageStructure
        .setSender(sender)
        .setRoom(room)
        .setText(message);

        await modify.getCreator().finish(messageStructure);
    }
}
```

> You can learn more about organising complex slash commands in our [Sub-command pattern](../recipes/sub-command-pattern.md) recipe

### Registering the slashcommand

After adding our slashcomamnd logic, we have to register the slashcommand in out app by extending its configuration:

```javascript
import {
    IAppAccessors,
    ILogger,
    IConfigurationExtend,
} from '@rocket.chat/apps-engine/definition/accessors';
import { App } from '@rocket.chat/apps-engine/definition/App';
import { IAppInfo } from '@rocket.chat/apps-engine/definition/metadata';

import { LiftoffCommand } from './commands/liftoff';

export class LiftoffApp extends App {
    constructor(info: IAppInfo, logger: ILogger, accessors: IAppAccessors) {
        super(info, logger, accessors);
    }

    protected async extendConfiguration(configuration: IConfigurationExtend): Promise<void> {
        await configuration.slashCommands.provideSlashCommand(new LiftoffCommand(this));
    }
}
```

We first had to import the `Liftoff` class and then register an instance of using the `provideSlashCommand` function. We pass the app's instance (`this`) so our slashcommand have access to all the functionalities of the app.

## Updating the app

If you want to, for example, change the message sent to the room from `Time to lift off!` to `Lift off now!`, you have to simply save the modifications and run:

```
rc-apps deploy --url http://localhost:3000 --username <your_username> --password <your_password> --update
```

The app will be updated and by sending `/liftoff`, the message will reflect the change you have made in the app.

## Testing the App

Now that you have your App ready, you can test it before submitting it.

To test your app, you need a Rocket.Chat server running locally on your machine and an admin user in it.

See [Installing Rocket.Chat for Developing](../../rocket.chat/rocket.chat-server/linux.md) to run Rocket.Chat in develop mode. Enable Apps development mode by navigating to `Administration > General` then scroll down to Apps and click on the `True` radio button over the Enable development mode.

or run it in preview mode with docker using the command:

```
docker run -it --rm -p 3000:3000 -v `pwd`/rocketdb:/var/lib/mongodb rocketchat/rocket.chat.preview
```

Having the server running, simply run inside the app project's directory:

```
rc-apps deploy --url http://localhost:3000 --username user_username --password user_password
```

Where:

`http://localhost:3000` is your local server URL (if you are running in another port, change the `3000` to the appropriate port)

`user_username` is the username of your admin user.

`user_password` is the password of your admin user.

If you want to update the app deployed in your Rocket.Chat instance after making changes to it, you can run:

```
rc-apps deploy --url http://localhost:3000 --username user_username --password user_password --update
```

> After version 1.9 of the App Engine CLI, the `--update` flag isn't strictly necessary for updating an existing App, you can just run the `deploy` command without it.
