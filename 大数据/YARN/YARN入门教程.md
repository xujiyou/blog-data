# YARN 入门教程

首先运行几个 Yarn 命令

查看版本号：

```bash
$ yarn version
```

查看 application 列表：

```bash
$ yarn application -list
```

查看某种状态的 application 的列表：

```bash
$ yarn application -list -appStates FINISHED
```

查看主机列表：

```bash
$ yarn node -list
```



## Java 远程连接 YARN

创建 Maven 项目，加入以下依赖：

```xml
dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-yarn-client</artifactId>
        <version>3.2.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>3.2.1</version>
    </dependency>
</dependencies>
```

编写 Java 代码，代码用于查看 YARN 中的 application 列表：

```java
package work.xujiyou;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.yarn.api.records.*;
import org.apache.hadoop.yarn.client.api.YarnClient;
import org.apache.hadoop.yarn.conf.YarnConfiguration;
import org.apache.hadoop.yarn.exceptions.YarnException;

import java.io.IOException;
import java.util.EnumSet;
import java.util.List;

/**
 * Main class
 *
 * @author jiyouxu
 * @date 2019/12/22
 */
public class Main {

    public static void main(String[] args) throws IOException, YarnException {
        YarnClient yarnClient = YarnClient.createYarnClient();
        Configuration conf = new YarnConfiguration();
        conf.set("yarn.resourcemanager.address", "fueltank-2.cloud.bbdops.com:8032");
        conf.set("yarn.resourcemanager.admin.address", "fueltank-2.cloud.bbdops.com:8033");
        conf.set("yarn.resourcemanager.scheduler.address", "fueltank-2.cloud.bbdops.com:8030");
        conf.set("yarn.resourcemanager.resource-tracker.address", "fueltank-2.cloud.bbdops.com:8031");
        conf.set("yarn.resourcemanager.webapp.address", "fueltank-2.cloud.bbdops.com:8088");
        conf.set("yarn.resourcemanager.webapp.https.address", "fueltank-2.cloud.bbdops.com:8090");
        yarnClient.init(conf);
        yarnClient.start();
        
        try {
            List<ApplicationReport> applications = yarnClient.getApplications(EnumSet.of(YarnApplicationState.RUNNING, YarnApplicationState.FINISHED));
            System.out.println(applications);
            if (applications.size() > 0) {
                System.out.println("ApplicationId ============> "+applications.get(0).getApplicationId());
                System.out.println("name ============> "+applications.get(0).getName());
                System.out.println("queue ============> "+applications.get(0).getQueue());
                System.out.println("queue ============> "+applications.get(0).getUser());
            }
        } catch (YarnException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        yarnClient.stop();
    }
}
```

