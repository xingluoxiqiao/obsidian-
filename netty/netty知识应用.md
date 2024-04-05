# 粘包与半包
## 简述
  
粘包（Packet Concatenation）和半包（Partial Packet）是在网络通信中常见的问题，特别是在基于流的传输协议（如TCP）中。它们可能导致接收端无法正确解析和处理发送端发送的数据。
发生粘包与半包现象的本质是**因为 TCP 是流式协议，消息无边界**
1. **粘包（Packet Concatenation）**：
    - 粘包指的是多个数据包被发送端连续发送并在接收端合并成一个或多个较大的数据包。这导致接收端无法分辨每个原始数据包的边界。
    - 原因：TCP是一个流协议，它不保证数据包的边界与发送端的`write`操作一一对应。
    - 示例：发送端依次发送"A"、"B"、"C"，接收端可能一次性接收到"ABC"，无法分辨每个数据包。
2. **半包（Partial Packet）**：
    - 半包指的是接收端只接收到部分数据包，没有接收到完整的数据包。这可能是因为数据包过大而被拆分成多个TCP段，也可能是因为接收端读取速度较慢。
    - 原因：TCP将数据划分为小的数据段（Segment），接收端可能需要多次读取才能完整接收一个数据包。
    - 示例：发送端发送一个大的数据包，接收端可能先接收到部分数据，后续再接收到剩余部分。
## 解决方案
### 短链接
**客户端每次向服务器发送数据以后，就与服务器断开连接，此时的消息边界为连接建立到连接断开**。这时便无需使用滑动窗口等技术来缓冲数据，则不会发生粘包现象。但如果一次性数据发送过多，接收方无法一次性容纳所有数据，还是会发生半包现象，所以**短链接无法解决半包现象**
代码改进
1.writeandflush后马上close
2.将发送步骤整体封装成send方法
### 定长解码器
客户端与服务器约定一个最大长度，保证客户端每次发送的数据长度都不会大于该长度。若发送数据长度不足则需要补齐至该长度

服务器接收数据时，将接收到的数据按照约定的最大长度进行拆分，即使发送过程中产生了粘包，也可以通过定长解码器将数据正确地进行拆分。服务端需要用到FixedLengthFrameDecoder对数据进行定长解码，具体使用方法如下
```
ch.pipeline().addLast(new FixedLengthFrameDecoder(16));
```
### 行解码器
行解码器的是通过分隔符对数据进行拆分来解决粘包半包问题的

可以通过LineBasedFrameDecoder(int maxLength)来拆分以换行符(\n)为分隔符的数据，也可以通过DelimiterBasedFrameDecoder(int maxFrameLength, ByteBuf... delimiters)来指定通过什么分隔符来拆分数据（可以传入多个分隔符）,一般以换行符 \\n 为分隔符

两种解码器都需要传入数据的最大长度，若超出最大长度，会抛出TooLongFrameException异常
### 长度字段解码器
在传送数据时可以在数据中**添加一个用于表示有用数据长度的字段**，在解码时读取出这个用于表明长度的字段，同时读取其他相关参数，即可知道最终需要的数据是什么样子的

