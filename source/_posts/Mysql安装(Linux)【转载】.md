---
title: Mysql安装(Linux)【转载】
date: 2020-09-11 23:13:40
categories:
- 技术博客
tags:
- Mysql
- Linux

---



转载内容：

[Linux下安装mysql-5.7.24](https://www.jianshu.com/p/276d59cbc529)



其中，在安装完成之后，即执行`service mysql restart`后再执行`mysql`时，若报以下错误时，可通过执行`yum install libncurses*`解决。

```err
mysql: error while loading shared libraries: libncurses.so.5: cannot open shared object file: No such file or directory
```



附带作者自己之前设置的mysql配置文件(my.cnf)：

```cnf
[client]
port=3357
default-character-set=utf8

[mysql]
default-character-set=gbk

[mysqld]
max_connections=200
default-storage-engine=INNODB
datadir=/usr/local/mysql/data
port=3357
character-set-server=utf8
default-storage-engine=INNODB

sql-mode="STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"



query_cache_size=0

table_open_cache=256


tmp_table_size=18M



thread_cache_size=8

myisam_max_sort_file_size=100G

myisam_sort_buffer_size=35M

key_buffer_size=25M

read_buffer_size=64K
read_rnd_buffer_size=256K

sort_buffer_size=256K

# innodb_additional_mem_pool_size=2M    --注释掉了
tmpdir="/usr/local/mysql/data"  # 新增的

innodb_flush_log_at_trx_commit=1

innodb_log_buffer_size=1M

innodb_buffer_pool_size=47M

innodb_log_file_size=24M

innodb_thread_concurrency=18
```

