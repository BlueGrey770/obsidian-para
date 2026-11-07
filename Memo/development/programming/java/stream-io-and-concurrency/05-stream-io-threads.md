---
type: memo
date: 2026-11-06
aliases:
  - "05-stream-io-xian-cheng"
  - "05 Stream, IO, 线程"
---
# 05 Stream, IO, 线程

## Collections补充

### StringBuilder 介绍

<callout emoji="⚠️">
StringBuilder是一个**可变的字符序列**
*作用: 在拼接字符串的时候效率很高!!!*
</callout>

构造:

```Java
StringBuilder()
StringBuilder(String str)
```

*功能:*

```Java
StringBuilder append(Object obj)   拼接内容, 并且返回自己，可以链式编程不断拼接
StringBuilder reverse() 反转字符串内容
String toString()   将StringBuilder转成String，因为String里面含有更多功能
```

**面试题:** 观察代码请说结果

<callout emoji="⚠️">
*StringBuilder sb1 = new StringBuilder("aaa");*  
*StringBuilder sb2 = new StringBuilder("aaa");*  
*sb1 == sb2 ? false*  
*sb1.equals(sb2) ? false*
*原因???*  
            *StringBuilder没有重写equals,走的默认是Object的equals方法,默认用==比较*
</callout>

***面试题:*** *为啥StringBuilder拼接效率高???*

<callout emoji="⚠️">
  
*String是一个固定的字符序列, 没有拼接能力, 所有的+,+=底层都是采用StringBuilder的append方法,*  
*而且拼接底层调用的是StringBuilder的toString方法转成String类型才行*  
*每一次拼接都会频繁向堆中申请空间, 而堆内存中申请空间很耗时*
StringBuilder自始至终创建一次对象
</callout>

**对比拼接效率的案例:**





## 一、 Stream流

### 1.1 介绍

*Stream流是用来简化集合或者数组的操作的用的一种技术*

<callout emoji="⚠️">
Stream是JDK8出现的技术, **主要是简化集合或者数组的操作的** , Stream是一个接口, 我们无法直接创建对象, 
可以通过集合或者数组转成Stream对象
</callout>

### 1.2 如何得到Stream流对象

格式：

```Java
集合转Stream对象
    只有单单列集合可以转成Stream流
        Stream <E> s1 = 集合.stream();
数组转Stream对象
    Stream <E> s1 = Arrays.stream(数组);
```

(1) 集合转Stream

单列集合底层接口Collection提供stream()方法可以将集合转成Stream  
default Stream<E> stream() 

```Java
private static void demo01() {
    //集合转Stream对象
    List<String> list = new ArrayList<>();
    list.add("aaa");
    list.add("bbbbbbbbb");
    list.add("cccc");
    list.add("ddddddd");

    Stream<String> stream = list.stream();
    //遍历元素
//        stream.forEach(s -> System.out.println(s));
    //获取最大长度的字符串
//        String s = stream.max((o1, o2) -> o1.length() - o2.length()).get();
    //获取最大小长度的字符串
    String s = stream.max((o1, o2) -> o2.length() - o1.length()).get();
    System.out.println(s);
   }
```

(2) 数组转Stream

数组的工具类Arrays提供stream(数组)方法可以将数组转成Stream  
static <T> Stream<T> stream(T[] array)

```TypeScript
private static void demo01() {
    //数组转Stream对象
    String[] arr = {"aaa", "bbb", "ccc", "ddd"};
    Stream<String> stream1 = Arrays.stream(arr);
    long count = stream1.count();
//        System.out.println(count);
}
```

### 1.3 Stream流的功能

#### 1.3.1 终结方法

*特点: 调用结束后, 不再返回Stream对象, 不能继续使用Stream功能*

