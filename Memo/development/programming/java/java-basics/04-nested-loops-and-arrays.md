---
type: memo
date: 2026-11-09
aliases:
  - "04-qian-tao-xun-huan-shu-zu-er-wei-shu-zu"
  - "04 嵌套循环、数组、二维数组·笔记"
---

# 嵌套循环与数组

1.3.5 嵌套循环

各位同学，接下来我们学习一种在实际工作中很常用的循环形式——循环嵌套。

## 嵌套循环语法

所谓循环嵌套，就是一个循环中又包含另一个循环（就是同学们常说的，套娃^\_^）。循环嵌套执行流程：**外部循环每循环一次，内部循环会全部执行完一轮**。

下面我们通过案例代码演示一下嵌套循环：

```Java
for(int i = 0; i < 3; i++) {

     for(int j = 0; j < 5; j++) {
        System.out.println("我爱你");
     }

 }
```

上面的代码使用了嵌套的`for`循环。外层循环控制变量`i`从0到2（共3次迭代），内层循环控制变量`j`从0到4（共5次迭代）。每次外层循环迭代时，内层循环会完整执行一次。最终会输出15行 “我爱你”。

**详细执行流程：**

1. 初始化外层循环：

   - `i` 被初始化为 0。
2. 检查外层循环条件：

   - 检查 `i < 3` 是否成立。如果成立，则进入外层循环体；如果不成立，则退出外层循环。
3. 进入内层循环：

   - `j` 被初始化为 0。
   - 检查 `j < 5` 是否成立。如果成立，则进入内层循环体；如果不成立，则退出内层循环。
4. 执行内层循环体：

   - 打印 "我爱你" 到控制台。
5. 更新内层循环变量：

   - `j` 增加 1 (`j++`)。
6. 再次检查内层循环条件：

   - 重复步骤 3 到 5，直到 `j` 不再小于 5。
7. 退出内层循环：

   - 当 `j` 达到 5 时，退出内层循环。
8. 更新外层循环变量：

   - `i` 增加 1 (`i++`)。
9. 再次检查外层循环条件：

   - 重复步骤 2 到 8，直到 `i` 不再小于 3。
10. 退出外层循环：

    - 当 `i` 达到 3 时，退出外层循环。

## 嵌套循环案例

### 打印矩形

需求：通过for循环，在屏幕上输出以下内容

![SIFLbemlPo7obrxzevvcDwMPnRb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/SIFLbemlPo7obrxzevvcDwMPnRb.png)

实现思路：

- 声明2个变量，一个用于控制行数（i），一个用于控制每行的列数（j）
- 每遇到一个新行，就循环j次，输出j个星号

```Java
package com.heima;

/**
 * 测试：循环嵌套
 *
 * 需求：使用*在控制台打印4行5列的矩形
 */
public class Demo20 {
    public static void main(String[] args) {
        //几行
        int n = 4;
        //几列
        int m = 5;

        //外循环
        for (int i=1; i<=n; i++){
            //System.out.println("*****");
            //内循环
            for(int j=1; j<=m; j++ ){
                System.out.print("*");
            }
            System.out.println();
        }
    }
}
```

### 打印三角形

 需求：通过for循环，在屏幕上输出以下内容

![MxBNbxEjBoVbGWxX6GVcpWGgnbg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/MxBNbxEjBoVbGWxX6GVcpWGgnbg.png)

实现思路：

- 声明2个变量，一个用于控制行数（i），一个用于控制每行的列数（j）
- 列的次数随着行的增加而增加, 所以列(j)需要和行(i)有关系才行

```Java
package com.heima;

/**
 * 测试：循环嵌套
 *
 * 需求：使用*在控制台打印4行5列的矩形
 */
public class Demo20 {
    public static void main(String[] args) {
        //几行
        int n = 4;
        //外循环
        for (int i = 0; i < n; i++) {
            //内循环的次数随之i的变大而增加
            for (int j = 0; j <= i; j++) {
                System.out.print("*");
            }
            System.out.println();
        }
    }
}
```

### 找质数

需求：判断101-200之间有多少素数，并输出所有质数。

说明：

- 除了1和它本身以外，不能被其他正整数整除的数，就是质数。
- 比如：3、7就是质数，而9、21不是质数。

思路分析：

- 由于判断范围是101-200之间的数字，循环次数已经确定，可以使用for循环进行每个数字的判断
- 定义一个方法isPrime(int num)，接收一个参数（数字），在方法内判断该数是否为质数，如果是返回true，否则返回false
- 方法内部逻辑：接收到数字后，判断 [2 \~ 该数-1] 区间内，是否有数能够整除该数，有则不是质数
- 在101-200循环中，对每一个数字调用isPrime方法，判断是否为质数

整体代码实现：

