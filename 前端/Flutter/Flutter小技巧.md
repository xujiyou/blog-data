# Flutter 小技巧



报错：

```
/Users/jiyouxu/.pub-cache/hosted/pub.flutter-io.cn/flutter_qiniu-0.1.3/ios/Classes/FlutterQiniuPlugin.m:2:9: fatal error: 'flutter_qiniu/flutter_qiniu-Swift.h' file not found
    #import <flutter_qiniu/flutter_qiniu-Swift.h>
```

参考：https://github.com/hui-z/flutter_install_plugin/issues/16

把代码改成：

```
#import <flutter_qiniu-Swift.h>
```

