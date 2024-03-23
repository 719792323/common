# 序列化实验

* 数据类型

  根据文档中的数据描述，构建如下JavgBean结构

  ```java
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class NtripData {
      private Integer messageType;
      private Integer satelliteId;
      private Double orbitCorrection;
      private Double clockCorrection;
      private Double phaseCorrection;
      private Double atmosphericCorrection;
  
  }
  ```

* proto文件

  对如下文件使用protoc进行编译生成对应java文件

  ```java
  syntax = "proto3";
  
  option java_package = "open.demo.common.pojo";
  option java_outer_classname = "NtripDataProto";
  
  
  message NtripData{
    int32 messsageType = 1;
    int32 satelliteId = 2;
    double orbitCorrection = 3;
    double clockCorrection = 4;
    double phaseCorrection = 5;
    double atmosphericCorrection = 6;
  }
  ```

* protobuffer与Json序列化性能比较

  测试代码：

  ```java
  public class ProtoMain {
      public static void main(String[] args) throws Exception {
          NtripData ntripData = new NtripData();
          ntripData.setMessageType(1301);
          ntripData.setSatelliteId(10);
          ntripData.setPhaseCorrection(0.5);
          //json测试
          ObjectMapper mapper = new ObjectMapper();
          long t1 = System.currentTimeMillis();
          String json = mapper.writeValueAsString(ntripData);
          double len1 = json.getBytes().length;
          mapper.readValue(json, NtripData.class);
          long t2 = System.currentTimeMillis();
          NtripDataProto.NtripData ntripDataProto = NtripDataProto.NtripData.newBuilder()
                  .setMesssageType(ntripData.getMessageType())
                  .setSatelliteId(ntripData.getSatelliteId())
                  .setPhaseCorrection(ntripData.getPhaseCorrection()).build();
          double len2 = ntripDataProto.toByteArray().length;
          NtripDataProto.NtripData.parseFrom(ntripDataProto.toByteArray());
          long t3 = System.currentTimeMillis();
          System.out.println(String.format("json长度:%s,proto长度:%s,压缩比:%s", len1, len2, len1 / len2));
          System.out.println(String.format("json耗时:%s,proto耗时:%s,耗时比:%s", t2 - t1, t3 - t2, (double) (t2 - t1) / (double) (t3 - t2)));
      }
  }
  ```

  **测试结果：**

  * **大小比较：json长度:134.0,proto长度:14.0,压缩比:9.571428571428571**
  * **耗时比较：json耗时:62,proto耗时:16,耗时比:3.875**

# NIO实验

**本实验效果不好（测试BIO普遍优于NIO，可能是单次传输数据较小的原因），感觉可以口头说提高了3倍左右即可（理论上NIO比BIO性能好，且阿里巴巴性能优化pdf中有案例支持），主要回答Netty相关八股文即可。**如下提供Server和Client的代码，选择一种Server模式和Client模式，先启动Server再启动Client可以完成性能测试。

* **NIO Server代码**

  ```java
  public class NtripServerNio {
      public static void main(String[] args) throws Exception {
          EventLoopGroup bossGroup = new NioEventLoopGroup(1);
          EventLoopGroup workerGroup = new NioEventLoopGroup(args.length >= 1 ? Integer.parseInt(args[0]) : 0);
          ServerBootstrap serverBootstrap = new ServerBootstrap()
                  .group(bossGroup, workerGroup)
                  .channel(NioServerSocketChannel.class)
                  .option(ChannelOption.SO_BACKLOG, 128)
                  .childOption(ChannelOption.SO_KEEPALIVE, true)
                  .childHandler(new ChannelInitializer<SocketChannel>() {
                      @Override
                      protected void initChannel(SocketChannel socketChannel) throws Exception {
                          socketChannel.pipeline().addLast(new JsonDecoder());
  //                        socketChannel.pipeline().addLast(new ProtoDecoder());
                      }
                  });
  
          serverBootstrap.bind(8088).sync();
  
      }
  }
  
  @Slf4j
  abstract class AbstractDecoder extends ByteToMessageDecoder {
  
      @Override
      protected void decode(ChannelHandlerContext context, ByteBuf in, List<Object> list) {
          //长度信息是否可读
          int availableBytes = in.readableBytes();
          if (availableBytes < 4) {
              return;
          }
  
          in.markReaderIndex();
          int payloadLength = in.readInt();
          //数据是否到齐
          if (in.readableBytes() < payloadLength) {
              in.resetReaderIndex();//重设readerIndex到markReaderIndex
              return;
          }
  
          byte[] payload = new byte[payloadLength];
          in.readBytes(payload);
          doRead(payload);
          context.channel().writeAndFlush(Unpooled.wrappedBuffer("bye".getBytes()));
      }
  
      protected abstract void doRead(byte[] data);
  }
  
  
  @Slf4j
  class JsonDecoder extends AbstractDecoder {
      private static ObjectMapper mapper = new ObjectMapper();
  
  
      @Override
      protected void doRead(byte[] data) {
          NtripData ntripData = null;
          try {
              ntripData = mapper.readValue(data, NtripData.class);
              log.info("ntripDataJson:{}", ntripData);
          } catch (IOException e) {
              throw new RuntimeException(e);
          }
      }
  }
  
  @Slf4j
  class ProtoDecoder extends AbstractDecoder {
  
      @Override
      protected void doRead(byte[] data) {
          NtripDataProto.NtripData ntripData = null;
          try {
              ntripData = NtripDataProto.NtripData.parseFrom(data);
              log.info("ntripDataProto:{}", ntripData);
          } catch (InvalidProtocolBufferException e) {
              throw new RuntimeException(e);
          }
      }
  }
  ```

