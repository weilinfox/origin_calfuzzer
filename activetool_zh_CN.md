# CalFuzzer: An Extensible Active Testing Framework for Concurrent Programs
# CalFuzzer：并行程序的可扩展主动测试框架
 
Pallavi Joshi 1 , Mayur Naik 2 , Chang-Seo Park 1 , and Koushik Sen 1

1 美国加州大学伯克利分校 {pallavi,parkcs,ksen}@eecs.berkeley.edu

2 英特尔研究院 mayur.naik@intel.com

**摘要** 最近引入了主动测试来有效测试并发程序。主动测试分为两个阶段，它首先使用不精确的现成静态或动态程序分析来识别潜在的并发错误，例如数据竞争，死锁和原子性违规。在第二阶段，主动测试使用来自这些不精确分析的报告来显式控制并发程序的基础调度程序，从而以非常高的概率和很少的开销准确、快速地发现实际的并发错误（如果有）。在本文中，我们提出了一个可扩展的框架，用于主动测试Java程序。该框架当前基于数据竞争，原子块，死锁和用户指定的断点实现四个活动测试器。

## 1 介绍

由于执行线程之间的意外干扰，多线程程序经常表现出错误的行为。此类并发错误（例如数据竞争和死锁）通常很难找到，因为它们通常是在执行线程的非常特定的交错下发生的。测试并发程序的传统方法是重复执行程序，希望不同的测试执行会导致不同的交错。这种方法存在一些问题。首先，在特定环境中执行的测试通常无法提出可能在其他环境中（例如在不同的系统负载下）可能发生的交错。其次，测试取决于底层操作系统或虚拟机的线程
调度-它不会尝试显式控制线程调度；因此，它通常最终会执行相同的交错多次。

已经开发的许多程序分析技术[2、5、1、4]，以通过检测对常用同步惯用法的违反来预测多线程程序中的并发错误。例如，在不持有公共锁的情况下访问内存位置可用于预测该位置上的数据竞争，而程序的锁顺序图中的周期可用于预测死锁。这些技术在发现并发错误方面非常有效，因为它们可以预测实际执行期间可能发生的错误-对于此类预测，它们不需要查看实际的执行情况（在静态分析的情况下），或者仅需要查看表现出违反惯用法而不必表现出错误本身的执行（在动态分析的情况下）。但是，尽管有最近的进步，但是这些技术经常报告许多错误警告。遍历所有这些警告并对其进行手动推理通常很耗时。

最近，我们提出了一种用于在并发程序中查找实际错误的新技术，称为主动测试[8，6，3]。主动测试使用随机线程调度程序来验证由不精确的静态或动态程序分析报告的警告是否为实际错误。该技术的工作原理如下。主动测试首先使用现有的不精确的现成静态或动态分析技术（例如Lockset [7，5]，Atomizer [1]或Goodlock [2]）来计算一组潜在的并发错误。每个潜在的并发错误通常由一组程序语句标识。例如，在数据争用的情况下，该集合包含两个程序语句，它们可能在某些并发执行中彼此竞争。对于每个潜在的并发错误，主动测试会在随机计划下运行被测并发程序。此外，当线程到达潜在并发错误中涉及的语句时，主动测试会通过暂停任何线程的执行来偏向随机调度。暂停线程后，活动测试还会检查一组暂停的线程是否可能创建了真正的并发错误。例如，在数据争用的情况下，主动测试会检查两个暂停的线程是否将要访问相同的内存位置，并且其中至少有一个是写操作。因此，主动测试试图迫使程序制定一个时间表，在该时间表中实际发生并发错误。在我们之前的工作中，我们已经开发了主动测试算法来检测实际数据竞争，原子性违规和死锁。


在本文中，我们描述了一种用于主动测试并发Java程序的可扩展工具，称为CalFuzzer。 CalFuzzer提供了一个框架，用于实施自定义计划程序以进行主动测试。我们将这些自定义计划程序称为活动检查器。该工具还提供了框架和库，用于实现各种不精确的动态分析技术，例如Lockset，Atomizer，和Goodlock。可以使用这些动态分析技术来计算与错误有关的一组程序语句。CalFuzzer还允许用户手动指定一组程序语句，在该语句中，活动检查器应暂停。这样的陈述可以被认为是“并发断点”。


我们在CalFuzzer中实现了三个活动检查器，用于检测实际数据竞争，原子性违规和死锁。 我们还在CalFuzzer中实现了三种动态分析技术。它们是hybrid race detector[5]，用于发现潜在原子违反的Atomizer [1]和用于检测潜在死锁的iGoodlock [3]。 CalFuzzer提供了一种 ``ActiveChecker.Check()`` 方法，用户可以在其并发程序中使用该方法在程序中指定任意“暂停”点（或“断点”）。

