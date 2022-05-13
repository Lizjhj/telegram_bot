
import discord
import credits
import os
import random
from discord.ext import commands
from PIL import Image
import numpy as np
token = credits.token
 
from discord.ext import commands

client = discord.Client()
bot = commands.Bot(command_prefix='!')
screen_dictionary = {}
photos = []
member_aktive = False


def white_black(file):
    img = Image.open(file)
    arr = np.asarray(img, dtype='uint8')
    k = np.array([0.2989, 0.587, 0.114])
    w, h, _ = arr.shape
    arr.reshape(w*h,3)
    out=np.matmul(arr, k)
    out.reshape(w,h)
    img2 = Image.fromarray(out.astype(np.uint8))
    img2.save('result_white.png')

def twist_image(input_file_name):
    from PIL import Image, ImageChops
    im = Image.open(input_file_name)
    x, _ = im.size
    im=ImageChops.offset(im, x//2, 0)
    im.save("twist2.jpg")
    
def makeanagliph(ﬁlename, sdvig):
    from PIL import Image
    image = Image.open(filename)
    width, height = image.size
    empty_image = Image.new('RGB', (width, height), (0, 0, 0))
    pixels2 = empty_image.load()
    pixels = image.load()
    for i in range(width):
        if i < sdvig:
            for j in range(height):
                r, g, b = pixels[i, j]
                pixels2[i, j] = 0, g, b
        else:
            for j in range(height):
                g, b = pixels[i, j][1:]
                r = pixels[i - sdvig, j][0]
                pixels2[i, j] = r, g, b
    empty_image.save("stereo2.jpg")


@bot.command(name="start")
async def start(ctx):
    global member_aktive 
    if member_aktive:
        await ctx.send("Ты уже со мной здоровался - я тебя знаю " + ctx.author.name)
    else:
        await ctx.send("Привет, " + ctx.author.name + "! Добро пожаловать ко мне!")
        await ctx.send("Я бот, который умеет работать с фотографиями(ничего полезного), вот, что я могу сделать:")
        await ctx.send(f"Отправь мне '!whiteblack', и я отпрвлю черно-белое изображение")
        await ctx.send(f"Отправь мне '!leftright', и я отправлю изображение, где будет помененнно местами левая и правая половины изображения.")
        await ctx.send(f"Отправь мне '!transparency', и я совмещу два изображения,  где одно станет полупрозрачным")
        await ctx.send(f"Отправь мне '!makeanagliph', и получишь изображение со стереопатическим эффектом ")
        member_aktive = True
        
@bot.command(name="makeanagliph")  # Отправляем через "Добавить комментарий"
async def makeanagliph(ctx, photo):
    makeanagliph(photo, 10)
    await ctx.send(file= discord.File("stereo2.jpg"))

@bot.command(name="leftright")  # Отправляем через "Добавить комментарий"
async def leftright(ctx, photo1):
    twist_image(photo1)
    await ctx.send(file= discord.File("twist2.jpg"))
    
@bot.command(name="whiteblack")  # Отправляем через "Добавить комментарий"
async def whiteblack(ctx, photo2):
    white_black(photo2)
    await ctx.send(file= discord.File("result_white.png"))

@bot.command(name="transparency")  # Отправляем через "Добавить комментарий"
async def transparency(ctx, photo3, photo4):
    def trans(filename1, filename2):
        image_1 = Image.open(filename1)
        image_2 = Image.open(filename2)
        picture_1 = image_1.load()
        picture_2 = image_2.load()
        width, height = image_1.size
        for i in range(width):
            for j in range(height):
                R1, G1, B1 = picture_1[i, j]
                R2, G2, B2 = picture_2[i, j]
                new_pixels = (int(0.5 * R1 + 0.5 * R2),
                              int(0.5 * G1 + 0.5 * G2),
                              int(0.5 * B1 + 0.5 * B2))
                picture_1[i, j] = new_pixels
        image_1.save("trans.png")

    trans(photo3, photo4)
    print(photo3)
    print(photo4)
    await ctx.send(file= discord.File("trans.png"))
    
bot.run(token)
