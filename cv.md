<HTML>
  <P>Egor Barinov</P>
  <H1>My contacts to contact me: lackory@gmail.com, t.me/lackory</H1>
  <B>My goal is to learn JAVASCRIPT and use libraries, despite how difficult it will be, I will still continue my studies. My work experience is Cook, Baker for 5 years. I would like to try myself in something new, in a field that I have not encountered before.</B>
  <P>I have partial knowledge of HTML and Python. I am also developing my Linux distribution Unison Linux, doing everything through trial and error t.me/unisonlinux , vk.com/unisonlinux . It took me more than 2 weeks to develop, and I'm currently working on finalizing the project. There is also a fork of the bot based on the vk battle library, https://github.com/lackory/bot</P>
  <P>import os, random, datetime, time, config, PIL, mc, requests, json, re, sqlite3 as sql
from vkbottle.bot import Bot, Message
from vkbottle import VideoUploader, Keyboard, KeyboardButtonColor, Text, OpenLink, VoiceMessageUploader, PhotoMessageUploader
from vkbottle.dispatch.rules.base import AttachmentTypeRule, ChatActionRule, FromUserRule
from requests import get
from PIL import Image, ImageDraw, ImageFont
from mc.builtin import validators
from wand.image import Image
from wand.display import display
from wand.font import Font
from orpl import *
from local import *
from config import * 
from requests import get
from loguru import *

bot=Bot(config.token)

def read_ff(file): # Read From File
    try:
        Ff = open(file, 'r', encoding='UTF-8')
        Contents = Ff.read()
        Ff.close()
        return Contents
    except:
        return None

print(f'\n{splash} {bot_ver} {build_code}\n') 

logger.info('Initialization...')

start_time = time.time()

connection = sql.connect('database.db', check_same_thread=False)  # Database connection
q = connection.cursor()
q.execute('CREATE TABLE IF NOT EXISTS players (id INTEGER DEFAULT NULL, status INTEGER DEFAULT 0, kolvo INTEGER DEFAULT 0, picgen INTEGER DEFAULT 3, txtgen INTEGER DEFAULT 25, dst STRING DEFAULT 0, rnd INTEGER DEFAULT 3, learn INTEGER DEFAULT 0, arrange INTEGER DEFAULT 0, gen_msgs INTEGER DEFAULT 0)')

logger.info('Loading files and directories...')
if not os.path.exists('Images'):
    os.mkdir('Images')

if not os.path.exists('Dialogs'):
    os.mkdir('Dialogs')

if not os.path.exists('modules'):
    os.mkdir('modules')

if not os.path.exists(config.lists_dir):
    os.mkdir(config.lists_dir)
        
for i in range(len(config.lists_files)):
    if not os.path.exists(f'{config.lists_dir}{config.lists_files[i]}'):
        f = open(f'{config.lists_dir}{config.lists_files[i]}', 'w', encoding='utf8')
        f.write('')
        f.close()

modules = os.listdir(config.modules_dir)
for i in range(len(modules)):
    if '.py' in modules[i]:
        logger.info(f'Starting module "{modules[i]}"...')
        exec(read_ff(f'{config.modules_dir}{modules[i]}'))
        
logger.info('One moment...')
logger.add(path_to_log, format='{time:YYYY-MM-DD at HH:mm:ss} | {level} | {message}', backtrace=False, encoding='utf-8', retention='10 days')   
start_menu = Keyboard(inline=True)
start_menu.add(Text(commands_alibave_in_pm, {"cmd": "daemon_funcs"}))
start_menu.add(Text(privacy_policy_button_text, {"cmd": "daemon_policy"}))

contact_with_admin = Keyboard(inline=True)
contact_with_admin.add(OpenLink(f'https://vk.com/{admin_username}', contact_with_admin_text))

logger.success('Ready!')

@bot.on.chat_message(AttachmentTypeRule("photo"))
async def photoadd(message: Message):
        await addtobd(message.peer_id)
        photo = message.attachments[0].photo.sizes[-1].url
        if message.from_id > 0:
            f = open(dir_to_pic + str(message.peer_id) + '.txt', 'r', encoding='utf8')
            l = f.read()
            f.close
            f = open(dir_to_pic + str(message.peer_id) + '.txt', 'w', encoding='utf8')
            f.write(f'{l}{photo}\n')
            f.close()       
            
