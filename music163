#!/usr/bin/python
# -*- coding:utf-8 -*-
# author:joel 18-6-26
import base64
import pymongo
import random
import re
import sys
import time
import requests
from Crypto.Cipher import AES

'''
这里有一个很有意思的地方，‘#’，在获取网页源码时需要将‘#’除去才可
'https://music.163.com/#/discover/playlist/?order=new'
'https://music.163.com/discover/playlist/?order=new'
                                                                                            0
https://music.163.com/#/discover/playlist/?order=new&cat=%E5%85%A8%E9%83%A8&limit=35&offset=35
                                                                                            70                                                                  
'https://music.163.com/weapi/v1/resource/comments/R_SO_4_{}?csrf_token='
'''

headers = {
    'User-Agent':'Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.87 Safari/537.36'
}

baseUrl = 'https://music.163.com'
# offset的取值为:(评论页数-1)*20,total第一页为true，其余页为false
# first_param = '{rid:"", offset:"0", total:"true", limit:"20", csrf_token:""}' # 第一个参数
# 第二个参数
second_param = "010001"
# 第三个参数
third_param = "00e0b509f6259df8642dbc35662901477df22677ec152b5ff68ace615bb7b725152b3ab17a876aea8a5aa76d2e417629ec4ee341f56135fccf695280104e0312ecbda92557c93870114af6c9d05c4f7f0c3685b7a46bee255932575cce10b424d813cfe4875d3e82047b97ddef52741d546b8e289dc6935b3ece0462db0a22b8e7"
# 第四个参数
forth_param = "0CoJUm6Qyw8W8jud"


def getHtml(url):
    r = requests.get(url, headers=headers)
    html = r.text
    return html

def getUrl():
    #从最新歌单开始
    startUrl = 'https://music.163.com/discover/playlist/?order=new'
    html = getHtml(startUrl)
    pattern =re.compile('<li>.*?<p.*?class="dec">.*?<.*?title="(.*?)".*?href="(.*?)".*?>.*?span class="s-fc4".*?title="(.*?)".*?href="(.*?)".*?</li>',re.S)
    result = re.findall(pattern,html)
    #获取歌单总页数
    pageNum = re.findall(r'<span class="zdot".*?class="zpgi">(.*?)</a>',html,re.S)[0]
    info = []
    #对第一页的歌单获取想要的信息
    for i in result:
        data = {}
        data['title'] = i[0]
        url = baseUrl+i[1]
        print url
        data['url'] = url
        data['author'] = i[2]
        data['authorUrl'] = baseUrl+i[3]
        info.append(data)
        #调用获取每个歌单里的歌曲的方法
        getSongSheet(url)
        time.sleep(random.randint(1,10))
        #这里暂时获取第一页的第一个歌单，所以用break
        break

# 获取参数
def get_params(page): # page为传入页数
    iv = "0102030405060708"
    first_key = forth_param
    second_key = 16 * 'F'
    if(page == 1): # 如果为第一页
        first_param = '{rid:"", offset:"0", total:"true", limit:"20", csrf_token:""}'
        h_encText = AES_encrypt(first_param, first_key, iv)
    else:
        offset = str((page-1)*20)
        first_param = '{rid:"", offset:"%s", total:"%s", limit:"20", csrf_token:""}' %(offset,'false')
        h_encText = AES_encrypt(first_param, first_key, iv)
    h_encText = AES_encrypt(h_encText, second_key, iv)
    return h_encText

# 获取 encSecKey
def get_encSecKey():
    encSecKey = "257348aecb5e556c066de214e531faadd1c55d814f9be95fd06d6bff9f4c7a41f831f6394d5a3fd2e3881736d94a02ca919d952872e7d0a50ebfa1769a7a62d512f5f1ca21aec60bc3819a9c3ffca5eca9a0dba6d6f7249b06f5965ecfff3695b54e1c28f3f624750ed39e7de08fc8493242e26dbc4484a01c76f739e135637c"
    return encSecKey

