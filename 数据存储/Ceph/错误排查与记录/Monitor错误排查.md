# Monitor错误排查

官方文档：https://ceph.readthedocs.io/en/latest/rados/troubleshooting/troubleshooting-mon/

当集群遇到与 Monitor 相关的问题时，就会出现恐慌的趋势，有时甚至是有充分理由的。 应该牢记，丢失一台或多台监视器并不一定意味着您的群集已关闭，只要大多数已启动，正在运行并具有适当的法定人数即可。 无论情况有多严重，您都应该做的第一件事是冷静下来，屏住呼吸，然后尝试回答我们的初始故障排除脚本。