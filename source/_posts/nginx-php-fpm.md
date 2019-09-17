---
title: nginx php-fpm
date: 2019-09-17 10:23:15
tags:
---

1. nginx worker 数指定为 CPU核心数的2倍

PHP-fpm
PHP-FPM是一个PHPFastCGI管理器，是只用于PHP的。
php-fpm 已经在 Linux、MacOSX、Solaris 和 FreeBSD 上测试通过。
确信 libxml2（在某些系统上叫做libxml2-devel）已经安装。

关于信号处理
SIGINT, SIGTERM	立刻终止
SIGQUIT	平滑终止
SIGUSR1	重新打开日志文件
SIGUSR2	平滑重载所有worker进程并重新载入配置和二进制模
参数调优
进程数
首先，我们关注一个前提设置： pm = static/dynamic,标识fpm子进程的产生模式

static(静态) ：表示在fpm运行时直接fork出pm.max_chindren个worker进程

dynamic(动态)：表示运行时fork出start_servers个进程，随着负载的情况，动态的调整，最多不超过max_children个进程。

一般推荐用static，优点是不用动态的判断负载情况，提升性能，缺点是多占用些系统内存资源。

static:worker进程	pm.max_children = 300	这个值原则上是越大越好
dynamic:worker进程	pm.start_servers = 20	
dynamic:空闲状态	pm.min_spare_servers = 5	最小php-fpm进程数量
dynamic:空闲状态	pm.max_spare_servers = 35	最大php-fpm进程数量
max_children

这个值原则上是越大越好，php-cgi的进程多了就会处理的很快，排队的请求就会很少。

设置”max_children”也需要根据服务器的性能进行设定

一般来说一台服务器正常情况下每一个php-cgi所耗费的内存在20M左右

假设“max_children”设置成100个，20M*100=2000M

也就是说在峰值的时候所有PHP-CGI所耗内存在2000M以内。

假设“max_children”设置的较小，比如5-10个，那么php-cgi就会“很累”，处理速度也很慢，等待的时间也较长。

如果长时间没有得到处理的请求就会出现504 Gateway Time-out这个错误，而正在处理的很累的那几个php-cgi如果遇到了问题就会出现502 Bad gateway这个错误。

start_servers

pm.start_servers的默认值为2。并且php-fpm中给的计算方式也为：
{（cpu空闲时等待连接的php的最小子进程数） + （cpu空闲时等待连接的php的最大子进程数 - cpu空闲时等待连接的php的最小子进程数）/ 2}；
用配置表示就是：min_spare_servers + (max_spare_servers - min_spare_servers) / 2；
一般而言，设置成10-20之间的数据足够满足需求了。
最大请求数max_requests
最大处理请求数是指一个php-fpm的worker进程在处理多少个请求后就终止掉，master进程会重新respawn一个新的。
这个配置的主要目的是避免php解释器或程序引用的第三方库造成的内存泄露。
pm.max_requests = 10240

当一个 PHP-CGI 进程处理的请求数累积到 max_requests 个后，自动重启该进程。

502，是后端 PHP-FPM 不可用造成的，间歇性的502一般认为是由于 PHP-FPM 进程重启造成的.

但是为什么要重启进程呢？

如果不定期重启 PHP-CGI 进程，势必造成内存使用量不断增长（比如第三方库有问题等）。因此 PHP-FPM 作为 PHP-CGI 的管理器，提供了这么一项监控功能，对请求达到指定次数的 PHP-CGI 进程进行重启，保证内存使用量不增长。

正是因为这个机制，在高并发中，经常导致 502 错误

目前我们解决方案是把这个值尽量设置大些，减少 PHP-CGI 重新 SPAWN 的次数，同时也能提高总体性能。PS：刚开始我们是500导致内存飙高，现在改成5120，当然可以再大一些，10240等，这个主要看测试结果，如果没有内存泄漏等问题，可以再大一些。

最长执行时间request_terminate_timeout
max_execution_time和request_terminate_timeout

; The timeout for serving a single request after which the worker process will
; be killed. This option should be used when the ‘max_execution_time’ ini option
; does not stop script execution for some reason. A value of ‘0’ means ‘off’.
; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
; Default Value: 0
;request_terminate_timeout = 0
＝＝＝＝＝＝＝＝＝＝＝＝
设置单个请求的超时中止时间. 该选项可能会对php.ini设置中的’max_execution_time’因为某些特殊原因没有中止运行的脚本有用. 设置为 ‘0’ 表示 ‘Off’.当经常出现502错误时可以尝试更改此选项。

