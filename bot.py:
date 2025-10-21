import discord
from discord.ext import commands, tasks
from datetime import datetime, timedelta
import asyncio
import os

intents = discord.Intents.all()
bot = commands.Bot(command_prefix='/', intents=intents)

# --- Custom Role Orders (Example) ---
WARNING_ROLES = [f"Warning {i}" for i in range(1, 11)]

@bot.event
async def on_ready():
    print(f"✅ {bot.user} is now active as The Imperial Enforcement Bot!")

# --- Enforce Warning Role Order ---
@bot.event
async def on_member_update(before, after):
    for i, role_name in enumerate(WARNING_ROLES):
        role = discord.utils.get(after.guild.roles, name=role_name)
        if role and role in after.roles:
            prev_role_name = WARNING_ROLES[i - 1] if i > 0 else None
            if prev_role_name and discord.utils.get(after.guild.roles, name=prev_role_name) not in after.roles:
                await after.remove_roles(role)
                await after.send(f"You cannot have **{role_name}** without **{prev_role_name}**. Role removed.")
                return

# --- Replace Ban with Temporary Suspension ---
@bot.slash_command(description="Suspend a user temporarily instead of banning.")
@commands.has_permissions(manage_roles=True)
async def suspend(ctx, member: discord.Member, hours: int, *, reason="No reason provided"):
    suspension_role = discord.utils.get(ctx.guild.roles, name="Temporary Suspension")
    if not suspension_role:
        suspension_role = await ctx.guild.create_role(name="Temporary Suspension", colour=discord.Colour.dark_gray())

    await member.add_roles(suspension_role)
    await ctx.respond(f"🔒 {member.mention} has been suspended for {hours} hours. Reason: {reason}")

    await asyncio.sleep(hours * 3600)
    await member.remove_roles(suspension_role)
    await ctx.send(f"✅ {member.mention}’s suspension has ended.")

# --- Auto-Role Based on Join Time ---
@tasks.loop(hours=24)
async def check_member_duration():
    for guild in bot.guilds:
        for member in guild.members:
            join_duration = (datetime.utcnow() - member.joined_at).days
            if join_duration >= 30:
                role = discord.utils.get(guild.roles, name="Veteran")
                if role and role not in member.roles:
                    await member.add_roles(role)
            elif join_duration >= 7:
                role = discord.utils.get(guild.roles, name="Active Member")
                if role and role not in member.roles:
                    await member.add_roles(role)

check_member_duration.start()

# --- Run the bot ---
bot.run(os.getenv("DISCORD_TOKEN"))
