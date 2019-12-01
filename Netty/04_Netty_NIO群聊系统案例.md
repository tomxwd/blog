---
title: 04_Netty_NIO群聊系统案例
date: 2019-12-01 18:30:21
tags: 
 - Netty
categories:
 - Netty
---

# 04_Netty_NIO群聊系统案例

实例要求：

1. 编写一个NIO群聊系统，实现服务器端和客户端之间的数据简单通讯（非阻塞）；
2. 实现多人群聊；
3. 服务器端：可以监测用户上线，离线，并实现消息转发功能；
4. 客户端：通过Channel可以无阻塞发送消息给其他所有用户，同时可以接受其他用户发送的消息（由服务器转发得到）；
5. 目的：进一步理解NIO非阻塞网络编程机制；

分析：

1. 编写服务器端
   - 服务器启动并监听6667端口
   - 服务器接收客户端信息，并实现转发【处理上线和离线】
2. 编写客户端
   - 连接服务器
   - 发送消息
   - 接收服务器消息

代码：

服务器端代码：

```java
public class GroupChatServer {

    private Selector selector;
    private ServerSocketChannel listenChannle;
    private static final int PORT = 6667;

    /**
     * 无参构造器
     */
    public GroupChatServer() {
        try {
            // 获取选择器
            selector = Selector.open();
            // 获取ServerSocketChannel
            listenChannle = ServerSocketChannel.open();
            // 绑定端口
            listenChannle.bind(new InetSocketAddress(PORT));
            // 设置非阻塞模式
            listenChannle.configureBlocking(false);
            // 将listenChannel注册到selector上去,关注的是accept事件
            listenChannle.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 监听
    public void listen() {
        try {
            // 循环处理
            while (true) {
                int count = selector.select(2000);
                // 有事件需要处理
                if (count > 0) {
                    // 遍历得到的SelectionKey集合
                    Set<SelectionKey> selectionKeys = selector.selectedKeys();
                    Iterator<SelectionKey> keyIterator = selectionKeys.iterator();
                    while (keyIterator.hasNext()) {
                        SelectionKey key = keyIterator.next();
                        // 监听到的是ACCPET事件
                        if (key.isAcceptable()) {
                            SocketChannel socketChannel = listenChannle.accept();
                            // 设置为非阻塞模式
                            socketChannel.configureBlocking(false);
                            // 将该通道注册到selector中
                            socketChannel.register(selector, SelectionKey.OP_READ, ByteBuffer.allocate(1024));
                            // 给出提示XXX上线
                            System.out.println(socketChannel.getRemoteAddress() + "上线");
                        }
                        // 监听到的是READ事件
                        if (key.isReadable()) {
                            // 处理读事件的方法
                            readData(key);
                        }
                        // 处理完成移除当前事件，防止重复
                        keyIterator.remove();
                    }
                } else {
                    System.out.println("等待");
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {

        }
    }

    /**
     * 读取客户端消息
     *
     * @param selectionKey
     */
    private void readData(SelectionKey selectionKey) {
        // 定义一个SocketChannel
        SocketChannel channel = null;
        try {
            // 取得关联的Channel
            channel = (SocketChannel) selectionKey.channel();
            // 创建Buffer
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            int read = channel.read(buffer);
            // 根据read的值做处理
            if (read > 0) {
                // 把Buffer转String
                String msg = new String(buffer.array());
                // 输出该消息
                System.out.println("from 客户端：" + msg);
                // 向其他客户端转发消息(排除自己)
                sendInfoToOtherClients(msg, channel);
            }
        } catch (IOException e) {
            try {
                System.out.println(channel.getRemoteAddress() + " 离线了...");
                // 取消注册
                selectionKey.cancel();
                // 关闭通道
                channel.close();
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }

    }

    /**
     * 转发消息给其他客户（通过通道）
     *
     * @param msg
     * @param self
     */
    private void sendInfoToOtherClients(String msg, SocketChannel self) throws IOException {
        System.out.println("服务器转发消息中...");
        // 遍历 所有注册到Selector上的SocketChannel并排除self
        for (SelectionKey key : selector.keys()) {
            // 通过key取出对应的SocketChannel
            Channel targetChannel = key.channel();
            // 排除self
            if (targetChannel instanceof SocketChannel && targetChannel != self) {
                // 转型为SocketChannel
                SocketChannel dest = (SocketChannel) targetChannel;
                // 将msg存储到buffer中
                ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes());
                // 将Buffer的数据写入通道
                dest.write(buffer);
            }

        }

    }

    public static void main(String[] args) {
        // 创建服务器对象
        GroupChatServer groupChatServer = new GroupChatServer();
        groupChatServer.listen();
    }

}
```

客户端代码：

```java
public class GroupChatClient {

    private final String HOST = "127.0.0.1";
    private final int PORT = 6667;
    private Selector selector;
    private SocketChannel socketChannel;
    private String username;

    /**
     * 无参构造器
     */
    public GroupChatClient() throws IOException {
        selector = Selector.open();
        // 连接服务器
        socketChannel = SocketChannel.open(new InetSocketAddress(HOST, PORT));
        // 设置非阻塞
        socketChannel.configureBlocking(false);
        // 将Channel注册到Selector
        socketChannel.register(selector, SelectionKey.OP_READ);
        // 得到username
        username = socketChannel.getLocalAddress().toString().substring(1);
        System.out.println(username + " 客户端准备完毕");
    }

    /**
     * 向服务器发送消息
     * @param info
     */
    public void sendInfo(String info) {
        info = username + " say: " + info;
        try{
            socketChannel.write(ByteBuffer.wrap(info.getBytes()));
        } catch (IOException e){
            e.printStackTrace();
        }
    }

    /**
     * 读取从服务器回复的信息
     */
    public void readInfo(){
        try{
            int readChannels = selector.select();
            if(readChannels>0){
                // 有事件发生的通道
                Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                while (iterator.hasNext()) {
                    SelectionKey key = iterator.next();
                    if(key.isReadable()){
                        // 得到相关的通道
                        SocketChannel sc = ((SocketChannel) key.channel());
                        // 得到一个Buffer
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        // 读取数据
                        sc.read(buffer);
                        // 把读取到的数据转成字符串
                        String msg = new String(buffer.array());
                        System.out.println(msg.trim());
                    }
                    // 删除当前SelectionKey，防止重复操作
                    iterator.remove();
                }
            } else {
                //System.out.println("没有可用的通道");
            }
        } catch (IOException e){
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws IOException {
        // 启动客户端
        GroupChatClient chatClient = new GroupChatClient();
        // 启动一个线程，每隔三秒，读取从服务器端发送的数据
        new Thread(()->{
            while (true){
                chatClient.readInfo();
                try{
                    Thread.currentThread().sleep(3000);
                } catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        }).start();
        // 客户端发送数据给服务器端
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()){
            String s = scanner.nextLine();
            chatClient.sendInfo(s);
        }
    }

}
```







