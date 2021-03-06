---                                                                                                                                                                                              
  layout: post
  title: HTTP_GZIP
---

# Web服务器处理HTTP压缩之gzip、deflate压缩

## 摘要

>现如今在处理http请求的时候，由于请求的资源较多，如果不启用压缩的话，那么页面请求的流量将会非常大。启用gzip压缩，在一定程度上会大大的提高页面性能。
- 一、什么是gzip         

>gzip是一种数据格式，默认且目前仅使用deflate算法压缩data部分；

>Gzip是一种流行的文件压缩算法，现在的应用十分广泛，尤其是在Linux平台。当应用Gzip压缩到一个纯文本文件时，效果是非常明显的，大约可以减少70％以上的文件大小。这取决于文件中的内容。
利用Apache中的Gzip模块，我们可以使用Gzip压缩算法来对Apache服务器发布的网页内容进行压缩后再传输到客户端浏览器。这样经过压缩后实际上降低了网络传输的字节数，最明显的好就是可以加快网页加载的速度。
网页加载速度加快的好处不言而喻，除了节省流量，改善用户的浏览体验外，另一个潜在的好处是Gzip与搜索引擎的抓取工具有着更好的关系。例如 Google就可以通过直接读取gzip文件来比普通手工抓取更快地检索网页。在Google网站管理员工具（Google Webmaster Tools）中你可以看到，sitemap.xml.gz 是直接作为Sitemap被提交的。
而这些好处并不仅仅限于静态内容，PHP动态页面和其他动态生成的内容均可以通过使用Apache压缩模块压缩，加上其他的性能调整机制和相应的服务器端 缓存规则，这可以大大提高网站的性能。因此，对于部署在Linux服务器上的PHP程序，在服务器支持的情况下，我们建议你开启使用Gzip Web压缩。

>PS：详情参考：http://baike.baidu.com/item/gzip?fr=aladdin

- 二、什么是deflate

>DEFLATE是同时使用了LZ77算法与哈夫曼编码（Huffman Coding）的一个无损数据压缩算法。

>它最初是由Phil Katz为他的PKZIP归档工具第二版所定义的，后来定义在RFC 1951规范中。

>人们普遍认为DEFLATE不受任何专利所制约，并且在LZW（GIF文件格式使用）相关的专利失效之前，这种格式除了在ZIP文件格式中得到应用之外也在gzip压缩文件以及PNG图像文件中得到了应用。

>DEFLATE压缩与解压的源代码可以在自由、通用的压缩库zlib上找到。

>更高压缩率的DEFLATE是7-zip所实现的。AdvanceCOMP也使用这种实现，它可以对gzip、PNG、MNG以及ZIP文件进行压缩从而得到比zlib更小的文件大小。在Ken Silverman的KZIP与PNGOUT中使用了一种更加高效同时要求更多用户输入的DEFLATE程序。

>deflate是一种压缩算法,是huffman编码的一种加强。

>deflate与gzip解压的代码几乎相同，可以合成一块代码。

- 三、web服务器处理http压缩的过程

    - 1. Web服务器接收到浏览器的HTTP请求后，检查浏览器是否支持HTTP压缩（Accept-Encoding 信息）；
    - 2. 如果浏览器支持HTTP压缩，Web服务器检查请求文件的后缀名；
    - 3. 如果请求文件是HTML、CSS等静态文件，Web服务器到压缩缓冲目录中检查是否已经存在请求文件的最新压缩文件；
    - 4. 如果请求文件的压缩文件不存在，Web服务器向浏览器返回未压缩的请求文件，并在压缩缓冲目录中存放请求文件的压缩文件；
    - 5. 如果请求文件的最新压缩文件已经存在，则直接返回请求文件的压缩文件；
    - 6. 如果请求文件是动态文件，Web服务器动态压缩内容并返回浏览器，压缩内容不存放到压缩缓存目录中。


- 四、gzip与deflate区别

