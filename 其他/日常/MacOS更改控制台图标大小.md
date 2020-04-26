# MacOS 更改控制台图表大小

见：https://www.jianshu.com/p/60315dfcda53https://www.jianshu.com/p/60315dfcda53

```bash
$ defaults write com.apple.dock springboard-columns -int 12
$ defaults write com.apple.dock ResetLaunchPad -bool TRUE;killall Dock
```