我们已将CalFuzzer应用到几个并发的Java基准测试程序中，这些程序总共包含60万行代码，并且检测到了先前已知和未知的数据竞争，原子性违规和死锁。CalFuzzer可以轻松扩展为检测其他种类的并发错误，例如丢失的通知和违反原子集的行为。

**算法 1** 带有用户定义 ``analysis()`` and ``check()`` 方法的CalFuzzer

```
Inputs: 初始状态s0和一组转换断点

paused := ∅
s := s0
while Enabled(s) 6= ∅ do
	t := a random transition in Enabled(s) \ paused
	analysis(t)
	if t ∈ breakpoints then
		paused := check(t, paused )
	end if
	if t 6∈ paused then
		s := Execute(s, t)
	end if
	if paused = Enabled(s) then
		remove a random element from paused
	end if
end while
if Alive(s) 6= ∅ then
	print “ERROR: system stall”
end if
```


## 2 主动测试框架

在本节中，我们对主动测试框架进行了高级描述。我们考虑由有限的一组线程组成的并发系统。给定并发状态s，让 ``Enabled(s)`` 表示在状态s中启用的过渡集。每个线程执行一系列转换，并通过共享对象与其他线程通信。我们假设每个线程在执行有限数量的转换后终止。并发系统通过从一种状态过渡到另一种状态而发展。如果s是并发状态，而t是过渡，则 ``Execute(s, t)`` 在状态s中执行过渡t并返回更新后的状态。

算法1中的伪代码描述了CalFuzzer算法。该算法将初始状态s0和一组称为断点的转换（表示潜在的并发错误）作为输入。暂停的过渡集将初始化为空集。从初始状态s0开始，在每个状态下，CalFuzzer都会随机选择一个在该状态下启用且未出现在已暂停集合中的过渡。然后，它调用用户定义的方法分析以执行用户定义的动态分析，例如Lockset，Atomizer或Goodlock。分析方法可以保持自己的局部状态；例如，在混合种族检测的情况下，本地状态可以保持锁集和向量时钟。如果转换t在设置的断点中，则CalFuzzer调用用户提供的方法检查，该方法将t和暂停的集合作为输入并返回更新的暂停的集合。该检查方法可用于实现各种活动检查器。check方法的典型实现可以将t添加到暂停的集合中，并从暂停的集合中删除一些过渡。调用check之后，如果尚未将过渡t添加到check方法的暂停集合中，则CalFuzzer将执行过渡t。在每次迭代结束时，如果所有启用的过渡都已暂停，则CalFuzzer将从暂停的集合中删除随机过渡。当系统达到没有启用的过渡的状态时，算法将终止。在终止时，如果至少有一个线程处于活动状态，则该算法将报告系统停顿。因此，CalFuzzer采用两种用户定义的方法：分析和检查。为了实施一种主动的测试技术，需要定义这两种方法。例如，一种针对数据竞争的主动测试技术[8]要求我们在分析方法和检查方法中实现混合种族检测，如算法2所示。

**算法 2** 主动测试数据竞争的检查方法

```
Inputs: 过渡t和一组过渡已暂停

if ∃t0 ∈ paused such that t and t0 access same location and one of them is write
then
	print “Real data race between t and t0 ” (* next resolve race randomly to check
if something could go wrong due to the race *)
	if random boolean then
		add t to paused and remove t0 from paused
	end if
else
	add t to paused
end if

return paused
```


## 3 实施细节

我们已经为Java实现了CalFuzzer主动测试框架。CalFuzzer（可从 <http://srl.cs.berkeley.edu/~ksen/calfuzzer/> 获得）使用Java字节码在各种同步操作和共享内存访问之前或之后插入以下回调函数：startBefore，joinAfter，waitAfter，notifyBefore， notifyAllBefore，lockBefore，unlockAfter，readBefore，writeBefore。这些回调函数实现为调用各种动态分析和活动检查器。不精确的动态分析通常是通过实现一个称为Analysis的接口来实现的。该接口包含上述回调函数。

通过扩展下面声明的类ActiveChecker来实现活动检查器。

```java
public class ActiveChecker {
	final public static Object lock = new Object ();
	final protected void block (int milliSeconds) { ... }
	final protected void unblock (int milliSeconds ) { ... }
	final public static void blockIfRequired () { ... }
	public void check (Collection<ActiveChecker> checkers) {
		block(0);
	}
	final public void check () { ... }
}
```


``ActiveChecker`` （例如RaceChecker）子类的实例等效于算法1中的转换。此类定义的 ``check()`` 方法等效于算法1中使用的check方法。依次调用方法 ``check(Collection<ActiveChecker> checkers)`` 。子类
ActiveChecker的方法应覆盖 ``check(Collection<ActiveChecker> checkers)`` 方法。默认实现会阻止（即暂停）当前线程（即ActiveChecker对象表示的过渡）。用户可以在所分析的代码中使用 ``ActiveChecker.Check()`` 来指示“断点”。该类还提供其他方法，例如 ``block(int milliSeconds)`` 暂停当前线程， ``unblock(int milliSeconds)`` 继续被暂停的线程。 这些方法应在 ``check(Collection<ActiveChecker> checkers)``方法的自定义实现中使用。