>deflate使用inflateInit()，而gzip使用inflateInit2()进行初始化，比 inflateInit()多一个参数: -MAX_WBITS，表示处理raw deflate数据。因为gzip数据中的zlib压缩数据块没有zlib header的两个字节。使用inflateInit2时要求zlib库忽略zlib header。在zlib手册中要求windowBits为8..15，但是实际上其它范围的数据有特殊作用，见zlib.h中的注释，如负数表示raw deflate。        

>Apache的deflate变种可能也没有zlib header，需要添加假头后处理。即MS的错误deflate (raw deflate).zlib头第1字节一般是0x78, 第2字节与第一字节合起来的双字节应能被31整除，详见rfc1950。例如Firefox的zlib假头为0x7801，python zlib.compress()结果头部为0x789c。        

>deflate 是最基础的算法，gzip 在 deflate 的 raw data 前增加了 10 个字节的 gzheader，尾部添加了 8 个字节的校验字节（可选 crc32 和 adler32） 和长度标识字节。

>安装它们的Apache Web服务器版本的差异。Apache 1.x系列没有内建网页压缩技术，所以才去用额外的第三方mod_gzip 模块来执¡压缩。而Apache 2.x官方在开发的时候，就把网页压缩考虑进去，内建了mod_deflate 这个模块，用以取代mod_gzip。虽然两者都是使用的Gzip压缩算法，它们的运作原理是类似的。     


>压缩质量。mod_deflate 压缩速度略快而mod_gzip 的压缩比略高。一般默认情况下，mod_gzip 会比mod_deflate 多出4%~6％的压缩量。

>对服务器资源的占用。 一般来说mod_gzip 对服务器CPU的占用要高一些。mod_deflate 是专门为确保服务器的性能而使用的一个压缩模块，mod_deflate 需要较少的资源来压缩文件。这意味着在高流量的服务器，使用mod_deflate 可能会比mod_gzip 加载速度更快。即在服务器性能足够的情况下，使用mod_gzip，虽然会耗费服务器性能，但是值得（压缩更快更好）；在服务器性能不足的情况下，使用mod_deflate 确保性能。


>从Apache 2.0.45开始，mod_deflate 可使用DeflateCompressionLevel 指令¥设置压缩级别。该指令的值可为1（压缩速度最快，最低的压缩质量）至9（最慢的压缩速度，压缩率最高）之间的整数，其默认值为6（压缩速度和压缩质 量较为平衡的值）。这个简单的变化更是使得mod_deflate 可以轻松媲美mod_gzip 的压缩。

- 五、开启mod_gzip、mod_deflate


>Apache上利用Gzip压缩算法进行压缩的模块有两种：mod_gzip 和mod_deflate。 要使用Gzip Web压缩，请首先确定你的服务器开启了对这两个组件之一的支持。在Linux服务器上，现在已经有越来越多的空间商开放了对它们的支持，有的甚至是同时 支持这两个模块的。例如目前Godaddy、Bluehost及DreamHosts等空间商的服务器都已同时支持mod_gzip 和mod_deflate。        


>通过查看HTTP头，我们可以快速判断使用的客户端浏览器是否支持接受gzip压缩。若发送的HTTP头中出现以下信息，则表明你的浏览器支持接受相应的gzip缩：

```c
Accept-Encoding: gzip 支持mod_gzip
Accept-Encoding: deflate 支持mod_deflate 
Accept-Encoding: gzip,deflate 同时支持mod_gzip 和mod_deflate
mod_deflate 是apache自带的模块,当然是在apache 2后支持的,以前1的时候是mod_gzip,启用mod_deflate可以很好的为节省网页大小,只不过是占用服务器的资源和内存.用户看到页面的速度会大大加快。在apache2.0以上（包括apache2.0）的版中gzip压缩使用的是mod_deflate模块
```
    - 1. 查看apache的安装模式
>apachectl -l
>发现 mod_so.c，ok可以动态加模块，不用重新编译。
    - 2. 安装mod_deflate
