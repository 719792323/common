# 基本使用

#### 1. 启动arthas

```java
java -jar arthas-boot.jar
```

- 执行该程序的用户需要和目标进程具有相同的权限。比如以`admin`用户来执行：`sudo su admin && java -jar arthas-boot.jar` 或 `sudo -u admin -EH java -jar arthas-boot.jar`。
- 如果 attach 不上目标进程，可以查看`~/logs/arthas/` 目录下的日志。

#### 2 选择需要attach的进程

```java
$ $ java -jar arthas-boot.jar
* [1]: 35542
  [2]: 71560 math-game.jar
```

`math-game`进程是第 2 个，则输入 2，再输入`回车/enter`。Arthas 会 attach 到目标进程上。

# 核心命令

#### trace

https://arthas.aliyun.com/doc/trace.html#%E5%8F%82%E6%95%B0%E8%AF%B4%E6%98%8E

方法内部调用路径，并输出方法路径上的每个节点上耗时

`trace` 命令能主动搜索 `class-pattern`／`method-pattern` 对应的方法调用路径，渲染和统计整个调用链路上的所有性能开销和追踪调用链路。

|            参数名称 | 参数说明                                                     |
| ------------------: | :----------------------------------------------------------- |
|     *class-pattern* | 类名表达式匹配                                               |
|    *method-pattern* | 方法名表达式匹配                                             |
| *condition-express* | 条件表达式                                                   |
|                 [E] | 开启正则表达式匹配，默认为通配符匹配                         |
|              `[n:]` | 命令执行次数（如果方法调用的次数很多，那么可以用`-n`参数指定捕捉结果的次数。比如下面的例子里，捕捉到一次调用就退出命令） |
|             `#cost` | 方法执行耗时（trace demo.MathGame run '#cost > 10'，只会展示耗时大于 10ms 的调用路径，有助于在排查问题的时候，只关注异常情况） |
|         `[m <arg>]` | 指定 Class 最大匹配数量，默认值为 50。长格式为`[maxMatch <arg>]`。 |

`trace` 能方便的帮助你定位和发现因 RT 高而导致的性能问题缺陷，但其每次只能跟踪一级方法的调用链路。

##### trace 多个类或者多个函数

trace 命令只会 trace 匹配到的函数里的子调用，并不会向下 trace 多层。因为 trace 是代价比较贵的，多层 trace 可能会导致最终要 trace 的类和函数非常多。

可以用正则表匹配路径上的多个类和函数，一定程度上达到多层 trace 的效果。

```bash
trace -E com.test.ClassA|org.test.ClassB method1|method2|method3
```

##### trace 结果时间不准确问题

比如下面的结果里：`0.705196 > (0.152743 + 0.145825)`

```bash
$ trace demo.MathGame run -n 1
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 66 ms, listenerId: 1
`---ts=2021-02-08 11:27:36;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@232204a1
    `---[0.705196ms] demo.MathGame:run()
        +---[0.152743ms] demo.MathGame:primeFactors() #24
        `---[0.145825ms] demo.MathGame:print() #25
```

那么其它的时间消耗在哪些地方？

1. 没有被 trace 到的函数。比如`java.*` 下的函数调用默认会忽略掉。通过增加`--skipJDKMethod false`参数可以打印出来。

   ```bash
   $ trace demo.MathGame run --skipJDKMethod false
   Press Q or Ctrl+C to abort.
   Affect(class count: 1 , method count: 1) cost in 35 ms, listenerId: 2
   `---ts=2021-02-08 11:27:48;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@232204a1
       `---[0.810591ms] demo.MathGame:run()
           +---[0.034568ms] java.util.Random:nextInt() #23
           +---[0.119367ms] demo.MathGame:primeFactors() #24 [throws Exception]
           +---[0.017407ms] java.lang.StringBuilder:<init>() #28
           +---[0.127922ms] java.lang.String:format() #57
           +---[min=0.01419ms,max=0.020221ms,total=0.034411ms,count=2] java.lang.StringBuilder:append() #57
           +---[0.021911ms] java.lang.Exception:getMessage() #57
           +---[0.015643ms] java.lang.StringBuilder:toString() #57
           `---[0.086622ms] java.io.PrintStream:println() #57
   ```