```Java
void forEach(Consumer<? super E > action) //遍历流中的元素
Optional max（Comparator<? super E > comparator)  //取出流中元素的最大值
    get()；+通过get获取最大值
Optional min(Comparator<? super E > comparator) //取出流中元素的最小值
    get()； //通过get获取最小值
long count()； //获取流中元素的个数
```

#### 1.3.2 中间方法

| 返回值 | 方法名 参数 | 作用 |
|-|-|-|
| *Stream<R>*  | *filter(Predicate<? super T> predicate)* | 根据条件过滤元素 |
| *Stream*  | *limit(long maxSize)* | 限制取前几个元素 |
| *Stream* |  *skip(long n)* | 跳过前几个元素 |
| *Stream*  | *distinct()* | 去重,采用的Hash去重 |
| *Stream*  | *sorted()* | 自然排序 |
| *Stream*  | *sorted(比较器)* | 比较器排序 |
| *Stream<T>*  | *static concat(Stream<? extends T> a, Stream<? extends T> b)* | 将两个流合并成一个流 |
| *Stream<R>*  | *map(Function<? super T,? extends R> mapper)* | 将流的元素修改或者修改成另外一种类型 |
| *Stream<R>*  | *flatMap(Function<? super T,? extends Stream<? extends R>> mapper)* | 处理嵌套集合或者数组,将所有元素处理成为一条流 |

```TypeScript
    private static void demo03() {
        List<String> list = new ArrayList<>();
        Collections.addAll(list,"杨过","郭靖","郭芙","杨康","杨国福","独孤求败","金轮法王","杨洋哥哥");
        String[] arr = {"徐志摩","戴望舒","令狐冲"};
        //将两个流合并成一个流
//        Stream.concat(list.stream(),Arrays.stream(arr)).forEach(s -> System.out.println(s));
        // 将所有名字打印出来
//        list.stream().map(s -> s.length() <=3 ? s.substring(1):s.substring(2)).forEach(s -> System.out.println(s));
        //将所有名字转换成Student对象
//        list.stream().map(s -> new  Stundent(s)).forEach(stundent -> System.out.println(stundent));
        String[][] arrs = {
                {"李清照", "李白", "房玄龄"},
                {"杜甫", "白居易", "王维"},
                {"苏轼", "辛弃疾", "欧阳修"}
        };
        Arrays.stream(arrs).flatMap(Arrays::stream).filter(s -> s.startsWith("李")).forEach(strings -> System.out.println(strings));
    }
```



#### 1.3.3 转换方法

| `<R,A> R` | `collect(Collector<? super T,A,R> collector)`  | 将流搜集成集合, 可以继续使用集合的功能 |
|-|-|-|
| `Object[]` | `toArray()`   | 将流转成数组 |

演示1：

```Java
private static void demo4() {
    List<String> list = new ArrayList<>();
    list.add("林青霞");
    list.add("张学友");
    list.add("梅军建");

    //将流转成集合后, 主要的目的可以使用集合的方法
    List<String> collect = list.stream().filter(name -> name.startsWith("张")).collect(Collectors.toList());
    //可以使用集合的功能
}
```

演示2：

```TypeScript
private static void demo01() {
        //集合转Stream对象
        List<String> list = new ArrayList<>();
        list.add("aaa");
        list.add("bbbbbbbbb");
        list.add("cccc");
        list.add("ddddddd");

        Stream<String> stream = list.stream();
        //遍历元素
//        stream.forEach(s -> System.out.println(s));
        //获取最大长度的字符串
//        String s = stream.max((o1, o2) -> o1.length() - o2.length()).get();
        //获取最大小长度的字符串
        String s = stream.max((o1, o2) -> o2.length() - o1.length()).get();
        System.out.println(s);
        //数组转Stream对象
        String[] arr = {"aaa", "bbb", "ccc", "ddd"};
        Stream<String> stream1 = Arrays.stream(arr);
        long count = stream1.count();
//        System.out.println(count);
    }
```

Stream流的注意事项:

<callout emoji="⚠️">
1. Stream流元素的操作"**不会影响**"原集合或者数组
2. Stream流对象只能使用一次, 但凡被操作了, 就不能再使用了, Stream主打的就是链式编程解决所有的问题, 所以不希望你分成两步, 所以禁止我们对流进行二次操作
</callout>

#### 1.3.4 综合案例

现有两个队伍的人员，分别存入两个List集合中，如下所示：

```TypeScript
public class Student {
    private String name;
    public Student(String name) {
        this.name = name;
    }
    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                '}';
    }
}

public class Test {
    public static void main(String[] args) {
//        List<Student> stu = new ArrayList<>();
        // 队伍1
        ArrayList<String> team1 = new ArrayList<>();
        team1.add("陈玄风");
        team1.add("梅超风");
        team1.add("风清扬");
        team1.add("解风");
        team1.add("风波恶");
        team1.add("陆乘风");
        team1.add("曲灵风");
        team1.add("武眠风");
        team1.add("冯默风");
        team1.add("陆小风");
        // 队伍2
        ArrayList<String> team2 = new ArrayList<>();
        team2.add("宋远桥");
        team2.add("俞莲舟");
        team2.add("俞岱岩");
        team2.add("张松溪");
        team2.add("张翠山");
        team2.add("殷梨亭");
        team2.add("莫声谷");
        //1、从第一个队伍中筛选出名字为3个字并且以“风”字结尾的人，筛选之后，只保留前6个。
        List<String> collect = team1.stream().filter(name -> name.length() == 3 && name.endsWith("风")).limit(6).collect(Collectors.toList());
        System.out.println(collect);

        //2、从第二个队伍中筛选出姓张的人员，筛选之后，不要第一个。
        List<String> collect1 = team2.stream().filter(name -> name.startsWith("张")).skip(1).collect(Collectors.toList());
        System.out.println(collect1);

        //3、将两个筛选结果合并为一个队伍。
        List<String> list = new ArrayList<>();
        list.addAll(collect);
        list.addAll(collect1);
        System.out.println(list);

        //4、将结果队伍中的字符串姓名作为Student对象的属性，创建Student对象，将集合中的元素都更改为Student类型。
        List<Student> collect2 = list.stream().map(s -> new Student(s)).collect(Collectors.toList());
        System.out.println(collect2);

        //5、打印结果队伍的Student对象信息
        collect2.stream().forEach(student -> System.out.println(student.toString()));
    }
}
```





## 二、IO

### 2.1 概述

<callout emoji="🌷">
IO是一种持久化存储技术
什么是临时存储? 什么是持久化存储? 各自特点是什么?
答: 
将数据保存到内存中的技术叫做临时存储, 临时存储内存释放后, 数据会丢失,比如之前学习的变量, 数组, 对象, 集合都是临时存储的方案
将数据保存到磁盘中的技术叫做持久化存储, 持久化存储程序结束后, 内存释放后, 数据依然在磁盘中存在
</callout>

### 2.2 File



#### 2.2.1 作用:

<callout emoji="⚠️">
用来**关联磁盘中的文件或者文件夹**用的
</callout>

#### 2.2.2 使用:(了解)

**构造:**

<callout emoji="⚠️">
*File(File parent, String child)*  
*从父抽象路径名和子路径名字符串创建新的 File实例。*  
*File(String pathname)*  
*通过将给定的路径名字符串转换为抽象路径名来创建新的 File实例。*  
*File(String parent, String child)*  
*从父路径名字符串和子路径名字符串创建新的 File实例。*
可以看出这里需要**传入路径**, java的路径写法:  
            *1.绝对路径: 带盘符的路径*  
                *举例:*  
                    *d:/a.txt*  
            *2.相对路径: 不带盘符的路径*  
                    *相对于****当前工程的目录****下关联*  
                    *d/a.txt*
</callout>

代码演示:

```Java
public class Test {

    public static void main(String[] args) {
        //1.创建File关联磁盘
        /*
            路径的写法:
            1.绝对路径: 带盘符的路径
                举例:
                    d:/a.txt
            2.相对路径: 不带盘符的路径
                    相对于当前工程的目录下关联
                    d/a.txt
         */
        //File(String pathname)  推荐!!!!!!!!!!!!!!!!!!!!!!!!
        //绝对路径
        File file = new File("d:/a.txt");
        System.out.println(file.getAbsolutePath());
        //相对路径
        File file1 = new File("d/a.txt");
        System.out.println(file1.getAbsolutePath());
        //File(String parent, String child)
        File file2 = new File("d:/aaa","a.txt");
        //File(File parent, String child)
        File file4 = new File("d:/aaa");
        File file3 = new File(file4,"a.txt");

    }
}
```

| 方法名称 | 说明 |
|-|-|
| public boolean exists() | 判断当前文件对象，对应的文件路径是否存在，存在返回true  |
| public boolean isFile() | 判断当前文件对象指代的是否是文件，是文件返回true，反之。 |
| public boolean isDirectory() | 判断当前文件对象指代的是否是文件夹，是文件夹返回true，反之 |
| public String getName() | 获取文件的名称 (包含后缀) |
| public long length() | 获取文件的大小，返回字节个数 |
| public long lastModified() | 获取文件的最后修改时间。 |
| public String getPath() | 获取创建文件对象时，使用的路径 |
| public String getAbsolutePath() | 获取绝对路径 |

| 方法名称 | 说明 |
|-|-|
| public boolean createNewFile() | 创建一个新的空的文件 |
| public boolean mkdir() | 只能创建一级文件夹 |
| public boolean mkdirs() | 可以创建多级文件夹 |
| public boolean delete() | 删除文件、空文件夹 |
| public String[] list() | 获取当前目录下所有的"一级文件名称"到一个字符串数组中去返回， |
| public File[] listFiles() | 获取当前目录下所有的"一级文件对象"到一个文件对象数组中去返回（重点） |

**功能演示:**

