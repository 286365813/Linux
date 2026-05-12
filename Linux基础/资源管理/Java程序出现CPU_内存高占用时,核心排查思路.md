当 Linux 下 Java 程序出现 CPU / 内存高占用时，核心排查思路是：先定位进程→再定位线程 / 内存问题→最后结合系统 / 业务逻辑溯源

# 1\. 定位高 CPU / 内存的 Java 进程、

用top或ps快速筛选目标进程，这是第一步：

```yaml
方法1：top交互式查看（按P排序CPU，按M排序内存，按c显示完整命令）
top -c

方法2：直接筛选Java进程（重点看%CPU、%MEM、PID列）
ps aux | grep java | grep -v grep
# 输出示例：USER  PID  %CPU %MEM  VSZ   RSS  TTY  STAT  START   TIME  COMMAND
#          root 1234  90.0 30.0 819200 49152 ?    Sl    10:00  1:20  java -jar app.jar

记录关键信息：PID（进程 ID）、%CPU/%MEM（占用率）、COMMAND（启动命令，确认是否是目标程序）。
```

# 2\. 定位进程内高 CPU 的线程（针对 CPU 高场景）

```yaml
1. 查看该Java进程下的所有线程（-Hp：显示线程，按P排序CPU）
top -Hp <Java进程PID>

2 记录高CPU的线程ID（TID列，十进制）

3. 将线程ID转为16进制（jstack日志中线程ID是16进制）
printf "%x\n" <高CPU线程TID>
# 示例：输入1234 → 输出4d2（注意小写，jstack中是0x4d2格式）

4. 导出 jstack
jstack <Java进程PID> > /tmp/jstack.log

5. 查找对应线程
grep -A 20 "0x303a" /tmp/jstack.log
grep -i "303a" /tmp/jstack.log
```

# 内存 &GC 分析（排查内存高场景）

```yaml
1. top / ps 看内存占用
        ↓
2. jstat 看GC状态
        ↓
3. jmap 看堆内存
        ↓
4. dump堆快照
        ↓
5. MAT分析内存泄漏
```

内存高可能是堆配置不足、内存泄漏、GC 频繁导致，用jstat+ jmap分析：

```yaml
1. jstat -gc <Java进程PID> 1000 10
# 重点：FGC频繁（比如每秒几次）、OU接近OC → 老年代满，内存泄漏/堆配置小

2. 导出堆内存快照（分析内存泄漏）
jmap -dump:format=b,file=heap_dump_$(date +%Y%m%d).hprof <Java进程PID>
# 用MAT（Memory Analyzer Tool）打开hprof文件，分析：
# - 大对象（比如超大List/Map）、重复对象；
# - 未释放的资源（比如数据库连接、线程池、定时任务实例）。
```