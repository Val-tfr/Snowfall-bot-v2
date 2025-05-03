import discord
from discord.ext import commands
from discord import app_commands
import os

intents = discord.Intents.default()
bot = commands.Bot(command_prefix="/", intents=intents)

user_data = {}

class ArmsView(discord.ui.View):
    def __init__(self, user_id):
        super().__init__(timeout=None)
        self.user_id = user_id

    @discord.ui.button(label=" Acheter le permis (500 $)", style=discord.ButtonStyle.primary)
    async def buy_permit(self, interaction: discord.Interaction, button: discord.ui.Button):
        user = user_data.setdefault(self.user_id, {"permit": False, "inventory": []})
        if user["permit"]:
            await interaction.response.send_message("Tu possèdes déjà un permis de port d’armes.", ephemeral=True)
        else:
            user["permit"] = True
            await interaction.response.send_message("✅ Permis de port d’armes acheté avec succès !", ephemeral=True)

    @discord.ui.button(label="📦 Ouvrir le catalogue", style=discord.ButtonStyle.secondary)
    async def open_catalogue(self, interaction: discord.Interaction, button: discord.ui.Button):
        user = user_data.get(self.user_id, {"permit": False})
        if not user["permit"]:
            await interaction.response.send_message("❌ Tu dois d’abord acheter un permis de port d’armes.", ephemeral=True)
            return
        embed = discord.Embed(title="Catalogue Ammu Nation", description="Voici les armes disponibles :", color=0x36393F)
        embed.set_image(url="https://cdn.discordapp.com/attachments/1204466945422895154/1241744711752253510/ammu.png")
        embed.add_field(name="**• Glock 17**", value="Prix : 2 500 $", inline=False)
        embed.add_field(name="**• Glock 18**", value="Prix : 3 200 $", inline=False)
        embed.add_field(name="**• Beretta M9**", value="Prix : 3 500 $", inline=False)
        embed.add_field(name="**• Colt 357 Magnum**", value="Prix : 4 000 $", inline=False)
        await interaction.response.send_message(embed=embed, ephemeral=True)

@bot.tree.command(name="arme", description="Ouvre le menu d'achat d'arme.")
async def arme(interaction: discord.Interaction):
    await interaction.response.send_message("Bienvenue chez Ammu Nation !", view=ArmsView(interaction.user.id), ephemeral=True)

@bot.event
async def on_ready():
    print("Bot en ligne.")
    try:
        synced = await bot.tree.sync()
        print(f"Commandes synchronisées : {len(synced)}")
    except Exception as e:
        print(f"Erreur de synchronisation : {e}")

bot.run(os.getenv("BOT_TOKEN"))
