# Redem And Subscribe System
To create a redeem and subscription system using discord.py, SQLite3, and random codes, you can follow the example code below. This example assumes that you have a Discord bot using the commands.Bot class from the discord.ext extension module.

## installation
```
pip install sqllite3
```
```
pip install discord
```
requirements : ```sqllite3```
requirements : ```discord```
# First, let's set up the SQLite database:
```python
import sqlite3

# Connect to SQLite database
conn = sqlite3.connect("subscription_codes.db")
cursor = conn.cursor()

# Create table if not exists
cursor.execute('''CREATE TABLE IF NOT EXISTS subscriptions (
                user_id INTEGER PRIMARY KEY,
                subscription_status TEXT NOT NULL,
                redeem_code TEXT)''')
conn.commit()


```

### Now, the Discord bot code:

```python
import discord
from discord.ext import commands
import random
import sqlite3

intents = discord.Intents.default()
client = commands.Bot(command_prefix='!', intents=intents)

# Connect to SQLite database
conn = sqlite3.connect("subscription_codes.db")
cursor = conn.cursor()

# Generate random redeem codes
def generate_redeem_code():
    return ''.join(random.choices('ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789', k=8))

@client.command()
async def generate_code(ctx):
    code = generate_redeem_code()
    cursor.execute("INSERT INTO subscriptions (redeem_code, subscription_status) VALUES (?, ?)", (code, 'Inactive'))
    conn.commit()
    await ctx.send(f"Generated Code: {code}")

@client.command()
async def redeem(ctx, code: str):
    cursor.execute("SELECT subscription_status FROM subscriptions WHERE redeem_code = ?", (code,))
    result = cursor.fetchone()
    if result:
        if result[0] == 'Inactive':
            cursor.execute("UPDATE subscriptions SET user_id = ?, subscription_status = 'Active' WHERE redeem_code = ?", (ctx.author.id, code))
            conn.commit()
            await ctx.send(f"Successfully redeemed the code! You are now subscribed.")
        else:
            await ctx.send("This code has already been used.")
    else:
        await ctx.send("Invalid redeem code.")

@client.command()
async def check_subscription(ctx):
    cursor.execute("SELECT subscription_status FROM subscriptions WHERE user_id = ?", (ctx.author.id,))
    result = cursor.fetchone()
    if result:
        await ctx.send(f"Your subscription status is: {result[0]}")
    else:
        await ctx.send("You are not subscribed.")

client.run('YOUR_BOT_TOKEN')
```
<h2>🛡️ License:</h2>

This project is licensed under the  [Hugs For Bugs Development](https://discord.gg/bFK5RR9upe)
