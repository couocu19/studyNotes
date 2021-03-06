# java序列化

## 序列化和反序列化

- ### 序列化

   ![img](https://pics2.baidu.com/feed/6d81800a19d8bc3eb6da307024085b1aaad345c5.jpeg?token=921868414de7a9b800d7ed553c4ca4bd&s=51023577138F41490AD581DB0000C0B1)

从上面这张图我们可以看到，两个进程进行通信时候，想要发送数据，要先要把数据发送到TCP缓冲区，然后形成报文再发送出去，同样的道理，接收端也是一样。我们可以相互发送各种类型的数据，包括文本、图片、音频、视频等， 而这些数据都会以二进制序列的形式在网络上传送。同样的，当两个Java进程进行通信时，也可以使用序列化技术实现对象之间的传递。



![img](https://pics2.baidu.com/feed/a6efce1b9d16fdfa29919576120c715095ee7b35.jpeg?token=f1635eea4738649d0275618c4289686c&s=60946F32A1E5491B085D54CA000090B3)



从这张图也可以清晰的看出，发送数据之前要序列化，接受数据要反序列化。到了这，我们再来看序列化的概念就比较好理解了，一句话：

**Java序列化是指把Java对象转换为字节序列的过程，而Java反序列化是指把字节序列恢复为Java对象的过程；**



- ### 序列化的应用场景

  这个使用场景应该算是最重要的一环了，因为我们学习序列化就是为了使用他，现在把他们归纳一下：

  （1）永久性保存对象，保存对象的字节序列到本地文件或者数据库中； 、

  （2）通过序列化以字节流的形式使对象在网络中进行传递和接收；

  （3）通过序列化在进程间传递对象；

- ### 序列化的好处

  其实好处是根据使用场景来的;

  (1)实现了数据的持久化，通过序列化可以把数据永久地保存到硬盘上

  (2)利用序列化实现远程通信，即在网络上传送对象的字节序列。

  

  

  

  

  

  

  

  