```Java
package com.heima.demo;

public class Test4 {
    public static void main(String[] args) {
        // 目标：找出101-200之间的全部质数。
        int count = 0;
        // 1、遍历101-200
        for (int i = 101; i <= 200; i++) {
            // i = 101 102 103 103 ... 200
            // 2、每遍历到一个数字，判断这个数字是否是质数，是则输出。（独立功能独立成方法）
            if (isPrime(i)) {
                System.out.println(i);
                count++;
            }
        }
        System.out.println("质数的个数为：" + count);
    }

    public static boolean isPrime(int num) {
        // num = 101
        // 定义一个循环从2开始找到该数的一半，只要发现有一个数字能整数num这个数，num就不是质数
        // 如果都没有找到，那么num就是质数
        for (int i = 2; i <= num / 2; i++) {
            // i = 2 3 4 5 ...
            // 2、判断num是否能被i整除，能则返回false，不能则返回true
            if (num % i == 0) {
                return false;
            }
        }
        return true; // 是质数
    }
}
```

## 数组

### 认识数组

好的，同学们，今天我们来学习Java程序里的数据存储方案，同学们都知道程序是用来处理数据的，那我们肯定需要把程序需要处理的数据拿到程序里存储起来才可以处理吧，目前大家其实已经接触过存储数据的一种存储方案，大家知道是啥吗？ 对，是变量。

但变量每次只能记录一个数据，而我们开发中可能需要同时处理大批量的数据，比如咱们全班所有学生的名字，或者说一张图片图片上的每个像素点都是一个颜色数据，那遇到这样大批量的数据，咱们再用变量肯定不太合适，比如你存储全班学生的名字你就需要定义60个变量，存储一张图片数据可能需要上万个变量，显然不合适。那Java已经为批量数据的存储提供了一些方案比如说数组，二维数组，后面还要学习的对象，集合等等。

今天我们先来带大家快速掌握数组的用法。

咱们还是以解决实际需求为导向来学习，就以这个随机点名的需求为例，咱们先考虑假如果用原来变量的方案来解决可能存在的问题，再看AI会怎么做？？

使用通义千问，向AI提问：“我是一名Java初学者，帮我开发一个随机点名的方法，假设有15个学生，请使用数组来存储学生姓名”，然后生成的代码中会有数组的语法，可以先对数组有个直观的认识。

![W1x1bVY2qokglDxJZHWcb9Y2nKb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/W1x1bVY2qokglDxJZHWcb9Y2nKb.png)

通过观察，我们得知：数组是一个数据容器，可用来存储一批同类型的数据。那么数组究竟应该怎么用呢？下面我们进行具体的学习。

### 数组的静态初始化

要使用数组，必须先创建数组，在java中有两种创建数据的方式：静态初始化、动态初始化，本小节我们先来学习静态初始化。

在Java中，数组的静态初始化是指在声明数组的同时直接给数组元素赋值。这种方式允许你在创建数组时就指定每个元素的具体值。数组的静态初始化语法如下：

```Java
数据类型[]  数组名 = new 数据类型[]{元素1，元素2 ，元素3… };
```

示例：

```Java
//定义数组，用来存储多个年龄
int[] ages = new int[]{12, 24, 36}

//定义数组，用来存储多个成绩
double[] scores = new double[]{89.9, 99.5, 59.5, 88.0};
```

以上是数组静态初始化的标准语法，此语法还可以简写（可以省略等号后面的 new int[] ）：

```Java
数据类型[] 数组名 = { 元素1，元素2 ，元素3，… };
int[] arr= {12, 24, 36};

```

完整格式和简化格式的区别?

```Java
完整格式可以先定义后赋值, 但是简化格式不行
例如:
  int[] ages = new int[]{12, 24, 36}
可以写成:
  int[] ages;
  ages =  int[]{12, 24, 36};
  但是简化格式不行:
  int[] ages;
  ages = {12, 24, 36}; //会摆错的
```

数组创建后，数据在内存中是连续存放的，如下图所示：

![MHXxb0EMroGRClx38rdcQEiunMb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/MHXxb0EMroGRClx38rdcQEiunMb.png)

### jvm的内存划分

1. **虚拟机栈（VM Stack）**

- 作用：描述 Java 方法执行的内存模型，每个方法调用时会创建一个栈帧（Stack Frame） 入栈，方法执行完毕后出栈。
- 栈帧包含：

  - 局部变量表（存放方法参数和局部变量）。
  - 操作数栈（方法执行时的临时数据栈）。
  - 动态链接（指向方法区中该方法的符号引用）。
  - 方法返回地址（方法执行完后回到调用处的地址）。

1. **堆（Heap）**

- 作用：存储 Java 对象实例（几乎所有的对象和数组都在这里分配内存），是 JVM 管理的内存中最大的一块。
- 特点：

  - 线程共享，全局唯一。
  - 是垃圾回收（GC）的主要区域（“GC 堆” 由此得名）。
  - 内存不足时会抛出`OutOfMemoryError`。
- 细分：

  - 年轻代（Young Generation）：

    - Eden 区：新对象优先分配到这里，大对象可能直接进入老年代（取决于 JVM 参数）。
    - Survivor 区：分为 From Survivor 和 To Survivor，用于存放 Eden 区 GC 后存活的对象，通过复制算法实现对象年龄增长。
  - 老年代（Old Generation）：存放存活时间较长的对象（默认年龄阈值为 15，可通过参数调整），GC 频率较低，采用标记 - 清除或标记 - 整理算法。

1. **方法区（Method Area）**

