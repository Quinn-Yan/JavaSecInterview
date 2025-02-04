## 内存马

- Tomcat和Spring内存马分别有哪些（★）

Tomcat内存马有：Filter型，Servlet型，Listener型

Spring内存马有：Controller型，Interceptor型

Java Agent内存马：这种方式不仅限于`Tomcat`或`Spring`



- Servlet/Filter内存马查杀手段是怎样的（★★★）

直接能想到的办法是利用Java Agent遍历所有JVM中的class，判断是否是内存马

例如使用阿里的arthas分析，查看是否存在恶意的类名，然后删除

或者使用c0ny1师傅的java-memshell-scanner项目，从Tomcat API角度删除



- Filter内存马查杀时候有什么明显特征吗（★★★）

首先是类名可能是恶意的，或者包名和项目名不符，可以一眼看出

其次优先级肯定是第一位的，这由内存马的特性决定，所以应该重点关注第一个Filter

观察ClassLoader是否是不正常的，以及是否存在对应的Class文件



- 如何实现无法删除的Servlet/Filter内存马（★★★★）

有一种思路是在`destroy`方法中加入再注册内存马的代码，但并不是所有删除方式都会触发`destroy`方法

所以另外的思路是跑一个不死线程，循环检测该内存马是否存在，以及注册的功能



- 内存马如何持久化（★★★）

内存马持久化这个问题必须要往本地写文件

一般来说可以往Tomcat里写字节码或者直接改写依赖的Jar，在`doFilter`等位置插入恶意字节码

4ra1n师傅提到的修改Tomcat的Lib也是一种手段，在默认开启的`WsFilter`中修改代码



- 内存马持久化写字节码方式除了`@Filter`标签还有什么办法（★★★）
使用`ServletContainerInitializer`用于在容器启动阶段注册三大组件，取代`web.xml`配置。其中`onStartup`方法会在`Tomcat`中间件重启加载当前`webapp`会优先执行这个方法。通过改方法，我们可以注册一个`webshell`的`filter`



- Java Agent内存马的查杀（★★★）

网上师傅提到用`sa-jdi.jar`工具来做，这是一个JVM性能检测工具，可以dump出JVM中可能有问题的Class文件，尤其重点关注`HttpServletr.service`方法，这是Agent内存马常用的手段



- 如果有一个陌生的框架你如何挖内存马（★★★）

核心是找到类似`Tomcat`和`Spring`中的`Context`对象，然后尝试从其中获取`request`和`response`对象以实现内存马的功能。

可以从常见的类名入手：Requst、ServletRequest、RequstGroup、RequestInfo、RequestGroupInfo等等

可以参考c0ny1师傅的`java-object-searcher`项目，半自动搜索`request`对象