from selenium import webdriver
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options
options = Options()
options.add_argument('--headless')
options.add_argument('--disable-gpu')
options.add_argument('--disable-dev-shm-usage')
options.add_argument("--no-sandbox")
# DRIVER_PATH = './chromedriver'
DRIVER_PATH = '/usr/lib/chromium-browser/chromedriver'
driver = webdriver.Chrome(executable_path=DRIVER_PATH, options=options)

import logging

from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackQueryHandler
from telegram import InlineKeyboardButton,InlineKeyboardMarkup,KeyboardButton,ReplyKeyboardMarkup

import sys
import urllib
import re
import requests
import datetime
from datetime import datetime
from tqdm import tqdm

import time

# read config.ini for token
import os
try:
    from configparser import ConfigParser
except ImportError:
    from ConfigParser import ConfigParser  # ver. < 3.0



# Enable logging
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
                    level=logging.INFO)

logger = logging.getLogger(__name__)


# Define a few command handlers. These usually take the two arguments update and
# context. Error handlers also receive the raised TelegramError object in error.
def start(update, context):
    """Send a message when the command /start is issued."""
    user = update.message.from_user
    print(user)
    send = f"Hi {user.username}, started your bot. \n First name {user.first_name} \n ID:{user.id}"
    context.bot.send_message(chat_id=update.message.chat_id, text=send)
    update.message.reply_text('Hi!')


def help(update, context):
    """Send a message when the command /help is issued."""
    update.message.reply_text('Help!')


# def echo(update, context):
#     """Echo the user message."""
#     update.message.reply_text(update.message.text)

def profile(update, context):
    """Send profile"""
    chat_id = update.message.chat_id
    url = update.message.text.split('?')[0]

    #PROFILE PICTURE
    profile_pic = re.match(r'^(https:)[\/][\/](www\.)?instagram.com[\/][a-zA-Z0-9._]*(\/)?$', url)

    if profile_pic:
        try:
            get_content = driver.get(url)
            print("\nDownloading the profile image...")
            src = driver.page_source
            find_pp = re.search(r'profile_pic_url_hd\":\"([^\'\" >]+)', src)
            pp_link = find_pp.group()
            pp_final = re.sub('profile_pic_url_hd":"', '', pp_link)
            file_size_request = requests.get(pp_final.replace('\\u0026','&'), stream=True)
            file_size = int(file_size_request.headers['Content-Length'])
            block_size = 1024
            filename = datetime.strftime(datetime.now(), '%Y-%m-%d-%H-%M-%S')
            t = tqdm(total=file_size, unit='B',
                        unit_scale=True, desc=filename, ascii=True)
            with open('./images/' + filename + '.jpg', 'wb') as f:
                for data in file_size_request.iter_content(block_size):
                    t.update(len(data))
                    f.write(data)
            t.close()
            print("Profile picture downloaded successfully")
            context.bot.send_photo(chat_id, photo=open('./images/'+filename+'.jpg','rb'))
        except Exception as e:
            print(e)
        
        return
    
    #PHOTOS OR VIDEOS
    normal_pic_or_video = re.match(r'^(https:)[\/][\/](www\.)?instagram.com[\/]p[\/][a-zA-Z0-9._]*[\/]?$', url)

    if normal_pic_or_video:
        request_image = driver.get(url)
        # time.sleep(3)
        try: # try to accept cookie
            driver.find_element_by_css_selector('.bIiDR').click()
            n = 0
        except:
            n = 0
        while True: # to emulate do while with fail_condition
            try:
                src = driver.page_source
                check_type = re.search(r'<meta name="medium" content=[\'"]?([^\'" >]+)', src)
                check_type_f = check_type.group()
                final = re.sub('<meta name="medium" content="', '', check_type_f)
                
                if final == "image":
                    print("\nDownloading the image...")
                    # extract_image_link = re.search(r'meta property="og:image" content=[\'"]?([^\'" >]+)', src)
                    # print(extract_image_link)
                    time.sleep(1)
                    try: # try to find image in carousel, if not
                        if (int(n) == int(0)):
                            image_link = driver.find_element_by_css_selector('article .vi798 li:nth-child(2) .KL4Bh img').get_attribute('src')
                        else:
                            image_link = driver.find_element_by_css_selector('article .vi798 li:nth-child(3) .KL4Bh img').get_attribute('src')
                            # final = re.sub('meta property="og:image" content="', '', image_link).replace('\\u0026','&').replace('&amp;','&')
                    except:
                        image_link = driver.find_element_by_css_selector('article .KL4Bh img').get_attribute('src')
                    final = image_link
                    _response = requests.get(final).content
                    file_size_request = requests.get(final, stream=True)
                    file_size = int(file_size_request.headers['Content-Length'])
                    block_size = 1024 
                    filename = datetime.strftime(datetime.now(), '%Y-%m-%d-%H-%M-%S')
                    t=tqdm(total=file_size, unit='B', unit_scale=True, desc=filename, ascii=True)
                    with open('./images/' +filename + '.jpg', 'wb') as f:
                        for data in file_size_request.iter_content(block_size):
                            t.update(len(data))
                            f.write(data)
                    t.close()
                    context.bot.send_photo(chat_id, photo=open('./images/'+filename+'.jpg','rb'))
                    print("Image downloaded successfully")

                if final == "video": 
                    print("Downloading the video...")
                    extract_video_link = re.search(r'meta property="og:video" content=[\'"]?([^\'" >]+)', src)
                    video_link = extract_video_link.group()
                    final = re.sub('meta property="og:video" content="', '', video_link).replace('\\u0026','&').replace('&amp;','&')
                    _response = requests.get(final).content
                    file_size_request = requests.get(final, stream=True)
                    file_size = int(file_size_request.headers['Content-Length'])
                    block_size = 1024 
                    filename = datetime.strftime(datetime.now(), '%Y-%m-%d-%H-%M-%S')
                    t=tqdm(total=file_size, unit='B', unit_scale=True, desc=filename, ascii=True)
                    with open('./videos/' +filename + '.mp4', 'wb') as f:
                        for data in file_size_request.iter_content(block_size):
                            t.update(len(data))
                            f.write(data)
                    t.close()
                    context.bot.send_video(chat_id=chat_id, video=open('./videos/' + filename + '.mp4', 'rb'))
                    print("Video downloaded successfully")
                
                try:
                    driver.find_element_by_css_selector('._6CZji').click()
                    n += 1
                except:
                    print('uscito')
                    break

            except AttributeError:
                print("Unknown URL")