该框架还提供了各种实用程序类，例如 ``VectorClockTracker`` 和 ``LocksetTracker`` 来计算矢量时钟和运行时的锁集。这些类的方法在上述各种回调函数中调用。这些实用程序类用于hybrid race detection[5]和iGoodlock[3]算法中。 其他用户定义的动态分析也可以使用这些实用程序类。

CalFuzzer的工具修改了除了用于实现CalFuzzer的类之外的，与Java程序相关联的所有字节码，包括其使用的库。这是因为CalFuzzer与正在分析的程序在相同的内存空间中运行。CalFuzzer无法通过本机代码跟踪锁的获取和释放。因此，如果在非仪器类或本机代码中进行同步操作，CalFuzzer可能会进入死锁。为避免此类情况，CalFuzzer运行了一个低优先级的监视线程，该线程会定期轮询以检查是否存在死锁。如果监视器发现死锁，它将从暂停的集中删除一个随机过渡。

CalFuzzer也可以进入活锁。当除了一个线程在循环中执行某些操作而不与其他线程进行同步之外，程序的所有线程最终都处于暂停集中时，就会发生活锁。我们在包括moldyn在内的几个基准中观察到了这种活锁。在存在活锁的情况下，这些基准测试可以正常工作，因为这些基准测试的正确性假定基础Java线程调度程序是公平的。为了避免出现活锁，CalFuzzer创建了一个监视线程，该线程会定期从等待较长时间的暂停集中删除那些过渡。

## 4 结论

表1总结了在多个实际Java程序上运行主动测试的一些结果。在[8，6，3]中有更多详细信息。请注意，活动检查程序报告的错误（例如RaceFuzzer，AtomFuzzer和DeadlockFuzzer）是真正的并发错误，而动态分析报告的错误（例如hybrid race detection，iGoodlock和Atomizer）可能是错误的警告。

|-|-|报告|bug|的|数量|||
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|基准测试|LoC|HRD|RaceFuzzer|iGoodlock|DeadlockFuzzer|Atomizer|AtomFuzzer|
|jspider|10,252|29|0|0|0|28|4|
|sor|17,718|8|0|0|0|0|0|
|hedc|25,024|9|1|0|0|3|0|
|jigsaw|160,388|547|36|283|29|60|2|
|Java Swing|337,291|-|-|1|1|-|-|

**表1** 基准测试结果: LoC is Lines of Code; HRD is Hybrid Race Detection

CalFuzzer提供了一个框架，可用于编写自定义的随机调度程序，从而可以快速找到实际的错误。目前，我们已经在此框架中实现了三个活动检查器，并且我们相信CalFuzzer提供了一个简单的可扩展框架，用于试验其他活动检查器和动态分析技术。

## 参考文献

1. C. Flanagan and S. N. Freund. Atomizer: a dynamic atomicity checker for multithreaded programs. In 31st ACM SIGPLAN-SIGACT Symposium on Principles of Programming Languages (POPL), pages 256–267, 2004.
2. K. Havelund. Using runtime analysis to guide model checking of java programs. In 7th International SPIN Workshop on Model Checking and Software Verification, pages 245–264, 2000.
3. P. Joshi, C.-S. Park, K. Sen, and M. Naik. A randomized dynamic program analysis technique for detecting real deadlocks. In ACM SIGPLAN Conference on Programming Language Design and Implementation (PLDI’09) (to appear), 2009.
4. M. Naik, A. Aiken, and J. Whaley. Effective static race detection for java. In ACM SIGPLAN Conference on Programming Language Design and Implementation, pages 308–319, 2006.
5. R. O’Callahan and J.-D. Choi. Hybrid dynamic data race detection. In ACM SIGPLAN symposium on Principles and practice of parallel programming, pages 167–178. ACM, 2003.
6. C.-S. Park and K. Sen. Randomized active atomicity violation detection in concurrent programs. In SIGSOFT ’08/FSE-16: Proceedings of the 16th ACM SIGSOFT International Symposium on Foundations of software engineering, pages 135–145, New York, NY, USA, 2008. ACM.
7. S. Savage, M. Burrows, G. Nelson, P. Sobalvarro, and T. E. Anderson. Eraser: A dynamic data race detector for multithreaded programs. ACM Trans. Comput. Syst., 15(4):391–411, 1997.
8. K. Sen. Race directed random testing of concurrent programs. In Programming Language Design and Implementation (PLDI’08), 2008.