- 作用：存储类信息（如类的结构、字段、方法、接口）、常量、静态变量、即时编译器（JIT）编译后的代码等。
- 特点：

  - 线程共享，逻辑上属于堆的一部分，但具体实现独立（如 HotSpot 的元空间）。
  - 内存不足时会抛出`OutOfMemoryError: Metaspace`（JDK 8 及以后）。
- 版本差异：

  - JDK 7 及之前：称为 “永久代（PermGen）”，物理上属于堆，有固定大小限制。
  - JDK 8 及之后：用元空间（Metaspace） 替代永久代，元空间默认使用本地内存（而非 JVM 堆内存），理论上受限于系统内存。

1. **本地方法栈（Native Method Stack）**

- 作用：与虚拟机栈类似，但专门为 native 方法（如 C/C++ 实现的方法）提供内存支持。
- 特点：

  - 线程私有，具体实现依赖 JVM（如 HotSpot 将虚拟机栈和本地方法栈合并实现）。
  - 可能抛出`StackOverflowError`或`OutOfMemoryError`。

1. **程序计数器（Program Counter Register）**

- 作用：记录当前线程执行的字节码指令的地址（行号），CPU 切换线程时通过它恢复执行位置。
- 特点：

  - 线程私有，每个线程有独立的程序计数器。
  - 是 JVM 中唯一不会抛出`OutOfMemoryError`的区域。
  - 如果线程执行的是 native 方法，计数器值为`undefined`（因为 native 方法由本地系统执行，不对应字节码）。

### 数组的访问

通过刚才的学习，我们知道数组是用来存储数据的。那么数组中存储的数据又如何访问呢？这里所说的访问，意思就是获取中数组中数据的值、或者给数组中的数据赋值。

这里先给大家统一几个概念，数组中存储的数据我们叫做元素；而且**数组中的每一个元素都有一个编号与之对应，我们把这个编号叫做索引**，这个索引是从0依次递增的整数。如下图所示

![EqBqbqTygo6r5gxlOD3cTx9TnUS](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/EqBqbqTygo6r5gxlOD3cTx9TnUS.png)

要想访问数组中的元素，格式如下：

```Plain Text
数组名[索引]
```

下面通过代码，演示数组元素的访问语法：

```Java
// 创建数组
int[] arr = {12, 24, 36};

// 1、访问数组的全部数据
System.out.println(arr[0]); //12
System.out.println(arr[1]); //24
System.out.println(arr[2]); //36

//下面代码没有3索引，会出现ArrayIndexOutOfBoundsException 索引越界异常
//System.out.println(arr[3]);

// 2、修改数组中的数据
arr[0] = 66;
arr[2] = 100;
System.out.println(arr[0]); //66
System.out.println(arr[1]); 0
System.out.println(arr[2]); //100
```

除了访问数组中的元素，我们可以获取数组中元素的个数，后面我们统称为**数组的长度**。

```Java
// 3、访问数组的元素个数：数组名.length
System.out.println(arr.length);

// 技巧：获取数组的最大索引: arr.length - 1(前提是数组中存在数据)
System.out.println(arr.length - 1);

int[] arr2 = {};
System.out.println(arr2.length - 1);
```

### 数组的动态初始化

刚才我们初始化数组时，都是直接将元素写出来。但有些场景下，创建数组时我们尚不知道具体数据。比如有一个需求：通过键盘录入8名学生的成绩，并存到数组中，且成绩是小数。此时我们只知道有8个数据，数据类型是小数（float 或 double），但具体成绩是多少我们并不知道。那么此时，我们应该如何创建这个数组呢？

这就需要用到另一个初始化数组的方式叫 **动态初始化**。动态初始化不需要我们写出具体的元素，而是指定元素类型和长度就行。格式如下：

```Java
数据类型[]  数组名 = new 数据类型[长度];
```

数组的动态初始化示例：

```Java
// 创建一个数组，可以存储3个整数，但具体的数值尚未可知
int[] arr = new int[3];
```

虽然我们创建数组时，并未指定数组中的数据。但数组在初始化时，会自动为元素提供默认值，具体规则如下图所示：

![YQYqbawfLofzCYxzWTfcX2sLnXe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/YQYqbawfLofzCYxzWTfcX2sLnXe.png)

### 数组的遍历

接下来我们学习一个对数组最最最常见的操作——数组遍历。所谓遍历意思就是将数组中的元素一个一个的取出来。

我们刚才学习了数组中元素的访问，访问元素必须用到索引，如下列代码。

```Java
int[] ages = {12, 24, 36};
System.out.println(ages[0]);
System.out.println(ages[1]);
System.out.println(ages[2]);
```

但是，如果数组中有很多很多元素，索引靠自己一个一个数肯定是不行的！我们可以使用for循环从0开始一直遍历到长度-1的位置，就可以获取所有的索引了。 当你有了每一个元素的索引，那么就可以获取到每一个元素值了。以下代码演示了数组的遍历：

```Java
int[] ages = {12, 24, 36};

for (int i = 0; i < ages.length; i++) {
    // i的取值 = 0,1,2
    System.out.println(ages[i]);
}
```

### 数组案例-求最值

前面我们学习了数组的基本语法，接下来我们来完成一个数组中最常见的功能：求最值。也就是找出数组中的最大值或最小值。

