# bot.py
Test
import discord
from discord.ext import commands, tasks
import aiohttp
import os
from dotenv import load_dotenv

load_dotenv()

TOKEN = os.getenv("DISCORD_TOKEN")
CHANNEL_ID = int(os.getenv("CHANNEL_ID", 0))
GROUP_ID = 12874996

bot = commands.Bot(command_prefix="!", intents=discord.Intents.default())

last_member_count = {}

@bot.event
async def on_ready():
    print(f"✅ Bot conectado: {bot.user}")
    monitor.start()

@tasks.loop(minutes=5)
async def monitor():
    try:
        async with aiohttp.ClientSession() as session:
            url = f"https://groups.roblox.com/v1/groups/{GROUP_ID}"
            async with session.get(url) as resp:
                if resp.status == 200:
                    data = await resp.json()
                    
                    if GROUP_ID not in last_member_count:
                        last_member_count[GROUP_ID] = data["memberCount"]
                    
                    if data["memberCount"] != last_member_count[GROUP_ID]:
                        embed = discord.Embed(
                            title="🔔 Atualização Detectada!",
                            description=f"**{data['name']}**",
                            color=discord.Color.green()
                        )
                        embed.add_field(name="👥 Membros", value=data["memberCount"], inline=False)
                        
                        channel = bot.get_channel(CHANNEL_ID)
                        if channel:
                            await channel.send(embed=embed)
                        
                        last_member_count[GROUP_ID] = data["memberCount"]
    except Exception as e:
        print(f"❌ Erro: {e}")

bot.run(TOKEN)