# 加密过程
def AES_encrypt(text, key, iv):
    pad = 16 - len(text) % 16
    text = text + pad * chr(pad)
    encryptor = AES.new(key, AES.MODE_CBC, iv)
    encrypt_text = encryptor.encrypt(text)
    encrypt_text = base64.b64encode(encrypt_text)
    return encrypt_text

#获取post得到的Json
def getPostApi(j ,postUrl, headers1):
    param = {
        # 获取对应页数的params
        'params': get_params(j),
        'encSecKey': get_encSecKey()
    }
    r = requests.post(postUrl, data=param, headers=headers1)
    return r

#获取歌曲评论
def getMusicComments(comment_TatalPage ,postUrl, headers1):
    commentinfo = []
    hotcommentinfo = []
    # 对每一页评论
    for j in range(1, comment_TatalPage + 1):
        # 热评只在第一页可抓取
        if j == 1:
            r = getPostApi(j , postUrl, headers1)
            comment_info = r.json()['comments']
            for i in comment_info:
                com_info = {}
                com_info['content'] = i['content']
                com_info['author'] = i['user']['nickname']
                com_info['likedCount'] = i['likedCount']
                commentinfo.append(com_info)
            hotcomment_info = r.json()['hotComments']
            for i in hotcomment_info:
                hot_info = {}
                hot_info['content'] = i['content']
                hot_info['author'] = i['user']['nickname']
                hot_info['likedCount'] = i['likedCount']
                hotcommentinfo.append(hot_info)
        else:
            r = getPostApi(j, postUrl, headers1)
            comment_info = r.json()['comments']
            for i in comment_info:
                com_info = {}
                com_info['content'] = i['content']
                com_info['author'] = i['user']['nickname']
                com_info['likedCount'] = i['likedCount']
                commentinfo.append(com_info)
        print u'第'+str(j)+u'页爬取完毕...'
        time.sleep(random.randint(1,10))
    print commentinfo
    print '\n-----------------------------------------------------------\n'
    print hotcommentinfo
    return commentinfo,hotcommentinfo

def saveToMongoDB(musicName,comment_data,hotComment_data):
    client = pymongo.MongoClient(host='localhost',port=27017)
    db = client['Music163']
    test = db[musicName]
    test.insert(hotComment_data)
    test.insert(comment_data)
    print musicName+u'已存入数据库...'

def getSongSheet(url):
    #获取每个歌单里的每首歌的id，作为接下来post获取的关键
    html = getHtml(url)
    result = re.findall(r'<li><a.*?href="/song\?id=(.*?)">(.*?)</a></li>',html,re.S)
    result.pop()
    musicList = []
    for i in result:
        data = {}
        headers1 = {
            'Referer': 'https://music.163.com/song?id={}'.format(i[0]),
            'User-Agent': 'Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.87 Safari/537.36'
        }
        musicUrl = baseUrl+'/song?id='+i[0]
        print musicUrl
        #歌曲url
        data['musicUrl'] = musicUrl
        #歌曲名
        data['title'] = i[1]
        musicList.append(data)
        postUrl = 'https://music.163.com/weapi/v1/resource/comments/R_SO_4_{}?csrf_token='.format(i[0])
        param = {
            'params': get_params(1),
            'encSecKey': get_encSecKey()
        }
        r = requests.post(postUrl,data = param,headers = headers1)
        total = r.json()
        # 总评论数
        total = int(total['total'])
        comment_TatalPage = total/20
        # 基础总页数
        print comment_TatalPage
        #判断评论页数，有余数则为多一页，整除则正好
        if total%20 != 0:
            comment_TatalPage = comment_TatalPage+1
            comment_data,hotComment_data = getMusicComments(comment_TatalPage, postUrl, headers1)
            #存入数据库的时候若出现ID重复，那么注意爬下来的数据是否只有一个
            saveToMongoDB(str(i[1]),comment_data,hotComment_data)
            print 'End!'
        else:
            comment_data, hotComment_data = getMusicComments(comment_TatalPage, postUrl, headers1)
            saveToMongoDB(str(i[1]),comment_data,hotComment_data)
            print 'End!'

        time.sleep(random.randint(1, 10))
        break

if __name__ == '__main__':
    getUrl()