在Java中，求一个数组中的最大值或最小值是一个常见的操作。这个过程可以通过遍历整个数组来完成，比较每个元素与当前已知的最大值或最小值。下面是求最值的基本思路：

**（1）求最大值**

1. 初始化：假设数组的第一个元素是当前最大值。
2. 遍历数组：从第二个元素开始遍历数组的每一个元素。
3. 比较和更新：对于数组中的每一个元素，如果该元素大于当前最大值，则更新当前最大值为该元素。
4. 结果：遍历完成后，当前最大值即为整个数组的最大值。

**（2）求最小值**

求最小值的过程与求最大值非常类似，只是在比较时寻找更小的值：

1. 初始化：假设数组的第一个元素是当前最小值。
2. 遍历数组：从第二个元素开始遍历数组的每一个元素。
3. 比较和更新：对于数组中的每一个元素，如果该元素小于当前最小值，则更新当前最小值为该元素。
4. 结果：遍历完成后，当前最小值即为整个数组的最小值。

求最值示例：在给定数组中找出最大值，实现代码如下：

```Java
package com.heima;

/**
 * 需求：数组元素求最大值
 */
public class Demo {
    public static void main(String[] args) {
        //声明数组
        int[] array = {11,55,33,88,22};

        //最大值
        int max = array[0];

        //遍历数组,寻找最大值
        for (int i = 0; i <array.length ; i++) {
            if( array[i]>max ){ //碰到比max大的值,就替换max的旧值
                max = array[i];
            }
        }

        //输出结果
        System.out.println("最大值是:" + max);

    }
}
```

### 数组案例-斗地主游戏

需求：开发一个简易版的斗地主游戏，要求只完成做牌（存储54张牌）、洗牌。

![Wi97bOPXposbTbxhC1DcRBtlnBe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Wi97bOPXposbTbxhC1DcRBtlnBe.png)

思路分析：

- 可以动态初始化一个数组，再存入54张牌到数组中，方便处理。
- 扑克牌中，每个数字包含4种花色（红桃、黑桃、梅花、方片），可以使用以下字符代表： ♦3 ♣3 ♥3 ♠3  ♦4 ♣4 ♥4 ♠4  ♦5 ♣5 ♥5 ♠5   ♦6 ♣6 ♥6 ♠6
- 洗牌：就是把数组中的牌顺序打乱。
- 打乱牌的顺序：可以遍历数组，针对每一张牌，随机与另一张牌交换位置。数组中的两个元素互换位置核心代码如下：

```Java
// 交换当前索引i与随机索引j处的元素
  String temp = array[i];
  array[i] = array[j];
  array[j] = temp;
```

此案例完整代码如下：

```Java
package com.heima;
import java.util.Arrays;
import java.util.Random;

public class PokerShuffler {
    public static void main(String[] args) {
        // 1. 创建扑克牌数组
        String[] suits = {"♠", "♥", "♦", "♣"};
        String[] ranks = {"2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"};

        // 生成一副不含大小王的扑克牌
        String[] deck = new String[52];
        int index = 0;
        for (String suit : suits) {
            for (String rank : ranks) {
                deck[index++] = rank + suit;
            }
        }

        // 添加大小王
        String[] fullDeck = Arrays.copyOf(deck, 54);
        fullDeck[52] = "小王";
        fullDeck[53] = "大王";

        // 打印原始牌组
        System.out.println("原始牌组:");
        System.out.println(Arrays.toString(fullDeck));

        // 2. 洗牌
        shuffle(fullDeck);

        // 打印洗后的牌组
        System.out.println("\n洗牌后的牌组:");
        System.out.println(Arrays.toString(fullDeck));
    }

    /**
     * 随机交换每张牌的位置
     * @param array 待洗牌的数组
     */
    public static void shuffle(String[] array) {
        Random random = new Random();
        for (int i = array.length - 1; i > 0; i--) {
            // 从0到i之间选择一个随机索引
            int j = random.nextInt(i + 1);

            // 交换当前索引i与随机索引j处的元素
            String temp = array[i];
            array[i] = array[j];
            array[j] = temp;
        }
    }
}
```

## 二维数组

### 认识二维数组

在Java中，二维数组可以被看作是一个“数组的数组”。这意味着每个元素本身也是一个数组。二维数组通常用来表示表格数据或者矩阵，其中数据以行和列的形式组织。

为了方便理解，假设我们有一个需求：开发一个程序可以记录咱们班级学生的座位信息。

![Rga4bqeMQoBbmzxGxpAcprpEnSf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Rga4bqeMQoBbmzxGxpAcprpEnSf.png)

为了存储座位信息，我们可以将同一排的人，放在一个数组中。 4排，共4个数组，然后再将每个数组作为一个元素，创建一个新数组，这个新数组就是二维数组。

![WDA9buAISovvKXxa58lc9khZnMd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/WDA9buAISovvKXxa58lc9khZnMd.png)

从上图可以看出，二维数组中的每一个元素，又是一个数组。

### 二维数组的定义

声明一个二维数组时，你需要指定两个维度：行数和列数。同时二维数组的创建，也分为静态初始化、动态初始化。语法如下：

