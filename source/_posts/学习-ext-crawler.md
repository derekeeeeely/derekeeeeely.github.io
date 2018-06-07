---
title: web crawler share
abbrlink: 835c8e5b
date: 2018-06-07 14:25:23
tags: [crawler]
categories: code
---

&emsp;&emsp;简单的爬虫分享，未完待续...
<!-- more -->

### 爬虫+pandora分享

#### spider

- 搜索引擎使用网络爬虫爬取网站信息，用于建立索引供用户搜索
  [google](https://www.google.com/search/howsearchworks/crawling-indexing/)
- robots.txt  [google](https://www.google.com/webmasters/tools/robots-testing-tool?hl=zh-CN&siteUrl=http://www.derekz.cn/)
  ![5b13aa65a3d27.png](https://i.loli.net/2018/06/03/5b13aa65a3d27.png)
- 爬取页面/接口，提取有效信息

#### 背景

- 快递业务
  - 各大快递网点信息 [yunda](http://www.yundaex.com/ydweb/fuwuwangdian_search.php)
- 饿了么、美团外卖商家
  - 商家信息及配送相关信息

#### 动手

- node-crawler [github](https://github.com/bda-research/node-crawler)
  - 访问页面，处理返回，提取数据
    ```js
    // 列表爬虫
    const spider = new Carwler({
      callback(err, res, done) {
        // do sth
        const detailQueue = []
        detailSpider.queue(detailQueue)
      }
    })
    // 详情爬虫
    const detailSpider = new Crawler()
    // 开始
    const run = () => {
      const listQueue = []
      // 需要爬取的内容队列
      spider.queue(listQueue)
    }
    ```
  - cheerio
    > 去除了浏览器端相关部分的jQuery，能够解析标签，在server端对html进行一些和客户端jQuery一样的操作

    ![5b13ae956b210.png](https://i.loli.net/2018/06/03/5b13ae956b210.png)

- puppeteer
  - 谷歌出品，必属精品
  - 打开一个headless chrome，模拟访问，页面执行
    ```js
    async start() {
      // launch browser
      this.browser = await puppeteer.launch();
      this.logger.info("Service puppeteerService Started");
    }
    async take(url) {
      // create page
      const page = await this.browser.newPage();
      page.setViewport({ width: 1920, height: 1080 });
      await page.goto(url);
      // screenshot
      const buf = await page.screenshot({ type: "jpeg", quality: 60 });
      await page.close();
      return {
        // 现在 IPC Hub 不能直接传递 Buffer，需要 base64。
        list: buf.toString("base64")
      };
    }
    ```
    - [example article](http://csbun.github.io/blog/2017/09/puppeteer/)
    - [api doc](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md)

#### 问题

- 饿了么商家
  - 区域覆盖
    ![5b13b6ef82642.png](https://i.loli.net/2018/06/03/5b13b6ef82642.png)
  - 代理
    - 重试，加日志
    - 代理动态替换

#### more

- pandora
  - service