```Java
import java.io.File;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;


/*
File(File parent, String child)
从父抽象路径名和子路径名字符串创建新的 File实例。
File(String pathname)
通过将给定的路径名字符串转换为抽象路径名来创建新的 File实例。
File(String parent, String child)
从父路径名字符串和子路径名字符串创建新的 File实例。

 */
public class Test {

    public static void main(String[] args) throws IOException {
        demo5();
    }

    private static void demo5() throws IOException {
        //创建文件 boolean createNewFile() 创建文件
        /*
            (1) 该方法只能创建文件, 不能创建文件夹, 所以你必须保证要创建的文件所在的文件夹存在才行,否则报错
                java.io.IOException: 系统找不到指定的路径。
            (2) 文件存在则创建失败, 文件不存在则创建成功
         */
        File file = new File("day06_File_Recursion_IO/a.txt");
        System.out.println(file.createNewFile());
        //创建文件夹 boolean mkdir() 创建文件夹  boolean mkdirs();
        /*
            mkdir() 只能创建单级目录, 不能创建多级目录,必须保证要创建文件夹所在的文件夹存在才能创建成功,否则创建失败
         */
        //File file1 = new File("day06_File_Recursion_IO/b/b/b");
        //System.out.println(file1.mkdir());
        //System.out.println(file1.mkdirs());

        /*
            删除:
                (1) 只能删除空文件夹或者文件
                (2) 删除不走回收站, 删除的文件无法找回
         */
        File file2 = new File("day06_File_Recursion_IO/a");
        System.out.println(file2.delete());
    }

    private static void demo4() {
        //String getPath() 获取构造中给的路径,一般不用这个
        File file = new File("a,txt");
        System.out.println(file.getPath());
        // 获取绝对路径 String getAbsolutePath() 获取绝对路径
        System.out.println(file.getAbsolutePath());
    }

    private static void demo3() {
        // long length(); 获取关联的文件的长度,文件夹没有大小的概念, 以字节大小返回
        /*
            \\: 反斜杠在java代表的转义功能, 需要表示真实斜杠, 需要成对写  \\目录分隔符只在window识别, window也识别/
            linux, mac:  只识别/
         */
        File file = new File("d:/a.txt");
        System.out.println("获取文件的大小:" + file.length());
        File file1 = new File("d:/aaa");
        System.out.println(file1.length());
        /*
           long lastModified(); 获取最后一次修改的时间毫秒值
           时间毫秒值: 时间距离时间原点的过得毫秒数
         */
        long start = file.lastModified();
        //将时间毫秒值转成时间对象
        Date date = new Date(start);
        /*
            月份0-11
            年: 1900开始算
         */
        // System.out.println(date.getYear()+"年"+date.getMonth()+"月"+date.getDate());
        //创建时间格式化对象
        /*
            yyyy: 代表年
            MM: 代表月
            dd: 代表日
            HH: 代表时
            mm: 代表分
            ss: 代表秒
         */
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
        String format = sdf.format(date);
        System.out.println(format);


    }

    private static void demo2() {
        //跟判断有关的方法
        // boolean exists(); 判断关联文件或目录是否存在
        File file = new File("d:/a/a/a/m");
        System.out.println(file.exists());
        // boolean isDirectory(); 判断是否是目录
        boolean directory = file.isDirectory();
        System.out.println(directory);
        // boolean isFile(); 判断关联的是否是文件
        System.out.println(file.isFile());
        // 获取关联的文件或者文件夹名字 String getName();   d:/a/a/a/a/a/a/a.txt
        System.out.println(file.getName());

    }

    private static void demo1() {
        //1.创建File关联磁盘
        /*
            路径的写法:
            1.绝对路径: 带盘符的路径
                举例:
                    d:/a.txt
            2.相对路径: 不带盘符的路径
                    相对于当前工程的目录下关联
                    d/a.txt
         */
        //File(String pathname)  推荐!!!!!!!!!!!!!!!!!!!!!!!!
        //绝对路径
        File file = new File("d:/a.txt");
        System.out.println(file.getAbsolutePath());
        //相对路径
        File file1 = new File("d/a.txt");
        System.out.println(file1.getAbsolutePath());
        //File(String parent, String child)
        File file2 = new File("d:/aaa", "a.txt");
        //File(File parent, String child)
        File file4 = new File("d:/aaa");
        File file3 = new File(file4, "a.txt");
    }
    private static void demo6() {
        /*
            public File[] listFiles()   
            获取当前目录下所有的"一级文件对象"到一个文件对象数组中去返回（重点）
            (1) 该方法只能是文件夹才能访问,如果文件强制访问,则会得到一个null的数组, 可能引发空指针异常
            (2) 有权限的文件夹不能使用,非要使用得到一个Null数组
                例如:
                    d:\System Volume Information
           (3) 如果是一个空文件夹, 强制访问,会得到一个长度为0的数组,不会出现Null指针异常
         */
        File file = new File("d:\\dd");
        System.out.println(file.isDirectory());
        //拿到D盘的一级子目录和文件
        File[] files = file.listFiles();
        System.out.println(files.length);
        for (File file1 : files) {
            System.out.println(file1);
        }
    }
}
```



### 2.2 IO的体系结构

<callout emoji="🌷">
IO流的体系
字节流:

字节输入流(InputStream(抽象类))

FileInputStream(实现类)

字节输出流(OutputStream(抽象类))

FileOutputStream(实现类)
字符流:

字符输入流(Reader(抽象类))

FileReader(实现类)

字符输入流(Writer(抽象类))

FileWriter(实现类)
</callout>

### 2.3 字节流

字节流概念: 以字节为单位读写"文件"中数据的流