```Java
// 二维数组的静态初始化
数据类型[][]  数组名 = new 数据类型[][]{元素1, 元素2, 元素3, ...};
int[][] arr = new int[][]{ {12, 24, 36} , {666, 888} , {10, 20, 30} , {999} };

// 二维数组的动态初始化
数据类型[][]  数组名 = new 数据类型[长度1][长度2];
int[][] arr = new int[3][5];

```

### 二维数组的访问

可以通过指定行索引和列索引来访问或修改二维数组中的特定元素。索引从0开始。例如：

```Java
// 获取第一行第二列的元素（即5）
int element = arr[0][1]; // 结果是 2

// 修改第三行第四列的元素
arr[2][3] = 100;
```

### 二维数组的遍历

二维数组的遍历与普通数组相似，就是通过下标（索引）访问数组中的每个元素，区别是二维数组中的下标由行下标、列下标组成，需要通过嵌套循环才能完成遍历。二维数组的遍历示例如下：

```Java
int[][] arr = new int[][]{ {12, 24, 36} , {666, 888} , {10, 20, 30} , {999} };

for (int i = 0; i < arr.length; i++) {
    for (int j = 0; j < arr[i].length; j++) {
        int data = arr[i][j];
        System.out.print(data + "\t");
    }
    System.out.println();
 }
```

**以上代码执行流程如下：**

- 定义二维数组：`int[][] arr = new int[][]{ {12, 24, 36}, {666, 888}, {10, 20, 30}, {999} };`

  - 这里创建了一个二维整型数组 `arr`，它包含4个一维数组，每个一维数组的长度可以不同。
- 外层循环：`for (int i = 0; i < arr.length; i++) { ... }`

  - 外层循环用于遍历二维数组中的每一行（即每一个一维数组）。`arr.length` 表示二维数组中一维数组的数量（这里是4）。
  - 循环变量 `i` 从0开始递增，直到小于 `arr.length`。
- 内层循环：`for (int j = 0; j < arr[i].length; j++) { ... }`

  - 内层循环用于遍历当前行（由外层循环变量 `i` 确定的一维数组）中的每一个元素。
  - `arr[i].length` 表示当前行中元素的数量。
  - 循环变量 `j` 从0开始递增，直到小于 `arr[i].length`。
- 获取并打印元素：

  - `int data = arr[i][j];` 获取当前行 `i` 和列 `j` 位置上的值，并将其存储在变量 `data` 中。
  - `System.out.print(data + "\t");` 打印出这个值，并使用制表符 `\t` 来分隔同一行的不同元素。

### 二维数组的案例

需求：在一个棋盘上显示4行4列的石块，每个石块上有一个数字，玩游戏时需要将数字顺序打乱，由玩家将石块恢复正常顺序。界面的实现我们暂不考虑（后面讲到gui编程时再实现），此处只要求将打乱顺序的数字存入一个二维数组，以模拟石块中的数据。

![KbiBb6b0foKPe5x4bpLcxO40nwh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/KbiBb6b0foKPe5x4bpLcxO40nwh.png)

思路分析：

- 创建二维数组，由4行4列的int数据组成
- 循环生成 1-15的数字，存入数组
- 通过循环，将数字顺序打乱（类似斗地主游戏中的洗牌逻辑，不同之处在于，此处要随机生成行、列两个值，并与当前元素进行交换）

![Ykk5bfDRBoJ710xSPzGcUsEknOb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Ykk5bfDRBoJ710xSPzGcUsEknOb.png)

实现代码如下：

```Java
package com.heima.demo;
import java.util.Random;

public class Shuffle2DArray {
    public static void main(String[] args) {
        // 创建一个4x4的二维数组
        int[][] array = new int[4][4];

        // 初始化数组，填入1到16的数字
        int number = 1;
        for (int i = 0; i < 4; i++) {
            for (int j = 0; j < 4; j++) {
                array[i][j] = number++;
            }
        }

        // 打印初始的二维数组
        System.out.println("原始数组:");
        printArray(array);

        // 打乱数组中的数字
        shuffleArray(array);

        // 打印打乱后的二维数组
        System.out.println("\n打乱后的数组:");
        printArray(array);
    }

    /**
     * 可参考AI生成
     * 请编写java代码，将以上数组元素顺序随机打乱
     */
    private static void shuffleArray(int[][] data) {
        // 打乱二维数组中的元素顺序
        for (int i = 0; i < data.length; i++) {
            for (int j = 0; j < data[i].length; j++) {
                // 随机两个行列位置，让这两个位置交换。
                int i1 = (int) (Math.random() * data.length);
                int j1 = (int) (Math.random() * data.length);

                int i2 = (int) (Math.random() * data.length);
                int j2 = (int) (Math.random() * data.length);

                int temp = data[i1][j1];
                data[i1][j1] = data[i2][j2];
                data[i2][j2] = temp;
            }
        }
    }

    /**
     * 打印二维数组
     * @param array 要打印的二维数组
     */
    public static void printArray(int[][] array) {
        for (int i = 0; i < array.length; i++) {
            for (int j = 0; j < array[i].length; j++) {
                System.out.printf("%-3d", array[i][j]);
            }
            System.out.println();
        }
    }
}
```

运行效果如下：

![NZRGbJeYZomM5hx53YWcPG19nOe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/NZRGbJeYZomM5hx53YWcPG19nOe.png)

