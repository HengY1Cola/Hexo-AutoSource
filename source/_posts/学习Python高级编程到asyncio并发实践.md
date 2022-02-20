---
title: 学习Python高级编程到asyncio并发实践
categories: Python
keywords: asyncio并发
top_img: /img/2610748.webp
cover: /img/1102008.webp
abbrlink: 4ba9700a
date: 2022-02-04 19:27:37
---

##  前言：

今天也顺利把**Python3高级核心技术97讲**看完了

课程链接我也贴一下：https://coding.imooc.com/class/chapter/200.html

总体说下感受：**很推荐吧，循序渐进学习，拓展了很多自己的不足**

正好我现在我看《Pyhton3网络爬虫开发实战(第二版)》也200页出头了

刚好来到了**aiohttp的使用**,这不正好？ 把崔佬的代码拿下来看看

那么接下来我简单讲解下自己的理解。图省事自己到最后的完成代码

> 注明：大部分我都写上了注释，是一个很好的实践代码
>
> 如果哪里我没理解对，记得来拍醒我下，请赐教，欢迎与我交流

##  图解：

> 首先得理解什么是多线程，什么是协程？ 以及相关的术语
>
> 简单来说：多线程就是多个线程同时并行；协程就是单个线程开启多个分支
>
> 协程将CPU更大利用化，简单比方：你去请求网络 那么CPU就在这里瞎等

![Untitled-2022-02-04-1909](https://img-blog.csdnimg.cn/img_convert/b482423210574f27c212e38af2fb4de3.png)

##  注意点

{% note orange 'fas fa-battery-half' flat %}
为什么将session设置在__init__中
{% endnote %}
免得一断一连。

{% note orange 'fas fa-battery-half' flat %}
gather与wait区别
{% endnote %}
简单理解gather的功能更高级，可以拿到结果。想了解更多直接谷歌QAQ

{% note orange 'fas fa-battery-half' flat %}
loop 的作用
{% endnote %}
可以看作这个程序的心跳，不停的循环。推荐看看我上面推荐的课程
从12章到13章，从最开始的源码程序实现到后面封装调用库
循序渐进讲的不错，还可以学学yield怎么回事

{% note orange 'fas fa-battery-half' flat %}
使用asyncio.sleep而不是用time.sleep 使用motor异步库而不是用平时的
{% endnote %}
使用time.sleep不是不行，会报错但可以拿到，具体的原因建议看看第12，13章

## 完整代码

> 原本的代码在 https://github.com/Python3WebSpider/ScrapeSpa5
>
> 我简单改了改小部分，学习的目的达到即可～

```python
# 使用aiohttp异步爬去
# 目标网站：https://spa5.scrape.center/

import asyncio
import aiohttp
import logging
from aiohttp import ContentTypeError
from motor.motor_asyncio import AsyncIOMotorClient

# 设置通用参数
PAGE_SIZE = 18  # 请求个数
PAGE_NUMBER = 1  # 请求页数
CONCURRENCY = 5  # 并发数量
# 设置日志格式
logging.basicConfig(filename='log.log', level=logging.INFO, format='%(asctime)s - %(levelname)s: %(message)s')
# 配置mongodb文件
MONGO_CONNECTION_STRING = 'mongodb://localhost:27017'
MONGO_DB_NAME = 'books'
MONGO_COLLECTION_NAME = 'books'
# 使用motor异步库
client = AsyncIOMotorClient(MONGO_CONNECTION_STRING)
db = client[MONGO_DB_NAME]
collection = db[MONGO_CONNECTION_STRING]


class Spider(object):
    def __init__(self):
        self.semaphore = asyncio.Semaphore(CONCURRENCY)  # 设置并发数量
        self.session = aiohttp.ClientSession()  # 设置session() 免得
        self.ids = []

    async def scrapeApi(self, url):
        async with self.semaphore:  # 限制并发
            try:
                logging.info('scraping %s', url)
                async with self.session.get(url) as response:  # 使用 async with 上下文会自动关闭
                    await asyncio.sleep(1)  # 必须使用asyncio.sleep而不是time.sleep
                    return await response.json()  # 为什么要使用await在pycharm中点进入发现是async申明的
            except ContentTypeError as e:
                logging.error('error occurred while scraping %s', url, exc_info=True)

    async def scrapeIndex(self, page):
        # todo 拼凑出来对应的url 然后教给下一个协程
        url = 'https://spa5.scrape.center/api/book/?limit=18&offset={offset}'
        url = url.format(offset=PAGE_SIZE * (page - 1))
        return await self.scrapeApi(url)

    async def scrapeDetail(self, id):
        url = 'https://spa5.scrape.center/api/book/{id}'
        url = url.format(id=id)
        data = await self.scrapeApi(url)  # 拿到结果就存入
        print(f"id: {id} : data: {data}")  # 具体内容自己操作写就好了(注意不是网络IO了不用await)
        # await self.saveData(data)  # 我是懒得去开个mongoQAQ

    @staticmethod
    async def saveData(data):
        logging.info('saving data %s', data)
        if data:
            return await collection.update_one({
                'id': data.get('id')
            }, {
                '$set': data
            }, upsert=True)

    async def main(self):
        # todo 设置任务
        # 从这一步的话可以拿到指定页数的列表
        scrape_index_tasks = [asyncio.ensure_future(self.scrapeIndex(page)) for page in range(1, PAGE_NUMBER + 1)]
        results = await asyncio.gather(*scrape_index_tasks)  # 使用gather而不是wait
        # todo 拿到每一页中的ids
        # 这一步就是从预览大概到每一页中获取详细信息
        for index_data in results:
            if not index_data:
                continue
            for item in index_data.get('results'):  # 使用字典的get方法
                self.ids.append(item.get('id'))
        # todo 根据拿到的id进入协程同步进行拿到想要的结果并存入
        scrape_detail_tasks = [asyncio.ensure_future(self.scrapeDetail(each)) for each in self.ids]
        await asyncio.wait(scrape_detail_tasks)
        # todo 最后关闭session会话
        await self.session.close()


if __name__ == '__main__':
    spider = Spider()
    loop = asyncio.get_event_loop()
    loop.run_until_complete(spider.main())
```
