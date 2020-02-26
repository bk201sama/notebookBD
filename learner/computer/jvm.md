# 内存数据区域划分
## 2图
![title](https://raw.githubusercontent.com/bk201sama/imagesBD/master/gitnote/2020/02/26/JVM%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA%E5%9F%9F-1582700740491.png)![title](https://raw.githubusercontent.com/bk201sama/imagesBD/master/gitnote/2020/02/26/2019-3Java%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA%E5%9F%9FJDK1.8-1582700751066.png)
## 要点
1. 程序计数器功能：实现代码的流程控制；实现多线程时的切换记录之前执行到的位置。
2. 程序计数器是唯一一个不会出现OutOfMemoryError的内存区域。
3. 虚拟机栈功能：放栈帧，每个栈帧上有局部变量表，操作数栈，动态链接，方法出口信息。
4. 虚拟机栈会出现StackOverFlowError(不许动态扩展，但是超过栈深度)
OutOfMemoryError（允许动态扩展，无法再动态扩展了)
5. 函数调用压入栈帧，return与抛异常栈帧被弹出。
6. 本地方法栈功能：存放Native方法的功能hotspot虚拟机中合二为一。