`LengthFieldBasedFrameDecoder`解码器可以提供更为丰富的拆分方法，其构造方法有五个参数
- maxFrameLength 数据最大长度
表示数据的最大长度（包括附加信息、长度标识等内容）
- lengthFieldOffset 数据长度标识的起始偏移量
用于指明数据第几个字节开始是用于标识有用字节长度的，因为前面可能还有其他附加信息
- lengthFieldLength 数据长度标识所占字节数（用于指明有用数据的长度）
数据中用于表示有用数据长度的标识所占的字节数
- lengthAdjustment 长度表示与有用数据的偏移量
用于指明数据长度标识和有用数据之间的距离，因为两者之间还可能有附加信息
- initialBytesToStrip 数据读取起点
读取起点，不读取 0 ~ initialBytesToStrip 之间的数据
```
public LengthFieldBasedFrameDecoder(
    int maxFrameLength, 		// 解析数据的最大长度
    int lengthFieldOffset, 		// 数据长度标识的起始偏移量
    int lengthFieldLength,		// 数据长度标识所占的字节数
    int lengthAdjustment,		// 有效数据与数据长度标识结束位置之间的偏移量
    int initialBytesToStrip     // 截取的报文数据起始偏移量，从头开始剥离几个字节
)	
```
![[Pasted image 20230930143604.png]]
# 协议设计及解析
#### 协议的作用
TCP/IP 中消息传输基于流的方式，没有边界
协议的目的就是划定消息的边界，制定通信双方要共同遵守的通信规则
## redis协议
```
// 该指令一共有3部分，每条指令之后都要添加回车与换行符
*3\r\n
// 第一个指令的长度是3
$3\r\n
// 第一个指令是set指令
set\r\n
// 下面的指令以此类推
$4\r\n
name\r\n
$4\r\n
test\r\n
```
netty通过遵守上述协议可以与向redis发送指令
构建指令并发送的代码如下
```
public class RedisClient {
    static final Logger log = LoggerFactory.getLogger(StudyServer.class);
    public static void main(String[] args) {
                NioEventLoopGroup eventExecutors = new NioEventLoopGroup();
        ChannelFuture channelFuture = new Bootstrap()
                .group(eventExecutors)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                        ch.pipeline().addLast(new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelActive(ChannelHandlerContext ctx) throws Exception {
                                // 定义换行符
                                final byte[] newLine = {'\r', '\n'};
                                // 获得ByteBuf
                                //创建了一个ByteBuf对象，用于构建Redis命令。这里使用了Redis的文本协议，按照协议规范构造了一条`SET`命令，将一个键值对存储到Redis中。然后使用`ctx.writeAndFlush(buffer)`将命令发送给Redis服务器
                                ByteBuf buffer = ctx.alloc().buffer();
                                buffer.writeBytes("*3".getBytes(StandardCharsets.UTF_8));
                                buffer.writeBytes(newLine);
                                buffer.writeBytes("$3".getBytes(StandardCharsets.UTF_8));
                                buffer.writeBytes(newLine);
                                buffer.writeBytes("set".getBytes(StandardCharsets.UTF_8));
                                buffer.writeBytes(newLine);
                                buffer.writeBytes("$19".getBytes(StandardCharsets.UTF_8));
                                buffer.writeBytes(newLine);
							    buffer.writeBytes("redis:protocol:name"
								    .getBytes(StandardCharsets.UTF_8));
                                buffer.writeBytes(newLine);
                                buffer.writeBytes("$4".getBytes(StandardCharsets.UTF_8));
                                buffer.writeBytes(newLine);
                                buffer.writeBytes("test".getBytes(StandardCharsets.UTF_8));
                                buffer.writeBytes(newLine);
                                ctx.writeAndFlush(buffer);
                            }
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                super.channelRead(ctx, msg);
                            }
                        });
                    }
                })
                .connect(new InetSocketAddress("localhost", 6379));
        try {
            ChannelFuture future = channelFuture.sync();
            ChannelFuture closeFuture = future.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            log.error("client error:", e);
        } finally {
            // 优雅的关闭事件组
            eventExecutors.shutdownGracefully();
        }
    }
}
```
## http协议
HTTP协议在请求行请求头中都有很多的内容，自己实现较为困难，可以使用`HttpServerCodec`作为**服务器端的解码器与编码器，来处理HTTP请求**
```
// HttpServerCodec 中既有请求的解码器 HttpRequestDecoder 又有响应的编码器 HttpResponseEncoder
// Codec(CodeCombine) 一般代表该类既作为 编码器 又作为 解码器

public final class HttpServerCodec extends CombinedChannelDuplexHandler<HttpRequestDecoder, HttpResponseEncoder>
        implements HttpServerUpgradeHandler.SourceCodec {...}
```
服务器代码示例：
```
public class HttpServer {
    static final Logger log = LoggerFactory.getLogger(StudyServer.class);

    public static void main(String[] args) {
        NioEventLoopGroup eventLoopGroup = new NioEventLoopGroup(2);
        new ServerBootstrap()
                .group(eventLoopGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                        ch.pipeline().addLast(new HttpServerCodec());
                        // 作为服务端，处理客户端发起的请求(只处理HttpRequest)
                        ch.pipeline().addLast(new SimpleChannelInboundHandler<HttpRequest>() {
                            @Override
                            protected void channelRead0(ChannelHandlerContext ctx, HttpRequest msg) throws Exception {
                                // 获取请求相关信息
                                log.info("request uri={}", msg.uri());
                                ..获得请求后，需要返回响应给浏览器。需要创建响应对象`DefaultFullHttpResponse`，设置HTTP版本号及状态码，为避免浏览器获得响应后，因为获得`CONTENT_LENGTH`而一直空转，需要添加`CONTENT_LENGTH`字段，表明响应体中数据的具体长度
                                // 创建相应数据对象
                                DefaultFullHttpResponse httpResponse = new
                                        DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK);
                                // 添加响应数据
                                byte[] responseByte = "<h1>hello, http protocol</h1>".getBytes(StandardCharsets.UTF_8);
                                // 写入响应数据
                                httpResponse.content().writeBytes(responseByte);
                                // 写入数据的响应长度
                                httpResponse.headers().setInt(HttpHeaderNames.CONTENT_LENGTH, responseByte.length);
                                // 将数据写出进行响应
                                ctx.writeAndFlush(httpResponse);
                            }
                        });
                    }
                })
                .bind(8089);
    }
}
```
## 自定义协议
### 组成要素
- 魔数：用来在第一时间判定接收的数据是否为无效数据包
- 版本号：可以支持协议的升级
- 序列化算法：消息正文到底采用哪种序列化反序列化方式
          如：json、protobuf、hessian、jdk