2. 非函数调用的指令消耗。比如 `i++`, `getfield`等指令。

3. 在代码执行过程中，JVM 可能出现停顿，比如 GC，进入同步块等

排除掉指定的类，使用 `--exclude-class-pattern` 参数可以排除掉指定的类，比如：

```bash
trace javax.servlet.Filter * --exclude-class-pattern com.demo.TestFilter
```

#### watch

https://arthas.aliyun.com/doc/watch.html#%E4%BD%BF%E7%94%A8%E5%8F%82%E8%80%83

函数执行数据观测（参数与返回值）

|            参数名称 | 参数说明                                                     |
| ------------------: | :----------------------------------------------------------- |
|     *class-pattern* | 类名表达式匹配                                               |
|    *method-pattern* | 函数名表达式匹配                                             |
|           *express* | 观察表达式，默认值：`{params, target, returnObj}`            |
| *condition-express* | 条件表达式                                                   |
|                 [b] | 在**函数调用之前**观察                                       |
|                 [e] | 在**函数异常之后**观察                                       |
|                 [s] | 在**函数返回之后**观察                                       |
|                 [f] | 在**函数结束之后**(正常返回和异常返回)观察                   |
|                 [E] | 开启正则表达式匹配，默认为通配符匹配                         |
|                [x:] | 指定输出结果的属性遍历深度，默认为 1，最大值是 4（观察入参和返回值时如果不显示想要的则加深x） |
|         `[m <arg>]` | 指定 Class 最大匹配数量，默认值为 50。长格式为`[maxMatch <arg>]`。 |

```shell
# location=AtExit表明函数正常返回，如果location=AtExceptionExit则说明函数抛出了异常
# 如下result是默认是{params, target, returnObj}
# 即 @Object[][isEmpty=true;size=0],为函数参数
# @Controller...为对应对象
# @String[hello]为返回参数
[arthas@15648]$ watch open.demo.task1.controller.Controller task1
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 79 ms, listenerId: 1
method=open.demo.task1.controller.Controller.task1 location=AtExit
ts=2024-02-03 15:39:46; [cost=0.7285ms] result=@ArrayList[
    @Object[][isEmpty=true;size=0],
    @Controller[open.demo.task1.controller.Controller@69164200],
    @String[hello],
]
```

##### 耗时

```shell
$ watch demo.MathGame primeFactors '{params, returnObj}' '#cost>200' -x 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 66 ms.
ts=2018-12-03 19:40:28; [cost=2112.168897ms] result=@ArrayList[
    @Object[][
        @Integer[1],
    ],
    @ArrayList[
        @Integer[5],
        @Integer[428379493],
    ],
]
```

- `#cost>200`(单位是`ms`)表示只有当耗时大于 200ms 时才会输出，过滤掉执行时间小于 200ms 的调用
- cost表达式之前一定要给观察表达式，否则没办法进行过滤

#### stack

输出当前方法被调用的调用路径

|            参数名称 | 参数说明                                                     |
| ------------------: | :----------------------------------------------------------- |
|     *class-pattern* | 类名表达式匹配                                               |
|    *method-pattern* | 方法名表达式匹配                                             |
| *condition-express* | 条件表达式                                                   |
|                 [E] | 开启正则表达式匹配，默认为通配符匹配                         |
|              `[n:]` | 执行次数限制                                                 |
|         `[m <arg>]` | 指定 Class 最大匹配数量，默认值为 50。长格式为`[maxMatch <arg>]`。 |

##### 据条件表达式来过滤

```bash
$ stack demo.MathGame primeFactors 'params[0]<0' -n 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 30 ms.
ts=2018-12-04 01:34:27;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@3d4eac69
    @demo.MathGame.run()
        at demo.MathGame.main(MathGame.java:16)

ts=2018-12-04 01:34:30;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@3d4eac69
    @demo.MathGame.run()
        at demo.MathGame.main(MathGame.java:16)

Command execution times exceed limit: 2, so command will exit. You can set it with -n option.
```

##### [#](https://arthas.aliyun.com/doc/stack.html#据执行时间来过滤)据执行时间来过