#### 2.3.1 字节输出流

**作用: 将程序中的数据以字节的方式写到磁盘中文件中**

**构造:**

```Java
FileOutputStream(File file)
创建文件输出流以写入由指定的 File对象表示的文件。

 FileOutputStream(String name) 推荐!!!!!!!!!
创建文件输出流以指定的名称写入文件。

FileOutputStream(File file, boolean append)
创建文件输出流以写入由指定的 File对象表示的文件。

FileOutputStream(String name, boolean append)
创建文件输出流以指定的名称写入文件。
```

**功能:**

```Java
void write(int by);    写单个字节
void write(byte[] arr);    写多个字节
void write(byte[] arr, int start, int len); 从start索引开始写, 写len个, 写多个字节的一部分
void close(); 关闭资源
```

**演示:** 

```Java
/*
构造：
    FileoutputStream(File file)
    创建文件输出流以写入由指定的File对象表示的文件。
    FileoutputStream(String name) 推荐!!!!!!!!!
    创建文件输出流以指定的名称写入文件。
    FileOutputStream(File file,boolean append)
    创建文件输出流以写入由指定的File对象表示的文件。
    FileoutputStream(String name,boolean append)
    创建文件输出流以指定的名称写入文件。
功能：
    void write(byte[]b)写多个字节
    void write(byte[] b， int off， int len)写多个字节的一部分
    void write(int b)写单个字节到磁盘
    void close() 关闭流，释放资源
    
1.创建文件字节输出流关联文件
    (1)创建一个输出流对象
    (2)关联的文件如果不存在，则会创建文件，底层采用FiLe类的创建功能创建的
    该方法只能创建文件不能创建文件夹，所以要求你必须保证文件所在的文件夹存在才可以否则会报错
    (3)IO只能关联文件不能关联文件夹，否则会报拒绝访问异常，因为文件夹没有存储数据的能力
    (4)如果文件存在，会清空文件，重新写入数据
*/
private static void demo01() throws IOException {
    FileOutputStream fos = new FileOutputStream("I:\\Resilio Sync\\test.txt");
    fos.write(97);
    fos.write(98);
    fos.write(99);
    byte[] arr = {100,101,102,103};
    fos.write(arr);

    fos.write(arr,0,2);
    fos.close();
}
```





#### 2.3.2 字节输入流

**作用: 将磁盘中文件中的数据以字节的方式读到程序中**

**构造:**

```Java
FileInputStream(String name);
FileInputStream(File name);

必须保证要读取的文件存在才行, 否则会报java.io.FileNotFoundException: day06_File_Recursion_IO\a\a\a.txt (系统找不到指定的路径。)
```

**功能:**

```Java
int read(); 第一次读一个字节, 返回读到的字节 ,一般不用,效率很低很低!!!
int read(byte[] arr); 一次读多个字节, 返回读取的有效个数
void close(); 关闭资源
```

**演示**

```Java
private static void demo3() throws IOException {

    /*
        FileInputStream:
            构造:
                FileOutputStream(String name);
                FileOutputStream(File name);
            必须保证要读取的文件存在才行, 否则会报java.io.FileNotFoundException: day06_File_Recursion_IO\a\a\a.txt (系统找不到指定的路径。)
            功能:
                int read(); 第一次读一个字节, 返回读到的字节 ,一般不用,效率很低很低!!!
                int read(byte[] arr); 一次读多个字节, 返回读取的有效个数
                void close(); 关闭资源

     */

    FileInputStream fis = new FileInputStream("day06_File_Recursion_IO/a.txt");
    byte[] arr = new byte[5];
    int len;
    while (len= fis.read(arr)!=-1){
        System.out.println(len+":::"+Arrays.toString(arr));
    }

    /*while (true){
        int len = fis.read(arr);
        if(len ==-1){
            break;
        }
        System.out.println(len+":::"+Arrays.toString(arr));
    }*/

   /* while (true){
        int by = fis.read();
        if(by ==-1){
            break;
        }
        System.out.println(by);
    }*/
    //关闭资源
    fis.close();


}
```





