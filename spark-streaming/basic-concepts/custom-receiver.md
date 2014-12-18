# 自定义receiver指南

Spark Streaming可以从包括内置数据源在内的任意数据源获取数据（其他数据源包括flume，kafka，kinesis，文件，套接字等等）。这需要开发者去实现一个定制`receiver`从具体的数据源接收
数据。这个指南介绍了实现自定义`receiver`的过程，以及怎样将`receiver`用到Spark Streaming应用程序中。

## 实现一个自定义的Receiver

这一节开始实现一个[Receiver](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.receiver.Receiver)。一个自定义的receiver必须继承
这个抽象类,实现它的两个方法`onStart()`（开始接收数据）以及`onStop()`（停止接收数据）。

需要注意的是`onStart()`和`onStop()`不能够无限期的阻塞。通常情况下，`onStart()`启动线程负责数据的接收，`onStop()`确保这个接收过程停止。接收线程也能够调用`receiver`的`isStopped`
方法去检查是否已经停止接收数据。

一旦接收了数据，这些数据就能够通过调用`store(data)`方法存到Spark中，`store(data)`是[Receiver]类中的方法。有几个重载的`store()`方法允许你存储接收到的数据（record-at-a-time or as whole collection of objects/serialized bytes）

在接收线程中出现的任何异常都应该被捕获或者妥善处理从而避免`receiver`在没有提示的情况下失败。`restart(<exception>)`方法将会重新启动`receiver`，它通过异步的方式首先调用`onStop()`方法，
然后在一段延迟之后调用`onStart()`方法。`stop(<exception>)`将会调用`onStop()`方法终止`receiver`。`reportError(<error>)`方法在不停止或者重启`receiver`的情况下打印错误消息到
驱动程序(driver)。

如下所示，是一个自定义的`receiver`，它通过套接字接收文本数据流。它用分界符'\n'把文本流分割为行记录，然后将它们存储到Spark中。如果接收线程碰到任何连接或者接收错误，`receiver`将会
重新启动以尝试再一次连接。

```scala

class CustomReceiver(host: String, port: Int)
  extends Receiver[String](StorageLevel.MEMORY_AND_DISK_2) with Logging {

  def onStart() {
    // Start the thread that receives data over a connection
    new Thread("Socket Receiver") {
      override def run() { receive() }
    }.start()
  }

  def onStop() {
   // There is nothing much to do as the thread calling receive()
   // is designed to stop by itself isStopped() returns false
  }

  //Create a socket connection and receive data until receiver is stopped
  private def receive() {
    var socket: Socket = null
    var userInput: String = null
    try {
     // Connect to host:port
     socket = new Socket(host, port)
     // Until stopped or connection broken continue reading
     val reader = new BufferedReader(new InputStreamReader(socket.getInputStream(), "UTF-8"))
     userInput = reader.readLine()
     while(!isStopped && userInput != null) {
       store(userInput)
       userInput = reader.readLine()
     }
     reader.close()
     socket.close()
     // Restart in an attempt to connect again when server is active again
     restart("Trying to connect again")
    } catch {
     case e: java.net.ConnectException =>
       // restart if could not connect to server
       restart("Error connecting to " + host + ":" + port, e)
     case t: Throwable =>
       // restart if there is any other error
       restart("Error receiving data", t)
    }
  }
}

```

## 在Spark流应用程序中使用自定义的receiver

在Spark流应用程序中，用`streamingContext.receiverStream(<instance of custom receiver>)`方法，可以使用自动用`receiver`。代码如下所示：

```scala
// Assuming ssc is the StreamingContext
val customReceiverStream = ssc.receiverStream(new CustomReceiver(host, port))
val words = lines.flatMap(_.split(" "))
...
```

完整的代码见例子[CustomReceiver.scala](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/CustomReceiver.scala)

## 实现和使用自定义的基于actor的receiver

自定义的Akka actor也能够拥有接收数据。[ActorHelper](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.receiver.ActorHelper)trait可以
应用于任何Akka actor中。它运行接收的数据通过调用`store()`方法存储于Spark中。可以配置这个actor的监控(supervisor)策略处理错误。

```scala
class CustomActor extends Actor with ActorHelper {
  def receive = {
   case data: String => store(data)
  }
}
```

利用这个actor，一个新的输入数据流就能够被创建。

```scala
// Assuming ssc is the StreamingContext
val lines = ssc.actorStream[String](Props(new CustomActor()), "CustomReceiver")
```

完整的代码间例子[ActorWordCount.scala](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/ActorWordCount.scala)
