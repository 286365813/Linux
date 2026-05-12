logrotate 程序是一个日志文件管理工具。用来把旧的日志文件删除，并创建新的日志文件，称为日志转 储或滚动。可以根据日志文件的大小，也可以根据其天数来转储，这个过程一般通过 cron 程序来执行

# 常用的配置文件

```markdown

[root@localhost ~]# rpm -qc logrotate
/etc/cron.daily/logrotate      # 计划任务
/etc/logrotate.conf            # 配置文件
/etc/rwtab.d/logrotate

```

# 配置参数

| 参数 | 描述 |
| --- | --- |
| `compress` | 启用日志文件压缩，通常使用 `gzip` 格式压缩日志文件。 |
| `delaycompress` | 延迟压缩，通常与 `compress` 一起使用，表示在下次轮换时才进行压缩。 |
| `daily` | 每天轮换日志文件。 |
| `weekly` | 每周轮换日志文件。 |
| `monthly` | 每月轮换日志文件。 |
| `size` | 根据指定的大小轮换日志文件，例如 `size 100M` 表示当日志文件大小达到 100MB 时进行轮换。 |
| `rotate` | 设置要保留的旧日志文件的数量，例如 `rotate 5` 表示保留最近的 5 个日志文件。 |
| `missingok` | 如果日志文件不存在时不报错。 |
| `notifempty` | 如果日志文件为空则不进行轮换。 |
| `create` | 设置新日志文件的权限、用户和组，例如 `create 0644 root root`。 |
| `dateext` | 启用日期扩展，在备份文件名中加上日期后缀（如 `.2025-02-23`）。 |
| `copytruncate` | 在轮换日志时先复制文件，然后截断原文件，适用于不能被直接删除的日志文件。 |
| `postrotate` 和 `prerotate` | 分别在日志轮换前或后执行的脚本或命令。 |
| `maxage` | 限制日志文件的最大年龄，超过指定天数的日志文件将被删除。 |
| `dateformat` | 定制日期格式，用于日志文件名的日期部分。 |
| `sharedscripts` | 如果多个日志文件一起轮换，指定只执行一次 `postrotate` 或 `prerotate` 脚本。 |

这些是 `logrotate` 配置文件中常用的一些参数，可以根据需要灵活组合使用来控制日志文件的轮换行为。

## logroate 配置范例

范例： 设置nginx的日志转储

```bash
[dwlitao7@a64-dg-yhjd-1054 ~]$ sudo vim /etc/logrotate.d/nginx 
/usr/local/nginx/logs/*.log {
    daily             
    rotate 180
    create 644 hcaiyun hcaiyun
    notifempty
    compress
    dateext
    dateyesterday
    dateformat -%Y%m%d
    sharedscripts
    postrotate
        if [ -f /usr/local/nginx/logs/nginx.pid ];then
            kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
        fi
    endscript
}
```

可以手动运行 `logrotate` 来测试配置文件是否正确：

```bash
[root@nginx ~]# sudo logrotate -d /etc/logrotate.d/nginx
```

​  

如果要强制执行轮转，可以使用：

```bash
[root@nginx ~]# sudo logrotate -f /etc/logrotate.d/nginx
```