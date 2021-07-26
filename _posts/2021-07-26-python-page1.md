---

layout: post
title: 'python,page1'
date: 2021-07-26
author: jerry
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll.theme-h2o-postcover.jpg'
tags: python

---
>逛52看到的一个超赞的 python 爬虫项目
### 下面放代码

code

```
 ```import asyncio
import random
import time 
import aiohttp
import aiofiles
import requests
from lxml import etree
import os
import re
from fake_useragent import UserAgent
from functools import wraps
from asyncio.proactor_events import _ProactorBasePipeTransport
 
 
def silence_event_loop_closed(func):
    @wraps(func)
    def wrapper(self, *args, **kwargs):
        try:
            return func(self, *args, **kwargs)
        except RuntimeError as e:
            if str(e) != 'Event loop is closed':
                raise
 
    return wrapper
 
 
_ProactorBasePipeTransport.__del__ = silence_event_loop_closed(_ProactorBasePipeTransport.__del__)
ua = UserAgent()
headers = {'User-Agent': ua.random, 'Referer': 'http://www.tulishe.com'}
 
 
class tulishe:
    def __init__(self):
        self.write_num = 0
 
    async def get_url(self, url):
        async with aiohttp.ClientSession() as client:
            async with client.get(url, headers=headers) as resp:
                if resp.status == 200:
                    return await resp.text()
 
    async def html_parse(self, html):
        semaphore = asyncio.Semaphore(5)
        html_parse = etree.HTML(html)
        url_list = html_parse.xpath('//div[@class="img"]//a[@rel="bookmark"]/@href')
        tasks = [asyncio.create_task(self.img_parse(url, semaphore)) for url in url_list]
        await asyncio.wait(tasks)
 
    async def img_parse(self, h_url, sem):
        async with sem:
            semaphore = asyncio.Semaphore(5)
            h_html = await self.get_url(h_url)
            h_html_parse = etree.HTML(h_html)
            title = h_html_parse.xpath('//h1[@class="article-title"]/text()')[0]
            img_demo_url = h_html_parse.xpath(
                '//*[@id="gallery-2"]/div[@class="gallery-item gallery-blur-item"]/img/@src')
            img_url_list = []
            for d_url in img_demo_url:
                img_url = d_url.split('=')[1].split('&')[0]
                img_url_list.append(img_url)
            i_u_l = h_html_parse.xpath(
                '//div[@class="gallery-item gallery-fancy-item"]/a/@href')
            full_list = i_u_l + img_url_list
            index_list = list(range(1, len(full_list) + 1))
            index_dict = dict(zip(full_list, index_list))
            tasks = [asyncio.create_task(self.img_con(i_url, i_num, title, semaphore)) for i_url, i_num in
                     index_dict.items()]
            await asyncio.wait(tasks)
 
    async def img_con(self, url, num, title, semaphore):
        async with semaphore:
            async with aiohttp.ClientSession() as client:
                async with client.get(url, headers=headers) as resp:
                    if resp.status == 200:
                        img_con = await resp.read()
                        await self.write_img(img_con, num, title)
                    else:
                        print('请求出错，请尝试调低并发数重新下载！！')
 
    async def write_img(self, img_con, num, title):
        if not os.path.exists(title):
            os.makedirs(title)
            async with aiofiles.open(title + '/' + f'{num}.jpg', 'wb') as f:
                print(f'正在下载{title}/{num}.jpg')
                await f.write(img_con)
                self.write_num += 1
        else:
            async with aiofiles.open(title + '/' + f'{num}.jpg', 'wb') as f:
                print(f'正在下载{title}/{num}.jpg')
                await f.write(img_con)
                self.write_num += 1
 
    async def main(self, ):
        q_start_num = input('输入要从第几页开始下载（按Entry默认为1）:') or '1'
        start_num = int(q_start_num)
        total_num = int(input('请输入要下载的页数：')) + start_num
        print('*' * 74)
        start_time = time.time()
        for num in range(start_num, total_num + 1):
            url = f'http://www.tulishe.com/all/page/{num}'
            html = await self.get_url(url)
            print('开始解析下载>>>')
            await self.html_parse(html)
        end_time = time.time()
        print(f'本次共下载写真图片{self.write_num}张，共耗时{end_time - start_time}秒。')
 
 
a = tulishe()
asyncio.run(a.main())
 ```
```
