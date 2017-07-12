## MariaDB 主从复制出错，重新开始同步

主库在某次因故挂掉，由于同步使用了基于 GTID（全局事务 ID）的方式，因此在主库恢复后从库数据和主库数据出现大量不对照的地方。

每一次从库的每条 GTID 执行失败都会导致整个同步过程停止，因此我们需要从主库重新拉一份完整的数据替换掉目前从库的数据，并从最后的 end_log_pos（最后的同步）位置开始进行同步。

关于备份，我们使用了 Percona 的 innobackupex 工具。

### 从主库拉数据到远程机器

> 参考文档：https://www.percona.com/doc/percona-xtrabackup/LATEST/innobackupex/streaming_backups_innobackupex.html#examples-using-xbstream

在主库所在机器上执行如下命令：

```
/usr/bin/innobackupex --user=$USER --password=$PASS --host=127.0.0.1 --stream=xbstream --compress ./ | ssh root@192.168.0.66 "xbstream -x -C /data/mysql_copy"
```

**命令详解：**

使用 innobackupex 把本机的数据库压缩后以 xbstream 的 stream 流的方式，通过 ssh 链接远端 B（192.168.0.66），发送到远端的 /data/mysql_copy 目录上

_根据数据库大小和网络原因，时间会比较久_

### 在远程机对备份进行解压缩

> 参考文档：https://www.percona.com/doc/percona-xtrabackup/2.1/howtos/recipes_ibkx_compressed.html#preparing-the-backup

我们首先登陆远端 B 机器，执行如下命令：

```
/usr/bin/innobackupex --decompress /data/mysql_copy 
```

**我们为什么要执行这个命令？** 因为目前我们备份过来的数据库现在是 *.qp 的压缩状态，需要通过这个来进行解压缩。
<br><br>

在解压缩之后，目前我们的数据目录仍然还不是可运行的数据目录，需要再执行如下命令：

```
/usr/bin/innobackupex --apply-log /data/mysql_copy
```

> 参考文档：https://www.percona.com/doc/percona-xtrabackup/2.1/howtos/recipes_ibkx_gtid.html#step-2-prepare-the-backup

> 关于 apply-log 参数及 innobackupex 的用法，可参见文档：
> https://www.percona.com/doc/percona-xtrabackup/LATEST/innobackupex/innobackupex_option_reference.html#cmdoption-innobackupex--apply-log

## 恢复同步

目前，我们的 /data/mysql_copy 目录已经是从主库拉过来的一份完整的数据了，是一份可运行的数据了。

那么，我们就要开始恢复从库的同步了。

```shell
$ cd /data/mysql_copy
$ cat xtrabackup_binlog_info
mariadb-bin.000023	73228	0-13-506268
(上述分别代表：当前数据库同步的 MASTER 日志文件是哪个、日志的最后位置、同步的 GTID)
```



现在，我们启动 SLAVE（从库）的 Server 端，指定数据目录是 /data/mysql_copy，然后链接从库，假设主库的地址为：192.168.0.33

(注: 目前从库的 Server 端虽然启动了，但是同步并没有开始，要搞清楚它们不是一个概念)

```bash
$ mysql -uroot -p123
MariaDB> CHANGE MASTER TO MASTER_HOST="192.168.0.33", MASTER_USER="root", MASTER_PASSWORD="123456", MASTER_LOG_FILE='mariadb-bin.000023', MASTER_LOG_POS=73228;
MariaDB> START SLAVE;
MariaDB> SHOW SLAVE STATUS \G
```

如此查看到从库的 IO 状态为 Yes 的话，则表明从库已经正常。


### 尾记
本篇文章不介绍如何搭建主从同步复制，如果后来有些的话，会附加到篇末。不介绍详细的原理，因为笔者也不是很懂，后来如果有去了解的话，会写文章附加到篇末。
