# Java 连接 Hive

Java 连接 Hive，就跟使用JDBC一样。

首先添加依赖：

```xml
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-jdbc</artifactId>
    <version>2.1.1</version>
</dependency>
```

然后写代码：

```java
package work.xujiyou;

import java.sql.*;

/**
 * Main class
 *
 * @author jiyouxu
 * @date 2019/12/26
 */
public class Main {

    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        Class.forName("org.apache.hive.jdbc.HiveDriver");
        Connection conn = DriverManager.getConnection("jdbc:hive2://fueltank-2:10000/default","","");
        PreparedStatement ps = conn.prepareStatement("select * from employee");
        ResultSet rs = ps.executeQuery();
        int columns = rs.getMetaData().getColumnCount();
        while(rs.next())
        {
            for(int i = 1; i <= columns; i++)
            {
                System.out.print(rs.getString(i));
                System.out.print("\t\t");
            }
            System.out.println();
        }
    }
}
```

这样就可以了。

Hive 的数据是存在 HDFS 中的，如：

```bash
$ hdfs dfs -ls /user/hive/warehouse
$ hdfs dfs -cat /user/hive/warehouse/employee/000000_0
```

跟JDBC中查出来的数据一毛一样。