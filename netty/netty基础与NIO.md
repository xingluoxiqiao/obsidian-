# NIO三大组件
NIO系统的核心在于通道和缓冲区
通道表示打开到IO设备的连接，
缓冲区用于容纳数据
若需要使用 NIO 系统，需要获取用于**连接 IO 设备的通道**以及用于**容纳数据的缓冲区**。然后操作缓冲区，对数据进行处理
## Channel
通道，负责传输
常见的Channel有FileChannel，DatagramChannel，SocketChannel，ServerSocketChannel，其中FileChannel用于文件传输，其余三种用于网络通信
## Selector
### 多线程方式处理连接
在Selector出现前，socket的并发连接通常采取多线程的方式，每个线程处理一个连接，有以下缺点：
内存占用高、线程上下文切换成本高、只适合连接数少的场景
![[Pasted image 20230925113204.png]]
### 线程池方式处理连接
使用线程池，让线程池中的线程去处理连接
有以下缺点：
1.阻塞模式下，线程只能处理一个连接（线程池中的线程获取任务（task）后，**只有当其执行完任务之后（断开连接后），才会去获取并执行下一个任务**；若socke连接一直未断开，则其对应的线程无法处理其他socke连接）
2.只适合短连接场景（短连接即建立连接发送请求并响应后就立即断开，使得线程池中的线程可以快速处理其他连接）
![[Pasted image 20230925113440.png]]
### 选择器方式处理连接
selector 的作用就是配合一个线程来管理多个 channel（fileChannel因为是阻塞式的，所以无法使用selector），获取这些 channel 上发生的事件，这些 channel 工作在非阻塞模式下，当一个channel中没有执行任务时，可以去执行其他channel中的任务。适合连接数多，但流量较少的场景

若事件未就绪，调用 selector 的 select() 方法会阻塞线程，直到 channel 发生了就绪事件。这些事件就绪后，select 方法就会返回这些事件交给 thread 来处理
![[Pasted image 20230925113447.png]]

## Buffer
缓冲区，负责存储
除boolean外的七种基本类型分别有自己的缓冲区
- ByteBuffer
    - MappedByteBuffer
    - DirectByteBuffer
    - HeapByteBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer
- CharBuffer
### 核心属性
```
//必须满足 mark <= position <= limit <= capacity
private int mark = -1;
private int position = 0;
private int limit;
private int capacity;Copy
```
capacity：缓冲区的容量。通过构造函数赋予，一旦设置，无法更改
limit：缓冲区的界限。位于limit 后的数据不可读写。缓冲区的限制不能为负，并且不能大于其容量
position：下一个读写位置的索引（类似PC）。缓冲区的位置不能为负，并且不能大于limit
mark：记录当前position的值。position被改变后，可以通过调用reset() 方法恢复到mark的位置。
![[Pasted image 20230925120413.png]]
## ByteBuffer
### buffer中的方法（被bytebuffer继承）
#### put
将一个数据放入到缓冲区中
进行该操作后，position的值加1，指向下一个可以放入的位置
capacity=limit，为缓冲区容量的值
#### flip
切换对缓冲区的操作模式，由写->读 / 读->写
进行该操作后，
- 如果是写模式->读模式，position = 0 ， limit 指向最后一个元素的下一个位置，capacity不变
- 如果是读->写，则恢复为put()方法中的值
#### get
读取缓冲区中的一个值
进行该操作后，position+1，如果超出limit会抛异常
如果是带参数的，表示取特定的某个值，position不变
#### rewind
只能在读模式下使用
进行该操作后，会恢复position、limit和capacity的值，变为进行get()前的值
#### clean
clean方法会将缓冲区中的各个属性恢复为最初的状态，position=0，capacity=limit
此时缓冲区中的数据依然存在，但处于“被遗忘”状态，下次进行写操作时会被覆盖
#### mark和reset
mark()方法会将postion的值保存到mark属性中
reset()方法会将position的值改为mark中保存的值
#### compact（bytebuffer独有）
compact会把未读完的数据向前压缩，然后切换到写模式
数据前移后，原位置的值并未清零，写时会**覆盖**之前的值
![[Pasted image 20230925121401.png]]
#### clean和compact的对比
clear只是对position、limit、mark进行重置，而compact在对position进行设置，以及limit、mark进行重置的同时，还涉及到数据在内存中拷贝（会调用arraycopy）。**所以compact比clear更耗性能。**
但compact能保存你未读取的数据，将新数据追加到为读取的数据之后；而clean则不行，若你调用了clean，则未读取的数据就无法再读取到了
**所以需要根据情况来判断使用哪种方法进行模式切换**
#### allocate和allocateDirect

