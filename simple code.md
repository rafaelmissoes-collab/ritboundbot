import discord
from discord.ext import commands
import requests
import os
import json

# -----------------------------
# Fetch card list from Riftbound API
# -----------------------------
API_URL = "https://api.riftbound.build/storage/v1/object/list/riftbound-cards"
BASE_URL = "https://riftbound.build/storage/v1/object/public/riftbound-cards/"

print("🔄 Downloading card list...")

try:
    response = requests.get(API_URL)
    response.raise_for_status()
    cards_data = response.json()
except Exception as e:
    print(f"❌ Failed to fetch card list: {e}")
    exit(1)

# Build dictionary: card name -> image URL
card_dict = {}
for card in cards_data:
    name = card["name"]
    if name.endswith("_full-desktop.jpg"):
        clean_name = name.replace("_full-desktop.jpg", "").replace("_alt", "")
        card_dict[clean_name.lower()] = BASE_URL + name

print(f"✅ Loaded {len(card_dict)} cards from {API_URL}")

# Optional: save locally
with open("cards.json", "w", encoding="utf-8") as f:
    json.dump(card_dict, f, ensure_ascii=False, indent=2)

# -----------------------------
# Setup bot
# -----------------------------
intents = discord.Intents.default()
intents.message_content = True  # Required to read message content

bot = commands.Bot(command_prefix="!", intents=intents)

@bot.event
async def on_ready():
    print(f"🤖 Bot connected as {bot.user} (ID: {bot.user.id})")

# -----------------------------
# Commands
# -----------------------------
@bot.command(name="status")
async def status(ctx):
    await ctx.send(f"✅ Loaded {len(card_dict)} cards.")

@bot.command(name="card")
async def get_card(ctx, *, card_name: str):
    key = card_name.lower()
    if key in card_dict:
        embed = discord.Embed(title=card_name)
        embed.set_image(url=card_dict[key])
        await ctx.send(embed=embed)
    else:
        await ctx.send(f"❌ Card not found: `{card_name}`")

# -----------------------------
# Run the bot
# -----------------------------
BOT_TOKEN = os.getenv("my token here")
if not BOT_TOKEN:
    print("❌ DISCORD_BOT_TOKEN environment variable not set.")
    exit(1)

bot.run(BOT_TOKEN)


