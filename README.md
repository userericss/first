import discord
from discord.ext import commands
import os, random
import requests
import pyttsx3
from settings import token
import deepl
import asyncio
from datetime import datetime
from sympy import sympify
import pytz

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix='!', intents=intents)


@bot.event
async def on_ready():
    print(f'Hemos iniciado sesión como {bot.user}')


def get_duck_image_url():
    url = 'https://random-d.uk/api/random'
    res = requests.get(url)
    data = res.json()
    return data['url']


@bot.command('duck')
async def duck(ctx):
    '''El comando "duck" devuelve la foto del pato'''
    print('hello')
    image_url = get_duck_image_url()
    await ctx.send(image_url)


engine = pyttsx3.init()
engine.setProperty('rate', 125)
engine.setProperty('volume', 2.0)
voices = engine.getProperty('voices')
engine.setProperty('voice', voices[0].id)
# Puedes cambiar a voices[1].id si quieres otra voz


def speak(text):
    engine.say(text)
    engine.runAndWait()


# Clima
def get_weather(city: str) -> str:
    format = "|Condicion : %C |Temperatura : %t |Sensación termica : %f |Humedad : %h |Viento : %w |"
    base_url = f"https://wttr.in/{city}?format={format}"

    response = requests.get(base_url)

    if response.status_code == 200:
        return response.text.strip()
    else:
        return "No se pudo obtener la información del clima. Por favor, inténtalo más tarde."
        
@bot.command()
async def weather(ctx, *, city: str):
    weather_info = get_weather(city)
    await ctx.send(f"Clima en {city}: {weather_info}")
    speak(weather_info)


# Datos random
def get_fact() -> str:
    url = "https://uselessfacts.jsph.pl/random.json"
    response = requests.get(url)
    if response.status_code == 200:
        data = response.json()
        return data.get("text")
    else:
        return "No se pudo obtener una respuesta."


@bot.command()
async def fact(ctx):
    random_fact = get_fact()
    await ctx.send(f"Aqui tienes un dato random: {random_fact}")
    speak(random_fact)

# Traductor 1:
DEEPL_API_KEY = "a095f285-5f80-4e4b-a1cb-ed41938284c1:fx"
translator = deepl.Translator(DEEPL_API_KEY)


@bot.command()
async def traducir(ctx, idioma_destino: str, *, texto: str):
    """Traduce un texto al idioma especificado"""
    try:
        resultado = translator.translate_text(texto, target_lang=idioma_destino.upper())
        await ctx.send(f"*Traducción ({idioma_destino}):* {resultado.text}")
    except Exception as e:
        await ctx.send(f"⚠️ Error al traducir: {e}")


# Bot tirar dado 
@bot.command(name='dado')
async def dado(ctx):
    await ctx.send(f"🎲 Has sacado un {random.randint(1, 6)}")


# Adivinar numero entre 1 y 10
@bot.command(name='adivina')
async def adivina(ctx, numero: int):
    secreto = random.randint(0, 10)
    if numero == secreto:
        await ctx.send("🎉 ¡Correcto!")
    else:
        await ctx.send(f"❌ Nope. El número era {secreto}")


# Recordatorio
@bot.command(name='recordar')
async def recordar(ctx, segundos: int, *, mensaje: str):
    await ctx.send(f"⏱️ Te recordaré esto en {segundos} segundos...")
    await asyncio.sleep(segundos)
    await ctx.send(f"🔔 Recordatorio: {mensaje}")


# Fecha y hora actual
@bot.command(name='hora')
async def hora(ctx):
    ahora = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    await ctx.send(f"🕒 Fecha y hora actual: {ahora}")


# Diccionario
@bot.command(name='definir')
async def definir(ctx, palabra: str):
    url = f"https://api.dictionaryapi.dev/api/v2/entries/en/{palabra}"
    response = requests.get(url)

    if response.status_code == 200:
        data = response.json()[0]
        definiciones = data['meanings'][0]['definitions'][0]
        definicion = definiciones.get('definition', 'No encontrada')
        ejemplo = definiciones.get('example', 'Sin ejemplo disponible')
        await ctx.send(f"📘 Definición de **{palabra}**:\n**Significado:** {definicion}\n**Ejemplo:** {ejemplo}")
    else:
        await ctx.send("❌ No encontré esa palabra.")


# Calculadora
@bot.command(name='calcula')
async def calc(ctx, *, expresion: str):
    try:
        resultado = sympify(expresion)
        await ctx.send(f"🧮 Resultado: {resultado}")
    except Exception as e:
        await ctx.send("❌ Expresión inválida.")


# Zonas horarias
intents = discord.Intents.default()
bot = commands.Bot(command_prefix='!', intents=intents)


@bot.command(name='zonahoraria')
async def convertir_zona_horaria(ctx, fecha: str, hora: str, zona_origen: str, zona_destino: str):
    try:
        fecha_hora_str = f"{fecha} {hora}"
        formato = "%Y-%m-%d %H:%M"
        fecha_hora = datetime.strptime(fecha_hora_str, formato)
        tz_origen = pytz.timezone(zona_origen)
        tz_destino = pytz.timezone(zona_destino)
        fecha_hora_origen = tz_origen.localize(fecha_hora)
        fecha_hora_destino = fecha_hora_origen.astimezone(tz_destino)
        await ctx.send(
            f"🕒 **Hora convertida:**\n"
            f"`{fecha_hora_origen.strftime('%Y-%m-%d %H:%M')} ({zona_origen})` ➡️ "
            f"`{fecha_hora_destino.strftime('%Y-%m-%d %H:%M')} ({zona_destino})`"
        )
    except Exception:
        await ctx.send(
            "❌ Formato inválido.\n"
            "Usa: `!zonahoraria YYYY-MM-DD HH:MM zona_origen zona_destino`\n"
            "Ejemplo: `!zonahoraria 2025-07-02 14:00 America/Mexico_City Europe/Madrid`"
        )

bot.run(token)
