# 内存数据区域划分
## 2图
![title](https://raw.githubusercontent.com/bk201sama/imagesBD/master/gitnote/2020/02/26/JVM%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA%E5%9F%9F-1582700740491.png)![title](https://raw.githubusercontent.com/bk201sama/imagesBD/master/gitnote/2020/02/26/2019-3Java%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA%E5%9F%9FJDK1.8-1582700751066.png)
## 要点
1. 程序计数器功能：实现代码的流程控制；实现多线程时的切换记录之前执行到的位置。
2. 程序计数器是唯一一个不会出现OutOfMemoryError的内存区域。
3. 虚拟机栈功能：存放函数栈帧，每个栈帧上有局部变量表，操作数栈，动态链接，方法出口信息。
4. 虚拟机栈会出现StackOverFlowError(不许动态扩展，但是超过栈深度)
OutOfMemoryError（允许动态扩展，无法再动态扩展了)
5. 函数调用压入栈帧，return与抛异常栈帧被弹出。
6. 本地方法栈功能：存放Native方法的栈帧,hotspot虚拟机中与虚拟机栈合二为一。
7. java堆（由于是垃圾收集器管理的主要区域也叫GC堆）的唯一目的就是存放对象实例与数组。
8. GC堆从采用分代垃圾收集算法细分为新生代和老年代
9. jdk7以及jdk之前堆分为新生代，老生代，永生代
10. 新生代分eden区，from Survivor和to Survivor 
11. 新生代升老年代有2种标准，取最小的那个。标准1-可配的老年代阀值；标准2-按年龄从小到大排，某年龄组超过survivor区一半，作为晋升年龄阈值的一个标准。
12. 堆最容易产生OutOfMemoryError；java.lang.OutOfMemoryError: Java heap space表示堆内存不足；OutOfMemoryError: GC Overhead Limit Exceeded 如果GC花费的时间超过 98%, 并且GC回收的内存少于 2%, JVM就会抛出这个错误。
13. 方法区（非堆）存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。在hotspot虚拟机的实现就是永久代。
14. 元空间默认最大内存只受系统内存的限制。可以通过调节参数进行限制。起始内存如果不设置动态调节。
# hotspot虚拟机对象
## 对象创建
![title](https://raw.githubusercontent.com/bk201sama/imagesBD/master/gitnote/2020/02/26/Java%E5%88%9B%E5%BB%BA%E5%AF%B9%E8%B1%A1%E7%9A%84%E8%BF%87%E7%A8%8B-1582710411187.png)

