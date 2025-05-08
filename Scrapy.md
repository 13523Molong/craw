# Scrapy 使用教程
前言:
其实爬虫所涉及的技术十分广泛，而当你接到一个系统爬虫的项目时，里面所涉及到很多模块，比如有些负责封装HTTP，有些负责解析网页内容，有些负责数据处理，有些负责调度等等。而Scrapy框架就是将这些模块封装起来，形成一个完整的爬虫系统。因此学习系统的框架十分有必要

而Scrapy爬虫的基本步骤大致可以分为:
1.创建一个Scrapy项目
2.创建一个爬虫脚本进行数据抓取
3.将抓取到数据写入你定义的Item数据模型中
4.利用Item Pipeline存储抓取Item,即数据实体对象

## 目录
1. [Scrapy 简介](#scrapy-简介)
2. [安装与配置](#安装与配置)
3. [项目创建与结构](#项目创建与结构)
4. [Spider 开发详解](#spider-开发详解)
5. [选择器与数据提取](#选择器与数据提取)
6. [Item 与数据处理](#item-与数据处理)
7. [中间件详解](#中间件详解)
8. [管道(Pipeline)开发](#管道pipeline开发)
9. [部署与调度](#部署与调度)
10. [常见问题与优化](#常见问题与优化)
11. [最佳实践](#最佳实践)

---

## Scrapy 简介
Scrapy 是一个用 Python 编写的开源网络爬虫框架，用于快速、高效地抓取网站数据。

**核心组件**：
- Scrapy Engine(引擎)：负责Spider、ItemPipeline、Downloader、Scheduler中间的通讯，信号、数据传递等。  
- Scheduler(调度器)：它负责接受引擎发送过来的Request请求，并按照一定的方式进行整理排列，入队，当引擎需要时，交还给引擎。
- Downloader(下载器)：负责下载Scrapy Engine(引擎)发送的所有Requests请求，并将其获取到的Responses交还给Scrapy Engine(引擎)，由引擎交给Spider来处理。
- Spiders(爬虫)：它负责处理所有Responses,从中分析提取数据，获取Item字段需要的数据，并将需要跟进的URL提交给引擎，再次进入Scheduler(调度器)。
- Item Pipeline：他负责处理收集的到的Item数据模型，并进行后期的处理或存储。

- Middlewares：中间件系统，又分为Spider Middleware 和 Downloader Middleware，分别处理Spider的输入与输出数据以及Downloader的输入与输出数据。


**特点**：
✔️ 异步处理架构  
✔️ 内置CSS/XPath选择器  
✔️ 交互式shell调试  
✔️ 完善的中间件系统  
✔️ 支持多种数据导出格式

---

## 安装与配置

### 基础安装
```bash
pip install scrapy

# 可选依赖
pip install scrapy[ssl]  # HTTPS支持
pip install scrapy[csv]  # CSV导出
pip install scrapy[stats]  # 统计扩展

# 炎症安装
scrapy version

```

配置设置 settings.py
```python
# 常用配置项
USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'
ROBOTSTXT_OBEY = False  # 是否遵守robots协议
CONCURRENT_REQUESTS = 16  # 并发请求数
DOWNLOAD_DELAY = 0.5  # 下载延迟(秒)
COOKIES_ENABLED = False  # 禁用Cookies可提高性能
DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml',
    'Accept-Language': 'en',
}
```



## 项目创建与结构

### 创建项目 
```bash
scrapy startproject myproject
cd myproject
scrapy genspider example example.com
```
### 创建爬虫文件
```bash
cd 项目名/项目名
# 创建爬虫文件
scrapy genspider -t basic 爬虫名称 域名
```
---

### 项目大致结构
    myproject/
    ├── scrapy.cfg            # 部署配置文件
    └── myproject/            # 项目模块
        ├── __init__.py
        ├── items.py          # 数据模型定义
        ├── middlewares.py    # 中间件
        ├── pipelines.py      # 数据处理管道
        ├── settings.py       # 项目设置
        └── spiders/          # 爬虫目录
            ├── __init__.py
            └── example.py    # 示例爬虫
---

## Spider 开发详解
### 基本结构
```python
import scrapy
class MySpider(scrapy.Spider):
    name = 'my_spider'   # 唯一标识符
    allowed_domains = ['example.com']  # 允许抓取的域名列表
    start_urls = [
        'http://www.example.com/',
        'http://www.example.com/page2',
    ]

    def parse(self, response):
        pass
```
- `name`：爬虫的唯一标识。
- `allowed_domains`：允许访问的域名列表。
- `start_urls`：起始URLs，用于开始爬取。
- `parse()`方法：解析响应并生成新的请求或提取数据项（Item）。

### 编写爬虫逻辑
在`parse`方法中实现具体的爬取和数据处理