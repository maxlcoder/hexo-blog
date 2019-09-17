---
title: CGI FastCGI PHP-CGI PHP-FPM
date: 2019-09-17 09:28:42
tags:
---

1. CGI
CGI（Common Gateway Interface）全称是“通用网关接口”，WEB 服务器与PHP应用进行“交谈”的一种工具，其程序须运行在网络服务器上。CGI可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。如php、perl、tcl等。

WEB服务器会传哪些数据给PHP解析器呢？URL、查询字符串、POST数据、HTTP header都会有。所以，CGI就是规定要传哪些数据，以什么样的格式传递给后方处理这个请求的协议。仔细想想，你在PHP代码中使用的用户从哪里来的。

也就是说，CGI就是专门用来和 web 服务器打交道的。web服务器收到用户请求，就会把请求提交给cgi程序（如php-cgi），cgi程序根据请求提交的参数作应处理（解析php），然后输出标准的html语句，返回给web服服务器，WEB服务器再返回给客户端，这就是普通cgi的工作原理。

CGI的好处就是完全独立于任何服务器，仅仅是做为中间分子。提供接口给apache和php。他们通过cgi搭线来完成数据传递。这样做的好处了尽量减少2个的关联，使他们2变得更独立。

但是CGI有个蛋疼的地方，就是每一次web请求都会有启动和退出过程，也就是最为人诟病的fork-and-execute模式，这样一在大规模并发下，就死翘翘了。

2. FastCGI介绍
FastCGI简单介绍
从根本上来说，FastCGI是用来提高CGI程序性能的。类似于CGI，FastCGI也可以说是一种协议。

FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一次。它还支持分布式的运算, 即 FastCGI 程序可以在网站服务器以外的主机上执行，并且接受来自其它网站服务器来的请求。

FastCGI是语言无关的、可伸缩架构的CGI开放扩展，其主要行为是将CGI解释器进程保持在内存中，并因此获得较高的性能。众所周知，CGI解释器的反复加载是CGI性能低下的主要原因，如果CGI解释器保持在内存中，并接受FastCGI进程管理器调度，则可以提供良好的性能、伸缩性、Fail- Over特性等等。

3. PHP-FPM介绍
要了解PHP-FPM，就得先说说PHP-CGI。

PHP-CGI就是PHP实现的自带的FastCGI管理器。 虽然是php官方出品，但是这丫的却一点也不给力，性能太差，而且也很麻烦不人性化，主要体现在：

php-cgi变更php.ini配置后，需重启php-cgi才能让新的php-ini生效，不可以平滑重启。
直接杀死php-cgi进程，php就不能运行了。
上面2个问题，一直让很多人病垢了很久，所以很多人一直还是在用 Module 方式。 直到 2004年一个叫 Andrei Nigmatulin的屌丝发明了PHP-FPM ，这神器的出现就彻底打破了这种局面，这是一个PHP专用的 fastcgi 管理器，它很爽的克服了上面2个问题，而且，还表现在其他方面更表现强劲。

也就是说，PHP-FPM 是对于 FastCGI 协议的具体实现，他负责管理一个进程池，来处理来自Web服务器的请求。目前，PHP5.3版本之后，PHP-FPM是内置于PHP的。

因为PHP-CGI只是个CGI程序，他自己本身只能解析请求，返回结果，不会进程管理。所以就出现了一些能够调度 php-cgi 进程的程序，比如说由lighthttpd分离出来的spawn-fcgi。同样，PHP-FPM也是用于调度管理PHP解析器php-cgi的管理程序。

PHP-FPM通过生成新的子进程可以实现php.ini修改后的平滑重启。