>找到原有的apache安装包安装mod_deflate
```c
cd httpd-2.0.59/modules/filters
/usr/local/apache2/bin/apxs -i -c -a mod_deflate.c
PS：apxs命令参数说明：
-i  此选项表示需要执行安装操作，以安装一个或多个动态共享对象到服务器的modules目录中。
-a  此选项自动增加一个LoadModule行到httpd.conf文件中，以激活此¡块，或者，如果此行已经存在，则启用之。
-A  与 -a 选项类似，但是它增加的LoadModule命令有一个井号前缀(#)，即此模块已经准备就绪但尚未启用。
-c  此选项表示需要执行编译操作。它首先会编译C源程序(.c)files为对应的目标代码文件(.o)，然后连接这些目标代码和files中其余的目标代码文件(.o和.a)，以生成动态共享对象dsofile 。如果没有指定 -o 选项，则此输出文件名由files中的第一个文件名推测得到，也就是默认为mod_name.so 。
```
    - 3、修改Apache的http.conf文件，去除mod_deflate.so前面的注释

>LoadModule deflate_module modules/mod_deflate.so
    - 4、在根目录中新建.htaccess文件，定制压缩规则

>GZIP压缩模块配置<ifmodule mod_deflate.c>
>启用对特定MIME类型内容的压缩
>SetOutputFilter DEFLATESetEnvIfNoCase Request_URI .(?:gif|jpe?g|png|exe|t?gz|zip|bz2|sit|rar|pdf|mov|avi|mp3|mp4|rm)$ no-gzip dont-vary #设置不对压缩的文件AddOutputFilterByType DEFLATE text/html text/css text/plain text/xml application/x-httpd-php application/x-javascript #设置对压缩的文件</ifmodule>
    - 5、对指定的文件配置缓存的生存时间，去除mod_headers.so模块前面的注释

>LoadModule headers_module modules/mod_headers.so
    - 6、在根目录中新建.htaccess文件，定制压缩规则
```c
#文件缓存时间配置
<FilesMatch ".(flv|gif|jpg|jpeg|png|ico|swf|js|css)$">
Header set Cache-Control "max-age=2592000"
</FilesMatch>
里面的文件MIME类型可以根据自己情况添加，至于PDF 、图片、音乐文档之类的这些本身都已经高度压缩格式，重复压缩的作用不大，反而可能会因为增加CPU的处理时间及浏览器的渲染问题而降低性能。所以就没必要再通过Gzip压缩。通过以上设置后再查看返回的HTTP头，出现以下信息则表明返回的数据已经过压缩。即网站程序所配置的Gzip压缩已生效。
Content-Encoding: gzip
```
>***注：***不管使用mod_gzip 还是mod_deflate，此处返回的信息都一样。因为它们都是实现的gzip压缩方式。

- 遇到的问题以及解决:

    - 1：

>apach2 安装mod_deflate后restart,直接

>load /opt/apache/modules/mod_deflate.so into server: /opt/apache/modules/mod_deflate.so: undefined symbol: deflate 异常的痛苦

>什么ldd mod_deflate.so后再export LIB_LIBRARY_PATH呀，都试了N次，google也go了N天

>终于在google上go出来一篇文章，终于解决，方法如下： vi /usr/local/apache2/bin/apr-config 修改LDFLAGS=" " 为 LDFLAGS="-lz" 然后再apxs -ica mod_deflate.c 就OK了.

    - 2：
>apach2 安装mod_deflate后restart,直接

>module deflate_module is built-in and can't be loaded ...

>这说明该模块已经安装，不必再LoadModule deflate_module启用它。

>只需做<ifmodule mod_deflate.c>配置

- 参考：

    - http://qbaok.blog.163.com/blog/static/101292652008101431233385/

    - http://blog.sina.com.cn/s/blog_a34721f00101f658.html

    - http://blog.csdn.net/zhangxinrun/article/details/5711307

    - http://baike.baidu.com/view/4795073.htm?fr=aladdin

    - http://baike.baidu.com/item/gzip?fr=aladdin

    - http://www.cnblogs.com/linzhenjie/archive/2013/03/05/2943635.html

    - http://www.cnblogs.com/zhangziqiu/archive/2009/05/17/gzip.html