allocate是class java.nio.HeapByteBuffer 堆内存中的对象，所以收到垃圾(GC)回收的影响；读写效率相对不高；
sllocateDirect是class java.nio.DirectByteBuffer 直接内存(系统内存)里面的对象，不会受到垃圾回收的影响；读写效率比较高，会少一次的复制拷贝；分配时的效率比较低，并且容易出现内存泄露，需要手动释放内存
### 字符串与ByteBuffer的相互转换
#### 字符串调用getbyte
编码：字符串调用getByte方法获得byte数组，将byte数组放入ByteBuffer中
解码：先调用ByteBuffer的flip方法，然后通过StandardCharsets的decoder方法解码
```
public class Translate {
    public static void main(String[] args) {
        // 准备两个字符串
        String str1 = "hello";
        String str2 = "";
        
        ByteBuffer buffer1 = ByteBuffer.allocate(16);
        // 通过字符串的getByte方法获得字节数组，放入缓冲区中
        buffer1.put(str1.getBytes());

        // 将缓冲区中的数据转化为字符串
        // 切换模式
        buffer1.flip();
        
        // 通过StandardCharsets解码，获得CharBuffer，再通过toString获得字符串
        str2 = StandardCharsets.UTF_8.decode(buffer1).toString();
    }
}
```
#### StandardCharsets的encode方法获得ByteBuffer
**编码**：通过StandardCharsets的encode方法获得ByteBuffer，此时获得的ByteBuffer为读模式，无需通过flip切换模式
**解码**：通过StandardCharsets的decoder方法解码
```
public class Translate {
    public static void main(String[] args) {
        // 准备两个字符串
        String str1 = "hello";
        String str2 = "";

        // 通过StandardCharsets的encode方法获得ByteBuffer
        // 此时获得的ByteBuffer为读模式，无需通过flip切换模式
        ByteBuffer buffer1 = StandardCharsets.UTF_8.encode(str1);

        // 将缓冲区中的数据转化为字符串
        // 通过StandardCharsets解码，获得CharBuffer，再通过toString获得字符串
        str2 = StandardCharsets.UTF_8.decode(buffer1).toString();
	}
}
```
#### 字节数组传给ByteBuffer的wrap()方法
**编码**：字符串调用getByte()方法获得字节数组，将字节数组传给**ByteBuffer的wrap()方法**，通过该方法获得ByteBuffer。**同样无需调用flip方法切换为读模式**
**解码**：通过StandardCharsets的decoder方法解码
```
public class Translate {
    public static void main(String[] args) {
        // 准备两个字符串
        String str1 = "hello";
        String str2 = "";

        // 通过StandardCharsets的encode方法获得ByteBuffer
        // 此时获得的ByteBuffer为读模式，无需通过flip切换模式
        ByteBuffer buffer1 = ByteBuffer.wrap(str1.getBytes());
        ByteBufferUtil.debugAll(buffer1);

        // 将缓冲区中的数据转化为字符串
        // 通过StandardCharsets解码，获得CharBuffer，再通过toString获得字符串
        str2 = StandardCharsets.UTF_8.decode(buffer1).toString();
    }
}
```
### 分散读集中写
减少数据在buffer和channel之间的复制，提升效率
channel的read和write方法都是从buffer中获取/写入数据
这两个方法都可以传入一个buffer数组作为参数
read就是将一个文件分散传递到多个缓冲区中，而write又可以将多个缓冲区中的数据集中写到一个文件中
分散读：
```
public class TestScatteringRead {
    public static void main(String[] args) {
        // 获取channel的两种方式 1. 通过输入输出流getChannel；2. 通过RandomAccessFile.getChannel()获取
        try (FileChannel channel = new RandomAccessFile("word1.txt", "r").getChannel()) {
            ByteBuffer allocate0 = ByteBuffer.allocate(3);
            ByteBuffer allocate1 = ByteBuffer.allocate(3);
            ByteBuffer allocate2 = ByteBuffer.allocate(5);

            // 将通道(channel)中的数据依次读入缓存区中
            channel.read(new ByteBuffer[]{allocate0, allocate1, allocate2});
            // 切换缓冲区读写模式
            allocate0.flip();
            allocate1.flip();
            allocate2.flip();

        } catch (IOException e) {
        }
    }
}
```
集中写：
```
public class TestGatheringWrite {
    public static void main(String[] args) {
        // 将指定字符串转换为bytebuffer
        ByteBuffer buffer1 = StandardCharsets.UTF_8.encode("hello");
        ByteBuffer buffer2 = StandardCharsets.UTF_8.encode("word");
        ByteBuffer buffer3 = StandardCharsets.UTF_8.encode("李雷");

        // 获取到文件对应的通道
        try (FileChannel channel = new RandomAccessFile("word2.txt", "rw").getChannel()) {
            // 将缓冲区中的数据写入通道中
            channel.write(new ByteBuffer[]{buffer1, buffer2, buffer3});
        } catch (IOException e) {
        }
    }
}
```
# 粘包与半包
粘包：
发送方在发送数据时，并不是一条一条地发送数据，而是将数据整合在一起，当数据达到一定的数量后再一起发送。这就会导致多条信息被放在一个缓冲区中被一起发送出去
半包：
接收方的缓冲区的大小是有限的，当接收方的缓冲区满了以后，就需要将信息截断，等缓冲区空了以后再继续放入数据。这就会发生一段完整的数据最后被截断的现象
解决办法：
通过get(index)方法遍历ByteBuffer，遇到分隔符时进行处理。
（1.记录该段数据长度，以便于申请对应大小的缓冲区
  2.将缓冲区的数据通过get()方法写入到target中）
