# 背景
>从3.18版本开始，Linux 内核提供了一种扩展的BPF虚拟机，被称为“extended BPF“，简称为eBPF。它能够被用于非网络相关的功能，比如附在不同的tracepoints上，从而获取当前内核运行的许多信息。
>
>传统的BPF，现在被称为“classical BPF”。主要用于过滤网络包，Android中的DHCP数据包，就使用BPF来进行过滤。
>
>eBPF由Alexei Starovoitov在PluMgrid工作时设计，这家公司专注于研究新的方法来设计软件定义网络解决方案。在它只是一个提议时，Daniel Borkmann——Red Hat公司的内核工程师，帮助修改使得它能够进入内核代码并完全替代已有的BPF实现。这是二十年来BPF首次主要的更新，使得BPF成为了一个通用的虚拟机。
>
>因为eBPF虚拟机使用的是类似于汇编语言的指令，对于程序编写来说直接使用难度非常大。和将C语言生成汇编语言类似，现在的编译器正在逐步完善从更高级的语言生成BPF虚拟机使用的指令。LLVM在3.7版本开始支持BPF作为后端输出。GCC 10也将会支持BPF作为后端。BCC是IOVisor项目下的编译器工具集，用于创建内核跟踪（tracing）工具。bpftrace是为eBPF设计的高级跟踪语言，在Linux内核（4.x）中提供。
>

*学习新知识的时候，先有一个宏观的认识，再去深究其中的细节。*

# 什么是eBPF?
>在讲eBPF之前，先简要说下BPF（Berkeley Packet Filter，缩写 BPF），其是类Unix系统上数据链路层的一种原始接口，提供原始链路层封包的收发。除此之外，如果网卡驱动支持混杂模式，那么它可以让网卡处于此种模式，这样可以收到网络上的所有包，不管他们的目的地是不是所在主机。
另外，BPF支持过滤数据包——用户态的进程可以提供一个过滤程序来声明它想收到哪些数据包。通过这种过滤可以避免从操作系统内核向用户态复制其他对用户态程序无用的数据包，从而极大地提高性能。

eBPF是一种内核注入技术，开发者可以使用高级语言（GO、Python、C等）来编写eBPF代码，然后注入到kernel中运行。

首先，我们需要编写代码，为了编码的方便，就出现了像[BCC](https://github.com/iovisor/bcc)、[bpftrace](https://github.com/iovisor/bpftrace)这样的框架来方便我们coding。

编写完代码后，我们需要编译，目前可以使用clang、llvm（GCC正在开发中），生成的elf文件，使用kernel提供的load接口loading到内核态。
*可以使用readelf来查看编译的.o*

其次，loading到kernel的程序，必须通过verify，这是必要且必须的操作，内核态的东西不是想怎么执行都可以的。[verifier.c 如果想详细了解检查机制，没有别阅读源码更有效的了](https://elixir.bootlin.com/linux/v5.6.2/source/kernel/bpf/verifier.c)

最后，“程序=数据结构+算法”，这是不变的真理；
eBPF的数据结构就是MAP，它新开辟了一个VFS（BPF_FS），用来用户态和内核态进行交互，你可以在/sys/fs/bpf下面找到定义的MAP和PROG。
eBPF的算法，可以有kprobe、tracepoint、perf、netfilter等等，这些都是原有的内核跟踪技术，eBPF就像ftrace一样，再一次把它们聚在了一起。

一句话，eBPF既是一种内核注入技术，又是一种内核跟踪技术，也是一种调试框架。你可以在学习理论的同时，使用它来跟踪Linux的行为，更为形象和直观的了解操作系统。

# 使用eBPF能做什么?
eBPF现在被应用于网络、跟踪、内核优化、硬件建模等领域。

# 如何学习eBPF?
* 诚然，现在网上已经很多关于eBPF的文章和资料了，但我还是推荐以下两本书。

[入门：Linux Observability with BPF Advanced Programming for Performance Analysis and Networking ](http://gen.lib.rus.ec/book/index.php?md5=B684AC84FE625207115FAFB72E322651)
[进阶：BPF Performance Tools](http://gen.lib.rus.ec/book/index.php?md5=4FF4065A0C77DD48CDE5A6FF0B14CFF2)
## 参考资料
阅读代码的同时，参考这两个也足够了。
[内核官方文档，没有比这更正宗的了，直接reading it](https://www.kernel.org/doc/Documentation)
[Brendan Gregg Blog](http://brendangregg.com/overview.html)


