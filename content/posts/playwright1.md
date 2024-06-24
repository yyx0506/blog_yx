---
title: "playwright 介绍和简单使用"
date: 2023-02-4
categories:
- tranquilpeak
- features
tags:
- playwright


thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->




### playwright 介绍和简单使用

#### 1 什么是playwright

```
Playwright 是微软在 2020 年初开源的新一代自动化测试工具，它的功能类似于 Selenium、Pyppeteer 等，都可以驱动浏览器进行各种自动化操作。它的功能也非常强大，对市面上的主流浏览器都提供了支持，API 功能简洁又强大。虽然诞生比较晚，但是现在发展得非常火热。
```

#### 2 **Playwright 的特点**

```
Playwright 支持当前所有主流浏览器，包括 Chrome 和 Edge（基于 Chromium）、Firefox、Safari（基于 WebKit） ，提供完善的自动化控制的 API。 一般来说大家都是使用 chromium 
Playwright 支持所有浏览器的 Headless 模式和非 Headless 模式的测试。
Playwright 的安装和配置非常简单，安装过程中会自动安装对应的浏览器和驱动，不需要额外配置 WebDriver 等。
Playwright 提供了自动等待相关的 API，当页面加载的时候会自动等待对应的节点加载，大大简化了 API 编写复杂度。
```

#### 3 playwright 的特点

```
要使用 Playwright，需要 Python 3.7 版本及以上
pip3 install playwright  
playwright install

这时候 Playwrigth 会安装 Chromium, Firefox and WebKit 浏览器并配置一些驱动，我们不必关心中间配置的过程，Playwright 会为我们配置好。

具体的安装说明可以参考：https://setup.scrape.center/playwright。

安装完成之后，我们便可以使用 Playwright 启动 Chromium 或 Firefox 或 WebKit 浏览器来进行自动化操作了。

我们也可以把 这个服务打包成镜像 随时在服务端启动


```

#### 4 基本使用

```
Playwright 支持两种编写模式，一种是类似 Pyppetter 一样的异步模式，另一种是像 Selenium 一样的同步模式，我们可以根据实际需要选择使用不同的模式。

我们先来看一个基本同步模式的例子：
from playwright.sync_api import sync_playwright

def run(playwright):
    chromium = playwright.chromium # or "firefox" or "webkit".
    browser = chromium.launch()
    page = browser.new_page()
    page.goto("http://example.com")
    # other actions...
    browser.close()

with sync_playwright() as playwright:
    run(playwright)

首先我们导入了 sync_playwright 方法，然后直接调用了这个方法，该方法返回的是一个 PlaywrightContextManager 对象，可以理解是一个浏览器上下文管理器，我们将其赋值为变量 p。

接着我们调用了 PlaywrightContextManager 对象的 chromium、firefox、webkit 属性依次创建了一个 Chromium、Firefox 以及 Webkit 浏览器实例，接着用一个 for 循环依次执行了它们的 launch 方法，同时设置了 headless 参数为 False。
注意：如果不设置为 False，默认是无头模式启动浏览器，我们看不到任何窗口。

launch 方法返回的是一个 Browser 对象，我们将其赋值为 browser 变量。然后调用 browser 的 new_page 方法，相当于新建了一个选项卡，返回的是一个 Page 对象，将其赋值为 page，这整个过程其实和 Pyppeteer 非常类似。接着我们就可以调用 page 的一系列 API 来进行各种自动化操作了，比如调用 goto，就是加载某个页面，这里我们访问的是百度的首页。接着我们调用了 page 的 screenshot 方法，参数传一个文件名称，这样截图就会自动保存为该图片名称，这里名称中我们加入了 browser_type 的 name 属性，代表浏览器的类型，结果分别就是 chromium, firefox, webkit。另外我们还调用了 title 方法，该方法会返回页面的标题，即 HTML 中 title 节点中的文字，也就是选项卡上的文字，我们将该结果打印输出到控制台。最后操作完毕，调用 browser 的 close 方法关闭整个浏览器，运行结束。


在 看一下异步的代码 
import asyncio
from playwright.async_api import async_playwright

async def run(playwright):
    chromium = playwright.chromium # or "firefox" or "webkit".
    browser = await chromium.launch()
    page = await browser.new_page()
    await page.goto("http://example.com")
    # other actions...
    await browser.close()

async def main():
    async with async_playwright() as playwright:
        await run(playwright)
asyncio.run(main())


```



#### 4 相关使用代码

```
截图 
>>> from playwright.sync_api import sync_playwright

>>> playwright = sync_playwright().start()

>>> browser = playwright.chromium.launch()
>>> page = browser.new_page()
>>> page.goto("http://whatsmyuseragent.org/")
>>> page.screenshot(path="example.png")
>>> browser.close()

>>> playwright.stop()


加载整个页面
import asyncio
from playwright.async_api import async_playwright

async def run(playwright):
    webkit = playwright.webkit
    iphone = playwright.devices["iPhone 6"]
    browser = await webkit.launch()
    context = await browser.new_context(**iphone)
    page = await context.new_page()
    await page.goto("http://example.com")
    # other actions...
    await browser.close()

async def main():
    async with async_playwright() as playwright:
        await run(playwright)
        
asyncio.run(main())

# 我们可以根据加载的链接url 去拦截我们想要的结果

class ServerBrowser(object):

    async def new_context_page(self,browser):
        headers = {}
        context = await browser.new_context(
            proxy=proxy,
            extra_http_headers=headers,
        )
        # await context.add_cookies([cookie])
        page = await context.new_page()
        return context,page

    async def new_browser_context_page(self, playwright):
        browser = await playwright.chromium.connect_over_cdp(
            endpoint_url=self.endpoint_url,
        )
        context, page = await self.new_context_page(browser=browser)
        return browser, context, page

    async def on_response(self, resp):
        url = resp.url
        if 'apsssss' in url:
            js = await resp.json()
						

    async def start_browser(self):

        async with async_playwright() as playwright:
            browser = await playwright.chromium.connect_over_cdp(
                endpoint_url='localhost',
            )
            proxy = None
            headers = {}
            context = await browser.new_context(
                proxy=proxy,
                extra_http_headers=headers,
                # locale='zh-CN',
                # timezone_id='Asia/Shanghai'
            )
            page = await context.new_page()
            await page.route('*', self.hook_route)
            page.on("response", self.on_response)
            while True:
                match_id = await self.queue_keyword.get()
                url = f"http://www.baidu.com"
                await page.goto(url)
```

