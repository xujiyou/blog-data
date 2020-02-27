# Helm 最佳实践

英文官方文档：https://helm.sh/docs/topics/chart_best_practices/conventions/

## 惯例

chart 名称应为小写字母和数字。单词*可以*用破折号（-）分隔。

chart 名称中不得使用大写字母或下划线。图表名称中不应使用圆点。

包含 chart 的目录必须与 chart 具有相同的名称。

YAML文件应使用*两个空格*缩进（并且不得使用制表符）。

术语`chart`不需要大写，因为它不是专有名词。

`Chart.yaml`确实需要大写，因为文件名区分大小写



## 变量命名约定

重点是 values.yaml 文件

变量名称应以小写字母开头，单词应以驼峰分隔。

每个变量名都应该有注释。