这两项都是用来配置一个PHP脚本的最大执行时间的。当超过这个时间时，PHP-FPM不只会终止脚本的执行，还会终止执行脚本的Worker进程。
Nginx会发现与自己通信的连接断掉了，就会返回给客户端502错误。
内存|CPU排查方法
top
命令格式：

top [-] [d] [p] [q] [c] [C] [S]    [n] 
参数说明：
d： 指定每两次屏幕信息刷新之间的时间间隔。当然用户可以使用s交互命令来改变之。
p： 通过指定监控进程ID来仅仅监控某个进程的状态。
q：该选项将使top没有任何延迟的进行刷新。如果调用程序有超级用户权限，那么top将以尽可能高的优先级运行。
S： 指定累计模式
s ： 使top命令在安全模式中运行。这将去除交互命令所带来的潜在危险。
i： 使top不显示任何闲置或者僵死进程。、
m：切换显示内存信息。
t：切换显示进程和CPU状态信息。
c：切换显示命令名称和完整命令行。
M： 根据驻留内存大小进行排序。
P：根据CPU使用百分比大小进行排序。
T：根据时间/累计时间进行排序。

sar
执行sar -P ALL 1 100。-P ALL表示监控所有核心，1表示每1秒采集，100表示采集100次。

开启慢日志
配置输出php-fpm慢日志，阀值为2秒：

request_slowlog_timeout = 2
slowlog = log/$pool.log.slow
利用sort/uniq命令分析汇总php-fpm慢日志：

grep -v “^$” www.log.slow.tmp | cut -d ” ” -f 3,2 | sort | uniq -c | sort -k1,1nr | head -n 50

参数解释：
sort: 对单词进行排序
uniq -c: 显示唯一的行，并在每行行首加上本行在文件中出现的次数
sort -k1,1nr: 按照第一个字段，数值排序，且为逆序
head -10: 取前10行数据

用strace跟踪进程
利用nohup将strace转为后台执行，直到attach上的php-fpm进程死掉为止：
nohup strace -T -p 13167 > 13167-strace.log &
参数说明:
-c 统计每一系统调用的所执行的时间,次数和出错的次数等.
-d 输出strace关于标准错误的调试信息.
-f 跟踪由fork调用所产生的子进程.
-o filename,则所有进程的跟踪结果输出到相应的filename
-F 尝试跟踪vfork调用.在-f时,vfork不被跟踪.
-h 输出简要的帮助信息.
-i 输出系统调用的入口指针.
-q 禁止输出关于脱离的消息.
-r 打印出相对时间关于,,每一个系统调用.
-t 在输出中的每一行前加上时间信息.
-tt 在输出中的每一行前加上时间信息,微秒级.
-ttt 微秒级输出,以秒了表示时间.
-T 显示每一调用所耗的时间.
-v 输出所有的系统调用.一些调用关于环境变量,状态,输入输出等调用由于使用频繁,默认不输出.
-V 输出strace的版本信息.
-x 以十六进制形式输出非标准字符串
-xx 所有字符串以十六进制形式输出.
-a column
设置返回值的输出位置.默认为40.
-e execve 只记录 execve 这类系统调用
-p 主进程号
也可以用利用-c参数让strace帮助汇总
strace -cp pid


nginx配置

正向代理代理的对象是客户端，反向代理代理的对象是服务端

正向典型案例 VPN 
反向典型案例 负载均衡，隐藏服务器

1. nginx与fastcgi通信的方式 Unix soket 后者 tcp, TODO 优缺比较 socket比较快 ，但是不能跨服务器，TCP高并发下比较稳定
2. 图片，js，css等文件缓存
3. 对图片，js，css等gzip压缩配置
4. 最大连接数 
5. 连接超时设置
6. 限制上传文件大小
7. 配置防盗链

nginx 负载均衡的配置

```
http {
    # 定义 upstream
    upstream test.com {
        server 192.168.0.1;
        server 192.168.0.2;
    }

    # 设置反向代理
    server {
        listen       80;
        server_name  test.com;
        location / {
            proxy_pass   http://test.com;
            proxy_set_header        Host    $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

    

}

# 其他服务器配置
http{
   

    server {
        listen       80;
        listen       [::]:80;
        server_name  test.com;
        index index.php index.html index.htm;
        root         /usr/share/nginx/html/test;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
        location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
}
```

