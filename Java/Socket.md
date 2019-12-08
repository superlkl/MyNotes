# 什么是Socket 

socket起源于Unix，而Unix/Linux基本哲学之一就是“一切皆文件”，都可以用“打开open –> 读写write/read –> 关闭close”模式来操作。我的理解就是Socket就是该模式的一个实现，socket即是一种特殊的文件，一些socket函数就是对其进行的操作（读/写IO、打开、关闭）。 

# Socket 编程

套接字使用TCP提供了两台计算机之间的通信机制。 客户端程序创建一个套接字，并尝试连接服务器的套接字。

当连接建立时，服务器会创建一个 Socket 对象。**<u>客户端和服务器现在可以通过对 Socket 对象的写入和读取来进行通信。</u>**

java.net.Socket 类代表一个套接字，并且 java.net.ServerSocket 类为服务器程序提供了一种来监听客户端，并与他们建立连接的机制。

以下步骤在两台计算机之间使用套接字建立TCP连接时会出现：

- 服务器实例化一个 ServerSocket 对象，表示通过服务器上的端口通信。
- 服务器调用 ServerSocket 类的 accept() 方法，该方法将一直等待，直到客户端连接到服务器上给定的端口。
- 服务器正在等待时，一个客户端实例化一个 Socket 对象，**<u>指定服务器名称和端口号来请求连接</u>。**
- Socket 类的构造函数试图将客户端连接到指定的服务器和端口号。如果通信被建立，则在客户端创建一个 Socket 对象能够与服务器进行通信。
- 在服务器端，accept() 方法返回服务器上一个新的 socket 引用，该 socket 连接到客户端的 socket。
- 连接建立后，通过使用 I/O 流在进行通信，**<u>每一个socket都有一个输出流和一个输入流，客户端的输出流连接到服务器端的输入流，而客户端的输入流连接到服务器端的输出流</u>。**

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

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204134817533.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MjU3Mzgz,size_16,color_FFFFFF,t_70)

既然socket是“open—write/read—close”模式的一种实现，那么socket就提供了这些操作对应的函数接口。下面以TCP为例，介绍几个基本的socket接口函数。 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204134946950.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MjU3Mzgz,size_16,color_FFFFFF,t_70)

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



# UDP发送广播

广播使用的特殊的IP地址：最后一位是255时的IP地址是给广播预留的IP地址，如：192.168.88.255 

udp广播包其实就是往整个局域网内发送udp数据报，而不是单个目的终端。发送udp广播包的时候要记住发送频率，否则很容易形成广播风暴，而导致局域网的网络瘫痪。如果局域网有多个网段，那不同的网段是接收不到其它局域网的广播包的。 

## 广播的实现原理

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128205701378.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MjU3Mzgz,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128205821798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MjU3Mzgz,size_16,color_FFFFFF,t_70)

##  UDP的实现 

- **客户端:**

1. 创建用于UDP通信的socket对象---DatagramSocket(用于UDP数据的发送和接收)---数据报套接字
2.  准备数据,封装包----DatagramPacket(数据包)
3. 发送数据,通过send方法
4. 关闭套接字对象--socket对象

- **服务器端: 接收数据**

1. 创建socket套接字对象,并绑定端口号
2. 创建包对象,创建空数组,准备接收数据
3. 接收数据
4. 关闭资源

- **UDP广播方式**

1. 同一网段所有主机都能接收，前提是端口要监听
2. 客户端发送广播，开启端口监听的服务端接收并打印消息
3.  广播的实现 :由客户端发出广播，服务器端接收 
4. String host = "255.255.255.255";  //广播地址--代表所有主机
5. 10.0.122.255----代表前三个网段是 10.0.122的所有主机 

### 测试一

**接收广播，先运行**

```java
public class UDPReceiveDemo{
    public static void main(String[] args) throws IOException {
        byte[] b=new byte[1024];
        //创建数据报包准备接收广播信息
        DatagramPacket dp=new DatagramPacket(b, b.length);
        MulticastSocket soc=new MulticastSocket(6666);
        //再多播的时候需要将相关的的IP地址加进来
        soc.joinGroup(InetAddress.getByName("225.0.0.1"));
        soc.receive(dp);
        String s=new String(b,0,dp.getLength());
        System.out.println("接收方收到的广播信息为： "+s);
    }
}
```

**发送广播，后运行**