* **BIO Server：Socket和Thread一对一**

  此版本的Server对一个Socket连接创建一个连接，这种方式本质上最耗费资源，对系统影响应该是最大，但是实际上测试结果显示吞吐量也很高。

  ```java
  @Slf4j
  public class NtripServerBio {
  
      public static void main(String[] args) throws Exception {
  		ServerSocket serverSocket = null;
          try {
              serverSocket = new ServerSocket(8088);
              while (true) {
                  Socket socket = serverSocket.accept();
                  NtripBioTask ntripBioTask = new NtripBioTask(socket);
                  //一个Socket启动一个线程
                  new Thread(ntripBioTask).start();
              }
          } catch (IOException e) {
              throw new RuntimeException(e);
          } finally {
              if (serverSocket != null) {
                  serverSocket.close();
              }
          }
      }
  
  }
  
  @Data
  @AllArgsConstructor
  @Slf4j
  class NtripBioTask implements Runnable {
      private static ObjectMapper mapper = new ObjectMapper();
      private Socket socket;
      private static final int maxWaitTime = 10;
      private static Random random = new Random();
  
      @Override
      public void run() {
          try {
              readAndWrite();
          } catch (Exception e) {
              e.printStackTrace();
          } finally {
              try {
                  socket.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
  
      private void readAndWrite() throws Exception {
          byte[] bytes = new byte[1024];
          InputStream inputStream = socket.getInputStream();
          OutputStream outputStream = socket.getOutputStream();
          ByteBuf buffer = Unpooled.buffer(1024);
          int count = 0;
          while (socket.isConnected() && !socket.isClosed() && count < maxWaitTime) {
              int len = inputStream.read(bytes);
              if (len == -1) {
                  count++;
                  //这里让bio线程阻塞，强行模拟bio的劣势
                  Thread.sleep(random.nextInt(10));
                  continue;
              }
              buffer.writeBytes(bytes, 0, len);
              buffer.markReaderIndex();
              if (buffer.readableBytes() < 4) {
                  continue;
              }
              int payloadLength = buffer.readInt();
              //数据是否到齐
              if (buffer.readableBytes() < payloadLength) {
                  buffer.resetReaderIndex();//重设readerIndex到markReaderIndex
                  continue;
              }
              byte[] data = new byte[payloadLength];
              buffer.readBytes(data);
              buffer.discardReadBytes();
              readJson(data);
              //readProto(data);
              outputStream.write("bye".getBytes());
              outputStream.flush();
              buffer.clear();
          }
      }
  
      private void readJson(byte[] data) throws Exception {
          NtripData ntripData = mapper.readValue(data, NtripData.class);
          log.info("ntripDataJson:{}", ntripData);
      }
  
      private void readProto(byte[] data) throws Exception {
          NtripDataProto.NtripData ntripData = NtripDataProto.NtripData.parseFrom(data);
          log.info("ntripDataProto:{}", ntripData);
      }
  }
  
  ```

