# CalFuzzer: 并行程序的可扩展主动测试框架

## 介绍

最近引入了主动测试来有效地测试并发程序。 主动测试可以迅速发现真实的数据争用，死锁和原子性违规。 主动测试分为两个阶段。 它首先使用不精确的现成静态或动态程序分析来识别潜在的并发错误，例如数据争用，死锁和原子性违规。 在第二阶段，主动测试使用来自这些不精确分析的报告来显式控制并发程序的基础调度程序，从而以非常高的概率和很少的开销准确，快速地发现实际的并发错误（如果有）。 CalFuzzer实现了一个可扩展框架，用于主动测试Java程序。

## 系统要求

Windows或Linux或Mac OSX。您需要预安装用于Windows或Linux的Sun的JDK 1.5，或Apple的Mac OS X的最新JDK。还需要Apache的[ANT](http://ant.apache.org/) 进行构建和安装并运行您的代码。

## 安装

```
git clone ...
cd calfuzzer
ant
```
    
## 测试
运行以下命令，来查看RaceFuzzer和DeadlockFuzzer的运行情况。

```    
ant -f run.xml racefuzzer
ant -f run.xml deadlockfuzzer
```

## 如何编写自定义分析？

查看工具文件activetool.pdf，我们还创建了一个简单的[作业分配](http://sp09.pbwiki.com/RaceFuzzer-Homework)，以指导您完成RaceFuzzer的实现。

查看

```
calfuzzer/src/javato/activetesting/HybridAnalysis.java
calfuzzer/src/javato/activetesting/RaceFuzzerAnalysis.java
```

来了解RaceFuzzer的实现。请注意，这不是PLDI’08论文中报告的优化实施。 有关如何调用RaceFuzzer的详细信息，请参见run.xml中的目标“racefuzzer”和目标“test_sor”。

类似的，查看

```
calfuzzer/src/javato/activetesting/IGoodlockAnalysis.java
calfuzzer/src/javato/activetesting/DeadlockFuzzerAnalysis.java
```

来了解DeadlockFuzzer的实现。请注意，这不是PLDI’09论文中报告的优化实施。 有关如何调用DeadlockFuzzer的详细信息，请参见run.xml中的目标“ deadlockfuzzer”和目标“test_6”。

类似的，查看

```
calfuzzer/src/javato/activetesting/PAtomicityAnalysis.java
calfuzzer/src/javato/activetesting/RaceFuzzerAnalysis.java
```

来了解AtomFuzzer的实现。请注意，这不是FSE'09论文中报告的实现，而是FSE 09和CAV 09中报告的技术的组合。有关如何调用AtomFuzzer的详细信息，参见run.xml中的目标“atomfuzzer”和目标“test_atomfuzzer15”。 

## 参考资料

 *  Pallavi Joshi, M. Naik, C.-S. Park, and K. Sen, "An Extensible Active Testing Framework for Concurrent Programs," in Proc. 21st International Conference on Computer Aided Verification (CAV'09), 2009. 

 *  Pallavi Joshi, C.-S. Park, K. Sen, and M. Naik, "A Randomized Dynamic Program Analysis Technique for Detecting Real Deadlocks," in Proc. ACM SIGPLAN Conference on Programming Language Design and Implementation (PLDI'09), 2009. 

 *  Chang-Seo Park and K. Sen, "Randomized Active Atomicity Violation Detection in Concurrent Programs," in Proc. 16th International Symposium on Foundations of Software Engineering (FSE'08), 2008. 

 * Koushik Sen, "Race Directed Random Testing of Concurrent Programs," in Proc. ACM SIGPLAN Conference on Programming Language Design and Implementation (PLDI'08), 2008, pp. 11-21. 

 *  Koushik Sen, "Effective Random Testing of Concurrent Programs," in Proc. 22nd IEEE/ACM nternational Conference on Automated Software Engineering (ASE'07), 2007, pp. 323-332. 
  
