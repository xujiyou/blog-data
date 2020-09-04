# MySQL 配置

## 设置时区

```mysql
mysql> set time_zone='+8:00';
mysql> set global time_zone='+8:00';
mysql> flush privileges;
mysql> show variables like "%time_zone%";
mysql> select now();
```

