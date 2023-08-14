+++
title = "开源应用程序架构（第二卷）：nginx"
date = 2023-08-14T16:57:39+08:00
weight = 200
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# The Architecture of Open Source Applications (Volume 2) nginx - 开源应用程序架构（第二卷）：nginx

https://aosabook.org/en/v2/nginx.html

Andrew Alexeev

安德鲁·亚历克谢夫

**[Software Design by Example in Python](https://third-bit.com/sdxpy/) is now in beta.
All the material is free to read and re-use under open licenses, and we would be very grateful for feedback and corrections.**

​	**[Python实例软件设计](https://third-bit.com/sdxpy/)现在处于测试版阶段。所有材料可以免费阅读和重用，采用开放许可发布，我们将非常感谢您的反馈和修正。**

If you enjoy these books, you may also enjoy [Software Design by Example in JavaScript](https://third-bit.com/sdxjs/), [Research Software Engineering with Python](https://merely-useful.tech/py-rse/), [JavaScript for Data Science](https://third-bit.com/js4ds/), [Teaching Tech Together](http://teachtogether.tech/), and [It Will Never Work in Theory](https://neverworkintheory.org/).

​	如果您喜欢这些书，您可能还会喜欢[JavaScript实例软件设计](https://third-bit.com/sdxjs/)、[使用Python进行研究软件工程](https://merely-useful.tech/py-rse/)、[数据科学的JavaScript](https://third-bit.com/js4ds/)、[共同教授技术](http://teachtogether.tech/)和[理论上永远行不通](https://neverworkintheory.org/)。

nginx (pronounced "engine x") is a free open source web server written by Igor Sysoev, a Russian software engineer. Since its public launch in 2004, nginx has focused on high performance, high concurrency and low memory usage. Additional features on top of the web server functionality, like load balancing, caching, access and bandwidth control, and the ability to integrate efficiently with a variety of applications, have helped to make nginx a good choice for modern website architectures. Currently nginx is the second most popular open source web server on the Internet.

​	nginx（发音为"engine x"）是由俄罗斯软件工程师伊戈尔·西索夫编写的免费开源网络服务器。自2004年公开发布以来，nginx专注于高性能、高并发和低内存使用率。在Web服务器功能之上，额外的功能，如负载均衡、缓存、访问和带宽控制，以及与各种应用程序高效集成的能力，已经使nginx成为现代网站架构的不错选择。目前，nginx是互联网上第二受欢迎的开源Web服务器。

## 14.1. 高并发为何重要？ - 14.1. Why Is High Concurrency Important?

These days the Internet is so widespread and ubiquitous it's hard to imagine it wasn't exactly there, as we know it, a decade ago. It has greatly evolved, from simple HTML producing clickable text, based on NCSA and then on Apache web servers, to an always-on communication medium used by more than 2 billion users worldwide. With the proliferation of permanently connected PCs, mobile devices and recently tablets, the Internet landscape is rapidly changing and entire economies have become digitally wired. Online services have become much more elaborate with a clear bias towards instantly available live information and entertainment. Security aspects of running online business have also significantly changed. Accordingly, websites are now much more complex than before, and generally require a lot more engineering efforts to be robust and scalable.

​	如今，互联网已经如此广泛和无处不在，很难想象十年前它并不存在，就像我们现在所知道的那样。它已经发展得非常多，从基于NCSA和Apache Web服务器的简单HTML产生可点击的文本，演变成了一个被全球超过20亿用户使用的始终连接的通信媒介。随着长期连接的个人计算机、移动设备和最近的平板电脑的普及，互联网的格局正在迅速变化，整个经济已经数字化。在线服务变得更加复杂，明显偏向于即时可用的实时信息和娱乐。运行在线业务的安全方面也发生了显著变化。因此，现在的网站比以前复杂得多，通常需要更多的工程努力来保持稳定和可扩展性。

One of the biggest challenges for a website architect has always been concurrency. Since the beginning of web services, the level of concurrency has been continuously growing. It's not uncommon for a popular website to serve hundreds of thousands and even millions of simultaneous users. A decade ago, the major cause of concurrency was slow clients—users with ADSL or dial-up connections. Nowadays, concurrency is caused by a combination of mobile clients and newer application architectures which are typically based on maintaining a persistent connection that allows the client to be updated with news, tweets, friend feeds, and so on. Another important factor contributing to increased concurrency is the changed behavior of modern browsers, which open four to six simultaneous connections to a website to improve page load speed.

​	对于网站架构师来说，最大的挑战之一始终是并发性。从Web服务的开始，并发级别一直在不断增长。一个受欢迎的网站为数十万甚至数百万个同时在线用户提供服务并不罕见。十年前，并发的主要原因是慢速客户端，即使用ADSL或拨号连接的用户。如今，并发性是由移动客户端和基于维护持久连接的较新的应用程序架构的组合引起的，该连接允许客户端随时更新新闻、推文、朋友动态等。增加并发性的另一个重要因素是现代浏览器的行为已经改变，它们会向网站开启4到6个同时连接以提高页面加载速度。

To illustrate the problem with slow clients, imagine a simple Apache-based web server which produces a relatively short 100 KB response—a web page with text or an image. It can be merely a fraction of a second to generate or retrieve this page, but it takes 10 seconds to transmit it to a client with a bandwidth of 80 kbps (10 KB/s). Essentially, the web server would relatively quickly pull 100 KB of content, and then it would be busy for 10 seconds slowly sending this content to the client before freeing its connection. Now imagine that you have 1,000 simultaneously connected clients who have requested similar content. If only 1 MB of additional memory is allocated per client, it would result in 1000 MB (about 1 GB) of extra memory devoted to serving just 1000 clients 100 KB of content. In reality, a typical web server based on Apache commonly allocates more than 1 MB of additional memory per connection, and regrettably tens of kbps is still often the effective speed of mobile communications. Although the situation with sending content to a slow client might be, to some extent, improved by increasing the size of operating system kernel socket buffers, it's not a general solution to the problem and can have undesirable side effects.

​	为了说明慢速客户端的问题，想象一个基于Apache的简单Web服务器，它生成一个相对较短的100 KB响应，即一个带有文本或图像的Web页面。生成或检索此页面可能只需一小部分时间，但是对于带宽为80 kbps（10 KB/s）的客户端传输需要10秒钟。基本上，Web服务器会相对较快地拉取100 KB的内容，然后在为客户端缓慢发送此内容时忙于10秒钟，然后才会释放其连接。现在想象一下，您有1000个同时连接的客户端请求类似的内容。如果每个客户端只分配了1 MB的额外内存，那么就会导致额外分配约1000 MB（约1 GB）的内存，仅为1000个客户端提供100 KB的内容。实际上，基于Apache的典型Web服务器通常为每个连接分配超过1 MB的额外内存，而令人遗憾的是，移动通信的有效速度仍然经常是十几kbps。虽然通过增加操作系统内核套接字缓冲区的大小可能在一定程度上改善向慢速客户端发送内容的情况，但这并不是解决问题的通用解决方案，可能会产生不良的副作用。

With persistent connections the problem of handling concurrency is even more pronounced, because to avoid latency associated with establishing new HTTP connections, clients would stay connected, and for each connected client there's a certain amount of memory allocated by the web server.

​	通过持久连接，处理并发问题变得更加明显，因为为了避免建立新的HTTP连接所带来的延迟，客户端将保持连接，而为每个连接的客户端分配了一定量的内存。

Consequently, to handle the increased workloads associated with growing audiences and hence higher levels of concurrency—and to be able to continuously do so—a website should be based on a number of very efficient building blocks. While the other parts of the equation such as hardware (CPU, memory, disks), network capacity, application and data storage architectures are obviously important, it is in the web server software that client connections are accepted and processed. Thus, the web server should be able to scale nonlinearly with the growing number of simultaneous connections and requests per second.

​	因此，为了处理与日益增长的受众和因此而增加的并发水平相关的增加的工作负载，并能够持续地这样做，一个网站应该基于一些非常高效的构建块。虽然方程的其他部分，如硬件（CPU、内存、磁盘）、网络容量、应用程序和数据存储架构显然也很重要，但在Web服务器软件中接受和处理客户端连接。因此，Web服务器应该能够与不断增长的同时连接数和每秒请求数非线性地扩展。

### Apache适用吗？ - Isn't Apache Suitable?

Apache, the web server software that still largely dominates the Internet today, has its roots in the beginning of the 1990s. Originally, its architecture matched the then-existing operating systems and hardware, but also the state of the Internet, where a website was typically a standalone physical server running a single instance of Apache. By the beginning of the 2000s it was obvious that the standalone web server model could not be easily replicated to satisfy the needs of growing web services. Although Apache provided a solid foundation for future development, it was architected to spawn a copy of itself for each new connection, which was not suitable for nonlinear scalability of a website. Eventually Apache became a general purpose web server focusing on having many different features, a variety of third-party extensions, and universal applicability to practically any kind of web application development. However, nothing comes without a price and the downside to having such a rich and universal combination of tools in a single piece of software is less scalability because of increased CPU and memory usage per connection.

​	Apache是今天仍然主导互联网的Web服务器软件，它起源于20世纪90年代初。最初，它的架构与当时的操作系统和硬件相匹配，但也与互联网的状态相匹配，在那个时候，一个网站通常是一个独立的物理服务器，运行一个单独的Apache实例。到了21世纪初，显然无法轻松复制独立的Web服务器模型以满足不断增长的Web服务需求。尽管Apache为未来的发展提供了坚实的基础，但它的架构是为每个新连接生成自身的副本，这对于网站的非线性可扩展性是不合适的。最终，Apache变成了一个通用的Web服务器，注重拥有许多不同的功能、各种第三方扩展和适用于几乎任何类型的Web应用程序开发的普遍适用性。然而，没有什么是没有代价的，单一软件中具有如此丰富和通用的工具组合的弊端是由于每个连接的CPU和内存使用率的增加而导致的可扩展性较差。

Thus, when server hardware, operating systems and network resources ceased to be major constraints for website growth, web developers worldwide started to look around for a more efficient means of running web servers. Around ten years ago, Daniel Kegel, a prominent software engineer, [proclaimed](http://www.kegel.com/c10k.html) that "it's time for web servers to handle ten thousand clients simultaneously" and predicted what we now call Internet cloud services. Kegel's C10K manifest spurred a number of attempts to solve the problem of web server optimization to handle a large number of clients at the same time, and nginx turned out to be one of the most successful ones.

​	因此，当服务器硬件、操作系统和网络资源不再是网站增长的主要限制因素时，全球的Web开发人员开始寻找更有效的运行Web服务器的方式。大约十年前，著名软件工程师丹尼尔·凯格尔 [宣称](http://www.kegel.com/c10k.html) "现在是Web服务器同时处理一万个客户端的时候了"，并预测了我们现在称之为互联网云服务的发展。凯格尔的C10K宣言促使许多尝试解决Web服务器优化问题，以处理大量同时连接的尝试，其中nginx被证明是最成功的之一。

Aimed at solving the C10K problem of 10,000 simultaneous connections, nginx was written with a different architecture in mind—one which is much more suitable for nonlinear scalability in both the number of simultaneous connections and requests per second. nginx is event-based, so it does not follow Apache's style of spawning new processes or threads for each web page request. The end result is that even as load increases, memory and CPU usage remain manageable. nginx can now deliver tens of thousands of concurrent connections on a server with typical hardware.

​	nginx旨在解决C10K问题，即10,000个同时连接的问题，它以不同的架构为基础编写，这种架构更适合于同时连接数和每秒请求数的非线性可扩展性。nginx是基于事件的，因此它不遵循Apache为每个Web页面请求生成新进程或线程的方式。结果是，即使负载增加，内存和CPU使用率仍然可以管理。现在，nginx可以在具有典型硬件的服务器上提供成千上万的并发连接。

When the first version of nginx was released, it was meant to be deployed alongside Apache such that static content like HTML, CSS, JavaScript and images were handled by nginx to offload concurrency and latency processing from Apache-based application servers. Over the course of its development, nginx has added integration with applications through the use of FastCGI, uswgi or SCGI protocols, and with distributed memory object caching systems like *memcached*. Other useful functionality like reverse proxy with load balancing and caching was added as well. These additional features have shaped nginx into an efficient combination of tools to build a scalable web infrastructure upon.

​	当第一个版本的nginx发布时，它被设计为与Apache一起部署，以便由nginx处理静态内容，如HTML、CSS、JavaScript和图像，以从基于Apache的应用服务器中卸载并发性和延迟处理。在其发展过程中，nginx通过使用FastCGI、uswgi或SCGI协议与应用程序集成，以及与分布式内存对象缓存系统*memcached*等进行了集成。还添加了反向代理和负载平衡以及缓存等其他有用的功能。这些附加功能将nginx塑造成一个有效的工具组合，用于构建可扩展的Web基础设施。

In February 2012, the Apache 2.4.x branch was released to the public. Although this latest release of Apache has added new multi-processing core modules and new proxy modules aimed at enhancing scalability and performance, it's too soon to tell if its performance, concurrency and resource utilization are now on par with, or better than, pure event-driven web servers. It would be very nice to see Apache application servers scale better with the new version, though, as it could potentially alleviate bottlenecks on the backend side which still often remain unsolved in typical nginx-plus-Apache web configurations.

​	2012年2月，Apache 2.4.x分支向公众发布。尽管此Apache的最新版本增加了新的多处理核心模块和新的代理模块，旨在提高可扩展性和性能，但现在尚不能确定其性能、并发性和资源利用率是否与纯事件驱动的Web服务器相当或更好。希望看到新版本的Apache应用服务器能够更好地扩展，因为它可能会在后端侧缓解瓶颈，而在典型的nginx加Apache Web配置中，这些瓶颈通常仍然没有解决。

### 使用nginx是否有更多优势？ - Are There More Advantages to Using nginx?

Handling high concurrency with high performance and efficiency has always been the key benefit of deploying nginx. However, there are now even more interesting benefits.

​	高性能和高效处理高并发始终是部署nginx的关键优势。然而，现在还有更多有趣的好处。

In the last few years, web architects have embraced the idea of decoupling and separating their application infrastructure from the web server. However, what would previously exist in the form of a LAMP (Linux, Apache, MySQL, PHP, Python or Perl)-based website, might now become not merely a LEMP-based one (`E' standing for `Engine x'), but more and more often an exercise in pushing the web server to the edge of the infrastructure and integrating the same or a revamped set of applications and database tools around it in a different way.

​	在过去的几年里，Web架构师已经接受了将应用程序基础设施与Web服务器分离和解耦的想法。然而，以前可能以LAMP（Linux、Apache、MySQL、PHP、Python或Perl）为基础的网站，现在可能不仅仅是基于LEMP的网站（'E'代表'Engine x'），而且越来越经常地是将Web服务器推到基础设施的边缘，并以不同的方式将相同或经过改进的应用程序和数据库工具集成到其周围。

nginx is very well suited for this, as it provides the key features necessary to conveniently offload concurrency, latency processing, SSL (secure sockets layer), static content, compression and caching, connections and requests throttling, and even HTTP media streaming from the application layer to a much more efficient edge web server layer. It also allows integrating directly with memcached/Redis or other "NoSQL" solutions, to boost performance when serving a large number of concurrent users.

​	nginx非常适合此目的，因为它提供了方便地将并发、延迟处理、SSL（安全套接字层）、静态内容、压缩和缓存、连接和请求限制，甚至HTTP媒体流从应用程序层转移到更高效的边缘Web服务器层所需的关键功能。它还允许直接集成memcached/Redis或其他“NoSQL”解决方案，以在提供大量并发用户服务时提高性能。

With recent flavors of development kits and programming languages gaining wide use, more and more companies are changing their application development and deployment habits. nginx has become one of the most important components of these changing paradigms, and it has already helped many companies start and develop their web services quickly and within their budgets.

​	随着最近版本的开发工具包和编程语言的广泛使用，越来越多的公司正在改变其应用程序开发和部署习惯。nginx已成为这些变化范例中最重要的组成部分之一，它已经帮助许多公司快速启动和开发他们的Web服务，而且还在预算内。

The first lines of nginx were written in 2002. In 2004 it was released to the public under the two-clause BSD license. The number of nginx users has been growing ever since, contributing ideas, and submitting bug reports, suggestions and observations that have been immensely helpful and beneficial for the entire community.

​	nginx的第一行代码是在2002年编写的。2004年，它在两条款的BSD许可证下向公众发布。自那以后，nginx用户的数量一直在增长，为整个社区贡献了想法，并提交了有助益的bug报告、建议和观察。

The nginx codebase is original and was written entirely from scratch in the C programming language. nginx has been ported to many architectures and operating systems, including Linux, FreeBSD, Solaris, Mac OS X, AIX and Microsoft Windows. nginx has its own libraries and with its standard modules does not use much beyond the system's C library, except for zlib, PCRE and OpenSSL which can be optionally excluded from a build if not needed or because of potential license conflicts.

​	nginx的代码库是原始的，并完全使用C编程语言从头编写。nginx已被移植到许多架构和操作系统，包括Linux、FreeBSD、Solaris、Mac OS X、AIX和Microsoft Windows。nginx拥有自己的库，与其标准模块一起，除了zlib、PCRE和OpenSSL，它几乎不使用系统的C库，如果不需要或由于潜在的许可证冲突，可以选择将它们从构建中排除在外。

A few words about the Windows version of nginx. While nginx works in a Windows environment, the Windows version of nginx is more like a proof-of-concept rather than a fully functional port. There are certain limitations of the nginx and Windows kernel architectures that do not interact well at this time. The known issues of the nginx version for Windows include a much lower number of concurrent connections, decreased performance, no caching and no bandwidth policing. Future versions of nginx for Windows will match the mainstream functionality more closely.

​	关于nginx的Windows版本，有一些问题需要解决。虽然nginx可以在Windows环境中运行，但Windows版本的nginx更像是一个概念验证，而不是一个完全功能的移植版本。nginx和Windows内核体系结构之间存在一些限制，目前它们之间的互动不太好。Windows版本的已知问题包括并发连接数较少、性能降低、没有缓存和没有带宽限制。将来的nginx版本为Windows将更加贴近主流功能。

## 14.2. nginx架构概述 - 14.2. Overview of nginx Architecture

Traditional process- or thread-based models of handling concurrent connections involve handling each connection with a separate process or thread, and blocking on network or input/output operations. Depending on the application, it can be very inefficient in terms of memory and CPU consumption. Spawning a separate process or thread requires preparation of a new runtime environment, including allocation of heap and stack memory, and the creation of a new execution context. Additional CPU time is also spent creating these items, which can eventually lead to poor performance due to thread thrashing on excessive context switching. All of these complications manifest themselves in older web server architectures like Apache's. This is a tradeoff between offering a rich set of generally applicable features and optimized usage of server resources.

​	处理并发连接的传统进程或线程模型涉及使用单独的进程或线程处理每个连接，并在网络或输入/输出操作上阻塞。根据应用程序，从内存和CPU消耗的角度来看，这可能非常低效。生成单独的进程或线程需要准备新的运行时环境，包括分配堆和堆栈内存以及创建新的执行上下文。还需要额外的CPU时间来创建这些项目，这最终可能导致由于过多的上下文切换而导致性能下降。所有这些复杂性都在旧的Web服务器架构（如Apache）中表现出来。这是在提供丰富的普适功能和优化服务器资源使用之间的权衡。

From the very beginning, nginx was meant to be a specialized tool to achieve more performance, density and economical use of server resources while enabling dynamic growth of a website, so it has followed a different model. It was actually inspired by the ongoing development of advanced event-based mechanisms in a variety of operating systems. What resulted is a modular, event-driven, asynchronous, single-threaded, non-blocking architecture which became the foundation of nginx code.

​	从一开始，nginx就旨在成为一个专业的工具，以实现更高性能、更高密度和更经济的服务器资源使用，同时实现网站的动态增长，因此它遵循了不同的模型。它实际上受到了各种操作系统中正在开发的先进事件驱动机制的启发。所产生的结果是模块化、事件驱动、异步、单线程、非阻塞架构，这成为nginx代码的基础。

nginx uses multiplexing and event notifications heavily, and dedicates specific tasks to separate processes. Connections are processed in a highly efficient run-loop in a limited number of single-threaded processes called `worker`s. Within each `worker` nginx can handle many thousands of concurrent connections and requests per second.

​	nginx大量使用多路复用和事件通知，并将特定任务分配给单独的进程。连接在高度高效的运行循环中在一组称为`worker`的单线程进程中进行处理。在每个`worker`中，nginx可以处理成千上万个并发连接和每秒请求数。

### 代码结构 - Code Structure

The nginx `worker` code includes the core and the functional modules. The core of nginx is responsible for maintaining a tight run-loop and executing appropriate sections of modules' code on each stage of request processing. Modules constitute most of the presentation and application layer functionality. Modules read from and write to the network and storage, transform content, do outbound filtering, apply server-side include actions and pass the requests to the upstream servers when proxying is activated.

​	nginx的`worker`代码包括核心和功能模块。nginx的核心负责维护紧密的运行循环，并在请求处理的每个阶段上执行模块代码的适当部分。模块构成了大部分演示和应用程序层功能。模块从网络和存储中读取和写入，转换内容，进行出站过滤，应用服务器端包含操作，并在激活代理时将请求传递给上游服务器。

nginx's modular architecture generally allows developers to extend the set of web server features without modifying the nginx core. nginx modules come in slightly different incarnations, namely core modules, event modules, phase handlers, protocols, variable handlers, filters, upstreams and load balancers. At this time, nginx doesn't support dynamically loaded modules; i.e., modules are compiled along with the core at build stage. However, support for loadable modules and ABI is planned for the future major releases. More detailed information about the roles of different modules can be found in [Section 14.4](https://aosabook.org/en/v2/nginx.html#sec.nginx.internals).

​	nginx的模块化架构通常允许开发人员在不修改nginx核心的情况下扩展Web服务器功能集。nginx模块有稍微不同的版本，即核心模块、事件模块、阶段处理程序、协议、变量处理程序、过滤器、上游和负载平衡器。目前，nginx不支持动态加载模块；即在构建阶段，模块会随着核心一起编译。然而，对于可加载模块和ABI的支持计划在未来的主要版本中实现。有关不同模块角色的更详细信息，请参见[第14.4节](https://aosabook.org/en/v2/nginx.html#sec.nginx.internals)。

While handling a variety of actions associated with accepting, processing and managing network connections and content retrieval, nginx uses event notification mechanisms and a number of disk I/O performance enhancements in Linux, Solaris and BSD-based operating systems, like `kqueue`, `epoll`, and `event ports`. The goal is to provide as many hints to the operating system as possible, in regards to obtaining timely asynchronous feedback for inbound and outbound traffic, disk operations, reading from or writing to sockets, timeouts and so on. The usage of different methods for multiplexing and advanced I/O operations is heavily optimized for every Unix-based operating system nginx runs on.

​	在处理与接受、处理和管理网络连接和内容检索相关的各种操作时，nginx使用事件通知机制和许多Linux、Solaris和基于BSD的操作系统中的磁盘I/O性能增强功能，例如`kqueue`、`epoll`和`event ports`。目标是为操作系统提供尽可能多的提示，以获取关于入站和出站流量、磁盘操作、从或写入套接字、超时等的及时异步反馈。对于基于Unix的操作系统上运行的nginx，不同方法的多路复用和高级I/O操作的使用都经过了优化。

A high-level overview of nginx architecture is presented in [Figure 14.1](https://aosabook.org/en/v2/nginx.html#fig.nginx.arch).

​	nginx架构的高级概述在[图14.1](https://aosabook.org/en/v2/nginx.html#fig.nginx.arch)中呈现。

![img](chapterNginxInTheArchitectureOfOpenSourceApplications_img/architecture.png)Figure 14.1: Diagram of nginx's architecture

图14.1：nginx架构图解

### Workers模型 - Workers Model

As previously mentioned, nginx doesn't spawn a process or thread for every connection. Instead, `worker` processes accept new requests from a shared "listen" socket and execute a highly efficient run-loop inside each `worker` to process thousands of connections per `worker`. There's no specialized arbitration or distribution of connections to the `worker`s in nginx; this work is done by the OS kernel mechanisms. Upon startup, an initial set of listening sockets is created. `worker`s then continuously accept, read from and write to the sockets while processing HTTP requests and responses.

​	如前所述，nginx不为每个连接生成进程或线程。相反，`worker`进程从共享的“监听”套接字接受新请求，并在每个`worker`内执行高效的运行循环，以处理每个`worker`的数千个连接。在nginx中，没有专门的仲裁或连接分发给`worker`的机制；这项工作由操作系统内核机制完成。启动时，会创建一组初始的监听套接字。然后，`worker`不断地接受、从套接字读取和写入，并在处理HTTP请求和响应时进行处理。

The run-loop is the most complicated part of the nginx `worker` code. It includes comprehensive inner calls and relies heavily on the idea of asynchronous task handling. Asynchronous operations are implemented through modularity, event notifications, extensive use of callback functions and fine-tuned timers. Overall, the key principle is to be as non-blocking as possible. The only situation where nginx can still block is when there's not enough disk storage performance for a `worker` process.

​	运行循环是nginx `worker` 代码中最复杂的部分。它包括全面的内部调用，并且在异步任务处理思想的基础上进行了大量的依赖。异步操作通过模块化、事件通知、广泛使用的回调函数和精细调整的计时器来实现。总体上，关键原则是尽可能地非阻塞。nginx仍然可能阻塞的唯一情况是当`worker`进程的磁盘存储性能不足时。

Because nginx does not fork a process or thread per connection, memory usage is very conservative and extremely efficient in the vast majority of cases. nginx conserves CPU cycles as well because there's no ongoing create-destroy pattern for processes or threads. What nginx does is check the state of the network and storage, initialize new connections, add them to the run-loop, and process asynchronously until completion, at which point the connection is deallocated and removed from the run-loop. Combined with the careful use of `syscall`s and an accurate implementation of supporting interfaces like pool and slab memory allocators, nginx typically achieves moderate-to-low CPU usage even under extreme workloads.

​	由于nginx不会为每个连接生成进程或线程，所以在绝大多数情况下，内存使用量非常保守，极其高效。nginx还节省CPU周期，因为没有为进程或线程持续创建-销毁模式。nginx要做的是检查网络和存储的状态，初始化新连接，将它们添加到运行循环中，并异步处理直到完成，此时连接将被释放并从运行循环中删除。结合对`syscall`的谨慎使用以及对支持接口（如内存池和slab内存分配器）的准确实现，即使在极端工作负载下，nginx通常也能实现中等到低CPU使用率。

Because nginx spawns several `worker`s to handle connections, it scales well across multiple cores. Generally, a separate `worker` per core allows full utilization of multicore architectures, and prevents thread thrashing and lock-ups. There's no resource starvation and the resource controlling mechanisms are isolated within single-threaded `worker` processes. This model also allows more scalability across physical storage devices, facilitates more disk utilization and avoids blocking on disk I/O. As a result, server resources are utilized more efficiently with the workload shared across several workers.

​	由于nginx生成了几个`worker`以处理连接，因此它在多个核心上的扩展性良好。通常，每个核心一个独立的`worker`可以充分利用多核架构，并防止线程抖动和锁死。资源饥饿现象是不存在的，资源控制机制被隔离在单线程的`worker`进程中。这种模型还允许更多的可伸缩性跨越物理存储设备，促进更多的磁盘利用，并避免了磁盘I/O的阻塞。因此，服务器资源以多个worker共享负载的方式更有效地利用。

With some disk use and CPU load patterns, the number of nginx `worker`s should be adjusted. The rules are somewhat basic here, and system administrators should try a couple of configurations for their workloads. General recommendations might be the following: if the load pattern is CPU intensive—for instance, handling a lot of TCP/IP, doing SSL, or compression—the number of nginx `worker`s should match the number of CPU cores; if the load is mostly disk I/O bound—for instance, serving different sets of content from storage, or heavy proxying—the number of `worker`s might be one and a half to two times the number of cores. Some engineers choose the number of `worker`s based on the number of individual storage units instead, though efficiency of this approach depends on the type and configuration of disk storage.

​	对于某些磁盘使用和CPU负载模式，nginx `worker`的数量应进行调整。规则在这里有点基础，系统管理员应该为他们的工作负载尝试几种配置。一般的建议可能如下：如果负载模式是CPU密集型——例如，处理大量的TCP/IP、执行SSL或压缩——nginx `worker`的数量应与CPU核心数量匹配；如果负载主要受限于磁盘I/O——例如，从存储中提供不同集合的内容，或者进行大量的代理——`worker`的数量可能是核心数的1.5到2倍。一些工程师根据个体存储单元的数量选择`worker`的数量，尽管这种方法的效率取决于磁盘存储的类型和配置。

One major problem that the developers of nginx will be solving in upcoming versions is how to avoid most of the blocking on disk I/O. At the moment, if there's not enough storage performance to serve disk operations generated by a particular `worker`, that `worker` may still block on reading/writing from disk. A number of mechanisms and configuration file directives exist to mitigate such disk I/O blocking scenarios. Most notably, combinations of options like sendfile and AIO typically produce a lot of headroom for disk performance. An nginx installation should be planned based on the data set, the amount of memory available for nginx, and the underlying storage architecture.

​	nginx开发人员在即将推出的版本中将解决一个主要问题，即如何避免大多数的磁盘I/O阻塞。目前，如果特定`worker`生成的磁盘操作的存储性能不足以提供磁盘I/O，那么该`worker`仍然可能在从磁盘读取/写入时阻塞。存在一些机制和配置文件指令来减轻这种磁盘I/O阻塞情况。尤其值得注意的是，诸如sendfile和AIO等选项的组合通常会为磁盘性能提供很大的余地。nginx安装应基于数据集、为nginx可用的内存量以及底层存储架构进行规划。

Another problem with the existing `worker` model is related to limited support for embedded scripting. For one, with the standard nginx distribution, only embedding Perl scripts is supported. There is a simple explanation for that: the key problem is the possibility of an embedded script to block on any operation or exit unexpectedly. Both types of behavior would immediately lead to a situation where the worker is hung, affecting many thousands of connections at once. More work is planned to make embedded scripting with nginx simpler, more reliable and suitable for a broader range of applications.

​	现有`worker`模型的另一个问题与嵌入式脚本的有限支持有关。首先，标准的nginx分发版本仅支持嵌入Perl脚本。这有一个简单的解释：主要问题是嵌入式脚本可能会在任何操作上阻塞或意外退出。这两种类型的行为都会立即导致worker挂起，一次性影响成千上万个连接。将来的工作计划是使nginx中的嵌入式脚本更简单、更可靠，并适用于更广泛的应用范围。

### nginx进程角色 - nginx Process Roles

nginx runs several processes in memory; there is a single master process and several `worker` processes. There are also a couple of special purpose processes, specifically a cache loader and cache manager. All processes are single-threaded in version 1.x of nginx. All processes primarily use shared-memory mechanisms for inter-process communication. The master process is run as the `root` user. The cache loader, cache manager and `worker`s run as an unprivileged user.

​	nginx在内存中运行了多个进程；有一个单独的主进程和若干个`worker`进程。还有一些专用进程，特别是缓存加载器和缓存管理器。1.x版本中的所有进程都是单线程的。所有进程主要使用共享内存机制进行进程间通信。主进程以`root`用户身份运行。缓存加载器、缓存管理器和`worker`以非特权用户身份运行。

The master process is responsible for the following tasks:

​	主进程负责以下任务：

- reading and validating configuration
- creating, binding and closing sockets
- starting, terminating and maintaining the configured number of `worker` processes
- reconfiguring without service interruption
- controlling non-stop binary upgrades (starting new binary and rolling back if necessary)
- re-opening log files
- compiling embedded Perl scripts
- 读取和验证配置
- 创建、绑定和关闭套接字
- 启动、终止和维护配置的`worker`进程数量
- 在不中断服务的情况下重新配置
- 控制非停止二进制升级（启动新的二进制文件并在必要时回滚）
- 重新打开日志文件
- 编译嵌入式Perl脚本

The `worker` processes accept, handle and process connections from clients, provide reverse proxying and filtering functionality and do almost everything else that nginx is capable of. In regards to monitoring the behavior of an nginx instance, a system administrator should keep an eye on `workers` as they are the processes reflecting the actual day-to-day operations of a web server.

​	`worker`进程从客户端接受、处理和处理连接，提供反向代理和过滤功能，以及nginx能够执行的几乎所有其他操作。在监控nginx实例的行为方面，系统管理员应该关注`workers`进程，因为它们是反映Web服务器实际日常运营的进程。

The cache loader process is responsible for checking the on-disk cache items and populating nginx's in-memory database with cache metadata. Essentially, the cache loader prepares nginx instances to work with files already stored on disk in a specially allocated directory structure. It traverses the directories, checks cache content metadata, updates the relevant entries in shared memory and then exits when everything is clean and ready for use.

​	缓存加载器进程负责检查磁盘缓存项，并将nginx的内存数据库填充满缓存元数据。基本上，缓存加载器准备了nginx实例，使其能够使用已经存储在特殊分配的目录结构中的文件。它遍历目录，检查缓存内容元数据，更新共享内存中的相关条目，然后在一切准备就绪时退出。

The cache manager is mostly responsible for cache expiration and invalidation. It stays in memory during normal nginx operation and it is restarted by the master process in the case of failure.

​	缓存管理器主要负责缓存的过期和使其无效。它在正常的nginx操作期间保持在内存中，并在出现故障的情况下由主进程重新启动。

### nginx缓存的简要概述 - Brief Overview of nginx Caching

Caching in nginx is implemented in the form of hierarchical data storage on a filesystem. Cache keys are configurable, and different request-specific parameters can be used to control what gets into the cache. Cache keys and cache metadata are stored in the shared memory segments, which the cache loader, cache manager and `worker`s can access. Currently there is not any in-memory caching of files, other than optimizations implied by the operating system's virtual filesystem mechanisms. Each cached response is placed in a different file on the filesystem. The hierarchy (levels and naming details) are controlled through nginx configuration directives. When a response is written to the cache directory structure, the path and the name of the file are derived from an MD5 hash of the proxy URL.

​	在nginx中，缓存是通过在文件系统上的分层数据存储形式来实现的。缓存键是可配置的，可以使用不同的请求特定参数来控制什么内容进入缓存。缓存键和缓存元数据存储在共享内存段中，缓存加载器、缓存管理器和`worker`进程可以访问这些内存段。目前，除了操作系统的虚拟文件系统机制所暗示的优化外，没有对文件进行内存缓存。每个缓存的响应都被放置在文件系统上的不同文件中。层次结构（级别和命名细节）通过nginx配置指令进行控制。当响应被写入缓存目录结构时，文件的路径和名称是从代理URL的MD5哈希派生的。

The process for placing content in the cache is as follows: When nginx reads the response from an upstream server, the content is first written to a temporary file outside of the cache directory structure. When nginx finishes processing the request it renames the temporary file and moves it to the cache directory. If the temporary files directory for proxying is on another file system, the file will be copied, thus it's recommended to keep both temporary and cache directories on the same file system. It is also quite safe to delete files from the cache directory structure when they need to be explicitly purged. There are third-party extensions for nginx which make it possible to control cached content remotely, and more work is planned to integrate this functionality in the main distribution.

​	将内容放入缓存的过程如下：当nginx从上游服务器读取响应时，内容首先被写入缓存目录结构之外的临时文件中。当nginx完成处理请求时，它会重命名临时文件并将其移动到缓存目录。如果用于代理的临时文件目录位于另一个文件系统上，则文件将被复制，因此建议将临时和缓存目录保持在同一个文件系统上。当需要显式清除文件时，从缓存目录结构中删除文件也是非常安全的。nginx存在第三方扩展，使得可以远程控制缓存内容，并计划在主要发行版中集成此功能。

## 14.3. nginx配置 - 14.3. nginx Configuration

nginx's configuration system was inspired by Igor Sysoev's experiences with Apache. His main insight was that a scalable configuration system is essential for a web server. The main scaling problem was encountered when maintaining large complicated configurations with lots of virtual servers, directories, locations and datasets. In a relatively big web setup it can be a nightmare if not done properly both at the application level and by the system engineer himself.

​	nginx的配置系统受到了Igor Sysoev在Apache上的经验的启发。他的主要洞察是，一个可扩展的配置系统对于一个Web服务器是至关重要的。主要的扩展问题出现在维护大型复杂配置时，其中有许多虚拟服务器、目录、位置和数据集。在相对大型的Web设置中，如果不正确地进行应用程序级别和系统工程师本身的操作，这可能会是一场噩梦。

As a result, nginx configuration was designed to simplify day-to-day operations and to provide an easy means for further expansion of web server configuration.

​	因此，nginx配置旨在简化日常操作，并为进一步扩展Web服务器配置提供简便的方法。

nginx configuration is kept in a number of plain text files which typically reside in `/usr/local/etc/nginx` or `/etc/nginx`. The main configuration file is usually called `nginx.conf`. To keep it uncluttered, parts of the configuration can be put in separate files which can be automatically included in the main one. However, it should be noted here that nginx does not currently support Apache-style distributed configurations (i.e., `.htaccess` files). All of the configuration relevant to nginx web server behavior should reside in a centralized set of configuration files.

​	nginx的配置保存在许多纯文本文件中，通常位于`/usr/local/etc/nginx`或`/etc/nginx`目录中。主配置文件通常称为`nginx.conf`。为了使其不混乱，配置的部分可以放在单独的文件中，这些文件可以自动包含在主文件中。然而，值得注意的是，nginx目前不支持类似Apache的分布式配置（即`.htaccess`文件）。与nginx Web服务器行为相关的所有配置都应该位于一组集中的配置文件中。

The configuration files are initially read and verified by the master process. A compiled read-only form of the nginx configuration is available to the `worker` processes as they are forked from the master process. Configuration structures are automatically shared by the usual virtual memory management mechanisms.

​	配置文件最初由主进程读取和验证。作为从主进程派生出来的`worker`进程，它们在运行时环境中自动共享编译后的只读nginx配置。

nginx configuration has several different contexts for `main`, `http`, `server`, `upstream`, `location` (and also `mail` for mail proxy) blocks of directives. Contexts never overlap. For instance, there is no such thing as putting a `location` block in the `main` block of directives. Also, to avoid unnecessary ambiguity there isn't anything like a "global web server" configuration. nginx configuration is meant to be clean and logical, allowing users to maintain complicated configuration files that comprise thousands of directives. In a private conversation, Sysoev said, "Locations, directories, and other blocks in the global server configuration are the features I never liked in Apache, so this is the reason why they were never implemented in nginx."

​	nginx配置对于`main`、`http`、`server`、`upstream`、`location`（还有`mail`用于邮件代理）指令的不同上下文有几个不同的上下文。这些上下文不会重叠。例如，在`main`指令块中放置`location`块是不可能的。此外，为了避免不必要的歧义，没有类似于“全局Web服务器”配置的东西。nginx的配置旨在简洁和逻辑清晰，允许用户维护复杂的配置文件，包括数千个指令。在私下的对话中，Sysoev说：“全局服务器配置中的位置、目录和其他块是我从未喜欢的Apache功能，所以这就是它们在nginx中从未实现的原因。”

Configuration syntax, formatting and definitions follow a so-called C-style convention. This particular approach to making configuration files is already being used by a variety of open source and commercial software applications. By design, C-style configuration is well-suited for nested descriptions, being logical and easy to create, read and maintain, and liked by many engineers. C-style configuration of nginx can also be easily automated.

​	配置语法、格式和定义遵循所谓的C样式约定。这种特定的配置文件制作方法已经被各种开源和商业软件应用采用。通过设计，C样式配置非常适合嵌套描述，逻辑清晰，易于创建、阅读和维护，并且受到许多工程师的喜爱。nginx的C样式配置也可以很容易地进行自动化。

While some of the nginx directives resemble certain parts of Apache configuration, setting up an nginx instance is quite a different experience. For instance, rewrite rules are supported by nginx, though it would require an administrator to manually adapt a legacy Apache rewrite configuration to match nginx style. The implementation of the rewrite engine differs too.

​	虽然一些nginx指令类似于Apache配置的某些部分，但设置nginx实例的经验是完全不同的。例如，nginx支持重写规则，但需要管理员手动调整传统的Apache重写配置以匹配nginx风格。重写引擎的实现也有所不同。

In general, nginx settings also provide support for several original mechanisms that can be very useful as part of a lean web server configuration. It makes sense to briefly mention variables and the `try_files` directive, which are somewhat unique to nginx. Variables in nginx were developed to provide an additional even-more-powerful mechanism to control run-time configuration of a web server. Variables are optimized for quick evaluation and are internally pre-compiled to indices. Evaluation is done on demand; i.e., the value of a variable is typically calculated only once and cached for the lifetime of a particular request. Variables can be used with different configuration directives, providing additional flexibility for describing conditional request processing behavior.

​	总的来说，nginx的设置还提供了几种非常有用的原始机制，可以作为精简的Web服务器配置的一部分。值得简要提及变量和`try_files`指令，这些在nginx中有些独特。nginx中的变量是为了提供一种额外的、更强大的机制来控制Web服务器的运行时配置。变量经过优化以进行快速评估，并在内部预编译为索引。求值是按需进行的；即变量的值通常只计算一次，并在特定请求的生命周期内进行缓存。变量可以与不同的配置指令一起使用，为描述条件请求处理行为提供了额外的灵活性。

The `try_files` directive was initially meant to gradually replace conditional `if` configuration statements in a more proper way, and it was designed to quickly and efficiently try/match against different URI-to-content mappings. Overall, the `try_files` directive works well and can be extremely efficient and useful. It is recommended that the reader thoroughly check the [`try_files` directive](http://nginx.org/en/docs/http/ngx_http_core_module.html#try_files) and adopt its use whenever applicable.

​	`try_files`指令最初旨在逐步以更适当的方式替代条件的`if`配置语句，并且它旨在快速有效地尝试/匹配不同的URI到内容映射。总体而言，`try_files`指令工作良好，可能非常高效和有用。建议读者在适用时仔细检查[`try_files`指令](http://nginx.org/en/docs/http/ngx_http_core_module.html#try_files)，并采用其使用。

## 14.4. nginx内部结构 - 14.4. nginx Internals

As was mentioned before, the nginx codebase consists of a core and a number of modules. The core of nginx is responsible for providing the foundation of the web server, web and mail reverse proxy functionalities; it enables the use of underlying network protocols, builds the necessary run-time environment, and ensures seamless interaction between different modules. However, most of the protocol- and application-specific features are done by nginx modules, not the core.

​	如前所述，nginx的代码库包括核心和许多模块。nginx的核心负责提供Web服务器、Web和邮件反向代理功能的基础；它使得使用底层网络协议成为可能，构建必要的运行时环境，并确保不同模块之间的无缝交互。然而，大多数协议和应用程序特定的功能都由nginx模块完成，而不是核心。

Internally, nginx processes connections through a pipeline, or chain, of modules. In other words, for every operation there's a module which is doing the relevant work; e.g., compression, modifying content, executing server-side includes, communicating to the upstream application servers through FastCGI or uwsgi protocols, or talking to memcached.

​	在内部，nginx通过模块的流水线或链处理连接。换句话说，对于每个操作，都有一个模块正在执行相关的工作；例如，压缩、修改内容、执行服务器端包含、通过FastCGI或uwsgi协议与上游应用程序服务器通信，或与memcached通信。

There are a couple of nginx modules that sit somewhere between the core and the real "functional" modules. These modules are `http` and `mail`. These two modules provide an additional level of abstraction between the core and lower-level components. In these modules, the handling of the sequence of events associated with a respective application layer protocol like HTTP, SMTP or IMAP is implemented. In combination with the nginx core, these upper-level modules are responsible for maintaining the right order of calls to the respective functional modules. While the HTTP protocol is currently implemented as part of the `http` module, there are plans to separate it into a functional module in the future, due to the need to support other protocols like SPDY (see "[SPDY: An experimental protocol for a faster web](http://www.chromium.org/spdy/spdy-whitepaper)").

​	有几个nginx模块位于核心和真正的“功能”模块之间。这些模块是`http`和`mail`。这两个模块在核心和低级组件之间提供了额外的抽象层次。在这些模块中，实现了与HTTP、SMTP或IMAP等应用层协议相关的事件序列处理。与nginx核心结合使用，这些上层模块负责在相应的功能模块之间保持正确的调用顺序。虽然HTTP协议目前作为`http`模块的一部分实现，但由于需要支持其他协议（如"[SPDY：更快速Web的实验性协议](http://www.chromium.org/spdy/spdy-whitepaper)"）的需要，计划将其分离为将来的功能模块。

The functional modules can be divided into event modules, phase handlers, output filters, variable handlers, protocols, upstreams and load balancers. Most of these modules complement the HTTP functionality of nginx, though event modules and protocols are also used for `mail`. Event modules provide a particular OS-dependent event notification mechanism like `kqueue` or `epoll`. The event module that nginx uses depends on the operating system capabilities and build configuration. Protocol modules allow nginx to communicate through HTTPS, TLS/SSL, SMTP, POP3 and IMAP.

​	功能模块可以分为事件模块、阶段处理程序、输出过滤器、变量处理程序、协议、上游和负载均衡器。其中大部分模块补充了nginx的HTTP功能，虽然事件模块和协议也用于`mail`。事件模块提供特定于操作系统的事件通知机制，如`kqueue`或`epoll`。nginx使用的事件模块取决于操作系统的功能和构建配置。协议模块允许nginx通过HTTPS、TLS/SSL、SMTP、POP3和IMAP进行通信。

A typical HTTP request processing cycle looks like the following.

​	一个典型的HTTP请求处理周期如下所示。

1. Client sends HTTP request.
2. nginx core chooses the appropriate phase handler based on the configured location matching the request.
3. If configured to do so, a load balancer picks an upstream server for proxying.
4. Phase handler does its job and passes each output buffer to the first filter.
5. First filter passes the output to the second filter.
6. Second filter passes the output to third (and so on).
7. Final response is sent to the client.
8. 客户端发送HTTP请求。
9. nginx核心根据配置的位置选择适当的阶段处理程序。
10. 如果配置为这样做，负载均衡器选择上游服务器进行代理。
11. 阶段处理程序完成其工作，并将每个输出缓冲区传递给第一个过滤器。
12. 第一个过滤器将输出传递给第二个过滤器。
13. 第二个过滤器将输出传递给第三个（以此类推）。
14. 最终响应发送给客户端。

nginx module invocation is extremely customizable. It is performed through a series of callbacks using pointers to the executable functions. However, the downside of this is that it may place a big burden on programmers who would like to write their own modules, because they must define exactly how and when the module should run. Both the nginx API and developers' documentation are being improved and made more available to alleviate this.

​	nginx模块调用非常可定制。它通过使用指向可执行函数的指针的一系列回调来执行。然而，这种方法的缺点是，它可能会对希望编写自己的模块的程序员造成很大的负担，因为他们必须明确地定义模块应该如何何时运行。nginx的API和开发人员文档正在不断改进，以减轻这一问题。

Some examples of where a module can attach are:

​	模块可以附加的一些示例位置包括：

- Before the configuration file is read and processed
- For each configuration directive for the location and the server where it appears
- When the main configuration is initialized
- When the server (i.e., host/port) is initialized
- When the server configuration is merged with the main configuration
- When the location configuration is initialized or merged with its parent server configuration
- When the master process starts or exits
- When a new worker process starts or exits
- When handling a request
- When filtering the response header and the body
- When picking, initiating and re-initiating a request to an upstream server
- When processing the response from an upstream server
- When finishing an interaction with an upstream server
- 在读取和处理配置文件之前
- 对于每个出现的位置和服务器的配置指令
- 初始化主配置时
- 初始化服务器（即主机/端口）时
- 当服务器配置与主配置合并时
- 初始化或将位置配置与其父服务器配置合并时
- 主进程启动或退出时
- 当新的工作进程启动或退出时
- 处理请求时
- 在过滤响应标头和正文时
- 选择、启动和重新启动对上游服务器的请求
- 处理来自上游服务器的响应
- 结束与上游服务器的交互时

Inside a `worker`, the sequence of actions leading to the run-loop where the response is generated looks like the following:

​	在`worker`内部，导致生成响应的运行循环的操作序列如下：

1. Begin `ngx_worker_process_cycle()`.
2. Process events with OS specific mechanisms (such as `epoll` or `kqueue`).
3. Accept events and dispatch the relevant actions.
4. Process/proxy request header and body.
5. Generate response content (header, body) and stream it to the client.
6. Finalize request.
7. Re-initialize timers and events.
8. 开始`ngx_worker_process_cycle()`。
9. 使用特定于操作系统的机制（如`epoll`或`kqueue`）处理事件。
10. 接受事件并分派相关操作。
11. 处理/代理请求标头和正文。
12. 生成响应内容（标头、正文）并将其流式传输到客户端。
13. 完成请求。
14. 重新初始化计时器和事件。

The run-loop itself (steps 5 and 6) ensures incremental generation of a response and streaming it to the client.

​	运行循环本身（步骤5和6）确保逐步生成响应并将其流式传输到客户端。

A more detailed view of processing an HTTP request might look like this:

​	处理HTTP请求的更详细视图可能如下所示：

1. Initialize request processing.
2. Process header.
3. Process body.
4. Call the associated handler.
5. Run through the processing phases.
6. 初始化请求处理。
7. 处理标头。
8. 处理正文。
9. 调用相关的处理程序。
10. 经过处理阶段。

Which brings us to the phases. When nginx handles an HTTP request, it passes it through a number of processing phases. At each phase there are handlers to call. In general, phase handlers process a request and produce the relevant output. Phase handlers are attached to the locations defined in the configuration file.

​	这使我们进入了阶段。当nginx处理HTTP请求时，它将其通过多个处理阶段。在每个阶段都有要调用的处理程序。一般来说，阶段处理程序处理请求并生成相关的输出。阶段处理程序附加到配置文件中定义的位置。

Phase handlers typically do four things: get the location configuration, generate an appropriate response, send the header, and send the body. A handler has one argument: a specific structure describing the request. A request structure has a lot of useful information about the client request, such as the request method, URI, and header.

​	阶段处理程序通常执行四项任务：获取位置配置、生成适当的响应、发送标头和发送正文。处理程序有一个参数：描述请求的特定结构。请求结构包含有关客户端请求的大量有用信息，例如请求方法、URI和标头。

When the HTTP request header is read, nginx does a lookup of the associated virtual server configuration. If the virtual server is found, the request goes through six phases:

​	当读取HTTP请求标头时，nginx会查找关联的虚拟服务器配置。如果找到虚拟服务器，则请求会经过六个阶段：

1. server rewrite phase
2. location phase
3. location rewrite phase (which can bring the request back to the previous phase)
4. access control phase
5. try_files phase
6. log phase
7. 服务器重写阶段
8. 位置阶段
9. 位置重写阶段（可能将请求带回到前一个阶段）
10. 访问控制阶段
11. try_files阶段
12. 日志阶段

In an attempt to generate the necessary content in response to the request, nginx passes the request to a suitable content handler. Depending on the exact location configuration, nginx may try so-called unconditional handlers first, like `perl`, `proxy_pass`, `flv`, `mp4`, etc. If the request does not match any of the above content handlers, it is picked by one of the following handlers, in this exact order: `random index`, `index`, `autoindex`, `gzip_static`, `static`.

​	为了生成响应所需的内容，nginx将请求传递给合适的内容处理程序。根据确切的位置配置，nginx可能首先尝试所谓的无条件处理程序，如`perl`、`proxy_pass`、`flv`、`mp4`等。如果请求与上述任何内容处理程序都不匹配，它将由以下处理程序之一选择，按照确切的顺序：`random index`、`index`、`autoindex`、`gzip_static`、`static`。

Indexing module details can be found in the nginx documentation, but these are the modules which handle requests with a trailing slash. If a specialized module like `mp4` or `autoindex` isn't appropriate, the content is considered to be just a file or directory on disk (that is, static) and is served by the `static` content handler. For a directory it would automatically rewrite the URI so that the trailing slash is always there (and then issue an HTTP redirect).

​	有关索引模块的详细信息可以在nginx文档中找到，但这些模块处理具有尾部斜杠的请求。如果专用模块如`mp4`或`autoindex`不合适，则认为内容只是磁盘上的文件或目录（即静态文件），并由`static`内容处理程序提供服务。对于目录，它会自动重写URI，以便始终存在尾部斜杠（然后发出HTTP重定向）。

The content handlers' content is then passed to the filters. Filters are also attached to locations, and there can be several filters configured for a location. Filters do the task of manipulating the output produced by a handler. The order of filter execution is determined at compile time. For the out-of-the-box filters it's predefined, and for a third-party filter it can be configured at the build stage. In the existing nginx implementation, filters can only do outbound changes and there is currently no mechanism to write and attach filters to do input content transformation. Input filtering will appear in future versions of nginx.

​	然后，内容处理程序的内容传递给过滤器。过滤器也附加到位置，并且可以为位置配置多个过滤器。过滤器执行操作是操纵处理程序产生的输出的任务。过滤器执行的顺序在编译时确定。对于开箱即用的过滤器，它是预定义的，对于第三方过滤器，它可以在构建阶段配置。在现有的nginx实现中，过滤器只能执行出站更改，目前没有机制编写和附加过滤器以进行输入内容转换。输入过滤将出现在nginx的将来版本中。

Filters follow a particular design pattern. A filter gets called, starts working, and calls the next filter until the final filter in the chain is called. After that, nginx finalizes the response. Filters don't have to wait for the previous filter to finish. The next filter in a chain can start its own work as soon as the input from the previous one is available (functionally much like the Unix pipeline). In turn, the output response being generated can be passed to the client before the entire response from the upstream server is received.

​	过滤器遵循特定的设计模式。过滤器被调用，开始工作，并调用下一个过滤器，直到调用链中的最终过滤器。在此之后，nginx完成响应的最终化。过滤器不必等待前一个过滤器完成。链中的下一个过滤器可以在前一个输入可用时立即开始自己的工作（在功能上很像Unix管道）。反过来，正在生成的输出响应可以在从上游服务器接收到整个响应之前传递给客户端。

There are header filters and body filters; nginx feeds the header and the body of the response to the associated filters separately.

​	有头部过滤器和正文过滤器；nginx将响应的标头和正文分别传递给相关的过滤器。

A header filter consists of three basic steps:

​	标头过滤器由三个基本步骤组成：

1. Decide whether to operate on this response.
2. Operate on the response.
3. Call the next filter.
4. 决定是否在此响应上进行操作。
5. 对响应进行操作。
6. 调用下一个过滤器。

Body filters transform the generated content. Examples of body filters include:

​	正文过滤器转换生成的内容。正文过滤器的示例包括： 

- server-side includes
- XSLT filtering
- image filtering (for instance, resizing images on the fly)
- charset modification
- `gzip` compression
- chunked encoding
- 服务器端包含
- XSLT过滤
- 图像过滤（例如，实时调整图像大小）
- 字符集修改
- `gzip`压缩
- 分块编码

After the filter chain, the response is passed to the writer. Along with the writer there are a couple of additional special purpose filters, namely the `copy` filter, and the `postpone` filter. The `copy` filter is responsible for filling memory buffers with the relevant response content which might be stored in a proxy temporary directory. The `postpone` filter is used for subrequests.

​	在过滤器链之后，响应传递给写入程序。除了写入程序外，还有一些额外的特殊用途过滤器，即`copy`过滤器和`postpone`过滤器。`copy`过滤器负责将相关响应内容填充到可能存储在代理临时目录中的内存缓冲区中。`postpone`过滤器用于子请求。

Subrequests are a very important mechanism for request/response processing. Subrequests are also one of the most powerful aspects of nginx. With subrequests nginx can return the results from a different URL than the one the client originally requested. Some web frameworks call this an internal redirect. However, nginx goes further—not only can filters perform multiple subrequests and combine the outputs into a single response, but subrequests can also be nested and hierarchical. A subrequest can perform its own sub-subrequest, and a sub-subrequest can initiate sub-sub-subrequests. Subrequests can map to files on the hard disk, other handlers, or upstream servers. Subrequests are most useful for inserting additional content based on data from the original response. For example, the SSI (server-side include) module uses a filter to parse the contents of the returned document, and then replaces `include` directives with the contents of specified URLs. Or, it can be an example of making a filter that treats the entire contents of a document as a URL to be retrieved, and then appends the new document to the URL itself.

​	子请求是请求/响应处理的一个非常重要的机制。子请求也是nginx最强大的方面之一。使用子请求，nginx可以返回与客户端最初请求的URL不同的URL的结果。一些Web框架称之为内部重定向。然而，nginx走得更远——不仅过滤器可以执行多个子请求并将输出合并为单个响应，而且子请求还可以嵌套和分层。子请求可以执行自己的子子请求，子子请求可以启动子子子请求。子请求可以映射到硬盘上的文件、其他处理程序或上游服务器。子请求最有用的是基于原始响应数据插入附加内容。例如，服务器端包含（SSI）模块使用过滤器解析返回文档的内容，然后将`include`指令替换为指定URL的内容。或者，它可以是一个示例，制作一个过滤器，将整个文档的内容视为要检索的URL，然后将新文档附加到URL本身。

Upstream and load balancers are also worth describing briefly. Upstreams are used to implement what can be identified as a content handler which is a reverse proxy (`proxy_pass` handler). Upstream modules mostly prepare the request to be sent to an upstream server (or "backend") and receive the response from the upstream server. There are no calls to output filters here. What an upstream module does exactly is set callbacks to be invoked when the upstream server is ready to be written to and read from. Callbacks implementing the following functionality exist:

​	上游和负载均衡器也值得简要描述一下。上游用于实现可以识别为反向代理（`proxy_pass`处理程序）的内容处理程序。上游模块主要准备将请求发送到上游服务器（或“后端”），并从上游服务器接收响应。这里没有输出过滤器的调用。上游模块的确切功能是在上游服务器准备就绪以便写入和读取时设置回调。存在实现以下功能的回调：

 

- Crafting a request buffer (or a chain of them) to be sent to the upstream server
- 创建要发送到上游服务器的请求缓冲区（或一系列缓冲区）
- Re-initializing/resetting the connection to the upstream server (which happens right before creating the request again)
- 重新初始化/重置与上游服务器的连接（在再次创建请求之前发生）
- Processing the first bits of an upstream response and saving pointers to the payload received from the upstream server
- 处理上游响应的前几位，并保存从上游服务器接收到的有效载荷的指针
- Aborting requests (which happens when the client terminates prematurely)
- 中止请求（当客户端提前终止时发生）
- Finalizing the request when nginx finishes reading from the upstream server
- 中止请求（当客户端提前终止时发生）
- Trimming the response body (e.g. removing a trailer)
- 在终止请求时发生

Load balancer modules attach to the `proxy_pass` handler to provide the ability to choose an upstream server when more than one upstream server is eligible. A load balancer registers an enabling configuration file directive, provides additional upstream initialization functions (to resolve upstream names in DNS, etc.), initializes the connection structures, decides where to route the requests, and updates stats information. Currently nginx supports two standard disciplines for load balancing to upstream servers: round-robin and ip-hash.

​	上游模块还支持分布式响应缓存，通过代理缓存上游服务器的响应来提供更好的性能。这些过滤器在请求/响应处理之间执行，以在上游服务器写入数据并将响应传递给客户端之前操纵响应。一个典型的过滤链包括一个将上游响应保存在内存中的过滤器（`cache`过滤器）、一个过滤器，允许获取从上游服务器获得的实际有效负载（`gzip`或`deflate`过滤器）、一个过滤器，用于获取缓存的有效负载并传递给客户端（`gzip`或`deflate`过滤器）。

Upstream and load balancing handling mechanisms include algorithms to detect failed upstream servers and to re-route new requests to the remaining ones—though a lot of additional work is planned to enhance this functionality. In general, more work on load balancers is planned, and in the next versions of nginx the mechanisms for distributing the load across different upstream servers as well as health checks will be greatly improved.

​	总之，nginx的内部结构允许它高效地处理请求和响应，同时提供了丰富的扩展性和定制性。这种结构允许使用不同的模块来处理各种不同的功能，从而使nginx适用于各种不同的应用场景和工作负载。由于nginx模块的灵活性和可定制性，它可以根据需求进行定制，以满足特定的需求和性能要求。这也使得nginx成为一个强大的工具，用于搭建高性能、可扩展的Web服务器和代理。

There are also a couple of other interesting modules which provide an additional set of variables for use in the configuration file. While the variables in nginx are created and updated across different modules, there are two modules that are entirely dedicated to variables: `geo` and `map`. The `geo` module is used to facilitate tracking of clients based on their IP addresses. This module can create arbitrary variables that depend on the client's IP address. The other module, `map`, allows for the creation of variables from other variables, essentially providing the ability to do flexible mappings of hostnames and other run-time variables. This kind of module may be called the variable handler.

​	此外，还有几个有趣的模块，提供了一组额外的变量，供配置文件使用。尽管nginx中的变量在不同的模块之间创建和更新，但有两个模块完全专门用于变量：`geo`和`map`。`geo`模块用于根据客户端的IP地址跟踪客户端。该模块可以创建依赖于客户端IP地址的任意变量。另一个模块`map`允许从其他变量创建变量，实际上提供了灵活的主机名和其他运行时变量的映射能力。这种类型的模块可以称为变量处理程序。

Memory allocation mechanisms implemented inside a single nginx `worker` were, to some extent, inspired by Apache. A high-level description of nginx memory management would be the following: For each connection, the necessary memory buffers are dynamically allocated, linked, used for storing and manipulating the header and body of the request and the response, and then freed upon connection release. It is very important to note that nginx tries to avoid copying data in memory as much as possible and most of the data is passed along by pointer values, not by calling `memcpy`.

​	单个nginx `worker`内部实现的内存分配机制，在某种程度上受到了Apache的启发。nginx内存管理的高级描述如下：对于每个连接，动态分配必要的内存缓冲区，链接，用于存储和操作请求和响应的标头和正文，然后在连接释放时释放。非常重要的一点是，nginx尽量避免在内存中复制数据，大部分数据都是通过指针值传递的，而不是通过调用`memcpy`来传递的。

Going a bit deeper, when the response is generated by a module, the retrieved content is put in a memory buffer which is then added to a buffer chain link. Subsequent processing works with this buffer chain link as well. Buffer chains are quite complicated in nginx because there are several processing scenarios which differ depending on the module type. For instance, it can be quite tricky to manage the buffers precisely while implementing a body filter module. Such a module can only operate on one buffer (chain link) at a time and it must decide whether to overwrite the input buffer, replace the buffer with a newly allocated buffer, or insert a new buffer before or after the buffer in question. To complicate things, sometimes a module will receive several buffers so that it has an incomplete buffer chain that it must operate on. However, at this time nginx provides only a low-level API for manipulating buffer chains, so before doing any actual implementation a third-party module developer should become really fluent with this arcane part of nginx.

​	稍微深入一点，当模块生成响应时，检索到的内容被放入一个内存缓冲区，然后添加到一个缓冲区链路中。随后的处理也使用这个缓冲链路。nginx的缓冲链路相当复杂，因为有几种处理场景，取决于模块类型的不同。例如，实现正文过滤模块时，精确地管理缓冲区可能相当棘手。这种模块一次只能操作一个缓冲区（链路），并且必须决定是覆盖输入缓冲区，用新分配的缓冲区替换缓冲区，还是在问题缓冲区之前或之后插入新缓冲区。为了使事情变得复杂，有时模块将接收多个缓冲区，以便它具有不完整的缓冲链，必须在上面操作。然而，此时nginx仅提供用于操纵缓冲区链的低级API，因此在进行任何实际实现之前，第三方模块开发人员应该真正熟悉nginx的这一神秘部分。

A note on the above approach is that there are memory buffers allocated for the entire life of a connection, thus for long-lived connections some extra memory is kept. At the same time, on an idle keepalive connection, nginx spends just 550 bytes of memory. A possible optimization for future releases of nginx would be to reuse and share memory buffers for long-lived connections.

​	关于上述方法的说明是，为整个连接的生命周期分配了内存缓冲区，因此对于长寿命的连接，会保留一些额外的内存。同时，在空闲的保持连接状态下，nginx仅花费550字节的内存。nginx未来版本的可能优化是重用和共享长寿命连接的内存缓冲区。

The task of managing memory allocation is done by the nginx pool allocator. Shared memory areas are used to accept mutex, cache metadata, the SSL session cache and the information associated with bandwidth policing and management (limits). There is a slab allocator implemented in nginx to manage shared memory allocation. To allow simultaneous safe use of shared memory, a number of locking mechanisms are available (mutexes and semaphores). In order to organize complex data structures, nginx also provides a red-black tree implementation. Red-black trees are used to keep cache metadata in shared memory, track non-regex location definitions and for a couple of other tasks.

​	管理内存分配的任务由nginx池分配器完成。共享内存区域用于接受互斥锁、缓存元数据、SSL会话缓存以及与带宽管控和管理（限制）相关的信息。nginx中实现了一个内存池分配器来管理共享内存分配。为了允许同时安全地使用共享内存，提供了许多锁定机制（互斥锁和信号量）。为了组织复杂的数据结构，nginx还提供了红黑树实现。红黑树用于在共享内存中保留缓存元数据、跟踪非正则位置定义和用于其他任务的一些情况。

Unfortunately, all of the above was never described in a consistent and simple manner, making the job of developing third-party extensions for nginx quite complicated. Although some good documents on nginx internals exist—for instance, those produced by Evan Miller—such documents required a huge reverse engineering effort, and the implementation of nginx modules is still a black art for many.

​	不幸的是，上述所有内容从未以一致且简单的方式进行描述，使得为nginx开发第三方扩展的工作相当复杂。尽管关于nginx内部的一些良好文档存在——例如Evan Miller等人制作的文档——但这些文档需要巨大的逆向工程工作，而nginx模块的实现对许多人来说仍然是一门黑魔法。

Despite certain difficulties associated with third-party module development, the nginx user community recently saw a lot of useful third-party modules. There is, for instance, an embedded Lua interpreter module for nginx, additional modules for load balancing, full WebDAV support, advanced cache control and other interesting third-party work that the authors of this chapter encourage and will support in the future.

​	尽管与第三方模块开发相关的某些困难，nginx用户社区最近看到了许多有用的第三方模块。例如，nginx中嵌入的Lua解释器模块、用于负载均衡的额外模块、完整的WebDAV支持、高级缓存控制以及其他有趣的第三方工作，这些都受到了nginx用户社区和原始开发人员的高度赞赏和支持。

## 14.5. 总结与经验 - 14.5. Lessons Learned

When Igor Sysoev started to write nginx, most of the software enabling the Internet already existed, and the architecture of such software typically followed definitions of legacy server and network hardware, operating systems, and old Internet architecture in general. However, this didn't prevent Igor from thinking he might be able to improve things in the web servers area. So, while the first lesson might seem obvious, it is this: there is always room for improvement.

​	当Igor Sysoev开始编写nginx时，大部分构建互联网的软件已经存在，这些软件的架构通常遵循传统服务器和网络硬件、操作系统以及旧的互联网架构的定义。然而，这并没有阻止Igor认为他可能能够改进Web服务器领域的事物。因此，虽然第一个教训可能显而易见，但是这个教训是：总是有改进的空间。

With the idea of better web software in mind, Igor spent a lot of time developing the initial code structure and studying different ways of optimizing the code for a variety of operating systems. Ten years later he is developing a prototype of nginx version 2.0, taking into account the years of active development on version 1. It is clear that the initial prototype of a new architecture, and the initial code structure, are vitally important for the future of a software product.

​	怀着改进Web软件的想法，Igor花了很多时间开发初始的代码结构，并研究了优化各种操作系统的代码的不同方法。十年后，他正在开发nginx 2.0版本的原型，考虑到在版本1上的多年积极开发。可以清楚地看出，新架构的初始原型和初始代码结构对于软件产品的未来至关重要。

Another point worth mentioning is that development should be focused. The Windows version of nginx is probably a good example of how it is worth avoiding the dilution of development efforts on something that is neither the developer's core competence or the target application. It is equally applicable to the rewrite engine that appeared during several attempts to enhance nginx with more features for backward compatibility with the existing legacy setups.

​	另一个值得提及的点是，开发应该是有重点的。nginx的Windows版本可能是一个很好的例子，它值得避免在不是开发者核心竞争力或目标应用程序的情况下分散开发工作。这同样适用于在几次尝试中出现的重写引擎，以增加nginx对现有遗留设置的向后兼容性的功能。

Last but not least, it is worth mentioning that despite the fact that the nginx developer community is not very large, third-party modules and extensions for nginx have always been a very important part of its popularity. The work done by Evan Miller, Piotr Sikora, Valery Kholodkov, Zhang Yichun (agentzh) and other talented software engineers has been much appreciated by the nginx user community and its original developers.

​	最后但同样重要的是，值得提到，尽管nginx的开发者社区规模不是很大，第三方模块和扩展一直是其受欢迎程度的一个非常重要的组成部分。Evan Miller、Piotr Sikora、Valery Kholodkov、张毅纯（agentzh）等其他才华横溢的软件工程师所做的工作，受到了nginx用户社区和其原始开发人员的高度赞赏。