#     update.send_photo(chat_id, photo=open('path', 'rb'))

    #STORIES
    # stories = re.match(r'^(https:)[\/][\/](www\.)?instagram.com[\/]stories[\/][a-zA-Z0-9._]*[\/][a-zA-Z0-9._]*[\/]?$', url)

    # if stories:
    #     request_image = driver.get(url)
    #     # time.sleep(3)
    #     try: # try to accept cookie
    #         driver.find_element_by_css_selector('.bIiDR').click()
    #         n = 0
    #     except:
    #         n = 0
    #     while True: # to emulate do while with fail_condition
    #         try:





def error(update, context):
    """Log Errors caused by Updates."""
    logger.warning('Update "%s" caused error "%s"', update, context.error)


def main():
    """Start the bot."""

    # instantiate
    config = ConfigParser()
    # parse existing file
    config.read('config.ini')
    tokenBot = config.get('Telegram', 'tokenBot')
    tokenBot = config.read(os.path.join(os.path.dirname(__file__), 'Telegram', 'tokenBot'))

    # Create the Und pass it your bot's token.
    # Make sure to set use_context=True to use the new context based callbacks
    # Post version 12 this will no longer be necessary
    updater = Updater(tokenBot, use_context=True)

    # Get the dispatcher to register handlers
    dp = updater.dispatcher

    # on different commands - answer in Telegram
    dp.add_handler(CommandHandler("start", start))
    dp.add_handler(CommandHandler("help", help))

    # dp.add_handler(CommandHandler("profile", profile))

    # on noncommand i.e message - echo the message on Telegram
    dp.add_handler(MessageHandler(Filters.text, profile))

    # log all errors
    dp.add_error_handler(error)

    # Start the Bot
    updater.start_polling()

    # Run the bot until you press Ctrl-C or the process receives SIGINT,
    # SIGTERM or SIGABRT. This should be used most of the time, since
    # start_polling() is non-blocking and will stop the bot gracefully.
    updater.idle()


if __name__ == '__main__':
    main()


