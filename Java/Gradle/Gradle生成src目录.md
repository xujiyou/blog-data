# Gradle 生成 src 目录

使用 IDEA 生成的 Gradle 项目没有 src 目录。

```groovy
def createDir = {
    path ->
        File dir = new File(path)
        if (!dir.exists()){
            dir.mkdirs()
        }
}

task makeJavaDir(){
    def paths=['src/main/java','src/main/resources','src/test/java','src/test/resources']
    doFirst{
        paths.forEach(createDir)
    }
}
```

