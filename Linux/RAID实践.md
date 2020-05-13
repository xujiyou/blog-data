# RAID 实践

在工作场景中，不乏遇到在线阵列配置或硬盘模式更改的。为了方便理解，打一个不是很恰当的比方：一台业务机器除系统盘外全部单盘raid0做数据挂载，在业务运行过程中，有一块硬盘坏了，我们在线更换硬盘后，因为硬盘模式的问题，此时系统是无法识别硬盘的，对不对，那怎么办？重启机器配置阵列？那如果业务较重要，不能停机呢？这时就是perccli和storccli大展身手的时候了。

其实perccli和storccli就是同一个工具，语法完全一样，只是命令名字不一样，适用的品牌不一样。perccli适用于dell机器，storccli适用于华为、浪潮（其它品牌没有测试过，不确认）



## 实操

下面查看硬件 RAID 信息，使用到一个名为 storcli 的工具。

storcli 文档：[storcli手册](https://wiki2.xbits.net:4430/hardware:lsi:storcli手册)

一个脚本：[利用storcli巡检raid](https://wiki2.xbits.net:4430/hardware:lsi:利用storcli巡检raid)

脚本内容如下：

```bash
#!/bin/bash
which storcli > /dev/null 2>&1
 
if [[ $? == 0 ]];then
# 检查整体情况
storcli /cALL show all J | jq -r '.Controllers[0]."Response Data" | {"阵列卡":.Basics.Controller, "型号":.Basics.Model, "SN":.Basics."Serial Number", "驱动":.Version."Driver Name", "接口类型":.Bus."Host Interface",  "设备带宽":.Bus."Device Interface", "有无电池":.HwCfg.BBU, "CacheSize":.HwCfg."On Board Memory Size"}, {"阵列卡状态":.Status."Controller Status", "蜂鸣器报警":.HwCfg."Alarm"}, {"硬盘总数":."Physical Drives"}, {"VD数":."Virtual Drives", "VD列表":."VD LIST"[]}'
# RAID状态
storcli /cALL/vALL show J | jq -r '.Controllers[0]."Response Data"."Virtual Drives"[] | {"VD":."DG/VD", "RAID状态":."State"}'
else
	echo "storcli NOT installed..."
fi
```