---

## 自测问答

> 覆盖本章考点与常见易错点；答不上来，回到上文对应小节重读。

### 问答：嵌套循环与数组

**题目: 请选择正确的静态初始化数组方式**

A、int[] arr = {1, 2, 3};

B、String arr[] = {1, 1, 1};

C、int arr[] = {"1", "22", '3'};

D、int[] arr = new int[]{1, 2, 3};

E、int arr[] = new int[3]{1, 2, 3};

F、int arr[] = new int[]{1, 2, 3};

```Java
A、D、E
```

**题目: 看程序说结果，请不要提前运行**

**目标: 掌握引用数据类型变量的特点**

```Java
 public class Test03_02 {
     public static void main(String[] args) {
       int[] arr1 = {1,2,3,4,5};
       int[] arr2 = {5,6,7};
       int[] temp = arr1;

       System.out.println("通过temp取出arr1中的元素: ");

       for(int i = 0;i<temp.length;i++) {
         System.out.print(temp[i]+" ");
       }

       temp = arr2;

       System.out.println("通过temp取出arr2中的元素: ");

       for(int i = 0;i<temp.length;i++) {
         System.out.print(temp[i]+" ");
       }
     }
   }
```

```Java
通过temp取出arr1中的元素: 
1 2 3 4 5
通过temp取出arr2中的元素:
5 6 7
```

**题目:看程序说结果，请不要提前运行**

**目标: 掌握引用数据类型变量的特点**

```Java
   public class Test03_01 {
     public static void main(String[] args) {
       int[] arr1 = {1,2,3,4,5};
       int[] temp1 = arr1;
       int[] temp2 = arr1;
       System.out.println("通过temp1取出数组值: ");

       for(int i = 0;i<temp1.length;i++) {
         System.out.print(temp1[i]+" ");
       }
       System.out.println();
       System.out.println("通过temp2取出数组值: ");
       for(int i = 0;i<temp2.length;i++) {
         System.out.print(temp2[i]+" ");
       }
       System.out.println();
       temp1[2] = 9;

       System.out.println("通过temp1取出数组值: ");
       for(int i = 0;i<temp1.length;i++) {
         System.out.print(temp1[i]+" ");
       }
       System.out.println();
       System.out.println("通过temp2取出数组值: ");
       for(int i = 0;i<temp2.length;i++) {
         System.out.print(temp2[i]+" ");
       }
       System.out.println();
     }
   }
```

```Java
通过temp1取出数组值:
1 2 3 4 5
通过temp1取出数组值:
1 2 3 4 5
通过temp1取出数组值:
1 2 9 4 5
通过temp1取出数组值:
1 2 9 4 5
```

**题目:**

**如何访问数组的元素?**

**如何访问数组的长度？**

**数组的最大索引是多少？**

**访问不存在的索引会出现什么现象?**

**目标: 掌握数组的访问基本格式**

```Java
如何访问数组的元素?
通过索引访问数组
格式1：
    for(int i = 0;i < 数组名.length;i++){
        System.out.println(数组名[i]);
    }
格式2：
    数组名[索引值]
如何访问数组的长度？
格式：
    //返回值是数组的元素个数
    数组名.length
数组的最大索引是多少？
格式：
    //由于数组是从0开始的，所以最大索引应该是 元素个数 -1
    数组名.length - 1
访问不存在的索引会出现什么现象?
    会报错，提示超出索引界限
```

**题目:**

**请叙述数组出现的好处 ?**

**数组的静态初始化的格式以及元素的特点?**

**数组的动态初始化的格式以及元素的特点?**

**目标: 掌握数组的特点**

```Java
请叙述数组出现的好处 ? 
    数组可以按顺序存储多个相同类型的数据
    数组通过索引可以快速访问和修改任意位置的元素
    数组便于使用循环批量处理数据
    数组提高代码的可读性和维护性
    内存空间连续，访问效率高
数组的静态初始化的格式以及元素的特点? 
简写格式：
    数组类型[] 数组名 = {元素1,元素2,...元素n}
完整格式：
    数组类型[] 数组名 = new 数组类型[]{元素1,元素2,...元素n}
元素特点：
    数组长度由大括号中元素个数决定
    元素类型必须与声明的数组类型一致
    初始化时直接为每个元素赋具体值
    一旦初始化完成，长度就固定不变
数组的动态初始化的格式以及元素的特点?
格式：
    数组类型[] 数组名 = new 数组类型[数组长度]
特点：
    只指定数组长度，不指定具体元素值
    系统会自动为数组元素分配默认值：
    整数类型（byte、short、int、long）：0
    浮点类型（float、double）：0.0
    字符类型（char）：'\u0000'（空字符）
    布尔类型（boolean）：false
    引用类型：null
    数组长度在创建时确定，不可改变
```

**循环嵌套概念: 循环中又包含循环。**

例如: 两者三天每天说五次"我爱你"

```Java

// 连续3天
for(int i = 0; i < 3; i++) { 
    // 每天5次 
    for(int j = 0; j < 5; j++) {
        System.out.println("我爱你"); 
    } 
} 

private static void loveEveryDay() {
    for (int i = 0; i < 3; i++) {
        System.out.print("第" + (i+1) + "天：");
        for (int j = 0; j < 5; j++) {
            System.out.print("我爱你！\t");
        }
        System.out.println();
    }
}

```

