# webscrapemint
Telegram Bot For Automatically Posting News with Thumbnail Heading and Body

import telegram.ext
from telegram import Bot
from bs4 import BeautifulSoup as soup
from urllib.request import Request, urlopen
import requests
from telegram.ext.updater import Updater
from telegram.update import Update
from telegram.ext.callbackcontext import CallbackContext
from telegram.ext.commandhandler import CommandHandler
from telegram.ext.messagehandler import MessageHandler
from telegram.ext.filters import Filters
import time,threading


url='http://www.livemint.com/rss/markets'
bot = Bot('5948865657:AAEk9z2cCIWCjKSy2SDMFsHAqZdZ5yxt3Mo')


sentheadline=[]
sentbody=[]
sentiamge=[]
updater = Updater("5948865657:AAEk9z2cCIWCjKSy2SDMFsHAqZdZ5yxt3Mo",use_context=True)

def fetch():
    headline = []
    body = []
    image = []
    html = requests.get(url)
    bsobj = soup(html.content, 'xml')
    # print(bsobj)
    j = 0
    a = bsobj.findAll('item')
    while j<5:
        for i in a:
            headline.append(i.find('title').text)
            #links.append(i.find('link').text)
            image.append(i.find('media:content')['url'])
            body.append(i.find('description').text)
            j+=1
    return headline,body,image

def autopost():
    print("running")
    while True:
        x=fetch()
        headline = x[0]
        body = x[1]
        image = x[2]
        k = 0
        while k < 5:
                if headline[k] not in sentheadline:
                    bot.send_photo(-1001877294509, image[k], headline[k])
                    bot.send_message(-1001877294509, body[k])
                    sentiamge.append(image[k])
                    sentheadline.append(headline[k])
                    sentbody.append(body[k])
                    print(sentheadline)
                    k += 1
                else:
                    k+=1
                    print("no new news")
        print('Sleeping')
        time.sleep(60*60)

def cleaner():
    while True:
        sentheadline.clear()
        time.sleep(60*60*24)

threading.Thread(target=autopost).start()
threading.Thread(target=cleaner).start()
updater.start_polling()
