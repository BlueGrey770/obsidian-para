---
type: memo
date: 2026-11-06
aliases:
  - "06-wang-luo-bian-cheng-ji-shu-yao-dian"
  - "06 网络编程技术要点"
---

# 06 网络编程技术要点

> 本章按「三要素 → UDP/TCP 收发 → JDK8 时间工具 → 群聊综合实战」的路径推进，是 IO 与多线程知识的第一次真正合流。

## 一、技术概念说明

### 1. 网络编程三要素

| 要素 | 作用 | Java 对应 |
|-|-|-|
| **IP** | 设备在网络中的唯一标识（IPv4/IPv6） | `InetAddress`，或点分十进制字符串（如 `127.0.0.1` 本机回环地址） |
| **端口** | 程序在设备上的编号（0\~65535） | 常用注册端口 1024\~49151 |
| **协议** | 通信规则 | UDP（`DatagramSocket/Packet`）、TCP（`ServerSocket/Socket`） |

记忆类比：IP 是「楼栋地址」，端口是「房间号」，协议是「送信方式」。

### 2. UDP：面向无连接的快速传输

- 特点：**不建立连接、不可靠**（不保证对方收到、不保证顺序），但速度快；单包上限 **64KB**；
- 核心类：`DatagramSocket`（收发站）、`DatagramPacket`（数据包裹，发送时打包地址端口，接收时只给缓冲区）；
- 适用：视频直播、语音通话等「宁丢勿慢」的场景。

### 3. TCP：面向连接的可靠传输

- 特点：**先连接后通信、可靠**（三次握手确认双方在线，四次挥手确保双方同时断开），数据大小无限制；
- 核心类：服务端 `ServerSocket`（监听端口，`accept()` 阻塞等待连接）→ 得到 `Socket`；客户端直接 `new Socket(ip, port)`；
- 读写分离：用 `DataInputStream/DataOutputStream` 包装 socket 的输入输出流，`readUTF/writeUTF` 收发字符串。

### 4. JDK8 时间类：LocalDateTime 家族

- `LocalDateTime.now()` 当前时间；`LocalDateTime.of(1949,10,1,0,0,0)` 指定时间；
- `DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")` 定义格式，`format()` 格式化、`parse()` 解析；
- `plusHours(1)` 加减运算；`Duration.between(t1, t2).toDays()` 计算间隔天数。

### 5. 群聊架构：多线程 + Socket 的综合应用

- **服务端**：`accept()` 循环接客，每个客户端连接存入 `List<Socket>`，并为每个连接**起一个线程**（`ServerRunnable`）负责收消息并**广播**给列表里其他客户端；
- **客户端**：两个线程各管一摊——线程 1 读键盘、发消息；线程 2 收服务器转发、打印——收发互不阻塞；
- **掉线处理**：`readUTF` 抛 `IOException` 说明对方断开，catch 中把该 socket 移出列表并广播离线通知。

## 二、技术点之间的联系

```
三要素（IP + 端口 + 协议）
  │
  ├─ UDP 路线：DatagramSocket + DatagramPacket
  │     一发一收 → 多发多收 → 同时收发（复杂度递增，但互不感知）
  │
  └─ TCP 路线：ServerSocket.accept() → Socket → IO流读写
        一发一收 → 多发多收 → 群聊（引入 多线程 + 集合管理连接）
              │
              ├─ 多线程：每客户端一个服务线程
              ├─ List<Socket>：连接登记簿，广播的数据基础
              └─ TimeUtils：LocalDateTime 工具化，给消息盖时间戳

JDK8 时间类 ──独立工具线──▶ 在群聊中承担「消息时间戳」职责
```

三条关键联系：

1. **群聊 = TCP 多发多收 + 多线程 + 集合的组合拳**：单独看每个案例都简单，群聊的价值在于展示「当服务端要同时服务多个客户端时，为什么必须一个连接一个线程」。
2. **IO 流是多线程的天然切入点**：`readUTF()` 是**阻塞**方法——没消息就卡住不动，所以客户端必须用两个线程分别收发，这正是「什么时候需要多线程」的最佳场景。
3. **时间类在实战中承担工具角色**：`TimeUtils.getNowTime()` 把「获取格式化当前时间」封装成静态工具方法（私有构造 + 静态方法），这是工具类的标准写法。

## 三、方法与实现要点

### 1. UDP 收发的固定四步