**特点: 特点外部循环每循环一次，内部循环会全部执行完一轮。**  
**1.题目: 在控制台使用 \* 打印出5行7列的矩形**  
\*\*\*\*\*\*\*  
\*\*\*\*\*\*\*  
\*\*\*\*\*\*\*  
\*\*\*\*\*\*\*  
\*\*\*\*\*\*\*\`

```Java
private static void matrixPrint() {
    //打印行
    for (int i = 0; i < 5; i++) {
        //打印列
        for (int j = 0; j < 7; j++) {
            //单行打印
            System.out.print("*" + " ");
        }
        //换行
        System.out.println();
    }
}
```

**2.题目: 在控制台使用 \* 打印出5行5列的阶梯如下**  
\*  
\*\*  
\*\*\*  
\*\*\*\*  
\*\*\*\*\*

```Java
private static void matrixPrint2() {
    //行
    for (int i = 0; i < 5; i++) {
        //列 控制列数不大于行 实现递增
        for (int j = 0; j <= i; j++) {
            //打印单行个数
            System.out.print("* \t");
        }
        //行满换行
        System.out.println();
    }
}
```

**题目: 请使用嵌套循环在控制输出九九乘法口诀表，如下图所示：**

![lz3bbzsqzonmtox46jhc2cjankg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/lz3bbzsqzonmtox46jhc2cjankg.png)
> 图注：图片展示的是九九乘法口诀表，从1×1=1到9×9=81，每行每列数字相乘的结果依次排列。该图片与文档中“请使用嵌套循环在控制输出九九乘法口诀表，如下图所示”的题目相关，是该题目答案的示例，直观呈现了九九乘法口诀表的结构，帮助学习者理解题目要求，即使用嵌套循环在控制台输出类似格式的九九乘法口诀表。

> 温馨提示: **\t**可以实现制表符的书写, 也就是键盘中tab键的效果

```Java
private static void mulNum() {
    //控制行 增长
    for (int i = 1; i <= 9; i++) {
        //控制列的最大值随着行增长
        for (int j = 1; j <= i; j++) {
            //打印输出每列 样式
            System.out.print(j + "*" + i + "=" + (j * i) + "\t");
        }
        //列满换行
        System.out.println();
    }
}
```

**题目: 下面哪个可以正确的获取数组a的第三个元素**

**目标: 掌握访问数组的基本格式**

A、a(3)

B、a[3]

C、a(2)

D、a[2]

```Java
D
```

**题目: 现有一个小数数组{12.9,53.54,75.0,99.1,3.14}。请编写代码，找出数组中的最小值并打印。**

**目标: 掌握数组的遍历以及和流程控制等综合应用**

```C++
private static void numPrint() {
    //定义小数类型数组
    double[] arr = new double[]{12.9,53.54,75.0,99.1,3.14};
    //定义最小值存储变量 初始值设置arr数组中索引为0的第一个元素
    double min_num = arr[0];
    //循环遍历数组
    for (int i = 0; i < arr.length; i++) {
        //判断遍历当前元素 和定义最小值大小
        if (arr[i] < min_num)
            //当前遍历元素小于 变量存储值，则执行替换当前变量值
            min_num = arr[i];
    }
    //打印输出
    System.out.println("最小值为：" + min_num);
}
```

**题目:**

**(1)定义一个长度为10的int类型数组，元素值使用随机数生成（范围为50-80）**

**(2)打印该数组中满足个位和十位不包含7的元素。**

**目标: 掌握数组的遍历以及和流程控制等综合应用**

```Java
private static void radomArray() {
    //定义长度为10的空数组
    int[] arr = new int[10];
    //调用随机数方法
    Random r = new Random();
    //遍历定义的空数组
    for (int i = 0; i < arr.length; i++) {
        //将随机范围为50-80的数 放入当前索引的数组位置，每轮放入一个
        arr[i] += r.nextInt(50,81);
        //打印检查
        System.out.print(arr[i] + "\t");
    }
    System.out.println();
    //循环遍历生成的随机数组
    for (int i = 0; i < arr.length; i++) {
        //定义变量并计算 当前索引元素的个位和十位的值
        int ge = arr[i] % 10;
        int shi = arr[i] /10 % 10;
        //判断个位和十位是否为7 
        if (ge != 7 && shi != 7)
            //个位十位均不满足为7 执行打印
            System.out.println(arr[i]);
    }
}
```

**题目:**

**定义一个数组其中包含多个数字。用自己的方式最终实现，奇数放在数组的左边，偶数放在数组的右边。（可以创建其他数组，不必须在原数组中改变）**

**目标: 掌握数组的遍历以及和流程控制等综合应用**

```SQL
private static void arrayLeftRight() {
    int[] arr = new int[]{3, 4,6,8,10,5, 7, 9, 9, 3, 4, 7};
    int left = 0;
    int right = arr.length - 1;
    while (left < right) {
        // 从左边找到第一个偶数
        while (left < right && arr[left] % 2 != 0) {
            left++;
        }
        // 从右边找到第一个奇数
        while (left < right && arr[right] % 2 == 0) {
            right--;
        }
        // 找到了，就交换
        if (left < right) {
            int temp = arr[left];
            arr[left] = arr[right];
            arr[right] = temp;
            left++;
            right--;
        }
    }
    for (int i = 0; i < arr.length; i++) {
        System.out.print(arr[i] + "\t");
    }
}
```

**题目:**

**某公司开发部5名开发人员，要进行项目进展汇报演讲，现在采取随机排名后进行汇报。**

**已知开发人员存放在数组中 String[] arr = { "张三", "李四", "王五", "赵六", "田七" };**

**目标: 掌握数组的遍历以及和流程控制等综合应用**

```Java
private static void nameRaking() {
    //定义字符串类型的数组 存放姓名
    String[] arr = { "张三", "李四", "王五", "赵六", "田七" };
    //调用随机数方法
    Random r = new Random();
    //循环遍历姓名数组
    for (int i = 0; i < arr.length / 2; i++) {
        //随机长度为0-4的索引
        int index = r.nextInt(5);
        //定义第三 变量 存储交换值 即当前遍历元素
        String name = arr[i];
        //将当前元素赋 随机索引元素值
        arr[i] = arr[index];
        //将当前元素值放回 随机索引位置
        arr[index] = name;
    }
    //循环遍历打印 该随机姓名数组
    for (int i = 0; i < arr.length; i++) {
        System.out.println(arr[i]);
    }
}
```

**题目:**

**需求某个数组有5个数据：10,20,30,40,50，请将这个数组中的数据进行反转。即：[10, 20, 30, 40, 50] 反转后 [50, 40, 30, 20, 10]请将代码区的数组进行反转。**

**目标: 掌握数组的遍历以及和流程控制等综合应用**

```C++
private static void reverseNum() {
    //定义数组
    int[] arr = new  int[]{10,20,30,40,50};
    //定义末尾索引
    int index = arr.length-1;
    for (int i = 0; i < arr.length; i++) {
        //判断是否索引是否发生碰撞
        if (i >= index)
            break;
        //当前元素赋值给 第三方变量
        int name = arr[i];
        //末尾索引元素值 赋值给当前元素赋 
        arr[i] = arr[index];
        //将当前元素值放回 末尾索引位置
        arr[index] = name;
        //末尾索引 向左推进
        index--;
    }
    //循环打印列表
    for (int i = 0; i < arr.length; i++) {
        System.out.println(arr[i]);
    }
}
```

题目: 某班级学生的Java考试成绩是：99, 100, 62, 15, 48, 65, 98, 99, 5, 59.5, 75; 请统计及格学生总人数。

**目标: 掌握数组的遍历以及和流程控制等综合应用**

```C++
private static void scorePeopleNum() {
    //定义数组
    double[] arr = new double[]{99, 100, 62, 15, 48, 65, 98, 99, 5, 59.5, 75};
    //定义计数变量
    int count = 0;
    //循环遍历成绩数组
    for (int i = 0; i < arr.length; i++) {
        //判断成绩是否及格 大于等于60为及格
        if (arr[i] >= 60){
            System.out.println("成绩及格：" + arr[i]);
            //及格则累加
            count++;
        }
    }
    //打印及格人数
    System.out.println("有：" + count + "人及格");
}
```

**题目: 需求某歌唱比赛，需要开发一个系统：可以录入6名评委的打分，录入完毕后立即输出平均分做为选手得分。**

**目标: 掌握数组的遍历以及和流程控制等综合应用**

```Java
private static void scorePeopleSong() {
    //调用键盘录入方法
    Scanner sc = new Scanner(System.in);
    //定义长度为6的数组
    double[] arr = new double[6];
    //循环遍历数组
    for (int i = 0; i < arr.length; i++) {
        //提示成绩输入
        System.out.println("请输入评分：");
        //键盘录入成绩放入当前索引的元素位置
        arr[i] = sc.nextInt();
    }
    //定义变量 接收累加值
    double count = 0;
    //循环遍历成绩数组
    for (int i = 0; i < arr.length; i++) {
        //累加元素值
        count += arr[i];
    }
    //打印计算平均成绩
    System.out.println("平均成绩为：" + count / 6);
}
```

**题目: 某部门5名员工的销售额分别是：16、26、36、6、100，请计算出他们部门的总销售额。**

**目标: 掌握数组的遍历以及和流程控制等综合应**

```C++
private static void allMoney() {
    //定义数组
    int[] arr = new int[]{16,26,36,6,100};
    //定义累加存储变量
    int count = 0;
    //循环遍历列表
    for (int i = 0; i < arr.length; i++) {
        //累加额度
        count +=arr[i];
    }
    //打印输出额度
    System.out.println("总销售额度为：" + count);
}
```

---

> 来源：飞书《讲义》知识库 · 第 04 章 · 嵌套循环、数组、二维数组 · [原文](https://baxsu7o100e.feishu.cn/wiki/H3NswyRUyi9lY2kvabFcr7N6nwc)
> 问答来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / 基础语法与流程控制 / 04 嵌套循环、数组、二维数组·笔记 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/MsAdwDfRQi14azkY29zcnVSonRc)
