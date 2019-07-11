# reactive-reference-impl

Five IO model in Linux
(1) Blocking IO
(2) Non-blocking IO
(3) IO multiplexing
(4) Signal driven I/O
(5) Asynchronous I/O
The first 4 are synchronous, only the last one is asynchronous IO.

Blocking IO

IO process takes two steps: (1) kernel data preparation (2) copying the data from kernel space to user space. Once the user thread invokes the IO method, the user thread will be blocked and has to wait until the data is returned. This is the blocking IO model.

Non-blocking IO
In the non-blocking IO model, after the IO function is invoked and before the kernel data is ready, the IO function could return immediately. The user thread calls the IO function many times (polling), once it finds the data is ready, it will take the second step: copying data from kernel space to user space, and in this step the user thread is blocked.

IO multiplexing
The user thread invokes methods like select, poll or epoll, to monitor multiple IO requests, it will be blocked until at least one IO request is ready (ready to read or write). The IO multiplexing model could process multiple IO requests by just one thread.

Signal driven I/O
Once the user thread invokes the IO function, it will return immediately even though the data is not ready. When the kernel data is ready, a signal function will be sent, to notify the user thread to carry on the second step, copying data from the kernel space to the user space.

Asynchronous I/O
In the asynchronous I/O model, the user thread will not be blocked in both of the two steps in IO operation. When the copying of data is finished in the second step, a signal function will be sent, to notify the user thread that the IO operation is complete.

Except the signal driven IO model, other four IO models are all supported in Java.

(1). the earliest traditional IO in Java is the blocking IO
(2). the Java NIO is the non-blocking IO
(3). the Reactor pattern in Java NIO is an implementation of IO multiplexing
(4). the Proactor pattern in Java AIO is an implementation of asynchronous IO

The traditional IO in Java is blocking IO, for example, if you want to read data from a socket and invoke read() method, the thread will get stuck at read() method until the data is returned. Therefore in the traditional web service design, the multi-thread (or thread pool) pattern are widely adopted. A thread is blocked when it serves one request, if another request comes at this moment, we have to start a new thread to serve it. The blockage in one thread shouldn't affect the works in other threads. This kind of server design seems straightforward, but it starts thread for every request, which will be resource-consuming, even though the thread pool pattern makes the thread reusable.

The multi-thread (or thread pool) pattern are suitable in the scenario that most of the connections are short connections. If most of the connections are long connections, and simultaneously there're only a few connections have IO operations, serving every connection with one thread will be quite wasteful[8][9]. Therefore the IO model with higher performance appears, the Java NIO model. It was designed on the basis of the Reactor pattern, which is also the most important concept in Reactive Programming.


Java NIO Model

Concepts in Java NIO: Channel, Buffer and Selector.

Channel

Channel is similar to Stream in traditional IO model. But Streams are unidirectional, you use InputStream to read or use OutputStream to write. Channels can be bidirectional, you can read and write through one Channel.

Some commonly used Channels are listed below:

FileChannel - read and write through file
SocketChannel - read and write through TCP socket
ServerSocketChannel - Listen the TCP connection from clients, create new SocketChannel for each TCP connection
DatagramChannel - read and write through UDP protocol
Buffer
Buffer can be considered as a data container, which could be implemented by an array. Either reading from a channel or writing to a channel, data must be put into the buffer first.

Selector

Selector is the core class in Java NIO, it monitors and processes interested IO events that happened in multiple registered Channels. Through this mechanism, we could maintain multiple connections by just one thread. Only when the IO events actually happen among these connections, the IO process logic are truly invoked. There's no need to start a new thread each time when a new connection comes in, therefore it would significantly reduce the system load.

In company with Selector, the SelectionKey is another important class, which represents an arrived event. These two classes constitute the key logic of the NIO server[10].

The figure below shows the monitoring of multiple channels (or connections) by a Selector running on a single thread.


How it works?

Creates selector
Creates Server Socket channel bind it to the port
Sets server socket channel as non-blocking
Register the server channel to the selector and set the OP_ACCEPT as the interested event
SocketChannel connected to the client
Set channel as non-blocking
Send/Read messages to the client
Register the client channel to the selector and configure the event type (read/write)
If channel as data to read, create buffer for reading 
