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
![channels]()
####    Buffers
####    Selectors
