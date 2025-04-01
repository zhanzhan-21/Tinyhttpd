A mirror for tinyhttpd(Tinyhttpd非官方镜像,Fork自[sourceForge](https://sourceforge.net/projects/tiny-httpd/),仅供学习)

测试CGI时需要本机安装PERL，同时安装perl-cgi

### Prepare 
Compile for Linux
```
 To compile for Linux:
  1) Comment out the #include <pthread.h> line.
  2) Comment out the line that defines the variable newthread.
  3) Comment out the two lines that run pthread_create().
  4) Uncomment the line that runs accept_request().
  5) Remove -lsocket from the Makefile.
```

<p>&nbsp; &nbsp; &nbsp;每个函数的作用：</p>
<p>&nbsp; &nbsp; &nbsp;accept_request: &nbsp;处理从套接字上监听到的一个 HTTP 请求，在这里可以很大一部分地体现服务器处理请求流程。</p>
<p>&nbsp; &nbsp; &nbsp;bad_request: 返回给客户端这是个错误请求，HTTP 状态吗 400 BAD REQUEST.</p>
<p>&nbsp; &nbsp; &nbsp;cat: 读取服务器上某个文件写到 socket 套接字。</p>
<p>&nbsp; &nbsp; &nbsp;cannot_execute: 主要处理发生在执行 cgi 程序时出现的错误。</p>
<p>&nbsp; &nbsp; &nbsp;error_die: 把错误信息写到 perror 并退出。</p>
<p>&nbsp; &nbsp; &nbsp;execute_cgi: 运行 cgi 程序的处理，也是个主要函数。</p>
<p>&nbsp; &nbsp; &nbsp;get_line: 读取套接字的一行，把回车换行等情况都统一为换行符结束。</p>
<p>&nbsp; &nbsp; &nbsp;headers: 把 HTTP 响应的头部写到套接字。</p>
<p>&nbsp; &nbsp; &nbsp;not_found: 主要处理找不到请求的文件时的情况。</p>
<p>&nbsp; &nbsp; &nbsp;sever_file: 调用 cat 把服务器文件返回给浏览器。</p>
<p>&nbsp; &nbsp; &nbsp;startup: 初始化 httpd 服务，包括建立套接字，绑定端口，进行监听等。</p>
<p>&nbsp; &nbsp; &nbsp;unimplemented: 返回给浏览器表明收到的 HTTP 请求所用的 method 不被支持。</p>
<p><br>
</p>
<p>&nbsp; &nbsp; &nbsp;建议源码阅读顺序： main -&gt; startup -&gt; accept_request -&gt; execute_cgi, 通晓主要工作流程后再仔细把每个函数的源码看一看。</p>
<p><br>
</p>
<h4>&nbsp; &nbsp; &nbsp;工作流程</h4>
<p>&nbsp; &nbsp; &nbsp;（1） 服务器启动：在指定端口或随机选取端口绑定 HTTP 服务。</p>
<p>&nbsp; &nbsp; &nbsp;（2） 接收请求：当接收到一个 HTTP 请求时，派生一个线程运行 accept_request 函数。</p>
<p>&nbsp; &nbsp; &nbsp;（3） 解析请求：提取 HTTP 请求中的方法（GET 或 POST）和 URL。对于 GET 方法，如果有携带参数，则 query_string 指针指向 URL 中 ? 后面的参数。</p>
<p>&nbsp; &nbsp; &nbsp;（4） 格式化路径：将 URL 格式化为服务器文件路径，默认在 htdocs 文件夹下。如果 URL 以 / 结尾或指向一个目录，则默认添加 index.html。</p>
<p>&nbsp; &nbsp; &nbsp;（5） 处理请求：</p>
<p>&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;如果文件路径合法且为无参数的 GET 请求，直接将服务器文件发送给客户端。</p>
<p>&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;其他情况（带参数 GET、POST 方式、URL 为可执行文件），调用 execute_cgi 函数执行 CGI 脚本。</p>
<p>&nbsp; &nbsp; &nbsp;（6） 执行 CGI 脚本：</p>
<p>&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;读取整个 HTTP 请求并丢弃，如果是 POST 请求则找出 Content-Length。</p>
<p>&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;建立两个管道 cgi_input 和 cgi_output，并创建一个子进程。</p>
<p>&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;在子进程中，重定向标准输入输出，设置环境变量，然后执行 CGI 程序。</p>
<p>&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;在父进程中，处理 POST 数据，读取 CGI 程序的输出并发送给客户端。</p>
<p>&nbsp; &nbsp; &nbsp;（7） 关闭连接：关闭与客户端的连接，完成一次 HTTP 请求和响应。</p>
<p><br>
</p>

以下内容来自源作者:

  This software is copyright 1999 by J. David Blackstone.  Permission
is granted to redistribute and modify this software under the terms of
the GNU General Public License, available at http://www.gnu.org/ .

  If you use this software or examine the code, I would appreciate
knowing and would be overjoyed to hear about it at
jdavidb@sourceforge.net .

  This software is not production quality.  It comes with no warranty
of any kind, not even an implied warranty of fitness for a particular
purpose.  I am not responsible for the damage that will likely result
if you use this software on your computer system.

  I wrote this webserver for an assignment in my networking class in
1999.  We were told that at a bare minimum the server had to serve
pages, and told that we would get extra credit for doing "extras."
Perl had introduced me to a whole lot of UNIX functionality (I learned
sockets and fork from Perl!), and O'Reilly's lion book on UNIX system
calls plus O'Reilly's books on CGI and writing web clients in Perl got
me thinking and I realized I could make my webserver support CGI with
little trouble.

  Now, if you're a member of the Apache core group, you might not be
impressed.  But my professor was blown over.  Try the color.cgi sample
script and type in "chartreuse."  Made me seem smarter than I am, at
any rate. :)

  Apache it's not.  But I do hope that this program is a good
educational tool for those interested in http/socket programming, as
well as UNIX system calls.  (There's some textbook uses of pipes,
environment variables, forks, and so on.)

  One last thing: if you look at my webserver or (are you out of
mind?!?) use it, I would just be overjoyed to hear about it.  Please
email me.  I probably won't really be releasing major updates, but if
I help you learn something, I'd love to know!

  Happy hacking!

                                   J. David Blackstone

