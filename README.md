# Discord Mobile Status

A minimal, well-documented guide and code reference for displaying the green mobile phone status indicator next to a Discord bot's avatar. Implementations are provided for both discord.js and discord.py.

## Table of Contents

1. Overview
2. How It Works
3. Important Notice
4. Requirements
5. Project Structure
6. Environment Configuration
7. Implementation: discord.js
8. Implementation: discord.py
9. Switching Between Android and iOS
10. Verifying the Result
11. Troubleshooting
12. Frequently Asked Questions
13. Security Guidelines
14. Contributing
15. License

---

## 1. Overview

By default, a Discord bot appears online with no device indicator. Regular users who are connected from a mobile device, however, are shown with a small phone icon beside their avatar in the member list and on their profile.

This project demonstrates how to make a bot appear as if it is connected from a mobile device, so the same phone icon is displayed beside the bot's avatar. The change is cosmetic only and does not affect how the bot handles commands, events, or permissions.

## 2. How It Works

When a bot connects to the Discord Gateway, it sends an `IDENTIFY` payload. This payload contains a `properties` object that describes the client, including the operating system, browser, and device.

Discord reads these values to decide which device indicator to show. A standard bot library sends its own library name (for example `discord.js` or `discord.py`). By replacing the `browser` value with `Discord Android` or `Discord iOS` before the connection is established, Discord treats the session as a mobile client and displays the mobile indicator.

Key points:

- The change is applied once, before the bot logs in.
- No additional permissions or intents are required.
- The bot continues to function exactly as before.

## 3. Important Notice

This technique is not an official Discord API feature. It relies on the way Discord currently interprets the identify properties, and it may change or stop working at any time without notice.

- Use it at your own risk.
- Do not depend on it for any critical functionality.
- Review the Discord Developer Terms of Service and Developer Policy before using it in a production or public bot.

## 4. Requirements

| Item | Requirement |
|---|---|
| Discord bot | A bot application created in the Discord Developer Portal |
| Bot token | Available from the Bot section of your application |
| Node.js (for discord.js) | Version 18 or higher |
| discord.js | Version 14 |
| Python (for discord.py) | Version 3.8 or higher |
| discord.py | Version 2.x |

## 5. Project Structure

Suggested layout for a repository using this project:

```
discord-mobile-status/
|-- discord.js/
|   |-- index.js
|   |-- package.json
|-- discord.py/
|   |-- bot.py
|   |-- requirements.txt
|-- .env.example
|-- .gitignore
|-- README.md
```

You only need the folder for the library you use.

## 6. Environment Configuration

Never place your bot token directly in source code. Store it in an environment file instead.

Create a file named `.env` in the project root:

```
DISCORD_TOKEN=your_bot_token_here
```

Create a file named `.gitignore` and add the following so the token is never committed:

```
.env
node_modules/
__pycache__/
```

Optionally, commit a `.env.example` file that shows the required variable without a real value:

```
DISCORD_TOKEN=
```

## 7. Implementation: discord.js

### 7.1 Installation

```bash
npm init -y
npm install discord.js @discordjs/ws dotenv
```

### 7.2 Code

Create a file named `index.js`:

```js
require('dotenv').config();

const { Client, GatewayIntentBits } = require('discord.js');
const { DefaultWebSocketManagerOptions } = require('@discordjs/ws');

// Set the device identifier before the client logs in.
// Use 'Discord Android' for Android or 'Discord iOS' for iOS.
DefaultWebSocketManagerOptions.identifyProperties.browser = 'Discord Android';

const client = new Client({
  intents: [GatewayIntentBits.Guilds],
});

client.once('ready', () => {
  console.log(`Logged in as ${client.user.tag} with mobile status enabled.`);
});

client.login(process.env.DISCORD_TOKEN);
```

### 7.3 Running the Bot

```bash
node index.js
```

### 7.4 Notes

- The line that sets `identifyProperties.browser` must run before `client.login()`.
- `@discordjs/ws` is a dependency of discord.js. Installing it explicitly makes sure it can be imported. Only one version should be present in your dependency tree. You can check this with `npm ls @discordjs/ws`.

## 8. Implementation: discord.py

### 8.1 Installation

```bash
pip install -U discord.py python-dotenv
```

If you prefer a requirements file, create `requirements.txt`:

```
discord.py>=2.0
python-dotenv
```

### 8.2 Code

Create a file named `bot.py`:

