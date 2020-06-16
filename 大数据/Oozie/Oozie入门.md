# Oozie 入门

Oozie 的作用是将多个作业组合成一个逻辑工作单元（一个工作流）。

我使用的 CDH 安装的 Oozie，拿来即用即可。

---

## Quick Start

根据官方的 [Running the Examples](https://oozie.apache.org/docs/5.2.0/DG_Examples.html) 来做的。

首先下载解压 oozie-examples.tar.gz ，把里面的 examples 目录上传到 HDFS：

```bash
$ hadoop fs -put examples examples
```

然后运行任务：

```bash
$ oozie job -oozie http://fueltank-2:11000/oozie -config examples/apps/map-reduce/job.properties -run
job: 14-20090525161321-oozie-tucu
```

过程中会遇到各种错误，根据错误信息纠错即可。

job.properties 内容如下：

```properties
nameNode=hdfs://fueltank-2.cloud.bbdops.com:8020
resourceManager=fueltank-2.cloud.bbdops.com:8032
queueName=default
examplesRoot=examples

oozie.wf.application.path=${nameNode}/user/${user.name}/${examplesRoot}/apps/map-reduce/workflow.xml
outputDir=map-reduce
```

配置文件的主要作用找到 workflow.xml ，下面看看这个文件：

```xml
<workflow-app xmlns="uri:oozie:workflow:1.0" name="map-reduce-wf">
    <start to="mr-node"/>
    <action name="mr-node">
        <map-reduce>
            <resource-manager>${resourceManager}</resource-manager>
            <name-node>${nameNode}</name-node>
            <prepare>
                <delete path="${nameNode}/user/${wf:user()}/${examplesRoot}/output-data/${outputDir}"/>
            </prepare>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
                <property>
                    <name>mapred.mapper.class</name>
                    <value>org.apache.oozie.example.SampleMapper</value>
                </property>
                <property>
                    <name>mapred.reducer.class</name>
                    <value>org.apache.oozie.example.SampleReducer</value>
                </property>
                <property>
                    <name>mapred.map.tasks</name>
                    <value>1</value>
                </property>
                <property>
                    <name>mapred.input.dir</name>
                    <value>/user/${wf:user()}/${examplesRoot}/input-data/text</value>
                </property>
                <property>
                    <name>mapred.output.dir</name>
                    <value>/user/${wf:user()}/${examplesRoot}/output-data/${outputDir}</value>
                </property>
            </configuration>
        </map-reduce>
        <ok to="end"/>
        <error to="fail"/>
    </action>
    <kill name="fail">
        <message>Map/Reduce failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name="end"/>
</workflow-app>
```



查看任务信息：

```bash
$ oozie job -oozie http://localhost:11000/oozie -info 14-20090525161321-oozie-tucu
----------------------------------------------------------------------------------------------------------------------------------------------------------------
Workflow Name :  map-reduce-wf
App Path      :  hdfs://localhost:8020/user/tucu/examples/apps/map-reduce
Status        :  SUCCEEDED
Run           :  0
User          :  tucu
Group         :  users
Created       :  2009-05-26 05:01 +0000
Started       :  2009-05-26 05:01 +0000
Ended         :  2009-05-26 05:01 +0000
Actions
.----------------------------------------------------------------------------------------------------------------------------------------------------------------
Action Name             Type        Status     Transition  External Id            External Status  Error Code    Start Time              End Time
.----------------------------------------------------------------------------------------------------------------------------------------------------------------
mr-node                 map-reduce  OK         end         job_200904281535_0254  SUCCEEDED        -             2009-05-26 05:01 +0000  2009-05-26 05:01 +0000
.----------------------------------------------------------------------------------------------------------------------------------------------------------------
```



### 使用 Java 代码来提交 Oozie 任务

新建 maven 项目，填入以下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.oozie</groupId>
        <artifactId>oozie-client</artifactId>
        <version>5.2.0</version>
    </dependency>
</dependencies>
```

然后写代码：

```java
package work.xujiyou;

import org.apache.oozie.client.OozieClient;
import org.apache.oozie.client.OozieClientException;
import org.apache.oozie.client.WorkflowJob;

import java.util.Properties;

/**
 * Main class
 *
 * @author jiyouxu
 * @date 2019/12/23
 */
public class Main {

    public static void main(String[] args) throws OozieClientException, InterruptedException {
        OozieClient wc = new OozieClient("http://fueltank-2:11000/oozie");
        Properties conf = wc.createConfiguration();
        conf.setProperty(OozieClient.APP_PATH, "hdfs://fueltank-2.cloud.bbdops.com:8020/user/root/examples/apps/map-reduce/workflow.xml");
        conf.setProperty("resourceManager", "fueltank-2.cloud.bbdops.com:8032");
        conf.setProperty("inputDir", "/user/root/inputdir");
        conf.setProperty("outputDir", "/user/root/outputdir");
        String jobId = wc.run(conf);
        System.out.println("Workflow job submitted");
        while (wc.getJobInfo(jobId).getStatus() == WorkflowJob.Status.RUNNING) {
            System.out.println("Workflow job running ...");
            Thread.sleep(10 * 1000);
        }
        System.out.println("Workflow job completed ...");
        System.out.println(jobId);
    }
}
```

主要就是配置 workflow.xml 的地址。

### 本地运行 Oozie

为提高开发速度，可以考虑使用本地 Oozie 环境。

首先下载 Oozie 包，然后在包内执行：

```
./bin/mkdistro.sh -DskipTests
```

这句话用来编译 Oozie ，前提是需要安装 Maven，macos 直接 brew install maven 即可。