```bash
$ stack demo.MathGame primeFactors '#cost>5'
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 35 ms.
ts=2018-12-04 01:35:58;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@3d4eac69
    @demo.MathGame.run()
        at demo.MathGame.main(MathGame.java:16)
```

#### jad

反编译已加载类的源码

|              参数名称 | 参数说明                                   |
| --------------------: | :----------------------------------------- |
|       *class-pattern* | 类名表达式匹配                             |
|                `[c:]` | 类所属 ClassLoader 的 hashcode             |
| `[classLoaderClass:]` | 指定执行表达式的 ClassLoader 的 class name |
|                   [E] | 开启正则表达式匹配，默认为通配符匹配       |

```shell
# 反编译
jad 对应类的全路径名称
# 只显示反编译对应的源代码，不显示多余内容
jad --source-only 对应类的全路径名称
# 反编译对应类中对应方法的代码
jad 类的全路径名称 方法名称
```

当有多个 `ClassLoader` 都加载了这个类时，`jad` 命令会输出对应 `ClassLoader` 实例的 `hashcode`，然后你只需要重新执行 `jad` 命令，并使用参数 `-c <hashcode>` 就可以反编译指定 ClassLoader 加载的那个类了；

```java
$ jad org.apache.log4j.Logger

Found more than one class for: org.apache.log4j.Logger, Please use jad -c hashcode org.apache.log4j.Logger
HASHCODE  CLASSLOADER
69dcaba4  +-monitor's ModuleClassLoader
6e51ad67  +-java.net.URLClassLoader@6e51ad67
            +-sun.misc.Launcher$AppClassLoader@6951a712
            +-sun.misc.Launcher$ExtClassLoader@6fafc4c2
2bdd9114  +-pandora-qos-service's ModuleClassLoader
4c0df5f8  +-pandora-framework's ModuleClassLoader

Affect(row-cnt:0) cost in 38 ms.
$ jad org.apache.log4j.Logger -c 69dcaba4

ClassLoader:
+-monitor's ModuleClassLoader

Location:
/Users/admin/app/log4j-1.2.14.jar

package org.apache.log4j;

import org.apache.log4j.spi.*;

public class Logger extends Category
{
    private static final String FQCN;

    protected Logger(String name)
    {
        super(name);
    }

...

Affect(row-cnt:1) cost in 190 ms.

```

#### thread

查看当前线程信息，查看线程的堆栈

|      参数名称 | 参数说明                                                |
| ------------: | :------------------------------------------------------ |
|          *id* | 线程 id                                                 |
|          [n:] | 指定最忙的前 N 个线程并打印堆栈                         |
|           [b] | 找出当前阻塞其他线程的线程                              |
| [i `<value>`] | 指定 cpu 使用率统计的采样间隔，单位为毫秒，默认值为 200 |
|       [--all] | 显示所有匹配的线程                                      |

```shell
# 打印当前最忙的3个线程的堆栈信息
thread -n 3
# 查看ID为1都线程的堆栈信息
thread 1
# 找出当前阻塞其他线程的线程
thread -b
# 查看指定状态的线程
thread -state WAITING
```



# 实践案例

### 1.定位当前高cpu占用线程与相关代码

dashboard+thread+jad命令

https://blog.csdn.net/oWhatever12/article/details/126677166

### 2.定位时间范围内CPU使用情况,进而分析高占用方法

profile命令

https://blog.csdn.net/weixin_31257709/article/details/131300641?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-131300641-blog-126677166.235%5Ev43%5Epc_blog_bottom_relevance_base5&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-131300641-blog-126677166.235%5Ev43%5Epc_blog_bottom_relevance_base5&utm_relevant_index=2

### 3.定位某方法一段时间内调用频率与一些统计数据

monitor命令

https://blog.csdn.net/weixin_31257709/article/details/131300641?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-131300641-blog-126677166.235%5Ev43%5Epc_blog_bottom_relevance_base5&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-131300641-blog-126677166.235%5Ev43%5Epc_blog_bottom_relevance_base5&utm_relevant_index=2

### 4.内存占用过大排查

可以使用heapdump命令导出当前堆情况再使用jprofile工具分析