发送方：`new DatagramSocket()` → 数据装 `DatagramPacket`（带目标 IP/端口）→ `send()` → `close()`；  
接收方：`new DatagramSocket(端口)`（**端口必须与发送目标一致**）→ 空包裹 `receive()` 阻塞等待 → `getData()+getLength()` 解码 → `close()`。

### 2. TCP 案例的调试顺序

先启动服务端（否则客户端 `new Socket` 直接报 Connection refused）→ 观察「等待连接…连接成功…」确认握手 → 再测收发。养成「服务端先行」的习惯。

### 3. 时间类的统一套路

「获取/指定时间对象 → 定义 Formatter → format 输出 / parse 读入 → 需要间隔用 Duration.between」。注意 `Duration.between` 计算**天数**时，若起点只有日期（LocalDate），要补一个 `LocalTime.of(0,0,0)` 拼成 LocalDateTime 再参与计算。

### 4. 群聊代码的三个工程化细节

- 广播时 `if (socket1 != socket)` 排除发送者自己；
- `DataOutputStream` 在循环里对每个客户端**新建包装流**而非长期持有，避免状态错乱；
- 掉线清理写在 catch 里（readUTF 异常 = 连接断开），保证 `List<Socket>` 不积累死连接。

## 四、重点与难点分析

### 重点 1：TCP 与 UDP 的区别

| 维度 | UDP | TCP |
|-|-|-|
| 连接 | 无连接 | 面向连接（三次握手建立、四次挥手断开） |
| 可靠性 | 不可靠（不保证送达与顺序） | 可靠（确认、重传、排序） |
| 数据大小 | 单包 ≤64KB | 无限制（流式） |
| 速度 | 快 | 相对慢（有握手确认开销） |
| Java API | DatagramSocket/Packet | ServerSocket/Socket |

一句话：TCP 用开销换可靠，UDP 用可靠换速度，选型看业务「怕丢」还是「怕慢」。

### 重点 2：accept 与 readUTF 的阻塞特性

`accept()` 没连接来就等，`readUTF()` 没数据来就等——理解「阻塞」是理解网络编程与多线程配合的前提：单线程服务器一次只能服务一个客户端，群聊必须每连接一线程。

### 难点 1：群聊广播的连接一致性

服务端用 `List<Socket>` 保存全部连接，消息来了遍历列表转发。难点在于**连接的生命周期管理**：新连接 add、掉线 remove，遗漏任何一端都会累积失效连接，下次广播时对死连接 `writeUTF` 抛异常拖垮线程。

### 难点 2：时间格式化的类型匹配

`yyyy-MM-dd HH:mm:ss` 的格式器**不能**格式化 `LocalTime`（没有日期字段可填，直接抛异常）——格式器包含什么字段，时间对象就必须有什么字段。

### 易错点清单

- 客户端先于服务端启动——Connection refused；
- UDP 接收方端口与发送目标端口不一致——永远收不到；
- 中文解码用 `new String(data)` 不带 `0, length`——缓冲区尾部残留旧数据造成乱码尾巴；
- 在客户端用同一个线程既读键盘又收消息——互相阻塞，表现为「发不出」或「收不到」。

## 五、示例

### 示例 1：UDP 一发一收

```java
// 发送端
public class UdpSender {
    public static void main(String[] args) throws IOException {
        DatagramSocket ds = new DatagramSocket();
        byte[] arr = "你好，景天！".getBytes();
        DatagramPacket dp = new DatagramPacket(
                arr, arr.length, InetAddress.getByName("127.0.0.1"), 10086);
        ds.send(dp);
        ds.close();
    }
}

// 接收端
public class UdpReceiver {
    public static void main(String[] args) throws IOException {
        DatagramSocket ds = new DatagramSocket(10086);        // 端口与发送目标一致
        DatagramPacket dp = new DatagramPacket(new byte[1024], 1024);
        ds.receive(dp);                                      // 阻塞等待
        byte[] data = dp.getData();
        int length = dp.getLength();                          // 必须用实际长度解码
        System.out.println(new String(data, 0, length));
        ds.close();
    }
}
```

要点：`new String(data)` 会把整个 1024 字节都解码，尾部空字节变成乱码尾巴，必须带 `0, length`。

### 示例 2：TCP 一发一收

