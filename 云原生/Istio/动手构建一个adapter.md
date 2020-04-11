# 动手构建一个 adapter

adapter 用于从 Istio Mixer 中导出 Metric。下面就开始动手构建一个简单的 adapter，以理解其原理。

官方教程：https://github.com/istio/istio/wiki/Mixer-Out-of-Process-Adapter-Walkthrough

首先去 Github 下载 Istio 的源码，这里我选的 1.4.7 版本的源码。

官方提供的几个 adapter 都在源码的 `mixer/adapter` 包中。

