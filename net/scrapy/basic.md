# Scrapy

## 基础

**1`安装命令**

```
pip install Scrapy
```

**2`生成项目**

```
scrapy startproject dianzikedaSpiders
```

**3`修改配置**

在settings.py中修改“君子协定”

```python
# Obey robots.txt rules
ROBOTSTXT_OBEY = False # 之前是True
```

修改默认请求头

```
# Override the default request headers:
DEFAULT_REQUEST_HEADERS = {
  'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
  'Accept-Language': 'en',
}
```

修改下载延迟

```python
DOWNLOAD_DELAY = 3
```

**4`生成爬虫**

执行下面这段命令后，会生成dianzikeda.py 这个爬虫文件

```
scrapy genspider dianzikeda https://yjsjy.uestc.edu.cn/gmis/jcsjgl/dsfc/index/
```

**5`pipeLine的预处理**

这样才能正确的写入中文

```python
import codecs
import json
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://docs.scrapy.org/en/latest/topics/item-pipeline.html


# useful for handling different item types with a single interface
from itemadapter import ItemAdapter


class DianzikedaspidersPipeline:

    def __init__(self):
        self.file = codecs.open('title.json', 'wb', encoding='utf-8')

    def process_item(self, item, spider):
        line = json.dumps(dict(item), ensure_ascii=False, sort_keys=False) + "\n"
        self.file.write(line)
        return item

    def close_spider(self, spider):
        self.file.close()
```

**6`网站**

在dianzikeda.py中，就是我们生成的爬虫

```python
class DianzikedaSpider(scrapy.Spider):
    name = 'dianzikeda'  # 爬虫名称必须唯一
    allowed_domains = ['uestc.edu.cn']  # 允许采集的域名
    start_urls = ['http://https://yjsjy.uestc.edu.cn/gmis/jcsjgl/dsfc/index//']  # 开始采集的网站

    # 解析响应数据 提取数据 以及 网址
    # response 就是网站源码
    def parse(self, response):
        print(response.xpath("//title/text()").extract())
```

## 使用

**编写爬虫**

![image-20201105104012117](D:\LearningNotes\net\scrapy\image-20201105104012117.png)

![image-20201105105209320](D:\LearningNotes\net\scrapy\image-20201105105209320.png)

**运行爬虫**

scrapy crawl [爬虫名称]

```
scrapy crawl dianzikeda
```

## 用法

### tbody

table里面的tbody在很多情况下，都没有占用一次选取器，即

```
"//table/tr" 就可以选取了
"//table/tbody/tr" 反而选不到元素
```

### **自定义请求标头**

一定要将空格都清理干净，尽量模仿正确的请求头

```python
class DianzikedaSpider(scrapy.Spider):
    name = 'dianzikeda'  # 爬虫名称必须唯一
    allowed_domains = ['uestc.edu.cn', '.baidu.com']  # 允许采集的域名
    start_urls = ['https://yjsjy.uestc.edu.cn/gmis/jcsjgl/dsfc/index/']  # 开始采集的网站
    custom_settings = {
        "DEFAULT_REQUEST_HEADERS": {
            'Accept': "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
            'Accept-Encoding': "gzip,deflate,br",
            'Accept-Language': "zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6",
            'Cache-Control': "max-age=0",
            'Connection': "keep-alive",
            'Cookie': "JSESSIONID=9F4EBC7418F05EAD61850A666752B726.gmis_server;UM_distinctid=17590f34d325a-020cc88fe47a16-7d677c6e-144000-17590f34d33679",
            'Host': "yjsjy.uestc.edu.cn",
            'Sec-Fetch-Dest': "document",
            'Sec-Fetch-Mode': "navigate",
            'Sec-Fetch-Site': "none",
            'Sec-Fetch-User': "?1",
            'Upgrade-Insecure-Requests': "1",
            "User-Agent": "Mozilla/5.0(WindowsNT10.0;Win64;x64)AppleWebKit/537.36(KHTML,likeGecko)Chrome/86.0.4240.111Safari/537.36Edg/86.0.622.61"
        }
    }

    # 解析响应数据 提取数据 以及 网址
    # response 就是网站源码
    def parse(self, response):
        print(response.xpath("//title/text()").extract())

```

### **循环遍历多个网站**

![image-20201105104924265](D:\LearningNotes\net\scrapy\image-20201105104924265.png)

### **翻页操作**

![image-20201105105139752](D:\LearningNotes\net\scrapy\image-20201105105139752.png)

### **保存爬虫数据**

![image-20201105105330378](D:\LearningNotes\net\scrapy\image-20201105105330378.png)

![image-20201105105343615](D:\LearningNotes\net\scrapy\image-20201105105343615.png)

![image-20201105105352049](D:\LearningNotes\net\scrapy\image-20201105105352049.png)

实际上，<u>如果在pipeline文件中，已经有对文件的操作了</u>的话

在这里就<u>不需要使用"-o"的相关选项</u>了，使用了反而多余。

### 匹配多个属性的元素

contains(@align,'left') 比

@align=‘left’更好！

因为align="left center"这样的话，使用contains 仍然可以匹配上

```
td_selector = selector.xpath("./td[contains(@align,'left') and contains(@valign,'center')]")
```

### 获取属性值

```
response.xpath("//a/@href").getall()
```

### 不再打印到控制台

```
scrpay crawl spider_name  -s LOG_FILE=all.log
```

