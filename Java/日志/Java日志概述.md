# Java 日志概述

关于 Java 这块的日志，很常见的三个东西是 slf4j log4j logback。

笼统的讲就是slf4j是一系列的日志接口，而log4j logback是具体实现了的日志框架。

而log4j和logback就是两个受欢迎的日志框架。但两者又有不同。

- log4j 是 apache 实现的一个开源日志组件。（Wrapped implementations）
- logback 同样是由 log4j 的作者设计完成的，拥有更好的特性，用来取代 log4j 的一个日志框架。是 slf4j 的原生实现。

现在很多大数据组件，包括 HDFS、Spark、Flink 等，都还在使用 log4j，所以还是了解学习一下 log4j 还是很有必要的。