调用compact方法切换模式，因为缓冲区中可能还有未读的数据
```
public class ByteBufferDemo {
    public static void main(String[] args) {
        ByteBuffer buffer = ByteBuffer.allocate(32);
        // 模拟粘包+半包
        buffer.put("Hello,world\nI'm Nyima\nHo".getBytes());
        // 调用split函数处理
        split(buffer);
        buffer.put("w are you?\n".getBytes());
        split(buffer);
    }

    private static void split(ByteBuffer buffer) {
        // 切换为读模式
        buffer.flip();
        for(int i = 0; i < buffer.limit(); i++) {

            // 遍历寻找分隔符
            // get(i)不会移动position
            if (buffer.get(i) == '\n') {
                // 缓冲区长度
                int length = i+1-buffer.position();
                ByteBuffer target = ByteBuffer.allocate(length);
                // 将前面的内容写入target缓冲区
                for(int j = 0; j < length; j++) {
                    // 将buffer中的数据写入target中
                    target.put(buffer.get());
                }
                // 打印查看结果
                ByteBufferUtil.debugAll(target);
            }
        }
        // 切换为写模式，但是缓冲区可能未读完，这里需要使用compact
        buffer.compact();
    }
}
```
# 文件编程
## FileChannel
FileChannel只能在阻塞模式下工作，所以无法搭配selector
FileChannel不能直接打开，只能通过getChannel方法获取
- 通过 FileInputStream 获取的 channel **只能读**
- 通过 FileOutputStream 获取的 channel **只能写**
- 通过 RandomAccessFile 是否能读写**根据构造 RandomAccessFile 时的读写模式决定**
### 读取read
通过 FileInputStream 获取channel，通过read方法将数据写入到ByteBuffer中
read方法的返回值表示读到了多少字节，若读到了文件末尾则返回-1，可根据返回值判断是否读取完毕
```
int readBytes = channel.read(buffer);
```
### 写入write
因为channel也是有大小的，所以 write 方法并不能保证一次将 buffer 中的内容全部写入 channel。必须**需要按照以下规则进行写入**
```
// 通过hasRemaining()方法查看缓冲区中是否还有数据未写入到通道中
while(buffer.hasRemaining()) {
	channel.write(buffer);
}
```
### 关闭close
通道需要close,调用流的close方法会间接调用channel的close
### 位置position
channel也有一个保存读取数据位置的属性
```
long pos=channel.position();
```
可以通过position（int pos）设置channel中position的值
```
long newPos=...;
channel.position(newPos);
```
设置当前位置时，如果设置为文件的末尾
- 这时读取会返回 -1
- 这时写入，会追加内容，但要注意如果 position 超过了文件末尾，再写入时在新内容和原末尾之间会有空洞（00）
### 获取文件大小size
### 强制写入
操作系统出于性能的考虑，会将数据缓存，不是立刻写入磁盘，而是等到缓存满了以后将所有数据一次性的写入磁盘。可以调用 **force(true)** 方法将文件内容和元数据（文件的权限等信息）立刻写入磁盘
## 两个Channel间传输数据
transferTo
使用transferTo方法可以快速、高效地将一个channel中的数据传输到另一个channel中，但**一次只能传输2G的内容**
当传输的文件**大于2G**时，需要使用以下方法进行多次传输
```
public class TestChannel {
    public static void main(String[] args){
        try (FileInputStream fis = new FileInputStream("stu.txt");
             FileOutputStream fos = new FileOutputStream("student.txt");
             FileChannel inputChannel = fis.getChannel();
             FileChannel outputChannel = fos.getChannel()) {
            long size = inputChannel.size();
            long capacity = inputChannel.size();
            // 分多次传输
            while (capacity > 0) {
                // transferTo返回值为传输了的字节数
                capacity -= inputChannel.transferTo(size-capacity, capacity, outputChannel);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
## Path与Paths
Path用来表示文件路径
Paths是工具类，用来获取Path实例
```
Path source = Paths.get("1.txt"); // 相对路径 不带盘符 使用 user.dir 环境变量来定位 1.txt

Path source = Paths.get("d:\\1.txt"); // 绝对路径 代表了  d:\1.txt 反斜杠需要转义

Path source = Paths.get("d:/1.txt"); // 绝对路径 同样代表了  d:\1.txt

