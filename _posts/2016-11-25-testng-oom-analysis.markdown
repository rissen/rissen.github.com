最近有个模块的web自动化测试Jenkins Job跑着跑着就OOM，使用的TestNG + WebDriver， 之前这个模块也偶尔遇到过OOM的问题，每次都是调整heap，已经调整到1536了，奇怪的是那么多模块，就单单那一个模块会OOM。鉴于之前屡次遇到OOM的问题，早先已在ant build.xml里TestNG对应的 HeapDumpOnOutOfMemoryError参数给加上了，一旦OOM发生了就会生成 heap dump，如"Dumping heap to java_pid2262.hprof ..."。

```
<jvmarg value="-XX:+HeapDumpOnOutOfMemoryError"/>
```

一直加heap也不是个好的解决方案，打算找到根本原因。可能是内存泄漏，也可能是性能问题。从Jenkins server上把.hprof文件下载到本地，文件大小2G多，也真够大的。Java heap dump分析工具有很多，我用的Eclipse Memory Analyzer，官网 http://www.eclipse.org/mat/ 。当heap dump很大上，Eclipse Memory Analyzer读取dump分析时也会出现OOM的问题，可以修改MemoryAnalyzer.ini调整JVM大小，这里调整到了4096。

```
-vmargs
-Xmx4096m
```

dump文件加载完之后，Eclipse Memory Analyzer并没有发现可疑的内存泄漏，所以这个模块的Jenkins Job还真可能需要更大的内存，然而这并不合理，别的的模块没有发生过OOM的问题，或许是这个模块的code有什么异常。

先分析Histogram，加了filter来过滤这个这个模块的package，发现一些bean的objects数量很惊人，数量最高的一个bean object数量竟然达到了43万多，最少的一个bean EC2UniqueEmployee也有7228个（剩下的bean object都是从EC2UniqueEmployee展开的，EC2UniqueEmployee的field基本都是bean ）。 选取了EC2UniqueEmployee这个bean，然后右键件菜单选择Merge shortest path of selection，这个TestNG用了5个线程执行用例，所以这里列出来5个线程，选取其中一个展开，发现有36个object，然后找到对应的class，找到对应的36个声明，发现在这个模块的testcase基类里面定义了36个EC2UniqueEmployee

```java
protected final EC2UniqueEmployee manager = new EC2UniqueEmployee("xxx","xxx","xxx");
```

展开剩下的4个线程，也都是36个同样的EC2UniqueEmployee声明，但是这并不能解释Object数量会有数万之多，5*36=180， 离7万还远着呢。

接着分析Dominator Tree，同样也加了filter来过滤这个模块的testcase，然后结果也很惊人，竟然有2247，同时只有5个TestNG线程，应该只有5个才对啊。2247刚好是这个模块的TestNG class数量，但是还有另外一个问题，这个Job只执行其中的320条case，为什么全部的TestNG class都被实例化了，而且是存活状态，后来在TestNG的 github上找到一个issue https://github.com/cbeust/TestNG/issues/411#issuecomment-22266087 

>The idea was to know before hand if the suite XML refers to valid classes or not. We weren't handling failure in test instantiation properly, so we ensure that all the classes are instantiated right at the beginning of the execution.

似乎TestNG就是这样设计的，之所以会有有全部testcase的instance，是因为我们使用了TestNG的Listener 通过groups来过滤需要跑的case。

一个简单的计算，(2247-282)*36=7074  刚好接近那个EC2UniqueEmployee object的数量（为什么减掉282，因为其中有282个class没有继承那个基类），这也就解释了为什么那些object的数量如此之多。

简单的解决了这个问题，把那36个final但是非static的EC2UniqueEmployee声明都改为了final static。加了log输出Runtime.getRuntime() 的totalMemory()和freeMemory(), 然后从启动job，发现totalMemory()稳定683M，freeMemory()波动于212M ~ 462M之间。Jenkins Job大概快执行完所有case的时候，登陆到server上生成了一份dump heap，然后同样的分析了下，EC2UniqueEmployee object数量降到了1680降到了只有几千个。Object的总数量从23.7百万降到了2.6百万。

```java
protected final static EC2UniqueEmployee manager = new EC2UniqueEmployee("xxx","xxx","xxx");
```

![image](../images/mat_overview.png)

![image](../images/mat_histogram.png)

