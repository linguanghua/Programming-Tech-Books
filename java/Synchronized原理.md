```
 无冥冥之志者，无昭昭之明；无惛惛之事者，无赫赫之功。
 解释：没有专心致志地刻苦学习，就没有融会贯通的智慧；没有埋头执着的工作，就不会取得显著的成就。
```

# Synchronized的原理
&ensp;&ensp;&ensp; 使用Synchronized关键字是Java多线程中比较常见的加锁方式。可以用于修饰对象、方法和代码块，达到加锁的目的。

## 一、使用方式

### 1、修饰实例方法
&ensp;&ensp;&ensp;
synchronized用于修饰一个对象的方法fun，当线程A访问该fun方法，且已经获得了该对象的锁时，此时线程B也想要访问该fun方法，则线程B将会进入阻塞等待状态；而线程C访问当前对象的funSolution方法则不需要阻塞等待。此时Synchronized可称为**对象锁**。

### 2、修饰静态方法
&ensp;&ensp;&ensp;
synchronized用于修饰一个类的静态方法时，当线程A访问该类的fun方法，且获得类锁时，线程B想要访问执行该fun方法，线程B将会进入阻塞等待状态，线程C想要访问该类的funSolution方法，如果funSolution没有被关键字static修饰，则不会进入阻塞等待状态。此时synchronized被称为**类锁**。

### 3、修饰代码块
&ensp;&ensp;&ensp;
synchronized可用于修饰代码块，根据代码块是否被static修饰，synchronized被称为**类锁**或**对象锁**。

## 二、对象头
&ensp;&ensp;&ensp;
在深入了解synchronized的原理之前需要先了解对象的对象头。

&ensp;&ensp;&ensp;
在Java（HotSpot）虚拟机中，对象在内存的存储布局可以分为三个区域：对象头、实例数据、对齐填充。
> 普通对象的对象头包括Mark Word和类型指针；

> 数组类型对象的对象头包括：Mark Word、类型指针和数组长度


&ensp;&ensp;&ensp;
**Mark Word**：用于存储Java对象在运行时的数据信息，包括：哈希码（Hash Code）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间。

&ensp;&ensp;&ensp;
**类型指针**：用于指向对象的类元数据，Java虚拟机将根据类型指针确定该对象属于哪个类的实例。

&ensp;&ensp;&ensp;
**数组长度**：数组类型对象的对象头中特有的数据，表示数组长度。如果是数组类型对象，虚拟机使用3个字宽（word）存储对象头数据。如果是普通对象，则使用两个字宽存储对象头。在32位虚拟机中，一个字宽等于4个字节，32位。

&ensp;&ensp;&ensp;
一个对象需要存储很多的对象头数据，存储这部分数据需要额外的存储成本。Java虚拟机为了更加节省存储空间，将Mark Word设计成了非固定的数据结构。根据对象当前锁状态的情况，存储不同的数据。

<table align="center">
    <tr>
        <td rowspan="2">锁状态</td>
        <td colspan="2" align="center">25bit</td>
        <td rowspan="2" align="center">4bit</td>
        <td align="center">1bit</td>
        <td align="center">2bit</td>
    </tr>
    <tr>
     <td>23bit</td>
     <td>2bit</td>
     <td>是否偏向锁</td>
     <td>锁标志位</td>
    </tr>
    <tr>
        <td>无状态</td>
        <td colspan="2">对象的Hash Code</td>
        <td>分代年龄</td>
        <td align="center">0</td>
        <td align="center">01</td>
    </tr>
    <tr>
        <td>偏向锁</td>
        <td>线程ID</td>
        <td>Epoch</td>
        <td>分代年龄</td>
        <td align="center">1</td>
        <td align="center">01</td>
    </tr>
    <tr>
        <td>轻量级锁</td>
        <td colspan="4" align="center">指向栈中锁记录的指针</td>
        <td align="center">00</td>
    </tr>
    <tr>
        <td>重量级锁</td>
        <td colspan="4" align="center">指向重量级锁的指针</td>
        <td align="center">10</td>
    </tr>
    <tr>
        <td>GC标记</td>
        <td colspan="4" align="center">空</td>
        <td align="center">11</td>
    </tr>
</table>

#### Monitor对象

