---
title: GrowingIO面试
categories:
- 面试
tags: [迁移自CSDN博客]
---

1.简历是一位老哥推荐的，都是网易在实习的，他也知道我后来转Scala了，刚好这家公司是全Scala公司，所以就问了有没有兴趣，此时我在趣头条实习，干的是测试开发，所以也想试试。

我大概是去年2017.11年开始逛牛客的，我基本只刷选择题，主要是计算机网络和Java基础。PS:即使后来基本没有被问到（仅网易实习生面试的时候被问到），当然了我是有基础的，刷题属于是复习。我在2017.7月之前就已经学完 java servlet jsp，但当时没有去学习SSM，直到开学才开始使用SSM做项目，
也就是大三，2017.10月（大三我没有选大数据，但此时已经在学Scala了，当初学习的目标是JVM上另一语言作为后手，也作为以后自转大数据的后手）。
经历了春招和秋招后我觉得自己不适合写Java，，但是又不能失业对吧，我就准备找一份稍微轻松又能有时间让我继续学Scala的公司，这是我发现了测开这个职位。此后简称自己是摸鱼鱼。趣头条测开分业务线，有的纯粹是点点点，有的是工具开发。
我一开始进去是分到了业务线，后来调到工具开发。在这一点我觉得公司很人性化，遵循求职者的意愿，后面实习几个月确实感觉很好（在我说毁约的时候HR还告诉我去试试GO PHP，所以我觉得就公司而言，企业文化和管理是不错的，
一开始自己都不好意思说要毁约，因为一开始也没想半吊子水平能这么快找到Scala开发，所以准备先干两年的），可能唯一缺陷就是公司主GO PHP了。实习的几个月基本是在做自动化（PS:就是兼容国内所有手机的自动化shell开发，写shell写的头皮发麻，简直比兼容WEB浏览器还难百倍）。
因为是实习而且是测开，所以我下班不是很晚（PS:不晚是和网易实习时每天10点多比。哈哈），于是晚上还会去学习Scala，Akka,这也是我一个半吊的Scala面过GrowingIo的一个因素。我计划是自学一到两年转岗，目前对我来说来的比较早，且还算合适。毕竟错误发现的越晚，需要的成本越高。
在这脚本写多了，根本不会写Java了，跟别说Scala和服务端开发了。这主要是测开对代码的性能 标准化 等等要求都不高，目测行业如此。我们组其他人都是全栈。。。

我个人的建议是，喜欢写写代码，但又不是很喜欢，还想轻松点，可以考虑去测开，如果以后想做开发，则不建议，测开这个岗位，我同事也提醒我说，公司就把你当测试，测开代码要求 版本控制 开发流程和 开发还是有很多区别，在这里很尴尬。。所以这个不中不下的确实不太符合我的个人意愿。准确的来说与我想象的有差异。
而且有的测开就是测试，点点点，选测开确实需要勇气。除非以后真的决定做自动化 脚本开发，，，那就去吧。测试开发基本是全栈加linux运维。。。但是都不会要求很深，也是最害怕的广而不精

面试约的都是晚上，毕竟白天上班，不好走开。

一、一面是一个技术小哥哥，手机信号不好，后来用了微信视频，我以为是我手机坏了，很迷。原来是房间没信号，大上海太可怕了...可能是因为我的是联通卡

一面很基础，不问项目

1.实习做什么

2.说说单例 ，Scala的单例 

3.动态代理了解吗 JDK CGLIB ,区别

4.CGLIB使用了什么？

5.用过的函数式编程的函数 map flatmap filter future

6.简述BIO NIO AIO

7.讨论BIO NIO区别

8.NIO运行有几个线程

9.CountDownLatch CyclicBarrier Semaphore 举例说明Semaphore

谈实际使用的

10.jvm如何调优  如何解决线上debug  jps jstack等使用 。（后来我发现线上可能是需要Btrace）

11.其他多线程的 好像是 wait notify之类的。。

12.小算法 括号匹配

13.有什么要问我的

之所以不问Spring SpringBoot我觉得是因为公司用不着。。。



二、二面是架构师的视频面 给我的感觉是压力面，有点紧张答得不好

这一面基本都是没有直接的问题，没有问题是最大的问题，因为他在看我的破绽，哈哈。我觉得这种让我说，他不先问的面试都很难回答，毕竟我已经出简历了，会什么写好了，我说的要是不符合面试官的兴趣，或者说太少都不太好

1.介绍自己的项目

2.把我GITHUB的IO小工具项目都问起来了，你的IO小工具解决了什么问题，为了解决什么问题，RabbitMQ在这里的作用，一连串问。。。

3.集合了解吗？挑一个自己说 hashmap  get put

4.多线程了解吗？自己说说

5.Linux熟悉吗 工作最常用哪些，你用这个命令解决了什么问题？

6.怎么学习的

7.算法 任务等待一定的序列进行依次执行 没太明白题目 说了join 但是好像是一个线程中的几个任务之间的等待序列，答错了

8.使用了Akka哪些功能，解决了什么问题

9.你的Actor如何设计，为什么，解决了什么问题

还有记不得了。。也是50多分钟。。FaceTime视频

三面、CTO面，HR提前给我发了CTO的博客，我去了解了看了看，害怕万一被问。一看叶玎玎。。我去 大佬。。

本来以为这一面不是技术面，结果还是，迷

首先面试官上来介绍自己，介绍的很详细。接下说想了解我，我就知道进入技术题了。。

1.大学最熟悉的课程？（挖坑）

2.挑一个最熟悉的，（数据结构）（c语言）

3.说说最常用的数据结构，说说很有用但是不常用的数据结构（堆栈 平衡树 B树 B+ 树 挖坑。。）

4.数据库索引  以及索引执行查找时具体流程 

5.情境题 数据库表结构设计  并讨论 理由 优缺点 优化方案 举出实际的record例子 和理由

6.你想成为什么样的人

7.你对自己的定位

8.看什么书 （不太记得了是不是这么问的）

9.有什么想问我的

三面聊了50多分钟，，都是开放式的题目，题目都忘了，只记得这几个了

。。站在窗户旁边甚至有的问题也没听清。室内信号不太好



HR面，谈面试官 面试感受 等等 扯皮子的。。。



面的杭州，杭州没有HC，准备调岗去大数据，后来考虑我没有Spark Hadoop基础，所以叫我去北京干服务器端，熟悉业务后让我转Spark 也可以回杭州。。（好像意思是说杭州没几个人，目前已经招到了，而且事情不多，暂时不需要更多人）

结束摸鱼生活~