#### 2.3.3 综合案例

**IO流拷贝: 单个字节拷贝:** 

```Java
/*
    IO流操作数据的, 只有文件可以存放数据, 所以IO不能关联文件夹必须关联文件才行
 */
public class Test2 {

    public static void main(String[] args) throws IOException {
        //1.创建字节输入流关联源(要拷贝的东西)
        FileInputStream fis = new FileInputStream("C:\\Users\\Lenovo\\Desktop\\颈椎操.flv");
        //2.创建输出流关联目的地(要拷贝的东西)
        FileOutputStream fos = new FileOutputStream("day06_File_Recursion_IO\\颈椎操.flv");
        //3.不断通过输入流读取数据, 将读到的数据通过输出流写出去
        long start = System.currentTimeMillis();
        while (true){
            int by = fis.read();
            if(by == -1){
                break;
            }
            fos.write(by);
        }
        long end = System.currentTimeMillis();
        System.out.println(end -start);
        fis.close();
        fos.close();
    }
}
```

**IO流拷贝: 多个字节的拷贝**

```Java
public class Test3 {

    public static void main(String[] args) throws IOException {
        //1.创建字节输入流关联源(要拷贝的东西)
        FileInputStream fis = new FileInputStream("C:\\Users\\Lenovo\\Desktop\\颈椎操.flv");
        //2.创建输出流关联目的地(要拷贝的东西)
        FileOutputStream fos = new FileOutputStream("day06_File_Recursion_IO\\颈椎操1.flv");
        //3.不断通过输入流读取数据, 将读到的数据通过输出流写出去
        long start = System.currentTimeMillis();
        byte[] arr = new byte[1024 * 8];
        int len;
        while ((len = fis.read(arr))!=-1) {
            fos.write(arr, 0, len);
        }
      
        long end = System.currentTimeMillis();
        System.out.println(end - start);
        fis.close();
        fos.close();
    }
}
```

总结:

<callout emoji="🧥">
对比效率, 多个字节拷贝的效率非常高, 原因是减少和磁盘交互的过程
</callout>

## 三、位运算

***前言:*** 

<callout emoji="🌷">
*底层是二进制运算:* ***按位运算****,'位'指的一个二进制, 在位运算中****0代表false,1代表true***
</callout>

**符号:**

<callout emoji="⚠️">
*& : 有0则0*  
*| : 有1则1*  
*^ : 不同为1, 相同为0*  
    *(1) 一个数 ^ 0 还是其本身*  
    *101*  
    *000*  
    *---*  
    *101*  
    *(2) 一个数 ^ 自己 等于0*  
    *111*  
    *111*  
    *---*  
    *000*  
    *(3) 一个数异或另外一个数两次还是其本身*  
    *5 ^3 ^ 3*  
*\~ : 按位取反, 1变0, 0变1, 包含符号位*  
*>>:  有符号右移,右移一位相当于/2 , 最高位原本是0就补0原本是1就补1*  
*>>>: 无符号右移,少用慎用, 最高位永远补0,可能一下将负数变成正数!!!*  
*<<: 左移, 相当于 \*2*  
    
</callout>

**加密案例:**

```Java
private static void demo1() throws IOException {
    int password = 1234;
    FileInputStream fis = new FileInputStream("d:/c.jpeg");
    FileOutputStream fos = new FileOutputStream("d:/b.jpeg");
    int by;
    while ((by = fis.read()) != -1) {
        fos.write(by ^ password);
    }
    fis.close();
    fos.close();
}
```

---
> 来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / Stream、IO 与多线程 / 05 Stream, IO, 线程 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/AF65wXbVliWP5kkJrE1cpbHKnob)