* **BIO  Server：使用线程池**

  优化Socket和Thread一对一的方式，这种方式是较多博客案例推荐的BIO处理连接的方式，但这种方式问题在于如果线程池中线程被阻塞，很容易导致线程池进入不可用状态。

  ```java
  @Slf4j
  public class NtripServerBio {
       public static void main(String[] args) throws Exception {
          ServerSocket serverSocket = null;
          ExecutorService service = Executors.newFixedThreadPool(args.length > 0 ? Integer.parseInt(args[0]) : Runtime.getRuntime().availableProcessors() * 2);
          try {
              serverSocket = new ServerSocket(8088);
              while (true) {
                  Socket socket = serverSocket.accept();
                  NtripBioTask ntripBioTask = new NtripBioTask(socket);
                  service.submit(ntripBioTask);
              }
          } catch (IOException e) {
              throw new RuntimeException(e);
          } finally {
              if (serverSocket != null) {
                  serverSocket.close();
              }
              service.shutdownNow();
          }
      }
  }
  
  @Data
  @AllArgsConstructor
  @Slf4j
  class NtripBioTask implements Runnable {
      private static ObjectMapper mapper = new ObjectMapper();
      private Socket socket;
      private static final int maxWaitTime = 10;
      private static Random random = new Random();
  
      @Override
      public void run() {
          try {
              readAndWrite();
          } catch (Exception e) {
              e.printStackTrace();
          } finally {
              try {
                  socket.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
  
      private void readAndWrite() throws Exception {
          byte[] bytes = new byte[1024];
          InputStream inputStream = socket.getInputStream();
          OutputStream outputStream = socket.getOutputStream();
          ByteBuf buffer = Unpooled.buffer(1024);
          int count = 0;
          while (socket.isConnected() && !socket.isClosed() && count < maxWaitTime) {
              int len = inputStream.read(bytes);
              if (len == -1) {
                  count++;
                  //这里让bio线程阻塞，强行模拟bio的劣势
                  Thread.sleep(random.nextInt(10));
                  continue;
              }
              buffer.writeBytes(bytes, 0, len);
              buffer.markReaderIndex();
              if (buffer.readableBytes() < 4) {
                  continue;
              }
              int payloadLength = buffer.readInt();
              //数据是否到齐
              if (buffer.readableBytes() < payloadLength) {
                  buffer.resetReaderIndex();//重设readerIndex到markReaderIndex
                  continue;
              }
              byte[] data = new byte[payloadLength];
              buffer.readBytes(data);
              buffer.discardReadBytes();
              readJson(data);
              //readProto(data);
              outputStream.write("bye".getBytes());
              outputStream.flush();
              buffer.clear();
          }
      }
  
      private void readJson(byte[] data) throws Exception {
          NtripData ntripData = mapper.readValue(data, NtripData.class);
          log.info("ntripDataJson:{}", ntripData);
      }
  
      private void readProto(byte[] data) throws Exception {
          NtripDataProto.NtripData ntripData = NtripDataProto.NtripData.parseFrom(data);
          log.info("ntripDataProto:{}", ntripData);
      }
  }
  ```

* **Client：一个线程创建一个Socket**

  ```java
  @Slf4j
  public class NtripClient {
      static volatile boolean stop = false;
      static AtomicInteger count = new AtomicInteger(0);
      static Random random = new Random();
      static ObjectMapper mapper = new ObjectMapper();
      static String host = "localhost";
      static int port = 8088;
      static int threadNums = 200;
      static int testTime = 60;
  
      static void writeAndRead(Socket socket, byte[] data, ByteBuf buffer) {
          try {
              if (socket.isConnected() && !socket.isClosed()) {
                  buffer.writeInt(data.length);
                  buffer.writeBytes(data);
                  data = new byte[buffer.readableBytes()];
                  buffer.readBytes(data);
                  OutputStream outputStream = socket.getOutputStream();
                  outputStream.write(data);
                  InputStream inputStream = socket.getInputStream();
                  byte bytes[] = new byte[1024];
                  int len = inputStream.read(bytes);
                  log.info("read:{}", new String(bytes, 0, len));
                  count.incrementAndGet();
              }
          } catch (Exception e) {
              throw new RuntimeException(e);
          }
      }
  	//随机生成NtripData
      static NtripData getData() {
          NtripData ntripData = new NtripData();
          ntripData.setMessageType(1001);
          ntripData.setSatelliteId(random.nextInt());
          ntripData.setPhaseCorrection(random.nextDouble());
          return ntripData;
      }
  
      static byte[] getJsonData() throws Exception {
          NtripData ntripData = getData();
          byte[] bytes = mapper.writeValueAsString(ntripData).getBytes();
          return bytes;
      }
  
      static byte[] getProtoData() throws Exception {
          NtripData ntripData = getData();
          NtripDataProto.NtripData ntripDataProto = NtripDataProto.NtripData.newBuilder()
                  .setMesssageType(ntripData.getMessageType())
                  .setSatelliteId(ntripData.getSatelliteId())
                  .setPhaseCorrection(ntripData.getPhaseCorrection()).build();
          return ntripDataProto.toByteArray();
      }
  
  
      static Socket getSocket() {
          while (true) {
              try {
                  Socket socket = new Socket(host, port);
                  return socket;
              } catch (Exception e) {
                  e.printStackTrace();
              }
          }
      }
  
  
  
      public static void main(String[] args) throws Exception {
          Thread[] threads = new Thread[threadNums];
          for (int i = 0; i < threadNums; i++) {
              threads[i] = new Thread(() -> {
                  ByteBuf buffer = Unpooled.buffer(1024);
                  Socket socket = getSocket();
                  while (!stop) {
                      try {
                          writeAndRead(socket, getJsonData(), buffer);
                          buffer.clear();
                          //模拟随机断开
                          if (random.nextInt(100) > 95) {
                              socket.close();
                              socket = getSocket();
                          }
                      } catch (Exception e) {
                          e.printStackTrace();
                          try {
                              socket.close();
                              socket = getSocket();
                          } catch (IOException ex) {
                              log.info("reconnect");
                          }
                      }
                  }
              });
              threads[i].setDaemon(true);
              threads[i].start();
          }
          TimeUnit.SECONDS.sleep(testTime);
          stop = true;
          log.info("完成测试次数:{},请求延迟:{}", count, count.get() / (double) (testTime * 1000));
      }
  }
  ```

