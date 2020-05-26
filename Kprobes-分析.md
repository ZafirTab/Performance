# Kernel Probes (Kprobes)

## 概念: Kprobes, and Return Probes
使用Kprobes,可以帮助你动态的打断内核的执行，并且可以无中断的收集调试和性能信息。你可以捕获几乎任何的内核态地址，并可以指定陷入时需要执行的程序。

目前有两种探测类型：kprobes, and kretprobes(also called return probes). kprobes可以插入到内核中的任一指令，而kretprobes是在函数返回点调用。

比较典型的例子是，基于kprobes的探测指令通常会编成一个ko，通过register_kprobe来注册，有注册就有解注册，这时候就需要用到unregister_*probes()。

## Kprobe是如何工作的？
当一个kprobe注册时，Kprobes会用一个中断指令（需要根据int3 on i386 and x86_64不同的架构）替换目标指令同时做一个指令拷贝，我们可以理解为是保护现场。
当CPU执行到替换后的中断指令时，会发生陷入，CPU的寄存器会被保存起来. 与Kprobes的交互是notifier_call_chain(通知链表是一个函数链表，链表上的每一个节点都注册了一个函数。
当某个事情发生时，链表上所有节点对应的函数就会被执行。有兴趣的可以看下源码kernel/notifier.c)

简单来说,Kprobes分为保护现场(保存当前的addr并替换成异常指令)和恢复现场(恢复之前替换的指令,继续执行),这其中会用到pre_handler和post_handler.

```
// 内核中的Kprobes定义
struct kprobe {
	struct hlist_node hlist;

	/* list of kprobes for multi-handler support */
	struct list_head list;

	/*count the number of times this probe was temporarily disarmed */
	unsigned long nmissed;

	/* location of the probe point */
	kprobe_opcode_t *addr;

	/* Allow user to indicate symbol name of the probe point */
	const char *symbol_name;

	/* Offset into the symbol */
	unsigned int offset;

	/* Called before addr is executed. */
	kprobe_pre_handler_t pre_handler;

	/* Called after addr is executed, unless... */
	kprobe_post_handler_t post_handler;

	/*
	 * ... called if executing addr causes a fault (eg. page fault).
	 * Return 1 if it handled fault, otherwise kernel will see it.
	 */
	kprobe_fault_handler_t fault_handler;

	/* Saved opcode (which has been replaced with breakpoint) */
	kprobe_opcode_t opcode;

	/* copy of the original instruction */
	struct arch_specific_insn ainsn;

	/*
	 * Indicates various status flags.
	 * Protected by kprobe_mutex after this kprobe is registered.
	 */
	u32 flags;
};

```

## 改变程序的执行路径
Kprobes可以替换你的指令,但这务必要小心.

## Return Probes


