- 指令类型：是登录、注册、单聊、群聊… 跟业务相关
- 请求序号：为了双工通信，提供异步能力
- 正文长度
- 消息正文
### 编码器与解码器
编码器与解码器方法源于**父类ByteToMessageCodec**，通过该类可以自定义编码器与解码器，**泛型类型为被编码与被解码的类**。此处使用了自定义类Message，代表消息
- 编码器**负责将附加信息与正文信息写入到ByteBuf中**，其中附加信息**总字节数最好为2的n次方，不足需要补齐**。正文内容如果为对象，需要通过**序列化**将其放入到ByteBuf中
- 解码器**负责将ByteBuf中的信息取出，并放入List中**，该List用于将信息传递给下一个handler
```
public class MessageCodec extends ByteToMessageCodec<Message> {

    @Override
    public void encode(ChannelHandlerContext ctx, Message msg, ByteBuf out) throws Exception {
        // 设置四字节 魔数
        out.writeBytes(new byte[]{'A', 'P', 'A', 'N'});
        // 设置一字节 版本号
        out.writeByte(1);
        // 设置一字节 序列化算法，此处使用jdk的序列化算法 jdk 0 , json 1
        out.writeByte(0);
        // 设置一字节 指令类型
        out.writeByte(msg.getMessageType());
        // 设置四字节 请求序号,目的提供双工通信，提供异步能力
        out.writeInt(msg.getSequenceId());
        // 附加信息最好是2的n次方位，4+1+1+1+4+4(内容长度字段)=15,最近的是16，因此添加一字节，补齐16
        out.writeByte(0x13);
        // 获取内容的字节数组
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
        objectOutputStream.writeObject(msg);
        byte[] bytes = outputStream.toByteArray();
        // 设置四字节 内容长度
        out.writeInt(bytes.length);
        // 写入内容
        out.writeBytes(bytes);
    }

    @Override
    public void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        // 根据加密进行获取相关的值
        // 获取魔数
        int magicNum = in.readInt();
        // 获取版本号
        byte version = in.readByte();
        // 获取序列化类型
        byte serializerType = in.readByte();
        // 获取指令类型
        byte messageType = in.readByte();
        // 获取请求序号
        int sequenceId = in.readInt();
        // 获取填充位
        byte fill = in.readByte();
        // 获取字段内容长度
        int length = in.readInt();
        // 获取传输的内容
        byte[] bytes = new byte[length];
        in.readBytes(bytes, 0, length);
        ObjectInputStream objectInputStream = new ObjectInputStream(new ByteArrayInputStream(bytes));
        Message message = (Message) objectInputStream.readObject();
        log.info("{},{},{},{},{},{},{}", magicNum, version, serializerType, messageType, sequenceId, fill, length);
        log.info("request message={}", message);
        // 将信息放入list，传递给下一个handler
        out.add(message);
    }
}
```
### @Sharable注解
为了提高handler的复用率，可以将handler创建为handler对象，然后在不同的channel中使用该handler对象进行处理操作
```
LoggingHandler loggingHandler = new LoggingHandler(LogLevel.DEBUG);
// 不同的channel中使用同一个handler对象，提高复用率
channel1.pipeline().addLast(loggingHandler);
channel2.pipeline().addLast(loggingHandler);
```
但是并不是所有的handler都能通过这种方法来提高复用率的，例如LengthFieldBasedFrameDecoder。
如果多个channel中使用同一个LengthFieldBasedFrameDecoder对象，则可能发生如下问题:
channel1中收到了一个半包，LengthFieldBasedFrameDecoder发现不是一条完整的数据，则没有继续向下传播;    此时channel2中也收到了一个半包，因为两个channel使用了同一个LengthFieldBasedFrameDecoder，存入其中的数据刚好拼凑成了一个完整的数据包。LengthFieldBasedFrameDecoder让该数据包继续向下传播，最终引发数据错误

为了提高handler的复用率，同时又避免出现一些并发问题，Netty中原生的handler中用@Sharable注解来标明，该handler能否在多个channel中共享。只有带有该注解，才能通过对象的方式被共享，否则无法被共享

自定义编解码器能否使用@Sharable注解
这需要根据自定义的handler的处理逻辑进行分析

上述MessageCodec本身接收的是LengthFieldBasedFrameDecoder处理之后的数据，那么数据肯定是完整的，按分析来说是可以添加@Sharable注解的，但是实际情况我们并**不能**添加该注解，会抛出异常信息`ChannelHandler cn.XXX.MessageCodec is not allowed to be shared`
这是因为因为MessageCodec**继承自ByteToMessageCodec**，它是**不能被多个channel所共享的**，该类的目标是将ByteBuf转化为Message，意味着传进该handler的数据还未被处理过。所以传过来的ByteBuf可能并不是完整的数据，如果共享则会出现问题
ByteToMessageCodec类的源码首先判断了其子类是否添加了@Sharable注解，添加了则会报错

如果想要共享的话，继承MessageToMessageDecoder即可。该类的目标是：将已经被处理的完整数据再次被处理。传过来的Message如果是被处理过的完整数据，那么被共享也就不会出现问题了，也就可以使用@Sharable注解了。实现方式与实现ByteToMessageCodec类似