```java
// 服务端：先启动
public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocket ss = new ServerSocket(10086);
        System.out.println("等待连接...");
        Socket socket = ss.accept();                 // 阻塞，直到有客户端连入
        System.out.println("连接成功...");
        DataInputStream in = new DataInputStream(socket.getInputStream());
        while (true) {
            System.out.println(in.readUTF());
        }
    }
}

// 客户端
public class Client {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 10086);
        DataOutputStream out = new DataOutputStream(socket.getOutputStream());
        Scanner sc = new Scanner(System.in);
        while (true) {
            System.out.println("请输入要发送的消息：");
            out.writeUTF(sc.next());
        }
    }
}
```

### 示例 3：JDK8 时间类的四种常用操作

```java
public class Test {
    public static void main(String[] args) {
        DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

        // 1. 当前时间
        System.out.println(dtf.format(LocalDateTime.now()));

        // 2. 指定时间
        LocalDateTime nationalDay = LocalDateTime.of(1949, 10, 1, 0, 0, 0);
        System.out.println(nationalDay.format(
                DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH点mm分ss秒")));

        // 3. 调快一小时
        System.out.println(dtf.format(LocalDateTime.now().plusHours(1)));

        // 4. 计算出生至今的天数
        LocalDate birthday = LocalDate.parse("1998-06-15",
                DateTimeFormatter.ofPattern("yyyy-MM-dd"));
        LocalDateTime start = LocalDateTime.of(birthday, LocalTime.of(0, 0, 0));
        long days = Duration.between(start, LocalDateTime.now()).toDays();
        System.out.println("您总共活了" + days + "天");
    }
}
```

要点：`LocalDate` 只有日期，参与 `Duration.between` 前要拼上 `LocalTime.of(0,0,0)` 转成 `LocalDateTime`。

### 示例 4：群聊服务端（每连接一线程 + 广播）

```java
public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocket ss = new ServerSocket(10086);
        List<Socket> clients = new ArrayList<>();     // 连接登记簿
        while (true) {
            Socket socket = ss.accept();
            clients.add(socket);                      // 新连接登记
            new Thread(new ServerRunnable(socket, clients)).start();
        }
    }
}

public class ServerRunnable implements Runnable {
    private final Socket socket;
    private final List<Socket> clients;
    private String nickname;

    ServerRunnable(Socket socket, List<Socket> clients) {
        this.socket = socket;
        this.clients = clients;
    }

    @Override public void run() {
        try {
            DataInputStream in = new DataInputStream(socket.getInputStream());
            nickname = in.readUTF();                  // 首条消息是昵称
            System.out.println(nickname + " 进入聊天室，当前 " + clients.size() + " 人");

            while (true) {
                String data = in.readUTF();
                for (Socket s : clients) {
                    if (s != socket) {                // 排除发送者自己
                        DataOutputStream out = new DataOutputStream(s.getOutputStream());
                        out.writeUTF("\t\t\t\t" + nickname + " " + TimeUtils.getNowTime());
                        out.writeUTF("\t\t\t\t" + data);
                    }
                }
            }
        } catch (IOException e) {                     // readUTF 异常 = 对方断开
            clients.remove(socket);
            System.out.println(nickname + " 退出聊天室，当前 " + clients.size() + " 人");
        }
    }
}
```

### 示例 5：群聊客户端（收发双线程）

```java
public class Client {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 10086);
        DataInputStream in = new DataInputStream(socket.getInputStream());
        DataOutputStream out = new DataOutputStream(socket.getOutputStream());
        Scanner sc = new Scanner(System.in);

        // 线程 1：读键盘 → 发消息
        new Thread(() -> {
            try {
                out.writeUTF(sc.next());               // 昵称
                while (true) out.writeUTF(sc.next());
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }).start();

        // 线程 2：收服务器转发 → 打印
        new Thread(() -> {
            while (true) {
                try {
                    System.out.println(in.readUTF());
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }).start();
    }
}
```

要点：`readUTF` 阻塞，若与键盘输入放在同一线程，打字时收不到消息、收消息时打不了字——两个线程各管一个方向是唯一解法。

### 示例 6：时间工具类（私有构造 + 静态方法）

```java
public class TimeUtils {
    private TimeUtils() { }                            // 禁止实例化，纯工具类

    public static String getNowTime() {
        return LocalDateTime.now()
                .format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
    }
}
```

---
> 来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / 网络编程 / 06 网络编程技术要点 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/QyMUwRUBui4oMUkXPc3cIrDOnug)