```python
import os
import sys

import discord
from discord.gateway import DiscordWebSocket
from dotenv import load_dotenv

load_dotenv()

# Device identifier. Use 'Discord Android' for Android or 'Discord iOS' for iOS.
DEVICE = 'Discord Android'


async def mobile_identify(self):
    """Replacement for DiscordWebSocket.identify that reports a mobile device."""
    payload = {
        'op': self.IDENTIFY,
        'd': {
            'token': self.token,
            'properties': {
                'os': sys.platform,
                'browser': DEVICE,
                'device': DEVICE,
            },
            'compress': True,
            'large_threshold': 250,
        },
    }

    if self.shard_id is not None and self.shard_count is not None:
        payload['d']['shard'] = [self.shard_id, self.shard_count]

    state = self._connection
    if state._activity is not None or state._status is not None:
        payload['d']['presence'] = {
            'status': state._status,
            'game': state._activity,
            'since': 0,
            'afk': False,
        }

    if state._intents is not None:
        payload['d']['intents'] = state._intents.value

    await self.call_hooks(
        'before_identify', self.shard_id, initial=self._initial_identify
    )
    await self.send_as_json(payload)


# Apply the patch before the client connects.
DiscordWebSocket.identify = mobile_identify

intents = discord.Intents.default()
client = discord.Client(intents=intents)


@client.event
async def on_ready():
    print(f'Logged in as {client.user} with mobile status enabled.')


client.run(os.getenv('DISCORD_TOKEN'))
```

### 8.3 Running the Bot

```bash
python bot.py
```

### 8.4 Notes

- The patch replaces the library's internal `identify` method. Because this is internal code, a future discord.py update may change its structure. If an update breaks the bot, compare the replacement function with the current `identify` method in `discord/gateway.py` and adjust accordingly.
- If you use `commands.Bot` instead of `discord.Client`, the same patch applies. Keep the patch above the bot creation.

## 9. Switching Between Android and iOS

Both platforms display the same phone icon. To change the reported device, edit one value:

| Platform | Value |
|---|---|
| Android | `Discord Android` |
| iOS | `Discord iOS` |

In discord.js, change the string assigned to `identifyProperties.browser`. In discord.py, change the `DEVICE` constant.

## 10. Verifying the Result

1. Start the bot and wait until it is online.
2. Open a server where the bot is a member.
3. Locate the bot in the member list.
4. Confirm that a small green phone icon appears next to the bot's avatar.

The indicator is displayed only while the bot is online. If the bot is offline, no device indicator is shown.

## 11. Troubleshooting

| Problem | Possible Cause | Solution |
|---|---|---|
| The icon does not appear | Discord client has cached the old state | Restart the bot and refresh the Discord client |
| The icon does not appear | Patch runs after login | Move the patch above `client.login()` or `client.run()` |
| `Cannot find module '@discordjs/ws'` | Package not installed | Run `npm install @discordjs/ws` |
| Unexpected behavior in discord.js | Multiple versions of `@discordjs/ws` | Run `npm ls @discordjs/ws` and reinstall so a single version remains |
| `AttributeError` in discord.py | Library internals changed | Update discord.py and compare against the current `identify` method |
| `Improper token has been passed` | Token missing or incorrect | Check the `.env` file and the variable name |
| Bot does not start | Missing dependencies | Re-run the installation commands |

## 12. Frequently Asked Questions

**Does this affect my bot's commands or permissions?**
No. Only the reported device type changes.

**Does this work with slash commands?**
Yes. Slash commands are unrelated to the identify properties.

**Can I use this with sharding?**
Yes. The discord.js approach applies globally. The discord.py replacement already handles shard information.

**Will this always work?**
There is no guarantee. It depends on current Discord behavior, which may change.

**Is a specific intent required?**
No. Use whichever intents your bot needs.

## 13. Security Guidelines

- Never commit your bot token to a public or private repository.
- Keep the token in a `.env` file and list it in `.gitignore`.
- If a token is ever exposed, regenerate it immediately from the Discord Developer Portal.
- Grant your bot only the permissions it requires.

## 14. Contributing

Contributions are welcome.

1. Fork the repository.
2. Create a feature branch.
3. Commit your changes with a clear message.
4. Open a pull request describing what was changed and why.

Please open an issue first for larger changes so they can be discussed.

## 15. License

This project is released under the MIT License. You are free to use, modify, and distribute it, provided the original copyright notice is kept.