Path projects = Paths.get("d:\\data", "projects"); // 代表了  d:\data\projects
```
## Files
### 查找
检查文件是否存在
```
Path path = Paths.get("helloword/data.txt");
System.out.println(Files.exists(path));
```
### 创建
创建一级目录
```
Path path = Paths.get("helloword/d1");
Files.createDirectory(path);
```
- 如果目录已存在，会抛异常 FileAlreadyExistsException
- 不能一次创建多级目录，否则会抛异常 NoSuchFileException
创建多级目录
```
Path path = Paths.get("helloword/d1/d2");
Files.createDirectories(path);
```
### 拷贝
```
Path source = Paths.get("helloword/data.txt");
Path target = Paths.get("helloword/target.txt");

Files.copy(source, target);
```
- 如果文件已存在，会抛异常 FileAlreadyExistsException
```
//如果希望用 source **覆盖**掉 target，需要用 StandardCopyOption 来控制
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
```
### 移动
```
Path source = Paths.get("helloword/data.txt");
Path target = Paths.get("helloword/data.txt");

Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);
```
### 删除
```
//删除文件，如果文件不存在，会抛异常 NoSuchFileException

Path target = Paths.get("helloword/target.txt");
Files.delete(target);

//删除目录，如果目录还有内容，会抛异常 DirectoryNotEmptyException