```java
public class UDPSendDemo{
    public static void main(String[] args) throws IOException {
        //下面的msg信息可以进行相关的拼接然后实现和feiq通信，只需要按照feiq的相关通信格式即可
        String msg="这是一条广播消息";
        InetAddress ips= InetAddress.getByName("225.0.0.1");//前面的地址在广播地址的范围内
        //创建多播数据报通道
        MulticastSocket socket=new MulticastSocket();
        //将IP加入多播组
        socket.joinGroup(ips);
        //准备好相应的数据报
        DatagramPacket dp=new DatagramPacket
                (msg.getBytes(), msg.getBytes().length,ips,6666
        );
        socket.send(dp);
        socket.close();
    }
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128214007830.png)

### 测试二

```java
public class UdpReceive {
    public static void main(String[] args)throws Exception
    {
        //创建UDP socket，建立端点
        DatagramSocket ds = new DatagramSocket(20000);	//监听20000端口

        //定义数据包，用于存储数据
        byte[] buf = new byte[1024];
        DatagramPacket dp = new DatagramPacket(buf,buf.length);

        ds.receive(dp);

        String ip = dp.getAddress().getHostAddress();	//数据提取
        String data = new String(dp.getData(),0,dp.getLength());
        int port = dp.getPort();
        System.out.println(data+"."+port+".."+ip);
        ds.close();
    }
}
```



```java
public class UdpSend {
    public static void main(String[] args)throws Exception
    {
        DatagramSocket ds  = new DatagramSocket();
        byte[] buf = "UDP Demo".getBytes();
        //20000为定义的端口
        DatagramPacket dp = new DatagramPacket
                (buf,buf.length, InetAddress.getByName("255.255.255.255"),20000);
        ds.send(dp);
        ds.close();
    }
}
```

 **DatagramSocket其中一个构造函数**

```java
public DatagramSocket(int port) throws SocketException {
        this(port, null);
    }
```

**DatagramPacket其中一个构造函数**

```java
  public DatagramPacket(byte buf[], int length,
                          InetAddress address, int port) {
        this(buf, 0, length, address, port);
    }
```

**分析：**

### InetAddress类

InetAddress类没有构造方法，所以不能直接new出一个对象，但是可以通过InetAddress类的静态方法获得InetAddress的对象；

- **InetAddress.getLocalHost();**

- **InetAddress.getByName("");**

```java
public class Test {
    public static void main(String[] args) throws UnknownHostException {
        //获取本机的InetAddress实例  
        InetAddress address = InetAddress.getLocalHost();

        System.out.println("计算机名：" + address.getHostName());

        System.out.println("IP地址：" + address.getHostAddress());

        //获取字节数组形式的IP地址
        byte[] bytes = address.getAddress();

        System.out.println("字节数组形式的IP：" + Arrays.toString(bytes));

        System.out.println("直接输出InetAddress对象："+address);//直接输出InetAddress对象  

        //根据机器名获取InetAddress实例
        InetAddress address3 = InetAddress.getByName("192.168.43.69");

        System.out.println("计算机名：" + address3.getHostName());

        System.out.println("IP地址：" + address3.getHostName());

    }
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129164635479.png)

>**String getHostName()：获取InetAddress对象的域名；**
>
>**String getHostAddress()：获取InetAddress对象的IP地址；**
>
>**getLocalHost()：获得一个InetAddress对象，该对象含有本地机的域名和IP地址。**
>
>**InetAddress.getByName("www.163.com")----在给定主机名的情况下确定主机的IP地址** 
>
>​                                                                               **----如果参数为null,获得的是本机的IP地址** 

## 数据电报套接字

- **DatagramSocket 和 DatagramPacket 两个类是 基于UDP 协议进行通信的包装类** 

- **实现两个客户端通过 UDP协议通信，使用DatagramSocket 和 DatagramPacket类** 

### 客户端

1.  实例化DatagramSocket类(带上指定端口)，创建客户端 
2.  准备数据，数据是以字节数组发送的 
3.  打包数据，使用 DatagramPacket 类 + 服务器地址+ 端口 
4.  发送数据  关闭连接 

```java
public class Client {
    public static void main(String[] args) throws IOException {
        // 1,创建服务端+端口
        DatagramSocket client = new DatagramSocket(614);

        // 2,准备数据
        String msg = "batman，I am you friends！";

        byte [] data = msg.getBytes();

        // 3,打包（发送的地点及端口）
        DatagramPacket packet = new DatagramPacket(data, data.length, new InetSocketAddress("127.0.0.1", 1219));

        // 4,发送资源
        client.send(packet);
        System.out.println("已发送数据包到服务端");
        // 5,关闭资源
        client.close();
    }
}
```

### 服务端

1.  实例化DatagramSocket类+指定端口 
2.  准备接收的字节数组，封装 DatagramPacket 
3.  接受数据 
4.  解析数据 
5.  关闭连接 

```java
public class Server {
    public static void main(String[] args) throws IOException {
        // 1,创建服务端+端口
        DatagramSocket server = new DatagramSocket(1219);

        System.out.println("等待客户端发送数据");
        // 2,准备接收容器
        byte[] container = new byte[1024];

        // 3,封装成包 new DatagramPacket(byte[] b,int length)
        DatagramPacket packet = new DatagramPacket(container, container.length);

        // 4,接收数据,使用 DatagramSocket的实例的 receive( DatagramPacket ) 方法进行接收
        server.receive(packet);

        // 5,分析数据
        byte[] data = packet.getData();
        int length = packet.getLength();
        String msg = new String(data, 0, length);
        System.out.println(msg);
        server.close();
    }
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129171332817.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129172150154.png)

注：

在不考虑子网掩码的情况下，192.168.1.255是一个子网的广播地址；255.255.255.255应该是全局的广播地址； 
