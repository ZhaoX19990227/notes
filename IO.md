#### BIO 一个服务端对应多个客户端

```java
public class Client {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1",8080);
        PrintStream printStream = new PrintStream(socket.getOutputStream());
        Scanner scanner = new Scanner(System.in);
        while(true) {
            System.out.println("请说：");
            String msg = scanner.nextLine();
            printStream.println(msg);
            printStream.flush();
        }
    }
}
```

```java
public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocket socket = new ServerSocket(8080);
        while(true) {
            Socket accept = socket.accept();
            new ServerThreadReader(accept).start();
        }
    }
}
```

```java
public class ServerThreadReader extends Thread{
    Socket socket;
    public ServerThreadReader(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        InputStream inputStream = null;
        try {
            inputStream = socket.getInputStream();
            BufferedReader bf = new BufferedReader(new InputStreamReader(inputStream));
            String msg;
            while ((msg = bf.readLine()) != null) {
                System.out.println(msg);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### 伪异步IO

```java
public class Server {
    public static void main(String[] args) {
        ServerSocket ss = null;
        try {
            ss = new ServerSocket(8080);
            HandleSocketServerPool pool = new HandleSocketServerPool(3, 10);
            while (true) {
                Socket socket = ss.accept();
                //将socket封装成一个任务对象交给线程池处理
                Runnable runnable = new ServerRunnableTask(socket);
                pool.execute(runnable);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class Client {
    public static void main(String[] args) throws IOException {
        try{
            Socket socket = new Socket("127.0.0.1",8080);
            PrintStream printStream = new PrintStream(socket.getOutputStream());
            Scanner scanner = new Scanner(System.in);
            while(true) {
                System.out.println("请说：");
                String msg = scanner.nextLine();
                printStream.println(msg);
                printStream.flush();
            }
        }catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```

```java
public class HandleSocketServerPool {
    private ExecutorService executorService;
    public HandleSocketServerPool(int maxThreadSize,int queueSize) {
        executorService =
                new ThreadPoolExecutor(3,maxThreadSize,120, TimeUnit.SECONDS,new ArrayBlockingQueue<Runnable>(queueSize));
    }
    public void execute(Runnable target){
        executorService.execute(target);
    }
}
```

```java
public class ServerRunnableTask implements Runnable{

    Socket socket;

    public ServerRunnableTask(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            InputStream inputStream = socket.getInputStream();
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
            String msg;
            while((msg = bufferedReader.readLine()) != null) {
                System.out.println("服务器收到：" + msg);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### 客户端上传文件

```java
public class Client {
    public static void main(String[] args) {
        try (InputStream is = new FileInputStream("/Users/zhaoxiang/WechatIMG325.png");
        ) {
            Socket socket = new Socket("127.0.0.1", 8082);
            DataOutputStream dos = new DataOutputStream(socket.getOutputStream());
            dos.writeUTF(".png");
            byte[] buffer = new byte[1024];
            int len;
            while ((len = is.read(buffer)) > 0) {
                dos.write(buffer, 0, len);
            }
            dos.flush();
          socket.shutdownOutput();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```

```java
public class Server {
    public static void main(String[] args) {
        try {
            ServerSocket ss = new ServerSocket(8082);
            while(true) {
                Socket socket = ss.accept();
                new ServerThreadReader(socket).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class ServerThreadReader extends Thread {
    Socket socket;

    public ServerThreadReader(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            DataInputStream dis = new DataInputStream(socket.getInputStream());
            //读取客户端发来文件的类型
            String suffix = dis.readUTF();
            System.out.println("服务器已接收到文件类型为：" + suffix);
            //定义一个字节输出管，文件写出去
            OutputStream ots = new FileOutputStream("/Users/zhaoxiang/serverPath/" + UUID.randomUUID().toString() + suffix);
            //从数据输入流中读取数据，输出到字节输出流中
            byte[] buffer = new byte[1024];
            int len;
            while((len = dis.read(buffer)) > 0) {
                ots.write(buffer,0,len);
            }
            ots.close();
            System.out.println("服务端接收文件保存成功");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### 群发消息

```java
public class Server {
    public static List<Socket> allOnlineClientSockets = Lists.newArrayList();
    public static void main(String[] args) {
        try {
            ServerSocket ss = new ServerSocket(8086);
            while (true) {
                Socket socket = ss.accept();
                allOnlineClientSockets.add(socket);
                new ServerReaderThread(socket).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class ServerReaderThread extends Thread {
    Socket socket;

    public ServerReaderThread(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            String msg;
            while((msg = br.readLine()) != null) {
                sendMsgToAllClient(msg);
            }
        } catch (IOException e) {
            System.out.println("有人下线");
            Server.allOnlineClientSockets.remove(socket);
        }
    }

    private void sendMsgToAllClient(String msg) throws IOException {
        for (Socket onlineClientSocket : Server.allOnlineClientSockets) {
            //通过socket流出给client
            PrintStream ps = new PrintStream(onlineClientSocket.getOutputStream());
            ps.println(msg);
            ps.flush();
        }
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        try {
            Socket socket = new Socket("127.0.0.1",8086);
            PrintStream printStream = new PrintStream(socket.getOutputStream());
            Scanner scanner = new Scanner(System.in);
            while(true) {
                System.out.println("请说：");
                String msg = scanner.nextLine();
                printStream.println(msg);
                printStream.flush();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class ClientReceive {
    public static void main(String[] args) {
        try {
            Socket socket = new Socket("127.0.0.1", 8086);
            BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            String msg;
            while ((msg = br.readLine()) != null) {
                System.out.println("client收到消息:" + msg);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### NIO和BIO区别

BIO是以流的方式处理数据，NIO是以块的方式处理数据。

BIO是阻塞的，NIO是非阻塞的。

BIO基于字节流、字符流进行操作，NIO基于Channel（通道）和Buffer（缓冲区）进行操作。

NIO中selector（选择器）用于监听多个通道的事件，因此单个线程就能监听多个客户端通道。 

<img src="/Users/zhaoxiang/Desktop/java系统笔记/IO.assets/image-20230206231548038.png" alt="image-20230206231548038" style="zoom:50%;" />

#### Nio写出文件

```java
public static void main(String[] args) {
    try {
        FileOutputStream fos = new FileOutputStream("data01.txt");
        FileChannel channel = fos.getChannel();
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        buffer.put("你好啊，小胖同学！".getBytes());
        buffer.flip();
        channel.write(buffer);
        channel.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

#### Nio读取数据

```java
public static void main(String[] args) {
    try {
        FileInputStream fileInputStream = new FileInputStream("data01.txt");
        FileChannel channel = fileInputStream.getChannel();
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        channel.read(buffer);
        buffer.flip();
        String result = new String(buffer.array(), 0, buffer.remaining());
        System.out.println(result);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

#### Nio复制文件

```java
public static void main(String[] args) {
    try {
        File srcFile = new File("/Users/zhaoxiang/WechatIMG325.png");
        File destFile = new File("/Users/zhaoxiang/xiaopang.png");
        FileInputStream fis = new FileInputStream(srcFile);
        FileOutputStream fos = new FileOutputStream(destFile);
        FileChannel fisChannel = fis.getChannel();
        FileChannel fosChannel = fos.getChannel();
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        while (true) {
            buffer.clear();
            int read = fisChannel.read(buffer);
            if (read == -1) {
                break;
            }
            buffer.flip();
            fosChannel.write(buffer);
        }
        fisChannel.close();
        fosChannel.close();
        System.out.println("复制成功");
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

#### TransferFrom&TransferTo 

```java
public static void main(String[] args) {
    try {
        FileInputStream fis = new FileInputStream("data01.txt");
        FileOutputStream fos = new FileOutputStream("data02.txt");
        FileChannel fisChannel = fis.getChannel();
        FileChannel fosChannel = fos.getChannel();

       //fosChannel.transferFrom(fisChannel,fisChannel.position(),fisChannel.size());
        fisChannel.transferTo(fisChannel.position(), fisChannel.size(),fosChannel);
        fisChannel.close();
        fosChannel.close();
        System.out.println("复制成功！");

    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### Selector轮询多个客户端通道

```java
public static void main(String[] args) throws IOException {
    SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 9999));
    socketChannel.configureBlocking(false);
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    Scanner sc = new Scanner(System.in);
    while (true) {
        System.out.println("请说：");
        String msg = sc.nextLine();
        buffer.put(("肥仔" + msg).getBytes());
        buffer.flip();
        socketChannel.write(buffer);
        buffer.clear();
    }
}
```

```java
public static void main(String[] args) throws IOException {
        ServerSocketChannel ssChannel = ServerSocketChannel.open();
        //非阻塞
        ssChannel.configureBlocking(false);
        //绑定端口
        ssChannel.bind(new InetSocketAddress(9999));
        //开启选择器
        Selector selector = Selector.open();
        //通道注册到选择器
        ssChannel.register(selector, SelectionKey.OP_ACCEPT);
        //选择器轮询准备好的事件
        while (selector.select() > 0) {
            //获取准备好的事件的集合
            Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();
            while (keyIterator.hasNext()) {
                //提取当前这个事件
                SelectionKey selectionKey = keyIterator.next();
                //如果是可获取的
                if (selectionKey.isAcceptable()) {
                    //获取当前接入的客户端通道
                    SocketChannel socketChannel = ssChannel.accept();
                    socketChannel.configureBlocking(false);
                    //服务端读取客户端的写事件
                    socketChannel.register(selector, SelectionKey.OP_READ);
                    //获取选择器上准备好的读事件
                } else if (selectionKey.isReadable()) {
                    SocketChannel channel = (SocketChannel) selectionKey.channel();
                    //读取事件
                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    int len = 0;
                    while ((len = channel.read(buffer)) > 0) {
                        buffer.flip();
                        System.out.println(new String(buffer.array(),0,len));
                        buffer.clear();
                    }
                }
                //移除当前事件
                keyIterator.remove();
            }
        }
    }
```

## NIO群聊

```java
public class Server {
    private Selector selector;
    private ServerSocketChannel ssChanel;
    private static final int PORT = 9999;

    public Server() {
        try {
            //开启选择器
            selector = Selector.open();
            //开启服务端通道
            ssChanel = ServerSocketChannel.open();
            //绑定端口
            ssChanel.bind(new InetSocketAddress(PORT));
            //服务端通道设置非阻塞
            ssChanel.configureBlocking(false);
            //服务端通道注册到选择器
            ssChanel.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 监听客户端
     */
    private void listen() {
        try {
            //如果选择器轮询到了事件
            while (selector.select() > 0) {
                Iterator<SelectionKey> it = selector.selectedKeys().iterator();
                while (it.hasNext()) {
                    //提取事件
                    SelectionKey selectionKey = it.next();
                    if (selectionKey.isAcceptable()) {
                        //获取客户端通道
                        SocketChannel socketChannel = ssChanel.accept();
                        socketChannel.configureBlocking(false);
                        //客户端通道注册到选择器上
                        socketChannel.register(selector, SelectionKey.OP_READ);
                    } else if (selectionKey.isReadable()) {
                        //接受客户端消息并转发给其他所有客户端
                        readClientData(selectionKey);
                    }
                    it.remove();//处理完毕后需要移除当前事件
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    /**
     * 接受当前客户端消息转发给其他所有客户端
     *
     * @param selectionKey
     */
    private void readClientData(SelectionKey selectionKey) {
        SocketChannel socketChannel = null;
        try {
            //得到当前客户端通道
            socketChannel = (SocketChannel) selectionKey.channel();
            //创建缓冲区接收客户端消息
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            int count = socketChannel.read(buffer);
            if (count > 0) {
                buffer.flip();
                //提取消息
                String msg = new String(buffer.array(), 0, buffer.remaining());
                System.out.println("接收客户端消息：" + msg);
                //消息转发
                sendMsgToAllClient(msg, socketChannel);
            }
        } catch (IOException e) {
            try {
                System.out.println("有人离线了：" + socketChannel.getRemoteAddress());
                //当前客户端离线
                selectionKey.cancel();
                socketChannel.close();
            } catch (Exception ex) {
                ex.printStackTrace();
            }

        }

    }

    private void sendMsgToAllClient(String msg, SocketChannel socketChannel) throws IOException {
        for (SelectionKey key : selector.keys()) {
            Channel channel = key.channel();
            //排除自己
            if (channel instanceof SocketChannel && channel != socketChannel) {
                ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes());
                ((SocketChannel) channel).write(buffer);
            }
        }
    }

    public static void main(String[] args) {
        Server server = new Server();
        server.listen();
    }
}
```

```java
public class Client {
    private Selector selector;
    private SocketChannel socketChannel;
    private static Integer port = 9999;

    public Client() {
        try {
            selector = Selector.open();
            socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", port));
            socketChannel.configureBlocking(false);
            socketChannel.register(selector, SelectionKey.OP_READ);
            System.out.println("当前客户端准备完成.....");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        Client client = new Client();
        //定义一个线程，监听服务端发过来的消息事件
        new Thread(new Runnable() {
            @Override
            public void run() {
                client.readInfo();
            }
        }).start();

        Scanner sc = new Scanner(System.in);
        while (true) {
            System.out.println("-------------");
            String s = sc.nextLine();
            client.sendToServer(s);
        }
    }

    private void sendToServer(String s) {
        try {
            socketChannel.write(ByteBuffer.wrap(("" + s).getBytes()));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void readInfo() {
        try {
            while (selector.select() > 0) {
                Iterator<SelectionKey> it = selector.selectedKeys().iterator();
                while (it.hasNext()) {
                    SelectionKey selectionKey = it.next();
                    if (selectionKey.isReadable()) {
                        SocketChannel sc = (SocketChannel) selectionKey.channel();
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        sc.read(buffer);
                        System.out.println(new String(buffer.array()).trim());
                    }
                    it.remove();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```