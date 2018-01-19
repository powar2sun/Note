#   IO
![io](https://github.com/powar2sun/Note/blob/master/Language/pictures/io.png)
##  File
![file](https://github.com/powar2sun/Note/blob/master/Language/pictures/file.png)
####    InputStream
![inputStream](https://github.com/powar2sun/Note/blob/master/Language/pictures/inputStream.png)
####    OutputSteam
![outputStream](https://github.com/powar2sun/Note/blob/master/Language/pictures/outputStream.png)
####    Reader
![reader](https://github.com/powar2sun/Note/blob/master/Language/pictures/reader.png)
####    Writer
![writer](https://github.com/powar2sun/Note/blob/master/Language/pictures/writer.png)
####    DataInput
![dataInput](https://github.com/powar2sun/Note/blob/master/Language/pictures/dataInput.png)
####    DataOutput
![dataOutput](https://github.com/powar2sun/Note/blob/master/Language/pictures/dataOutput.png)
##  IO Type
*   BIO 阻塞式

![bio](https://github.com/powar2sun/Note/blob/master/Language/pictures/bio.png)

*   Non-BIO 非阻塞式

![nonBio](https://github.com/powar2sun/Note/blob/master/Language/pictures/nonBio.png)

*   IO(select,poll) 复用

![ioRep](https://github.com/powar2sun/Note/blob/master/Language/pictures/ioRep.png)

*   IO 信号驱动式

![ioSinga](https://github.com/powar2sun/Note/blob/master/Language/pictures/ioSinga.png)

*   IO 异步

![ioAio](https://github.com/powar2sun/Note/blob/master/Language/pictures/ioAio.png)

#   NIO
IO模式会按照Blocking/Non-Blocking、Synchronous/Asynchronous这两个标准进行分类，其中Blocking与Synchronous基本上一个意思，
而NIO与Async的区别在于NIO强调的是Polling(轮询)，而Async强调的是Notification(通知)

*   同步阻塞：用户进程在发起一个IO操作以后，必须等待IO操作的完成，只有当真正完成了IO操作以后，用户进程才能运行
*   同步非阻塞：用户进程发起一个IO操作以后，边可返回做其它事情，但是用户进程需要时不时的询问IO操作是否就绪，这就要求用户进程不停的去询问，从而引入不必要的CPU资源浪费
*   异步非阻塞：用户进程只需要发起一个IO操作然后立即返回，等IO操作真正的完成以后，应用程序会得到IO操作完成的通知，此时用户进程只需要对数据进行处理就好了，不需要进行实际的IO读写操作，因为真正的IO读取或者写入操作已经由内核完成了。

####    Channels
是一个通道，可以通过它读取和写入数据,通道与流的不同之处在于通道是双向的，流只是在一个方向上移动（InputStream或OutputStream的子类），而且通道可以用于读、写或者同时用于读写。
*   一个 Channel 可以读和写，而一个流一般只能读或者写
*   Channel 可以异步(asynchronously)的读和写
*   Channel 总是需要一个 Buffer，不管是读到 Buffer 还是从 Buffer 写到 Channel
![channels](https://github.com/powar2sun/Note/blob/master/Language/pictures/channels.png)

```java
public class DemoChannel {
    public void readC(String path){
        try {
            FileInputStream fileInputStream = new FileInputStream(path);
            FileChannel channel = fileInputStream.getChannel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            channel.read(buffer);

            buffer.flip();
            while (buffer.remaining() > 0) {
                System.out.print((char) buffer.get());
            }

            fileInputStream.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void writeC(String path){
        String[] message = {"hello,", "world!", "powar..."};
        try {
            FileOutputStream fileOutputStream = new FileOutputStream(path);
            FileChannel channel = fileOutputStream.getChannel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);

            for (int i = 0; i < message.length; i++) {
                byte[] tem = message[i].getBytes();
                buffer.put(tem);
            }

            buffer.flip();
            channel.write(buffer);

            fileOutputStream.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    public static void main(String[] args) {
        DemoChannel dc = new DemoChannel();
        dc.readC("/home/powar/Downloads/test.txt");
        dc.writeC("/home/powar/Documents/demo.txt");
    }
}

```
####    Buffers
缓冲区实际上是一个容器对象，更直接的说，其实就是一个数组，在NIO库中，所有数据都是用缓冲区处理的
在NIO中，所有的缓冲区类型都继承于抽象类Buffer，最常用的就是ByteBuffer，对于Java中的基本类型，基本都有一个具体Buffer类型与之相对应
```java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
while (bytesRead != -1) {
  System.out.println("Read " + bytesRead);
  buf.flip();
  while(buf.hasRemaining()){
      System.out.print((char) buf.get());
  }
  buf.clear();
  bytesRead = inChannel.read(buf);
}
aFile.close();
```
打开一个文件的连接，然后获取到一个 Channel，开辟一个 Buffer，从 channel 读取数据到 buffer，然后再从 Buffer 中获取到读取的数据。
####    Selectors
多路复用器Selector是Java NIO编程的基础。Selector会不断地轮询注册在其上的Channel，如果某个Channel上面有新的TCP连接接入、读和写事件，
这个Channel就处于就绪状态，会被Selector轮询出来，然后通过SelectionKey可以获取就绪Channel的集合，进行后续的I/O操作
![reactor](https://github.com/powar2sun/Note/blob/master/Language/pictures/reactor.png)

#   Linux NIO
![spe](https://github.com/powar2sun/Note/blob/master/Language/pictures/spe.png)
####    Select
select能监控的描述符个数由内核中的FD_SETSIZE限制，仅为1024（32位，64位为2048），这也是select最大的缺点，因为现在的服务器并发量远远不止1024。
即使能重新编译内核改变FD_SETSIZE的值，但这并不能提高select的性能。
每次调用select都会线性扫描所有描述符的状态，在select结束后，用户也要线性扫描fd_set数组才知道哪些描述符准备就绪，等于说每次调用复杂度都是O（n）的，
在并发量大的情况下，每次扫描都是相当耗时的，很有可能有未处理的连接等待超时。
每次调用select都要在用户空间和内核空间里进行内存复制fd描述符等信息
####    poll
poll使用pollfd结构来存储fd，突破了select中描述符数目的限制。
与select的后两点类似，poll仍然需要将pollfd数组拷贝到内核空间，之后依次扫描fd的状态，整体复杂度依然是O（n）的，
在并发量大的情况下服务器性能会快速下降。 
####    epoll
epoll维护的描述符数目不受到限制，而且性能不会随着描述符数目的增加而下降。
服务器的特点是经常维护着大量连接，epoll先通过epoll_ctl注册一个描述符到内核中，并一直维护着而不像poll每次操作都将所有要监控的描述符传递给内核；
在描述符读写就绪时，通过回掉函数将自己加入就绪队列中，之后epoll_wait返回该就绪队列。也就是说，epoll基本不做无用的操作，时间复杂度仅与活跃的客户端数有关，而不会随着描述符数目的增加而下降。
epoll在传递内核与用户空间的消息时使用了内存共享，而不是内存拷贝，这也使得epoll的效率比poll和select更高。 

*   selector
![select](https://github.com/powar2sun/Note/blob/master/Language/pictures/select.png)
