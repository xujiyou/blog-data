# Elasticsearch 分析大乐透

使用 scrapy 爬取历史大乐透数据，然后放入 ES 中，再利用 Kibana 进行可视化展示走势。

大乐透的历史开奖页：http://www.lottery.gov.cn/historykj/history.jspx?_ltype=dlt

可选好查询期数后，将网页保存到本地。

## 爬虫

生成爬虫代码：

```bash
$ pip3 install scrapy
$ scrapy startproject letou_spider
$ cd letou_spider/
$ scrapy genspider letou www.lottery.gov.cn
```

安装 ES 依赖：

```bash
$ pip3 install elasticsearch
```

编写代码:

```python
# -*- coding: utf-8 -*-
import scrapy
from elasticsearch import Elasticsearch
from elasticsearch import helpers


def gendata(trs):
    for tr in trs:
        tds = tr.css("td")
        stage = tds[0].css("::text").get()
        front_one = tds[1].css("::text").get()
        front_two = tds[2].css("::text").get()
        front_three = tds[3].css("::text").get()
        front_four = tds[4].css("::text").get()
        front_five = tds[5].css("::text").get()
        after_one = tds[6].css("::text").get()
        after_two = tds[7].css("::text").get()
        first_prize_count = tds[8].css("::text").get()
        first_prize_money = tds[9].css("::text").get()
        first_prize_append_count = tds[10].css("::text").get()
        first_prize_append_money = tds[11].css("::text").get()
        second_prize_count = tds[12].css("::text").get()
        second_prize_money = tds[13].css("::text").get()
        second_prize_append_count = tds[14].css("::text").get()
        second_prize_append_money = tds[15].css("::text").get()
        sales_volume = tds[17].css("::text").get()
        prize_pool = tds[18].css("::text").get()
        date = tds[19].css("::text").get()

        letou_map = {"stage": stage, "front_one": front_one, "front_two": front_two, "front_three": front_three,
                     "front_four": front_four, "front_five": front_five, "after_one": after_one, "after_two": after_two,
                     "first_prize_count": first_prize_count, "first_prize_money": first_prize_money,
                     "first_prize_append_count": first_prize_append_count,
                     "first_prize_append_money": first_prize_append_money,
                     "second_prize_count": second_prize_count, "second_prize_money": second_prize_money,
                     "second_prize_append_count": second_prize_append_count,
                     "second_prize_append_money": second_prize_append_money,
                     "sales_volume": sales_volume, "prize_pool": prize_pool, "date": date}
        index_map = {
            "_index": "daletou",
            "_source": letou_map
        }
        yield index_map


class LetouSpider(scrapy.Spider):
    es = Elasticsearch(
        hosts=["http://fueltank-4:9200/"]
    )
    name = 'letou'
    allowed_domains = ['www.lottery.gov.cn']
    start_urls = ['file:///Users/jiyouxu/Downloads/%E4%B8%AD%E5%9B%BD%E4%BD%93%E5%BD%A9%E7%BD%91%20-%20%E5%BC%80%E5%A5%96%E5%8E%86%E5%8F%B2%E9%A1%B5-%E5%A4%A7%E4%B9%90%E9%80%8F.htm']

    def parse(self, response):
        trs = response.css("tbody tr")
        helpers.bulk(self.es, gendata(trs))

```

搞定，一个 bulk 操作，将数据全部插入到 ES 中了，大约有 1900 多条数据：



