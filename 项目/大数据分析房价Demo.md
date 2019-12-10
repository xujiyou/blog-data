# 大数据分析房价Demo

项目采用 scrapy 采集链家的数据，然后采用 Spark 对数据进行分析

## Scrapy 采集数据

采集的是成都市链家的数据

先创建 python 文件：spider.py

```python
import json
import scrapy


class BlogSpider(scrapy.Spider):
    name = 'home spider'
    start_urls = ['https://cd.fang.lianjia.com/loupan/pg1/']

    for page in range(1, 101):
        url = "https://cd.fang.lianjia.com/loupan/pg{0}/".format(page)
        start_urls.append(url)

    def parse(self, response):

        file_name = "home_list.txt"
        with open(file_name, 'a') as f:
            for community_class in response.css('.resblock-list'):
                desc_wrapper_class = community_class.css(".resblock-desc-wrapper")
                name_class = desc_wrapper_class.css(".resblock-name")
                name = name_class.css("a ::text").get()
                location_class = community_class.css(".resblock-location")
                zone = location_class.css("span:first-child ::text").get()
                street = location_class.css("span:nth-child(3) ::text").get()
                door = location_class.css("a ::text").get()
                area_class = community_class.css(".resblock-area")
                area = area_class.css("span ::text").get()
                price_class = community_class.css(".resblock-price")
                unit_price = price_class.css(".main-price .number ::text").get()
                all_price = price_class.css(".second ::text").get()

                f.write(json.dumps({
                    '小区': name,
                    "地址": zone + " - " + street + " - " + door,
                    "建面": area,
                    "单价": unit_price,
                    "总价": all_price}).encode("utf-8").decode('unicode_escape') + "\n")
```

然后执行：

```
scrapy runspider spider.py
```

随后会在当前目录生成一个 home_list.txt 的文件。可以将这个文件收到放入 HDFS ，也可以通过 Flume 自动放入。

这里为了简单，就是通过本地文件的方式让 Spark 读取这个文件

下面是这个文件头部的几条数据

```json
{"小区": "融创玖樾台邸", "地址": "锦江 - 三圣乡 - 三圣花乡银木街与茶花街交汇处", "建面": "建面 50-63㎡", "单价": "10000", "总价": "总价63万/套"}
{"小区": "中南樾府", "地址": "金牛 - 天回镇 - 金牛区聚霞路933号", "建面": "建面 120-148㎡", "单价": "13800", "总价": "总价155万/套"}
{"小区": "成都文旅城", "地址": "都江堰 - 都江堰 - 成都市都江堰市青外江大桥与青城山大道交汇处(玉堂LG博爱中学旁)", "建面": "建面 75-142㎡", "单价": "9000", "总价": "总价72万/套"}
{"小区": "万科公园都会", "地址": "天府新区 - 麓湖生态城 - 天府新区鹭湖南路东段360-361号", "建面": "建面 105-136㎡", "单价": "21000", "总价": "总价210万/套"}
{"小区": "东湖郡", "地址": "新都 - 新都城区 - 新繁镇文庙街88号", "建面": null, "单价": "19000", "总价": null}
```



## 使用 Spark 对数据进行分析

这里只是简单的对房单价进行排序，排序之前，需要先将字符串转换为Json

```scala
package work.xujiyou.spark

import org.apache.spark.{SparkConf, SparkContext}

import scala.util.parsing.json.JSON

object WordCount {

  def strToInt(str: String): Int = {
    val regex = """([0-9]+)""".r
    val res = str match {
      case regex(num) => num
      case _ => "0"
    }
    val resInt = Integer.parseInt(res)
    resInt
  }

  def main(args: Array[String]) {
    val inputFile = "/Users/jiyouxu/PycharmProjects/spider/home_list.txt"
    val conf = new SparkConf().setAppName("WordCount").setMaster("local")
    val sc = new SparkContext(conf)
    val textFile = sc.textFile(inputFile)
    val wordCount = textFile
      .map(line => JSON.parseFull(line).get.asInstanceOf[Map[String, String]])
      .sortBy(map => strToInt(map("单价")))
    wordCount.foreach(println)
  }
}
```

打印的部分日志，房价最高的10条：

```
Map(地址 -> 高新 - 金融城 - 交子大道33号, 建面 -> 建面 160-186㎡, 小区 -> 中国华商交子公馆, 总价 -> 总价800万/套, 单价 -> 60000)
Map(地址 -> 成华 - 万年场 - 建工路龙湖梵城, 建面 -> null, 小区 -> 龙湖梵城, 总价 -> null, 单价 -> 65000)
Map(地址 -> 高新 - 金融城 - 天府大道北段1199号（高新国际广场旁）, 建面 -> 建面 580-660㎡, 小区 -> 银泰中心, 总价 -> 总价4000万/套, 单价 -> 70000)
Map(地址 -> 锦江 - 三官堂 - 三官堂街与龙舟路交汇处（望江公园、望江楼正对面）, 建面 -> 建面 443-660㎡, 小区 -> 望江名门, 总价 -> 总价4500万/套, 单价 -> 70000)
Map(地址 -> 锦江 - 东大路 - 通源街588号, 建面 -> null, 小区 -> 乐天圣苑, 总价 -> null, 单价 -> 70000)
Map(地址 -> 武侯 - 双楠 - 二环路西一段80号, 建面 -> null, 小区 -> 金科双楠天都, 总价 -> null, 单价 -> 70000)
Map(地址 -> 锦江 - 东大路 - 通源街588号, 建面 -> null, 小区 -> 乐天圣苑, 总价 -> null, 单价 -> 74000)
Map(地址 -> 高新 - 新会展 - 益州大道中段555号, 建面 -> null, 小区 -> 星宸国际, 总价 -> null, 单价 -> 80000)
Map(地址 -> 成华 - 李家沱 - 二环路与府青路交汇处（原512建材市场旁）, 建面 -> null, 小区 -> 永立星城都, 总价 -> null, 单价 -> 80000)
Map(地址 -> 高新 - 芳草 - 永丰路7号（衣冠庙地铁口B出口）, 建面 -> null, 小区 -> 和雅嘉御, 总价 -> null, 单价 -> 80000)
```

