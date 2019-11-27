# 什么是Socket 

socket起源于Unix，而Unix/Linux基本哲学之一就是“一切皆文件”，都可以用“打开open –> 读写write/read –> 关闭close”模式来操作。我的理解就是Socket就是该模式的一个实现，socket即是一种特殊的文件，一些socket函数就是对其进行的操作（读/写IO、打开、关闭）。 

# Socket 编程

套接字使用TCP提供了两台计算机之间的通信机制。 客户端程序创建一个套接字，并尝试连接服务器的套接字。

当连接建立时，服务器会创建一个 Socket 对象。<u>客户端和服务器现在可以通过对 Socket 对象的写入和读取来进行通信。</u>

java.net.Socket 类代表一个套接字，并且 java.net.ServerSocket 类为服务器程序提供了一种来监听客户端，并与他们建立连接的机制。

以下步骤在两台计算机之间使用套接字建立TCP连接时会出现：

- 服务器实例化一个 ServerSocket 对象，表示通过服务器上的端口通信。
- 服务器调用 ServerSocket 类的 accept() 方法，该方法将一直等待，直到客户端连接到服务器上给定的端口。
- 服务器正在等待时，一个客户端实例化一个 Socket 对象，<u>指定服务器名称和端口号来请求连接</u>。
- Socket 类的构造函数试图将客户端连接到指定的服务器和端口号。如果通信被建立，则在客户端创建一个 Socket 对象能够与服务器进行通信。
- 在服务器端，accept() 方法返回服务器上一个新的 socket 引用，该 socket 连接到客户端的 socket。
- 连接建立后，通过使用 I/O 流在进行通信，<u>每一个socket都有一个输出流和一个输入流，客户端的输出流连接到服务器端的输入流，而客户端的输入流连接到服务器端的输出流</u>。

TCP 是一个双向的通信协议，因此数据可以通过两个数据流在同一时间发送.以下是一些类提供的一套完整的有用的方法来实现 socket。

**Socket的构造函数**

```java
   public Socket(String host, int port)
        throws UnknownHostException, IOException
    {
        this(host != null ? new InetSocketAddress(host, port) :
             new InetSocketAddress(InetAddress.getByName(null), port),
             (SocketAddress) null, true);
}
```

第一个参数传入服务器的IP地址，第二个参数传入端口号

---



```java
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class SocketServer {
    public static void main(String[] args) throws Exception {
        // 创建服务接口
        ServerSocket socket = new ServerSocket(8888);
        System.out.println("由服务器发送消息");
        // 接收请求
        Socket a = socket.accept();
        // 调用服务端的业务逻辑
        String result = "努力学习，尽量不要玩游戏，容易玩物丧志！加油！！";
        // 获得输出流
        OutputStream out = a.getOutputStream();
        // 发送数据
        out.write(result.getBytes());
        // 关闭资源
        out.close();// 关闭输出流
        a.close();// 关闭接收
        socket.close();// 关闭服务
    }
}
```

```java
public class SocketClient {
    public static void main(String[] args) throws Exception {
        // 创建socket
        Socket s = new Socket("127.0.0.1", 8888);
        // 获得输入流
        java.io.InputStream in = s.getInputStream();
        //读取字节流
        BufferedReader reader = new BufferedReader(new InputStreamReader(in));
        String line = null;
        while ((line = reader.readLine()) != null) {
            System.out.println("服务器给的忠告！");
            System.out.println(line);
        }
        in.close();
        s.close();
    }
}
```

先启动服务端

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127195107351.png)

再启动客服端

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127195204481.png)

# socket的基本操作 

既然socket是“open—write/read—close”模式的一种实现，那么socket就提供了这些操作对应的函数接口。下面以TCP为例，介绍几个基本的socket接口函数。 

## 测试一

```java
public class Client {
    //客户端（Socket）
    public static void main(String[] args) throws IOException {
//设置发送给服务器的ip和端口号
        Socket s=new Socket("127.0.0.1",12345);
        //创建输出字节流
        OutputStream os=s.getOutputStream();
        //包装成数据输出流
        DataOutputStream dos=new DataOutputStream(os);
        //写入内容
        dos.writeUTF("hello world");
    }
}
```

```java
import java.io.DataInputStream;
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {
    public static void main(String[] args) throws IOException {
//创建ServerSocket并监听6666端口
        ServerSocket ss=new ServerSocket(12345);
        //无限循环，用于不断接收请求
        while (true){
            //调用accept方法，接受客户端连接对象，阻塞式方法（本连接不结束，其他连接无法被占用）
            Socket s=ss.accept();
            System.out.println("已连接");
            //接收客户端传入的数据流
            DataInputStream dis=new DataInputStream(s.getInputStream());
            //阻塞式方法，客户不传消息，就一直堵塞，程序不能往下执行，直到有消息读入
            String str=dis.readUTF();
            System.out.println(str);
        }
    }
}
```

先运行客户端，再运行服务端

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127192746140.png)





# 麻烦

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127192401914.png)

问题出现的原因是：端口已经被占用。 (`eg`:端口号：10000)
解决方法思路是：
1、重新分配一个端口号 (`eg`:端口号：12345)
2、断开该端口号连接。再重新使用端口号(10000) 

