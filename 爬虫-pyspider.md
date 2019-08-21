---
title: 爬虫-pyspider
date: 2018-02-05 16:19:14
categories: 
- 爬虫
tags:
- pyspider
---





爬虫

<!--more-->

## 一、python





## 二、charles

抓包软件

### 使用

手机和电脑登录同一个wifi，help-local ip address获取本地ip，

在手机选择对应的wifi-修改网络-高级选项-代理-手动-填写ip和端口（一般为888）

https需要SSL

参考：[Charles 从入门到精通](http://blog.devtang.com/2015/11/14/charles-introduction/#%E6%88%AA%E5%8F%96-Https-%E9%80%9A%E8%AE%AF%E4%BF%A1%E6%81%AF)

## 三、pyspider

#### 3.1windows安装:

3.1.1安装python

[下载地址](https://www.python.org/downloads/windows/)

3.1.2安装对应python版本的pycurl 

[下载地址](https://bintray.com/pycurl/pycurl/pycurl/view#files)

3.1.3配置config文件

```json
{
  "taskdb": "mysql+taskdb://root:0120@127.0.0.1:3306/taskdb",
  "projectdb": "mysql+projectdb://root:0120@127.0.0.1:3306/projectdb",
  "resultdb": "mysql+resultdb://root:0120@127.0.0.1:3306/resultdb",
  "queue-maxsize": 2000,
  "webui": {
    "port": 5000,
    "process-time-limit": 300
  },
  "scheduler": {
    "fail-pause-num": 0,
    "active-tasks": 2000,
    "loop-limit":2000,
    "threads":8
  },
  "processor": {
    "process-time-limit": 300
  },
  "fetcher": {
    "poolsize":2000
  },
  "all": {
    "fetcher-num": 20,
    "processor-num": 20
  }
}


#json文件放到pyspider的安装地址
C:\Users\Administrator\AppData\Local\Programs\Python\Python35\Lib\site-packages\pyspider
```



3.1.4通过配置文件启动pyspider

```
# pyspider的安装目录下启动
pyspider -c pyspider-config.json
```

3.1.5打开localhost:5000



### 3.2MAC

启动pyspider虚拟环境

```
source venv/bin/activate
```

使用自己的配置启动pyspider

```
pyspider -c pyspider-config.json
```



打开http://localhost:5000/进入pyspider页面

点击create-把页面右边的代码复制下来，修改为自己的爬虫。

参考：[pyspider官方](http://docs.pyspider.org/en/latest/)

参考：[pyspider文档](https://www.kancloud.cn/manbuheiniu/pyspider)

[pyspider中内容选择器常用方法汇总](http://www.pyspider.cn/jiaocheng/pyspider-pyquery-14.html)

[Python中PyQuery库的使用总结](http://www.cnblogs.com/shaosks/p/6845655.html)

[在Mac上安装phantomjs及运行](http://blog.csdn.net/u010267996/article/details/59480578)



### MySQL

参考：https://segmentfault.com/q/1010000006507996

```
pip search mysql-connector | grep --color mysql-connector-python
然后选择安装版本
pip install mysql-connector-python-rf==2.1.3
```





## 四、爬虫代码

### B站热门

```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-
# Created on 2018-02-03 16:36:48
# Project: netease

from pyspider.libs.base_handler import *
import time
import mysql.connector
import json


class Handler(BaseHandler):
    crawl_config = {
    }

    mysql_config = {
        'host': 'localhost',
        'user': '***',
        'password': '******',
        'port': 3306,
        'database': '*****',
        'charset': 'utf8'
    }

    cnx = mysql.connector.connect(**mysql_config)
    cursor = cnx.cursor()

    table_name = 'bilibili_ranking'

    @every(minutes=1 * 60)
    def on_start(self):
            self.crawl(u'https://api.bilibili.com/x/web-interface/ranking?rid=0&day=3&jsonp=jsonp',
                       callback=self.index_page,)

    @config(age=0)
    def index_page(self, response):
        data1 = json.loads(response.text)
        data = data1['data']
        article_list = data['list']
        result_list = []
        for article in article_list:
            result = {}
            result['aid'] = article['aid'].encode('utf8', 'ignore')
            result['author'] = article['author'].encode('utf8', 'ignore')
            result['coins'] = str(article['coins']).encode('utf8', 'ignore')
            result['duration'] = article['duration'].encode('utf8', 'ignore')
            result['mid'] = str(article['mid'])
            result['pic'] = article['pic'].encode('utf8', 'ignore')
            result['play'] = str(article['play'])
            result['pts'] = str(article['pts'])
            result['title'] = article['title'].encode('utf8', 'ignore')
            result['trend'] = str(article['trend'])
            result['video_review'] = str(article['video_review'])
            result['update_time'] = time.strftime('%Y-%m-%d %H:%M:%S').encode('utf8', 'ignore')
            result['table_name'] = self.table_name
            result_list.append(result)
        return result_list

    def on_result(self, result):
        if not result:
            return
        print(result)
        for item in result:
            self.process_result(item)

    def process_result(self, result):
        table_name = result.pop('table_name')
        col_str = ''
        row_str = ''
        for key in result.keys():
            col_str = col_str + " " + key + ","
            row_str = "{}'{}',".format(row_str,
                                       result[key] if "'" not in result[key] else result[key].replace("'", "\\'"))
            sql = "insert ignore INTO {} ({}) VALUES ({}) ON DUPLICATE KEY UPDATE ".format(table_name, col_str[1:-1],
                                                                                           row_str[:-1])
        for (key, value) in six.iteritems(result):
            sql += "{} = '{}', ".format(key, value if "'" not in value else value.replace("'", "\\'"))
        sql = sql[:-2]
        self.cursor.execute(sql)
        self.cnx.commit()

```

数据库（MySQL）

```mysql
CREATE TABLE `bilibili_ranking` (
  `aid` varchar(32) NOT NULL DEFAULT '',
  `author` varchar(32) DEFAULT NULL,
  `coins` varchar(32) DEFAULT NULL,
  `duration` varchar(32) DEFAULT NULL,
  `mid` varchar(32) DEFAULT NULL,
  `pic` varchar(128) DEFAULT NULL,
  `play` varchar(32) DEFAULT NULL,
  `pts` varchar(128) DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  `title` varchar(32) DEFAULT NULL,
  `trend` varchar(32) DEFAULT NULL,
  `video_review` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`aid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

------



### 链家二手房

以杭州为例

```python
from pyspider.libs.base_handler import *

start_page = 1
stop_page = 100


class Handler(BaseHandler):
    crawl_config = {
    }

    def __init__(self):
        self.url = "https://m.lianjia.com/hz/ershoufang/index/pg"
        self.start_page = start_page
        self.stop_page = stop_page

    @every(minutes=24 * 60)
    def on_start(self):
        while self.start_page < self.stop_page:
            url = self.url + str(self.start_page)
            self.crawl(url, callback=self.index_page)
            self.start_page += 1

    @config(age=0)
    def index_page(self, response):
        for each in response.doc('li[class="pictext"]').items():
            detail_url = each('a').attr.href
            print (detail_url)
            self.crawl(detail_url, callback=self.detail_page)

    @config(priority=2)
    def detail_page(self, response):
        return {
            "url": response.url,
            "标题": response.doc('title').text().encode('utf8', 'ignore'),
            "售价": response.doc('.similar_data > .similar_data_detail > p > span').text().encode('utf8', 'ignore'),
            "单价": response.doc('ul > .short').eq(0).text().encode('utf8', 'ignore').split("：")[1],
            "挂牌": response.doc('ul > .short').eq(1).text().encode('utf8', 'ignore').split("：")[1],
            "朝向": response.doc('ul > .short').eq(2).text().encode('utf8', 'ignore').split("：")[1],
            "楼层": response.doc('ul > .short').eq(3).text().encode('utf8', 'ignore').split("：")[1],
            "楼型": response.doc('ul > .short').eq(4).text().encode('utf8', 'ignore').split("：")[1],
            "电梯": response.doc('ul > .short').eq(5).text().encode('utf8', 'ignore').split("：")[1],
            "装修": response.doc('ul > .short').eq(6).text().encode('utf8', 'ignore').split("：")[1],
            "年代": response.doc('ul > .short').eq(7).text().encode('utf8', 'ignore').split("：")[1],
            "用途": response.doc('ul > .short').eq(8).text().encode('utf8', 'ignore').split("：")[1],
            "权属": response.doc('ul > .short').eq(9).text().encode('utf8', 'ignore').split("：")[1],
            "房源编码": response.doc('ul > .long').eq(0).text().encode('utf8', 'ignore').split("：")[1],
            "首付预算": response.doc('ul > .long').eq(1).text().encode('utf8', 'ignore').split("：")[1],
            "标签": response.doc('.tag_group').text().encode('utf8', 'ignore'),
            "小区": response.doc('ul > .long').eq(2).text().encode('utf8', 'ignore').split("：")[1],
            "房源介绍": response.doc('.sub_mod_box > .mod_cont').eq(2).text().encode('utf8', 'ignore'),
            "经纪人带看反馈": response.doc('.sub_mod_box > .mod_cont').eq(3).text().encode('utf8', 'ignore'),
            "房源户型": response.doc('.info_li > .info_content').eq(0).text().encode('utf8', 'ignore'),
            "建筑面积": response.doc('.info_li > .info_content').eq(1).text().encode('utf8', 'ignore'),
            "套内面积": response.doc('.info_li > .info_content').eq(2).text().encode('utf8', 'ignore'),
            "户型结构": response.doc('.info_li > .info_content').eq(3).text().encode('utf8', 'ignore'),
            "梯户比例": response.doc('.info_li > .info_content').eq(4).text().encode('utf8', 'ignore'),
            "供暖方式": response.doc('.info_li > .info_content').eq(5).text().encode('utf8', 'ignore'),
            "上次交易": response.doc('.info_li > .info_content').eq(6).text().encode('utf8', 'ignore'),
            "购房年限": response.doc('.info_li > .info_content').eq(7).text().encode('utf8', 'ignore'),
            "房屋用途": response.doc('.info_li > .info_content').eq(8).text().encode('utf8', 'ignore'),
            "交易权属": response.doc('.info_li > .info_content').eq(9).text().encode('utf8', 'ignore'),
            "产权所属": response.doc('.info_li > .info_content').eq(10).text().encode('utf8', 'ignore'),
            "抵押信息": response.doc('.info_li > .info_content').eq(11).text().encode('utf8', 'ignore'),
            "房本备件": response.doc('.info_li > .info_content').eq(12).text().encode('utf8', 'ignore'),
            "看房时间": response.doc('.info_li > .info_content').eq(13).text().encode('utf8', 'ignore'),
            "链家编号": response.doc('.info_li > .info_content').eq(14).text().encode('utf8', 'ignore'),
            "土地年限": response.doc('.info_li > .info_content').eq(15).text().encode('utf8', 'ignore'),
            "近7日带看(次)":
                response.doc('.mod_box > .mod_cont > .data > .box_col').eq(0).text().encode('utf8', 'ignore').split(
                    ' ')[1],
            "30日带看(次)":
                response.doc('.mod_box > .mod_cont > .data > .box_col').eq(1).text().encode('utf8', 'ignore').split(
                    ' ')[1],
            "关注(人)":
                response.doc('.mod_box > .mod_cont > .data > .box_col').eq(2).text().encode('utf8', 'ignore').split(
                    ' ')[1],
            "户型分间": response.doc('.sub_mod_box').eq(0).text().encode('utf8', 'ignore'),
            "户型分间img": response.doc('.sub_mod_box > .mod_cont > .pictext > .mod_media > .media_main > .lazyload').attr(
                'origin-src')
        }

```

------

### 链家小区

以杭州为例

```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-
# Created on 2018-02-03 16:36:48
# Project: netease

from pyspider.libs.base_handler import *
import time
import mysql.connector
import json


class Handler(BaseHandler):
    crawl_config = {
    }

    mysql_config = {
        'host': 'localhost',
        'user': 'root',
        'password': '******',
        'port': 3306,
        'database': '******',
        'charset': 'utf8'
    }

    cnx = mysql.connector.connect(**mysql_config)
    cursor = cnx.cursor()

    table_name = 'bilibili_ranking'

    @every(minutes=1 * 60)
    def on_start(self):
            self.crawl(u'https://api.bilibili.com/x/web-interface/ranking?rid=0&day=3&jsonp=jsonp',
                       callback=self.index_page,)

    @config(age=0)
    def index_page(self, response):
        data1 = json.loads(response.text)
        data = data1['data']
        article_list = data['list']
        result_list = []
        for article in article_list:
            result = {}
            result['aid'] = article['aid'].encode('utf8', 'ignore')
            result['author'] = article['author'].encode('utf8', 'ignore')
            result['coins'] = str(article['coins']).encode('utf8', 'ignore')
            result['duration'] = article['duration'].encode('utf8', 'ignore')
            result['mid'] = str(article['mid'])
            result['pic'] = article['pic'].encode('utf8', 'ignore')
            result['play'] = str(article['play'])
            result['pts'] = str(article['pts'])
            result['title'] = article['title'].encode('utf8', 'ignore')
            result['trend'] = str(article['trend'])
            result['video_review'] = str(article['video_review'])
            result['update_time'] = time.strftime('%Y-%m-%d %H:%M:%S').encode('utf8', 'ignore')
            result['table_name'] = self.table_name
            result_list.append(result)
        return result_list

    def on_result(self, result):
        if not result:
            return
        print(result)
        for item in result:
            self.process_result(item)

    def process_result(self, result):
        table_name = result.pop('table_name')
        col_str = ''
        row_str = ''
        for key in result.keys():
            col_str = col_str + " " + key + ","
            row_str = "{}'{}',".format(row_str,
                                       result[key] if "'" not in result[key] else result[key].replace("'", "\\'"))
            sql = "insert ignore INTO {} ({}) VALUES ({}) ON DUPLICATE KEY UPDATE ".format(table_name, col_str[1:-1],
                                                                                           row_str[:-1])
        for (key, value) in six.iteritems(result):
            sql += "{} = '{}', ".format(key, value if "'" not in value else value.replace("'", "\\'"))
        sql = sql[:-2]
        self.cursor.execute(sql)
        self.cnx.commit()

```

数据库（MySQL）

```mysql
CREATE TABLE `lianjia_xiaoqu` (
  `housecode` varchar(32) NOT NULL DEFAULT '',
  `title` varchar(32) DEFAULT NULL,
  `totalPrice` int(11) DEFAULT NULL,
  `build_time` int(11) DEFAULT NULL,
  `district` varchar(11) DEFAULT NULL,
  `bizcircle` varchar(11) DEFAULT NULL,
  `totalSellCount` int(11) DEFAULT NULL,
  `url` varchar(128) DEFAULT NULL,
  `subwary` varchar(32) DEFAULT NULL COMMENT '现有地铁',
  `resblockPosition` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`housecode`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```





## 五、参考链接

[Pyspider windows下的安装](https://blog.csdn.net/liyadongcn/article/details/55211137)