mysql 配置优化

配置分类 

1. 数量，连接数，最大xx数量

2. 内存，占用内存大小

3. 时间，超时时间等

4. 事务，事务隔离级别

5. 基本配置，默认存储引擎，日志配置，数据存储位置等

配置优化注意事项

1. 重点首先要清楚，每项配置针对点是什么，例如某项内存配置是适配整体的，还是针对单个连接的，要考虑内存乘积结果对整个服务器的内存的影响

2. 更新配置需要针对业务痛点进行相应的调整，一般来说默认配置能够保持最优的稳定性，针对性的调整能够在维持稳定性的同时不断的提升性能

3. 优化项，需经过相应的模拟和压力测试之后才可以更新到线上

```
[client]
port = 3306  
socket = /var/lib/mysql/mysql.sock

[mysql]
#这个配置段设置启动MySQL服务的条件；在这种情况下，no-auto-rehash确保这个服务启动得比较快。
no-auto-rehash

[mysqld]
user = mysql  
port = 3306  
socket = /var/lib/mysql/mysql.sock  
basedir = /usr/local/mysql  
datadir = /data/mysql/data/  
open_files_limit = 10240

back_log = 600  
#在MYSQL暂时停止响应新请求之前，短时间内的多少个请求可以被存在堆栈中。如果系统在短时间内有很多连接，则需要增大该参数的值，该参数值指定到来的TCP/IP连接的监听队列的大小。默认值80。

max_connections = 3000  
#MySQL允许最大的进程连接数，如果经常出现Too Many Connections的错误提示，则需要增大此值。默认151

max_connect_errors = 6000  
#设置每个主机的连接请求异常中断的最大次数，当超过该次数，MYSQL服务器将禁止host的连接请求，直到mysql服务器重启或通过flush hosts命令清空此host的相关信息。默认100

external-locking = FALSE  
#使用–skip-external-locking MySQL选项以避免外部锁定。该选项默认开启

max_allowed_packet = 32M  
#设置在网络传输中一次消息传输量的最大值。系统默认值 为4MB，最大值是1GB，必须设置1024的倍数。

#sort_buffer_size = 2M  
# Sort_Buffer_Size 是一个connection级参数，在每个connection（session）第一次需要使用这个buffer的时候，一次性分配设置的内存。
#Sort_Buffer_Size 并不是越大越好，由于是connection级的参数，过大的设置+高并发可能会耗尽系统内存资源。例如：500个连接将会消耗 500*sort_buffer_size(8M)=4G内存
#Sort_Buffer_Size 超过2KB的时候，就会使用mmap() 而不是 malloc() 来进行内存分配，导致效率降低。 系统默认2M，使用默认值即可

#join_buffer_size = 2M  
#用于表间关联缓存的大小，和sort_buffer_size一样，该参数对应的分配内存也是每个连接独享。系统默认2M，使用默认值即可

thread_cache_size = 300  
#默认38
# 服务器线程缓存这个值表示可以重新利用保存在缓存中线程的数量,当断开连接时如果缓存中还有空间,那么客户端的线程将被放到缓存中,如果线程重新被请求，那么请求将从缓存中读取,如果缓存中是空的或者是新的请求，那么这个线程将被重新创建,如果有很多新的线程，增加这个值可以改善系统性能.通过比较 Connections 和 Threads_created 状态的变量，可以看到这个变量的作用。设置规则如下：1GB 内存配置为8，2GB配置为16，3GB配置为32，4GB或更高内存，可配置更大。

#thread_concurrency = 8  
#系统默认为10，使用10先观察
# 设置thread_concurrency的值的正确与否, 对mysql的性能影响很大, 在多个cpu(或多核)的情况下，错误设置了thread_concurrency的值, 会导致mysql不能充分利用多cpu(或多核), 出现同一时刻只能一个cpu(或核)在工作的情况。thread_concurrency应设为CPU核数的2倍. 比如有一个双核的CPU, 那么thread_concurrency的应该为4; 2个双核的cpu, thread_concurrency的值应为8

query_cache_size = 64M  
#在MyISAM引擎优化中，这个参数也是一个重要的优化参数。但也爆露出来一些问题。机器的内存越来越大，习惯性把参数分配的值越来越大。这个参数加大后也引发了一系列问题。我们首先分析一下 query_cache_size的工作原理：一个SELECT查询在DB中工作后，DB会把该语句缓存下来，当同样的一个SQL再次来到DB里调用时，DB在该表没发生变化的情况下把结果从缓存中返回给Client。这里有一个关建点，就是DB在利用Query_cache工作时，要求该语句涉及的表在这段时间内没有发生变更。那如果该表在发生变更时，Query_cache里的数据又怎么处理呢？首先要把Query_cache和该表相关的语句全部置为失效，然后在写入更新。那么如果Query_cache非常大，该表的查询结构又比较多，查询语句失效也慢，一个更新或是Insert就会很慢，这样看到的就是Update或是Insert怎么这么慢了。所以在数据库写入量或是更新量也比较大的系统，该参数不适合分配过大。而且在高并发，写入量大的系统，建议把该功能禁掉。

query_cache_limit = 4M  
#指定单个查询能够使用的缓冲区大小，缺省为1M

query_cache_min_res_unit = 2k  
#默认是4KB，设置值大对大数据查询有好处，但如果你的查询都是小数据查询，就容易造成内存碎片和浪费
#查询缓存碎片率 = Qcache_free_blocks / Qcache_total_blocks * 100%
#如果查询缓存碎片率超过20%，可以用FLUSH QUERY CACHE整理缓存碎片，或者试试减小query_cache_min_res_unit，如果你的查询都是小数据量的话。
#查询缓存利用率 = (query_cache_size – Qcache_free_memory) / query_cache_size * 100%
#查询缓存利用率在25%以下的话说明query_cache_size设置的过大，可适当减小;查询缓存利用率在80%以上而且Qcache_lowmem_prunes > 50的话说明query_cache_size可能有点小，要不就是碎片太多。
#查询缓存命中率 = (Qcache_hits – Qcache_inserts) / Qcache_hits * 100%

#default-storage-engine = MyISAM
#default_table_type = InnoDB #开启失败

#thread_stack = 192K  
#设置MYSQL每个线程的堆栈大小，默认值足够大，可满足普通操作。可设置范围为128K至4GB，默认为256KB，使用默认观察

transaction_isolation = READ-COMMITTED  
# 设定默认的事务隔离级别.可用的级别如下:READ UNCOMMITTED-读未提交 READ COMMITTE-读已提交 REPEATABLE READ -可重复读 SERIALIZABLE -串行

tmp_table_size = 256M  
# tmp_table_size 的默认大小是 32M。如果一张临时表超出该大小，MySQL产生一个 The table tbl_name is full 形式的错误，如果你做很多高级 GROUP BY 查询，增加 tmp_table_size 值。如果超过该值，则会将临时表写入磁盘。
max_heap_table_size = 256M

expire_logs_days = 7  
key_buffer_size = 2048M  
#批定用于索引的缓冲区大小，增加它可以得到更好的索引处理性能，对于内存在4GB左右的服务器来说，该参数可设置为256MB或384MB。

read_buffer_size = 1M  
#默认128K
# MySql读入缓冲区大小。对表进行顺序扫描的请求将分配一个读入缓冲区，MySql会为它分配一段内存缓冲区。read_buffer_size变量控制这一缓冲区的大小。如果对表的顺序扫描请求非常频繁，并且你认为频繁扫描进行得太慢，可以通过增加该变量值以及内存缓冲区大小提高其性能。和sort_buffer_size一样，该参数对应的分配内存也是每个连接独享。

read_rnd_buffer_size = 16M  
# MySql的随机读（查询操作）缓冲区大小。当按任意顺序读取行时(例如，按照排序顺序)，将分配一个随机读缓存区。进行排序查询时，MySql会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。但MySql会为每个客户连接发放该缓冲空间，所以应尽量适当设置该值，以避免内存开销过大。

bulk_insert_buffer_size = 64M  
#批量插入数据缓存大小，可以有效提高插入效率，默认为8M

myisam_sort_buffer_size = 128M  
# MyISAM表发生变化时重新排序所需的缓冲 默认8M

myisam_max_sort_file_size = 10G  
# MySQL重建索引时所允许的最大临时文件的大小 (当 REPAIR, ALTER TABLE 或者 LOAD DATA INFILE).
# 如果文件大小比此值更大,索引会通过键值缓冲创建(更慢)

#myisam_max_extra_sort_file_size = 10G 5.6无此值设置
#myisam_repair_threads = 1   默认为1
# 如果一个表拥有超过一个索引, MyISAM 可以通过并行排序使用超过一个线程去修复他们.
# 这对于拥有多个CPU以及大量内存情况的用户,是一个很好的选择.

myisam_recover  
#自动检查和修复没有适当关闭的 MyISAM 表
skip-name-resolve  
lower_case_table_names = 1  
server-id = 1

innodb_additional_mem_pool_size = 16M  
#这个参数用来设置 InnoDB 存储的数据目录信息和其它内部数据结构的内存池大小，类似于Oracle的library cache。这不是一个强制参数，可以被突破。

innodb_buffer_pool_size = 2048M  
# 这对Innodb表来说非常重要。Innodb相比MyISAM表对缓冲更为敏感。MyISAM可以在默认的 key_buffer_size 设置下运行的可以，然而Innodb在默认的 innodb_buffer_pool_size 设置下却跟蜗牛似的。由于Innodb把数据和索引都缓存起来，无需留给操作系统太多的内存，因此如果只需要用Innodb的话则可以设置它高达 70-80% 的可用内存。一些应用于 key_buffer 的规则有 — 如果你的数据量不大，并且不会暴增，那么无需把 innodb_buffer_pool_size 设置的太大了

#innodb_data_file_path = ibdata1:1024M:autoextend 设置过大导致报错，默认12M观察
#表空间文件 重要数据

#innodb_file_io_threads = 4   不明确，使用默认值
#文件IO的线程数，一般为 4，但是在 Windows 下，可以设置得较大。


innodb_thread_concurrency = 8  
#服务器有几个CPU就设置为几，建议用默认设置，一般为8.

innodb_flush_log_at_trx_commit = 2  
# 如果将此参数设置为1，将在每次提交事务后将日志写入磁盘。为提供性能，可以设置为0或2，但要承担在发生故障时丢失数据的风险。设置为0表示事务日志写入日志文件，而日志文件每秒刷新到磁盘一次。设置为2表示事务日志将在提交时写入日志，但日志文件每次刷新到磁盘一次。

#innodb_log_buffer_size = 16M   使用默认8M
#此参数确定些日志文件所用的内存大小，以M为单位。缓冲区更大能提高性能，但意外的故障将会丢失数据.MySQL开发人员建议设置为1－8M之间

#innodb_log_file_size = 128M  使用默认48M
#此参数确定数据日志文件的大小，以M为单位，更大的设置可以提高性能，但也会增加恢复故障数据库所需的时间

#innodb_log_files_in_group = 3   使用默认2
#为提高性能，MySQL可以以循环方式将日志文件写到多个文件。推荐设置为3M

#innodb_max_dirty_pages_pct = 90  使用默认75观察
#推荐阅读 http://www.taobaodba.com/html/221_innodb_max_dirty_pages_pct_checkpoint.html
# Buffer_Pool中Dirty_Page所占的数量，直接影响InnoDB的关闭时间。参数innodb_max_dirty_pages_pct 可以直接控制了Dirty_Page在Buffer_Pool中所占的比率，而且幸运的是innodb_max_dirty_pages_pct是可以动态改变的。所以，在关闭InnoDB之前先将innodb_max_dirty_pages_pct调小，强制数据块Flush一段时间，则能够大大缩短 MySQL关闭的时间。

innodb_lock_wait_timeout = 120  
#默认为50秒 
# InnoDB 有其内置的死锁检测机制，能导致未完成的事务回滚。但是，如果结合InnoDB使用MyISAM的lock tables 语句或第三方事务引擎,则InnoDB无法识别死锁。为消除这种可能性，可以将innodb_lock_wait_timeout设置为一个整数值，指示 MySQL在允许其他事务修改那些最终受事务回滚的数据之前要等待多长时间(秒数)

innodb_file_per_table = 0  
#默认为No
#独享表空间（关闭）

[mysqldump]
quick  
# max_allowed_packet = 32M

[mysqld_safe]
log-error=/data/mysql/mysql_oldboy.err  
pid-file=/data/mysql/mysqld.pid

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
```