* **Client：Socket的个数>线程个数，线程随机选取Socket读取**

  这个案例如果搭配BIO Server：Socket和Thread一对一的模式一起使用，案例来说性能会比NIO Server差很多，因为BIO Server创建了大量线程，但是有效线程很少，因为Client的线程个数远小于socket的个数，但是实际结果BIO性能依旧比NIO好。

  ```java
  @Slf4j
  public class NtripClient {
      static volatile boolean stop = false;
      static AtomicInteger count = new AtomicInteger(0);
      static Random random = new Random();
      static ObjectMapper mapper = new ObjectMapper();
      static String host = "localhost";
      static int port = 8088;
      static int threadNums = 200;
      static int testTime = 60;
      static int socketNums = 2000;
      static List<Socket> sockets = new ArrayList<>();
      static List<ReentrantLock> locks = new ArrayList<>();
  
      static void writeAndRead(Socket socket, byte[] data, ByteBuf buffer) {
          try {
              if (socket.isConnected() && !socket.isClosed()) {
                  buffer.writeInt(data.length);
                  buffer.writeBytes(data);
                  data = new byte[buffer.readableBytes()];
                  buffer.readBytes(data);
                  OutputStream outputStream = socket.getOutputStream();
                  outputStream.write(data);
                  InputStream inputStream = socket.getInputStream();
                  byte bytes[] = new byte[1024];
                  int len = inputStream.read(bytes);
                  log.info("read:{}", new String(bytes, 0, len));
                  count.incrementAndGet();
              }
          } catch (Exception e) {
              throw new RuntimeException(e);
          }
      }
  
      static NtripData getData() {
          NtripData ntripData = new NtripData();
          ntripData.setMessageType(1001);
          ntripData.setSatelliteId(random.nextInt());
          ntripData.setPhaseCorrection(random.nextDouble());
          return ntripData;
      }
  
      static byte[] getJsonData() throws Exception {
          NtripData ntripData = getData();
          byte[] bytes = mapper.writeValueAsString(ntripData).getBytes();
          return bytes;
      }
  
      static byte[] getProtoData() throws Exception {
          NtripData ntripData = getData();
          NtripDataProto.NtripData ntripDataProto = NtripDataProto.NtripData.newBuilder()
                  .setMesssageType(ntripData.getMessageType())
                  .setSatelliteId(ntripData.getSatelliteId())
                  .setPhaseCorrection(ntripData.getPhaseCorrection()).build();
          return ntripDataProto.toByteArray();
      }
  
  
      static Socket getSocket() {
          while (true) {
              try {
                  Socket socket = new Socket(host, port);
                  return socket;
              } catch (Exception e) {
                  e.printStackTrace();
              }
          }
      }
  
  
  public static void main(String[] args) throws Exception {
          //创建指定个数socket，socket的个数大于线程数
          for (int i = 0; i < socketNums; i++) {
              sockets.add(getSocket());
              locks.add(new ReentrantLock());
          }
          Thread[] threads = new Thread[threadNums];
          for (int i = 0; i < threadNums; i++) {
              threads[i] = new Thread(() -> {
                  ByteBuf buffer = Unpooled.buffer(1024);
                  Socket socket = getSocket();
                  while (!stop) {
                      //线程随机挑选一个socket进行数据传输
                      int index = random.nextInt(socketNums);
                      if (locks.get(index).tryLock()) {
                          try {
                              writeAndRead(socket, getJsonData(), buffer);
                              //writeAndRead(socket, getProtoData(), buffer);
                              buffer.clear();
                          } catch (Exception e) {
                              e.printStackTrace();
                          } finally {
                              locks.get(index).unlock();
                          }
                      }
                  }
              });
              threads[i].setDaemon(true);
              threads[i].start();
          }
          TimeUnit.SECONDS.sleep(testTime);
          stop = true;
          log.info("完成测试次数:{},请求延迟:{}", count, count.get() / (double) (testTime * 1000));
      }
  }
  
  
  ```

  