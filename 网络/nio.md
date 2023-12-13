## BIO

[C10K问题](http://www.kegel.com/c10k.html "C10K问题")


BIO随着连接数的过大，C10K要抛出10K的线程

### 原理图

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_cdc103b33773204be291eb3e3b57ced6.jpg)


### 网络架构图

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_b6c8ef1f10212bc5bf55498c27eedd34.jpg)



### BIO代码实例

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @author xingzhiwei
 * @createBy IntelliJ IDEA
 * @time 2021/3/16 3:56 下午
 * @email jsjxzw@163.com
 * 服务端
 */
public class SocketIOPropertites {
    //server socket listen property:
    private static final int RECEIVE_BUFFER = 10;
    private static final int SO_TIMEOUT = 0;
    private static final boolean REUSE_ADDR = false;
    private static final int BACK_LOG = 2;
    //client socket listen property on server endpoint:
    private static final boolean CLI_KEEPALIVE = false;
    private static final boolean CLI_OOB = false;
    private static final int CLI_REC_BUF = 20;
    private static final boolean CLI_REUSE_ADDR = false;
    private static final int CLI_SEND_BUF = 20;
    private static final boolean CLI_LINGER = true;
    private static final int CLI_LINGER_N = 0;
    private static final int CLI_TIMEOUT = 0;
    private static final boolean CLI_NO_DELAY = false;

    public static void main(String[] args) {

        ServerSocket server = null;
        try {
            server = new ServerSocket();
            server.bind(new InetSocketAddress(9090), BACK_LOG);
            server.setReceiveBufferSize(RECEIVE_BUFFER);
            server.setReuseAddress(REUSE_ADDR);
            server.setSoTimeout(SO_TIMEOUT);

        } catch (IOException e) {
            e.printStackTrace();
        }
        System.out.println("server up use 9090!");
        try {
            while (true) {

                final Socket client = server.accept();  //阻塞的，没有 -1  一直卡着不动  accept(4,
                System.out.println("client port: " + client.getPort());

                client.setKeepAlive(CLI_KEEPALIVE);
                client.setOOBInline(CLI_OOB);
                client.setReceiveBufferSize(CLI_REC_BUF);
                client.setReuseAddress(CLI_REUSE_ADDR);
                client.setSendBufferSize(CLI_SEND_BUF);
                client.setSoLinger(CLI_LINGER, CLI_LINGER_N);
                client.setSoTimeout(CLI_TIMEOUT);
                client.setTcpNoDelay(CLI_NO_DELAY);

                //client.read   //阻塞   没有  -1 0
                new Thread(
                        new Runnable() {
                            @Override
                            public void run() {
                                try {
                                    InputStream in = client.getInputStream();
                                    BufferedReader reader = new BufferedReader(new InputStreamReader(in));
                                    char[] data = new char[1024];
                                    while (true) {

                                        int num = reader.read(data);

                                        if (num > 0) {
                                            System.out.println("client read some data is :" + num + " val :" + new String(data, 0, num));
                                        } else if (num == 0) {
                                            System.out.println("client readed nothing!");
                                            continue;
                                        } else {
                                            System.out.println("client readed -1...");
                                            System.in.read();
                                            client.close();
                                            break;
                                        }
                                    }

                                } catch (IOException e) {
                                    e.printStackTrace();
                                }

                            }
                        }
                ).start();

            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                server.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
}

```


![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_ea564a62bdaff5d7ec30f8337bbf7eb0.jpg)


### 弊端

BIO 慢,为了解决这个问题。
要找到弊端：阻塞，accept阻塞，read阻塞。因为阻塞，抛出线程，每个线程互不影响，抛出多个线程。java中的线程是linux的子进程，上下文切换（用户态内核态切换）非常消耗性能。
阻塞是内核限制的，只有内核能解决。内核新增 NOBLOCKING


## NIO

### NIO代码实例

```java
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.LinkedList;
/**
 * @author xingzhiwei
 * @createBy IntelliJ IDEA
 * @time 2021/3/17 6:17 下午
 * @email jsjxzw@163.com
 */
public class SocketNIO {
    //  what   why  how
    public static void main(String[] args) throws Exception {
        LinkedList<SocketChannel> clients = new LinkedList<>();
        ServerSocketChannel ss = ServerSocketChannel.open();  //服务端开启监听：接受客户端
        ss.bind(new InetSocketAddress(9090));
        ss.configureBlocking(false); //重点  OS  NONBLOCKING!!!  //只让接受客户端  不阻塞


        while (true) {
            //接受客户端的连接
            Thread.sleep(1000);
            SocketChannel client = ss.accept(); //不会阻塞？  操作系统 -1  JAVA面向对象是 NULL
            //accept  调用内核了：1，没有客户端连接进来，返回值？在BIO 的时候一直卡着，但是在NIO ，不卡着，返回-1，NULL
            //如果来客户端的连接，accept 返回的是这个客户端的fd  5，client  object
            //NONBLOCKING 就是代码能往下走了，只不过有不同的情况

            if (client == null) {
                //   System.out.println("null.....");
            } else {
                client.configureBlocking(false); //重点  socket（服务端的listen socket<连接请求三次握手后，往我这里扔，我去通过accept 得到  连接的socket>，连接socket<连接后的数据读写使用的> ）
                int port = client.socket().getPort();
                System.out.println("client..port: " + port);
                clients.add(client);
            }

            ByteBuffer buffer = ByteBuffer.allocateDirect(4096);  //可以在堆里   堆外

            //遍历已经链接进来的客户端能不能读写数据
            for (SocketChannel c : clients) {   //串行化！！！！  多线程！！
                int num = c.read(buffer);  // >0  -1  0   //不会阻塞
                if (num > 0) {
                    buffer.flip();
                    byte[] aaa = new byte[buffer.limit()];
                    buffer.get(aaa);

                    String b = new String(aaa);
                    System.out.println(c.socket().getPort() + " : " + b);
                    buffer.clear();
                }


            }
        }
    }
}
```


### 原理图

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_192430a25afeaf99ef8b475a08857cd7.jpg)


### 优点

NOI 解决了不停抛出许多线程的问题，可以在一个线程里面处理 accept 和 read。

### 弊端
一直不停的调accept 和 read。随着建立的连接越来越多，read 越来越多。read 调用了系统调用 recv，用户态内核态切换消耗性能。（read无罪，无效的无用的read别调起）

全量遍历，用户态内核态不停切换