Path target = Paths.get("helloword/d1");
Files.delete(target);
```
### 遍历
可以**使用Files工具类中的walkFileTree(Path, FileVisitor)方法**，其中需要传入两个参数
其中Path是文件起始路径，FileVisitor是文件访问器接口，实现类SimpleFileVisitor可选重写四个方法
- preVisitDirectory：访问目录前的操作
- visitFile：访问文件的操作
- visitFileFailed：访问文件失败时的操作
- postVisitDirectory：访问目录后的操作
```
public class TestWalkFileTree {
    public static void main(String[] args) throws IOException {
        Path path = Paths.get("D:\\JDK 8");
        // 文件目录数目
        AtomicInteger dirCount = new AtomicInteger();
        // 文件数目
        AtomicInteger fileCount = new AtomicInteger();
        Files.walkFileTree(path, new SimpleFileVisitor<Path>(){
            @Override
            public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
                System.out.println("===>"+dir);
                // 增加文件目录数
                dirCount.incrementAndGet();
                return super.preVisitDirectory(dir, attrs);
            }

            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                System.out.println(file);
                // 增加文件数
                fileCount.incrementAndGet();
                return super.visitFile(file, attrs);
            }
        });
        // 打印数目
        System.out.println("文件目录数:"+dirCount.get());
        System.out.println("文件数:"+fileCount.get());
    }
}
```
# 网络编程
## 阻塞
1.阻塞模式下，相关方法都会导致线程暂停
- ServerSocketChannel.accept 会在没有连接建立时让线程暂停
- SocketChannel.read 会在通道中没有数据可读时让线程暂停
2.阻塞的表现其实就是线程暂停了，暂停期间不会占用 cpu，但线程相当于闲置，不能处理其他的任务
3.单线程下，阻塞方法之间相互影响，几乎不能正常工作，需要多线程支持
4.但多线程下，有新的问题，体现在以下方面
- 32 位 jvm 一个线程 320k，64 位 jvm 一个线程 1024k，如果连接数过多，必然导致 OOM，并且线程太多，反而会因为频繁上下文切换导致性能降低
- 可以采用线程池技术来减少线程数和线程上下文切换，但治标不治本，如果有很多连接建立，但长时间 inactive，会阻塞线程池中所有线程，因此不适合长连接，只适合短连接
## 非阻塞
- 可以通过ServerSocketChannel的configureBlocking(false)方法将获得连接设置为非阻塞的。此时若没有连接，accept会返回null
- 可以通过SocketChannel的configureBlocking(false)方法将从通道中读取数据设置为非阻塞的。若此时通道中没有数据可读，read会返回-1
```
public class Server {
 public static void main(String[] args) {
        // 创建缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(16);
        // 获得服务器通道
        try (ServerSocketChannel serverSocketChannel = ServerSocketChannel.open()) {
            // 为服务器通道绑定端口
            serverSocketChannel.bind(new InetSocketAddress(8089));
            // 将服务端连接通道设置为非阻塞模式，此种状模式，server.accept()在没有客户端连接请求建立时，返回值是null此时若没有连接，accept会返回null
            serverSocketChannel.configureBlocking(false);
            // 用户存放连接的集合
            List<SocketChannel> channelList = new ArrayList<>();
            // 循环接收连接
            while (true) {
                // configureBlocking = false （非阻塞模式）下，执行到此代码，如没有客户端连接请求建立，返回值为null
                SocketChannel socketChannel = serverSocketChannel.accept();
                // 通道不为空时才将连接放入到集合中
                if (null != socketChannel) {
                    System.out.println("client connecting...");
                    // 客户端socket通道，设置为非阻塞模式，则使用channel.read()是非阻塞，不会阻塞线程的执行
                    socketChannel.configureBlocking(false);
                    channelList.add(socketChannel);
                }
                for (SocketChannel channel : channelList) {
                    // 处理通道中的数据
                    // 在通道channel的configureBlocking = false （非阻塞模式）下，若通道中没有数据，则返回值是0，不会阻塞线程的执行
                    int read = channel.read(buffer);
                    if (read > 0) {
                        buffer.flip();
                        ByteBufferUtil.debugRead(buffer);
                        buffer.clear();
                        System.out.println("after reading");
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
这样写存在一个问题，因为设置为了非阻塞，会一直执行while(true)中的代码，CPU一直处于忙碌状态，会使得性能变低，所以实际情况中不使用这种方法处理请求
## Selector
多路复用
单线程可以配合 Selector 完成对多个 Channel 可读写事件的监控，这称之为多路复用

1.多路复用仅针对网络 IO，普通文件 IO 无法利用多路复用
2.如果不用 Selector 的非阻塞模式，线程大部分时间都在做无用功，而 Selector 能够保证
- 有可连接事件时才去连接
- 有可读事件才去读取
- 有可写事件才去写入
3.限于网络传输能力，Channel 未必时时可写，一旦 Channel 可写，会触发 Selector 的可写事件
### 使用代码示例
```
public class SelectorServer {
    public static void main(String[] args) {
        // 创建缓冲区
        ByteBuffer allocate = ByteBuffer.allocate(32);
        // 开启服务器通信通道
        try (ServerSocketChannel serverSocketChannel = ServerSocketChannel.open()) {
            // 为服务器绑定端口
            serverSocketChannel.bind(new InetSocketAddress(8089));
            // 设置服务器通道为非阻塞性形式
            serverSocketChannel.configureBlocking(false);
            // 获取selector,实现多路复用，注意fileChannel没有非阻塞形式，所以不能配合selector使用，进行非阻塞多路复用
            Selector selector = Selector.open();
            // 将服务器通信通道注册到selector中，并设置关注(感兴趣的)的事件
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
            // 循环处理通道事件
            while (true) {
                System.out.println("selector before ready,thread is block");
                // 获取到选择器selector中的事件，若没有事件，则线程会处于阻塞状态；反之不会阻塞。此类现象避免了CPU空转
                // 返回值为已经就绪的事件数
                int readySelect = selector.select();
                System.out.println("selector ready counts : " + readySelect);
                // 获取选择器所有事件
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                // 获取事件的迭代器
                Iterator<SelectionKey> iterator = selectionKeys.iterator();
                while (iterator.hasNext()) {
                    // 获取当前事件的key
                    SelectionKey selectionKey = iterator.next();
                    // 判断事件的类型，不同的事件做出不同的处理
                    if (selectionKey.isAcceptable()) {
                        // 服务端连接事件
                        ServerSocketChannel channel = (ServerSocketChannel) selectionKey.channel();
                        System.out.println("before accepting...");
                        // 获取链接通道
                        SocketChannel socketChannel = channel.accept();
                        System.out.println("after accepting..." + socketChannel);
                        // 需要将当前的选择器事件key进行移除，否则下次处理事件会出现 nullPointException
                        iterator.remove();
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
### 代码解析
1.获得选择器Selector
```
Selector selector = Selector.open();
```
2.将通道设置为非阻塞模式，并注册到选择器中，并设置感兴趣的事件
- channel 必须工作在非阻塞模式
- FileChannel 没有非阻塞模式，因此不能配合 selector 一起使用
```
// 通道必须设置为非阻塞模式
server.configureBlocking(false);
// 将通道注册到选择器中，并设置感兴趣的事件
server.register(selector, SelectionKey.OP_ACCEPT);
```
其中，可绑定的事件类型有：
- connect - 客户端连接成功时触发
- accept - 服务器端成功接受连接时触发
- read - 数据可读入时触发，有因为接收能力弱，数据暂不能读入的情况
- write - 数据可写出时触发，有因为发送能力弱，数据暂不能写出的情况
3.通过Selector监听事件，并获得就绪的通道个数，若没有通道就绪，线程会被阻塞
- 阻塞直到绑定事件发生
```
int count = selector.select();
```
- 阻塞直到绑定事件发生，或是超时（时间单位为 ms）
```
int count = selector.select(long timeout);
```
- 不会阻塞，也就是不管有没有事件，立刻返回，自己根据返回值检查是否有事件
```
int count = selector.selectNow();
```
4.使用迭代器获取就绪事件并得到对应的通道，然后进行可以根据事件类型（调用key的is... 方法）进行判断，对不同类型的事件进行不同的处理
事件发生后，要么处理，要么取消（cancel），不能什么都不做，否则下次该事件仍会触发，这是因为 nio 底层使用的是水平触发
```
// 获取所有事件
Set<SelectionKey> selectionKeys = selector.selectedKeys();
                
// 使用迭代器遍历事件
Iterator<SelectionKey> iterator = selectionKeys.iterator();

while (iterator.hasNext()) {
	SelectionKey key = iterator.next();
                    
	// 判断key的类型，此处为Accept类型
	if(key.isAcceptable()) {
        // 获得key对应的channel
        ServerSocketChannel channel = (ServerSocketChannel) key.channel();

        // 获取连接并处理，而且是必须处理，否则需要取消
        SocketChannel socketChannel = channel.accept();

        // 处理完毕后移除
        iterator.remove();
	}
}
```
#### read事件
- 在Accept事件中，若有客户端与服务器端建立了连接，**需要将其对应的SocketChannel设置为非阻塞，并注册到选择其中**
- 添加Read事件，触发后进行读取操作
#### 删除事件
当处理完一个事件后，一定要调用迭代器的remove方法移除对应事件，否则会出现错误
当调用了 server.register(selector, SelectionKey.OP_ACCEPT)后，Selector中维护了一个集合，**用于存放SelectionKey以及其对应的通道**，当**选择器中的通道对应的事件发生后**，selecionKey会被放到另一个集合中，但是**selecionKey不会自动移除**，所以需要我们在处理完一个事件后，通过迭代器手动移除其中的selecionKey。否则会导致已被处理过的事件再次被处理，就会引发错误
#### write事件
服务器通过Buffer向通道中写入数据时，可能因为通道容量小于Buffer中的数据大小，导致无法一次性将Buffer中的数据全部写入到Channel中，这时便需要分多次写入
1.执行一次写操作，向将buffer中的内容写入到SocketChannel中，然后判断Buffer中是否还有数据
2.若Buffer中还有数据，则需要将SockerChannel注册到Seletor中，并关注写事件，同时将未写完的Buffer作为附件一起放入到SelectionKey中
```
 int write = socket.write(buffer);
// 通道中可能无法放入缓冲区中的所有数据
if (buffer.hasRemaining()) {
    // 注册到Selector中，关注可写事件，并将buffer添加到key的附件中
    socket.configureBlocking(false);
    socket.register(selector, SelectionKey.OP_WRITE, buffer);
}
```
3.添加写事件的相关操作`key.isWritable()`，对Buffer再次进行写操作,每次写后需要判断Buffer中是否还有数据（是否写完）。若写完，需要移除SelecionKey中的Buffer附件，避免其占用过多内存，同时还需移除对写事件的关注
```
SocketChannel socket = (SocketChannel) key.channel();
// 获得buffer
ByteBuffer buffer = (ByteBuffer) key.attachment();
// 执行写操作
int write = socket.write(buffer);
System.out.println(write);
// 如果已经完成了写操作，需要移除key中的附件，同时不再对写事件感兴趣
if (!buffer.hasRemaining()) {
    key.attach(null);
    key.interestOps(0);
}
```
### 断开处理
当客户端与服务器之间的连接断开时，会给服务器端发送一个读事件，对异常断开和正常断开需要加以不同的方式进行处理
#### 正常断开
正常断开时，服务器端的channel.read(buffer)方法的返回值为-1，所以当结束到返回值为-1时，需要调用key的cancel方法取消此事件，并在取消后移除该事件
```
int read = channel.read(buffer);
// 断开连接时，客户端会向服务器发送一个写事件，此时read的返回值为-1
if(read == -1) {
    // 取消该事件的处理
	key.cancel();
    channel.close();
} else {
    ...
}
// 取消或者处理，都需要移除key
iterator.remove();
```
#### 异常断开
异常断开时，会抛出IOException异常，在try-catch的catch块中捕获异常并调用key的cancel方法即可
### 消息边界的处理
若不处理消息边界，可能出现乱码
或文本大于缓冲区大小以及半包和粘包
解决思路大致有三种：
1.固定消息长度，数据包大小一样，服务器按预定长度读取，当发送的数据较少时，需要将数据进行填充，直到长度与消息规定长度一致。缺点是浪费带宽
2.按分隔符拆分，缺点是效率低，需要一个一个字符地去匹配分隔符
3.TLV 格式，即 Type 类型、Length 长度、Value 数据
（也就是在消息开头用一些空间存放后面数据的长度），如HTTP请求头中的Content-Type与Content-Length。类型和长度已知的情况下，就可以方便获取消息大小，分配合适的 buffer，缺点是 buffer 需要提前分配，如果内容过大，则影响 server 吞吐量
### 附件与扩容
Channel的register方法还有第三个参数：附件，可以向其中放入一个Object类型的对象，该对象会与登记的Channel以及其对应的SelectionKey绑定，可以从SelectionKey获取到对应通道的附件
```
public final SelectionKey register(Selector sel, int ops, Object att)
```
可通过SelectionKey的attachment()方法获得附件
```
ByteBuffer buffer = (ByteBuffer) key.attachment();
```
我们需要在Accept事件发生后，将通道注册到Selector中时，对每个通道添加一个ByteBuffer附件，让每个通道发生读事件时都使用自己的通道，避免与其他通道发生冲突而导致问题
```
// 设置为非阻塞模式，同时将连接的通道也注册到选择其中，同时设置附件
socketChannel.configureBlocking(false);
ByteBuffer buffer = ByteBuffer.allocate(16);
// 添加通道对应的Buffer附件
socketChannel.register(selector, SelectionKey.OP_READ, buffer);
```
当Channel中的数据大于缓冲区时，需要对缓冲区进行扩容操作。此代码中的扩容的判定方法：Channel调用compact方法后position与limit相等，说明缓冲区中的数据并未被读取（容量太小），此时创建新的缓冲区，其大小扩大为两倍。同时还要将旧缓冲区中的数据拷贝到新的缓冲区中，同时调用SelectionKey的attach方法将新的缓冲区作为新的附件放入SelectionKey中
```
// 如果缓冲区太小，就进行扩容
if (buffer.position() == buffer.limit()) {
    ByteBuffer newBuffer = ByteBuffer.allocate(buffer.capacity()*2);
    // 将旧buffer中的内容放入新的buffer中
    ewBuffer.put(buffer);
    // 将新buffer作为附件放到key中
    key.attach(newBuffer);
}
```
### ByteBuffer的大小分配
- 每个 channel 都需要记录可能被切分的消息，因为 ByteBuffer 不能被多个 channel 共同使用，因此需要为每个 channel 维护一个独立的 ByteBuffer
- ByteBuffer 不能太大，比如一个 ByteBuffer 1Mb 的话，要支持百万连接就要 1Tb 内存，因此需要设计大小可变的 ByteBuffer
分配思路可以参考：
- 一种思路是首先分配一个较小的 buffer，例如 4k，如果发现数据不够，再分配 8k 的 buffer，将 4k buffer 内容拷贝至 8k buffer，优点是消息连续容易处理，缺点是数据拷贝耗费性能
- 另一种思路是用多个数组组成 buffer，一个数组不够，把多出来的内容写入新的数组，与前面的区别是消息存储不连续解析复杂，优点是避免了拷贝引起的性能损耗
### 多线程优化
充分利用多核CPU，分两组选择器
单线程配一个选择器（Boss），专门处理 accept 事件
创建 cpu 核心数的线程（Worker），每个线程配一个选择器，轮流处理 read 事件
实现思路:
1.创建一个负责处理Accept事件的Boss线程，与多个负责处理Read事件的Worker线程
2.Boss线程执行的操作
接受并处理Accepet事件，当Accept事件发生后，调用Worker的register(SocketChannel socket)方法，让Worker去处理Read事件，其中需要根据标识robin去判断将任务分配给哪个Worker
```
// 创建固定数量的Worker
Worker[] workers = new Worker[4];
// 用于负载均衡的原子整数
AtomicInteger robin = new AtomicInteger(0);
// 负载均衡，轮询分配Worker
workers[robin.getAndIncrement()% workers.length].register(socket);Copy
```
register(SocketChannel socket)方法会通过同步队列完成Boss线程与Worker线程之间的通信，让SocketChannel的注册任务被Worker线程执行。添加任务后需要调用selector.wakeup()来唤醒被阻塞的Selector
```
public void register(final SocketChannel socket) throws IOException {
    // 只启动一次
    if (!started) {
       // 初始化操作
         // 将当前的工作者(处理客户端链接通道的)和线程关联连起来，当前工作者需要实现runnable接口
                thread = new Thread(this);
                thread.setName(name);
                thread.start();
                // 一定要对worker进行开启操作，防止后续操作出现空指针
                worker = Selector.open();
                start = !start;
    }
    // 向同步队列中添加SocketChannel的注册事件
    // 在Worker线程中执行注册事件
    queue.add(new Runnable() {
        @Override
        public void run() {
            try {
                socket.register(selector, SelectionKey.OP_READ);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    });
    // 唤醒被阻塞的Selector
    // select类似LockSupport中的park，wakeup的原理类似LockSupport中的unpark
    selector.wakeup();
```
3.Worker线程执行的操作
从同步队列中获取注册任务，并处理Read事件
# NIO与BIO
## stream与channel
1.stream 不会自动缓冲数据，channel 会利用系统提供的发送缓冲、接收缓冲区（更为底层）
2.stream 仅支持阻塞 API，channel 同时支持阻塞、非阻塞 API，网络 channel 可配合 selector 实现多路复用
3.二者均为全双工，即读写可以同时进行
虽然Stream是单向流动的，但是它也是全双工的
## IO模型
同步：线程自己去获取结果（一个线程）
例如：线程调用一个方法后，需要等待方法返回结果
异步：线程自己不去获取结果，而是由其它线程返回结果（至少两个线程）
例如：线程A调用一个方法后，继续向下运行，运行结果由线程B返回
当调用一次 channel.read或 stream.read后，会由用户态切换至操作系统内核态来完成真正数据读取，而读取又分为两个阶段，分别为：
等待数据阶段
复制数据阶段
### 阻塞IO
用户线程进行read操作时，需要等待操作系统执行实际的read操作
![[Pasted image 20230925175544.png]]
### 非阻塞IO
- 用户线程在一个循环中一直调用read方法，若内核空间中还没有数据可读，立即返回  
- **只是在等待阶段非阻塞**
- 用户线程发现内核空间中有数据后，等待内核空间执行复制数据，待复制结束后返回结果
![[Pasted image 20230925175656.png]]
### 多路复用
java中通过selector实现多路复用
当没有事件时，调用select方法会被阻塞住
一旦有一个或多个事件发生后，就会处理对应的事件，从而实现多路复用
![[Pasted image 20230925175719.png]]
### 多路复用与阻塞IO的区别
- 阻塞IO模式下，**若线程因accept事件被阻塞，发生read事件后，仍需等待accept事件执行完成后**，才能去处理read事件
- 多路复用模式下，一个事件发生后，若另一个事件处于阻塞状态，不会影响该事件的执行
### 异步IO
- 线程1调用方法后立即返回，**不会被阻塞也不需要立即获取结果**
- 当方法的运行结果出来以后，由线程2将结果返回给线程1
![[Pasted image 20230925175932.png]]
## 零拷贝
零拷贝指的时数据无需拷贝到JVM内存中，有以下三个优点
更少的用户态与内核态的切换
不利用cpu计算，减少cpu缓存伪共享
零拷贝适合小文件传输
#### 传统io工作流程
![[Pasted image 20230925180234.png]]
1.Java 本身并不具备 IO 读写能力，因此 read 方法调用后，要从 Java 程序的用户态切换至内核态，去调用操作系统（Kernel）的读能力，将数据读入内核缓冲区。这期间用户线程阻塞，操作系统使用 DMA（Direct Memory Access）来实现文件读，期间也不会使用 CPU（DMA 也可以理解为硬件单元，用来解放 cpu 完成文件 IO）
2.从内核态切换回用户态，将数据从内核缓冲区读入用户缓冲区（即 byte[ ] buf），这期间 CPU 会参与拷贝，无法利用 DMA
3.调用 write 方法，这时将数据从用户缓冲区（byte[] buf）写入 socket 缓冲区，CPU 会参与拷贝
4.接下来要向网卡写数据，这项能力 Java 又不具备，因此又得从用户态切换至内核态，调用操作系统的写能力，使用 DMA 将 socket 缓冲区的数据写入网卡，不会使用 CPU
可以看到中间环节较多，java 的 IO 实际不是物理设备级别的读写，而是缓存的复制，底层的真正读写是操作系统来完成的
- 用户态与内核态的切换发生了 3 次，这个操作比较重量级
- 数据拷贝了共 4 次
#### nio优化
ByteBuffer.allocate(10) 底层对应 HeapByteBuffer，使用的还是 Java 内存
ByteBuffer.allocateDirect(10) 底层对应DirectByteBuffer，使用的是操作系统内存

大部分步骤与优化前相同，唯有一点：Java 可以使用 DirectByteBuffer 将堆外内存映射到 JVM 内存中来直接访问使用，这块内存不受 JVM 垃圾回收的影响，因此内存地址固定，有助于 IO 读写

Java 中的 DirectByteBuf 对象仅维护了此内存的虚引用，内存回收分成两步：
1.DirectByteBuffer 对象被垃圾回收，将虚引用加入引用队列
当引用的对象ByteBuffer被垃圾回收以后，虚引用对象Cleaner就会被放入引用队列中，然后调用Cleaner的clean方法来释放直接内存；DirectByteBuffer 的释放底层调用的是 Unsafe 的 freeMemory 方法
2.通过专门线程访问引用队列，根据虚引用释放堆外内存

这样做**减少了一次数据拷贝，用户态与内核态的切换次数没有减少**
##### 优化1
底层采用了 **linux 2.1** 后提供的 **sendFile** 方法，Java 中对应着两个 channel 调用 **transferTo/transferFrom**方法拷贝数据

- Java 调用 transferTo 方法后，要从 Java 程序的**用户态**切换至**内核态**，使用 DMA将数据读入**内核缓冲区**，不会使用 CPU
- 数据从**内核缓冲区**传输到 **socket 缓冲区**，CPU 会参与拷贝
- 最后使用 DMA 将 **socket 缓冲区**的数据写入网卡，不会使用 CPU

这种方法下只发生了1次用户态与内核态的切换，数据拷贝了 3 次
##### 优化2
linux2.4继续优化

- Java 调用 transferTo 方法后，要从 Java 程序的用户态切换至内核态，使用 DMA将数据读入内核缓冲区，不会使用 CPU
- 只会将一些 offset 和 length 信息拷入 socket 缓冲区，几乎无消耗
- 使用 DMA 将 内核缓冲区的数据写入网卡，不会使用 CPU
- 
**整个过程仅只发生了1次用户态与内核态的切换，数据拷贝了 2 次**



