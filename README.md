# 📱 Discord Bot Mobile Status

Apne Discord bot ke avatar ke neeche **green phone icon** (mobile status) dikhane ka simple tarika.

> Example: bot ke naam ke saath 📱 icon dikhega, bilkul real mobile user jaisa.

---

## ❓ Ye kaise kaam karta hai?

Jab bot Discord se connect hota hai, wo `IDENTIFY` payload me apna device/browser batata hai.
Normally bot `Discord Client` / `discord.js` / `discord.py` bhejta hai.
Hum use badal ke `Discord Android` (ya `Discord iOS`) bhej dete hain, to Discord bot ko **mobile** pe online maan leta hai.

- Bot ke normal kaam (commands, events) pe **koi farak nahi** padta.
- Icon **member list aur profile** me dikhta hai, tab hi jab bot **online** ho.

---

## ⚠️ Zaroori baat

- Ye **unofficial trick** hai, official Discord API feature nahi.
- Discord future me ise change ya band kar sakta hai.
- Apne risk pe use karo.

---

## 🟨 Option 1: discord.js (v14)

### 1. Install

```bash
npm install discord.js @discordjs/ws
```

### 2. Code (`index.js`)

```js
const { Client, GatewayIntentBits } = require('discord.js');
const { DefaultWebSocketManagerOptions } = require('@discordjs/ws');

// ✅ Mobile status: ye line client.login() se PEHLE honi chahiye
DefaultWebSocketManagerOptions.identifyProperties.browser = 'Discord Android';
// iOS chahiye to: 'Discord iOS'

const client = new Client({
  intents: [GatewayIntentBits.Guilds],
});

client.once('ready', () => {
  console.log(`✅ Logged in as ${client.user.tag} (Mobile)`);
});

client.login('YOUR_BOT_TOKEN');
```

### 3. Run

```bash
node index.js
```

---

## 🐍 Option 2: discord.py (v2.x)

### 1. Install

```bash
pip install -U discord.py
```

### 2. Code (`bot.py`)

```python
import sys
import discord
from discord.gateway import DiscordWebSocket


# ✅ Mobile status: identify function ko replace kar rahe hain
async def mobile_identify(self):
    payload = {
        'op': self.IDENTIFY,
        'd': {
            'token': self.token,
            'properties': {
                'os': sys.platform,
                'browser': 'Discord Android',  # iOS ke liye: 'Discord iOS'
                'device': 'Discord Android',
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


DiscordWebSocket.identify = mobile_identify

intents = discord.Intents.default()
client = discord.Client(intents=intents)


@client.event
async def on_ready():
    print(f'✅ Logged in as {client.user} (Mobile)')


client.run('YOUR_BOT_TOKEN')
```

### 3. Run

```bash
python bot.py
```

---

## ✅ Check kaise karein?

1. Bot ko apne server me online karo.
2. Member list me bot ka avatar dekho, neeche **green phone icon** dikhna chahiye.
3. Agar nahi dikha, Discord app restart karo ya bot dobara start karo.

---

## 🛠️ Problem aaye to

| Problem | Solution |
|---|---|
| Icon nahi dikh raha | Bot ko restart karo aur Discord refresh (`Ctrl + R`) karo |
| discord.js me error | `npm ls @discordjs/ws` check karo, ek hi version install hona chahiye |
| discord.py me error | `pip install -U discord.py` karke latest version lo |
| Pehle jaisa hi dikh raha | Patch line `login()` / `run()` se pehle likhi hai ya nahi check karo |

---

## 🔒 Security

- Bot token **kabhi GitHub pe mat daalo**.
- `.env` file use karo aur `.gitignore` me add karo.

---

## 📄 License

Free to use. Credit dena na bhoolo ❤️