@bot.on.chat_message()
async def allmsg(message: Message):
    await addtobd(message.peer_id)
    if await check_bl_wl(message) != False:
        file_id = time.time() 
        if len(message.text) <= 60 and message.text != '' and message.from_id > 0 and message.text[:3] != '[id' and message.text[:1] != '/':
            f = open(f'{dir_to_txt}{message.peer_id}.txt', 'r', encoding='utf8')
            l = f.read()
            f.close()
            f = open(f'{dir_to_txt}{message.peer_id}.txt', 'w', encoding='utf8')
            f.write(f'{l}{message.text}\n')
            f.close()
            q.execute(f"SELECT * FROM players WHERE id = {message.peer_id}")
            result = q.fetchall()
            status = result[0][1]
            kolvo = result[0][2]
            picgen = result[0][3]
            txtgen = result[0][4]
            learn = result[0][7]
            q.execute(f"UPDATE players SET kolvo = '{kolvo + 1}' WHERE id = '{message.peer_id}'")
            connection.commit()
            pic2 = 0
            with open(f'{dir_to_pic}{message.peer_id}.txt', encoding='utf8') as f:
                for line in f:
                    pic2 = pic2 + 1
            lines = 0
            with open(f'{dir_to_txt}{message.peer_id}.txt', encoding='utf8') as f:
                for line in f:
                    lines = lines + 1       
            if lines >= txtgen and pic2 >= picgen and kolvo >=txtgen and learn != 1:
                if status == 0: 
                    with open(f'{dir_to_txt}{message.peer_id}.txt', encoding='utf8') as file:
                        content = file.read().splitlines()
                    generator = mc.PhraseGenerator(samples=content)
                    rndtxt = generator.generate_phrase(attempts=20, validators=[validators.words_count(minimal=1, maximal=5)])
                    rndtxt2 = generator.generate_phrase(attempts=20, validators=[validators.words_count(minimal=1, maximal=10)])
                    with open(f'{dir_to_pic}{message.peer_id}.txt', encoding='utf8') as file:
                        content2 = file.read().splitlines()
                    rndpic = random.choice(content2)
                    p = requests.get(rndpic)
                    out = open(fr'randomimg_{message.peer_id}_{file_id}.jpg', "wb")
                    out.write(p.content)
                    out.close()
                    q.execute(f"SELECT * FROM players WHERE id = {message.peer_id}")
                    result = q.fetchall()
                    kolvo = result[0][2]
                    mode = result[0][5]
                    arrange = result[0][8]
                    q.execute(f"UPDATE players SET kolvo = '0' WHERE id = '{message.peer_id}'")
                    connection.commit()
                    if mode == 1:
                        user_img = PIL.Image.open(f'randomimg_{message.peer_id}_{file_id}.jpg').convert("RGBA")
                        (width, height) = user_img.size
                        image = Image(filename=f'randomimg_{message.peer_id}_{file_id}.jpg')
                        with image.clone() as liquid:
                            liquid.liquid_rescale(int(width*0.5), int(height*0.5))
                            liquid.save(filename=f'randomimg_{message.peer_id}_{file_id}.jpg')
                            liquid.size
                        user_img = PIL.Image.open(f'randomimg_{message.peer_id}_{file_id}.jpg').resize((width, height))
                        user_img.save(f'randomimg_{message.peer_id}_{file_id}.jpg')
                    if arrange == 1:
                        dem = Demotivator(rndtxt, rndtxt2)
                        dem.create(f'randomimg_{message.peer_id}_{file_id}.jpg', watermark=watermarkwrong, result_filename=f'dem_{message.peer_id}_{file_id}.jpg')
                    else:
                        dem =  Demotivator(rndtxt, rndtxt2)
                        dem.create(f'randomimg_{message.peer_id}_{file_id}.jpg', watermark=watermarkwrong, result_filename=f'dem_{message.peer_id}_{file_id}.jpg')
                    photo = await PhotoMessageUploader(bot.api).upload(f'dem_{message.peer_id}_{file_id}.jpg')
                    await message.answer(attachment=photo)
                    await update_stats(message)
                    os.remove(f'randomimg_{message.peer_id}_{file_id}.jpg')
                    os.remove(f'dem_{message.peer_id}_{file_id}.jpg')       
                else:
                    with open(f'{dir_to_txt}{message.peer_id}.txt', encoding='utf8') as file:
                        content = file.read().splitlines()
                    q.execute(f"SELECT * FROM players WHERE id = {message.peer_id}")
                    result = q.fetchall()
                    kolvo = result[0][2]
                    q.execute(f"UPDATE players SET kolvo = '0' WHERE id = '{message.peer_id}'")
                    connection.commit()
                    generator = mc.PhraseGenerator(samples=content)
                    rndtxt = generator.generate_phrase(attempts=20, validators=[validators.words_count(minimal=1, maximal=5)])
                    rndtxt2 = generator.generate_phrase(attempts=20, validators=[validators.words_count(minimal=1, maximal=10)])
                    long2 = len(rndtxt2)
                    await update_stats(message)
                    if len(rndtxt2) <= 60:
                        await message.answer(f'{rndtxt} {rndtxt2}')
                    else:
                        rndtxt2 = rndtxt2[:60]
                        await message.answer(f'{rndtxt} {rndtxt2}')

@bot.on.private_message(payload={"cmd": "daemon_funcs"})
async def func_info_msg(message: Message):
    if await check_bl_wl(message) != False:
        await message.reply(func_info_pm)

@bot.on.private_message(payload={"cmd": "daemon_policy"})
async def privacy_policy(message: Message):
    if await check_bl_wl(message) != False:
        await message.reply(privacy_policy_msg)     

@bot.on.private_message()
async def get_started(message: Message):
    if await check_bl_wl(message) != False:
        await message.reply(hello_msg, keyboard=start_menu)

if __name__ == '__main__':    
    bot.run_forever()</P>
    <P>I didn't have any training projects.</P>
    <P>Education Cook, Pastry Chef.</P>
    <P>I don't speak English very well, but I can partially understand the meaning of the English text.</P>
</HTML>
