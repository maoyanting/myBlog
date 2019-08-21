---
title: Java-I/O
date: 2018-03-02 15:29:20
categories: 
- Java
tags:
- Java
---

I/O资料整理

<!--more-->

原文链接：

[java回忆录—输入输出流详细讲解（入门经典）](http://blog.csdn.net/qq_22063697/article/details/52137369)

[java回忆录—I/O流详解补充](http://blog.csdn.net/qq_22063697/article/details/52154916)

采用数据流的目的就是使得输出输入独立于设备。

## 一、I/O体系结构

![img](http://img.blog.csdn.net/20160806165130911)

简单介绍下上图：

如果是 Stream 的话就是字节流，如果是 Reader/Writer 的话就是字符流。

在整个Java.io包中最重要的就是5个类和一个接口。5个类指的是File、OutputStream、InputStream、Writer、Reader；一个接口指的是Serializable.

### 流式部分主要类：

Java中字符是采用Unicode标准，一个字符是16位，即一个字符使用两个字节来表示。为此，JAVA中引入了处理字符的流。

| 分类           | 详细                                                         | 其他                                                         |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 对文件进行操作 | FileInputStream， FileOutputStream， FileReader， FileWriter |                                                              |
| 对管道进行操作 | PipedInputStream, PipedOutStream， PipedReader， PipedWriter | PipedInputStream的一个实例要和PipedOutputStream的一个实例共同使用，共同完成管道的读取写入操作。主要用于线程操作。 |
| 字节/字符数组  | ByteArrayInputStream， ByteArrayOutputStream， CharArrayReader， CharArrayWriter | 在内存中开辟了一个字节或字符数组                             |
| Buffered缓冲流 | BufferedInputStream， BufferedOutputStream， BufferedReader, BufferedWriter | 是带缓冲区的处理流，缓冲区的作用的主要目的是：避免每次和硬盘打交道，提高数据访问的效率。 |
| 转化流         | InputStreamReader， OutputStreamWriter                       | 把字节转化成字符。                                           |
| 数据流         | DataInputStream， DataOutputStream。                         | 因为平时若是我们输出一个8个字节的long类型或4个字节的float类型，那怎么办呢？可以一个字节一个字节输出，也可以把转换成字符串输出，但是这样转换费时间，若是直接输出该多好啊，因此这个数据流就解决了我们输出数据类型的困难。 |
| 打印流         | printStream， printWriter，                                  | 一般是打印到控制台，可以进行控制打印的地方。                 |
| 对象流         | ObjectInputStream， ObjectOutputStream                       | 把封装的对象直接输出                                         |
| 序列化流       | SequenceInputStream。                                        | 对象序列化：把对象直接转换成二进制，写入介质中。             |

### 非流式部分主要类：

RandomAccessFile（随机文件操作）：它的功能丰富，**可以从文件的任意位置进行存取（输入输出）操作。**

#### File类：

作用：File类主要用于命名文件、查询文件属性和处理文件目录。

| 构造器                              | 解释                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| File (String pathname)              | e.g:`File  f1=new File("FileTest1.txt");`                    |
| File（URI uri）                     |                                                              |
| File (String parent , String child) | e.g:`File f2=new  File(“D:\\dir1","FileTest2.txt") ;`//D:\\dir1目录事先必须存在，否则异常 |
| File (File parent , String child)   | e.g: ` File  f4=new File("E：\\dir3");File  f5=new File(f4,"FileTest5.txt");`//在如果 E：\\dir3目录不存在则需要先使用f4.mkdir()先创建 |

一个对应于某磁盘文件或目录的File对象一经创建， 就可以通过调用它的方法来获得文件或目录的属性。

>   1）public boolean exists( ) 判断文件或目录是否存在
>
>   2）public boolean isFile( ) 判断是文件还是目录 
>
>   3）public boolean isDirectory( ) 判断是文件还是目录
>
>   4）public String getName( ) 返回文件名或目录名
>
>   5）public String getPath( ) 返回文件或目录的路径。
>
>   6）public long length( ) 获取文件的长度 
>
>   7）public String[ ] list ( ) 将目录中所有文件名和目录名保存在字符串数组中返回。 
>
>   8）public File[] listFiles() 返回某个目录下所有文件和目录的绝对路径，返回的是File数组
>
>   9）public String getAbsolutePath() 返回文件或目录的绝对路径
>
>   ....
>   File类中还定义了一些对文件或目录进行管理、操作的方法，常用的方法有：
>   1） public boolean renameTo( File newFile );    重命名文件
>
>   2） public void delete( );   删除文件
>
>   3） public boolean mkdir( ); 创建目录
>
>   4）public boolean createNewFile(); 创建文件



## 二、几种流的介绍

### 1、InputStream抽象类

InputStream 为字节输入流，它本身为一个抽象类，必须依靠其子类实现各种功能，此抽象类是表示字节输入流的所有类的超类。**数据单位为字节（8bit）**；

| method                                | 解释                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| abstract int read( )                  | 读取一个byte的数据，返回值是高位补0的int类型值。若返回值=-1说明没有读取到任何字节读取工作结束。 |
| int read(byte b[ ])                   | 读取b.length个字节的数据放到b数组中。返回值是读取的字节数。该方法实际上是调用下一个方法实现的 |
| int read(byte b[ ], int off, int len) | 从输入流中最多读取len个字节的数据，存放到偏移量为off的b数组中。 |
| int available( )                      | 返回输入流中可以读取的字节数。注意：若输入阻塞，当前线程将被挂起，如果InputStream对象调用这个方法的话，它只会返回0，这个方法必须由继承InputStream类的子类对象调用才有用， |
| long skip(long n)                     | 忽略输入流中的n个字节，返回值是实际忽略的字节数, 跳过一些字节来读取 |
| int close( )                          | 在使用完后，必须对打开的流进行关闭.                          |

主要的子类：

> 1） FileInputStream：把一个文件作为InputStream，实现对文件的读取操作     
>
> 2） ByteArrayInputStream：把内存中的一个缓冲区作为InputStream使用 
>
> 3） StringBufferInputStream：把一个String对象作为InputStream 
>
> 4） PipedInputStream：实现了pipe的概念，主要在线程中使用 
>
> 5） SequenceInputStream：把多个InputStream合并为一个InputStream 
>



### 2、OutputStream抽象类

OutputStream提供了3个write方法来做数据的输出，这个是和InputStream是相对应的。 　　

| method                                         | 解释                                            |
| ---------------------------------------------- | ----------------------------------------------- |
| public void write(byte b[ ])                   | 将参数b中的字节写到输出流                       |
| public void write(byte b[ ], int off, int len) | 将参数b的从偏移量off开始的len个字节写到输出流。 |
| public abstract void write(int b)              | 先将int转换为byte类型，把低字节写入到输出流中。 |
| public void flush( )                           | 将数据缓冲区中数据全部输出，并清空缓冲区。      |
| public void close( )                           | 关闭输出流并释放与流相关的系统资源。            |

主要的子类：

> 1) ByteArrayOutputStream：把信息存入内存中的一个缓冲区中 
>
> 2) FileOutputStream：把信息存入文件中 
>
> 3) PipedOutputStream：实现了pipe的概念，主要在线程中使用 
>
> 4) SequenceOutputStream：把多个OutStream合并为一个OutStream 
>

**注：流结束的判断：方法read()的返回值为-1时；readLine()的返回值为null时。**

### 3、FileInputStream文件输入流

如果调用`read(byte b[])`，在将整个文件读取完成或写入完毕的过程中，这么一个byte数组通常被当作缓冲区

```java
public class FileInputStreamDemo {
    public static void main(String[] args) throws IOException {
        FileInputStream fis = new FileInputStream("a.txt");
        int a;
        while ((a = fis.read()) != -1) {
            System.out.print((char)a);
        }
    }
}
```

输出来的应该是这个数据的 ASCII 码。



### 4、FileOutputStream文件输出流

一个表示文件名的字符串，也可以是File或FileDescriptor对象。

| 构造器                                        | 解释                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| FileOutputStream(File file)                   | 创建一个向指定 File 对象表示的文件中写入数据的文件输出流。 例： `File f=new File (“d:/myjava/write.txt “); FileOutputStream out= new FileOutputStream (f);` |
| FileOutputStream(File file, boolean append)   | 创建一个向指定 File 对象表示的文件中写入数据的文件输出流。 append表示内容是否追加 |
| FileOutputStream(FileDescriptor fdObj)        | 创建一个向指定文件描述符处写入数据的输出文件流，该文件描述符表示一 个到文件系统中的某个实际文件的现有连接。 |
| FileOutputStream(String name)                 | 创建一个向具有指定名称的文件中写入数据的输出文件流。 例：`FileOutputStream out=new FileOutputStream(“d:/myjava/write.txt “);` |
| FileOutputStream(String name, boolean append) | 创建一个向具有指定 name 的文件中写入数据的输出文件流。 append表示内容是否追加 |

>注：
>
>（1）文件中写数据时，若文件已经存在，则覆盖存在的文件；
>
>（2）读/写操作结束时，应调用close方法关闭流。

例子：

使用键盘输入一段内容，将内容保存在文件write.txt中

```java
public class FileOutputStreamDemo {
    public static void main(String[] args) throws IOException {
        FileOutputStream fos = new FileOutputStream("write.txt");
        int a;
        while ((a = System.in.read()) != -1) {
            fos.write(a);
        }
    }
}
```

从键盘读取的时候确实是一个字节一个字节读取，存的时候也是一个字节一个字节存，只不过它会加点标志（不同的编码集标记的方式可能不一样）。到时候输出的时候它就会根据标记和汉字所占字节数来拼接成汉字。

我们来输出看一下：

```java
public class FileInputStreamDemo {

    public static void main(String[] args) throws IOException {
        //哈：   229 147 136 13 10   
        //哈哈：  229 147 136 229 147 136 13 10  
        //哈哈哈： 229 147 136 229 147 136 229 147 136 13 10  
        //哈哈（两行）哈哈：229 147 136 229 147 136 13 10 229 147 136 229 147 136 13 10 
        FileInputStream fis = new  FileInputStream("a.txt");
        int a;
        while ((a = fis.read()) != -1) {
            System.out.print(a+" ");
        }
    }
}
```

可以发现标记一行的是13 10 这两个字节。不要问我这明明就是个整数不是占4个字节，为什么说是一个字节？read() 的返回值为 int，取出的 byte 高位补0成int。那么怎么拼接的呢？比如：内容为哈的时候，首先找到标记的位置，用（所占字节数-2）/3=汉字的个数，而每个汉字占三个字节，这不就可以三个三个字节拼接出来了吗。???

### 5、缓冲输入输出流 BufferedInputStream/ BufferedOutputStream（也称包装流）

计算机访问外部设备非常耗时。访问外存的频率越高，造成CPU闲置的概率就越大。为了减少访问外存的次数，应该在一次对外设的访问中，读写更多的数据。为此，除了程序和流节点间交换数据必需的读写机制外，还应该增加缓冲机制。

缓冲流就是每一个数据流分配一个缓冲区，一个缓冲区就是一个临时存储数据的内存。这样可以减少访问硬盘的次数,提高传输效率。

#### a、将文件读入内存：

将BufferedInputStream与FileInputStream相接

```java
FileInputStream in=new FileInputStream( “file1.txt “);
BufferedInputStream bin=new BufferedInputStream(in);
```

#### b、将内存写入文件：

将BufferedOutputStream与 FileOutputStream相接

```java
FileOutputStreamout=new FileOutputStream(“file2.txt”);
BufferedOutputStream bin=new BufferedInputStream(out);
```

#### c、键盘输入流读到内存

将BufferedReader与标准的数据流相接

```java
InputStreamReader sin=new InputStreamReader (System.in) ；
BufferedReader bin=new BufferedReader(sin);
```

例子：从键盘输入一串内容存到file1.txt文件中

```java
public class FileInputStreamDemo {

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        BufferedWriter bw = new BufferedWriter(new FileWriter(new File("file1.txt")));
        String s;
        while ((s = br.readLine()).length() > 0) {
            bw.write(s, 0, s.length());
        }
    }
}结果为空
```

包装流的一个特性即因为它是包装流拥有缓存区，每次的读取的数据都存在缓存区中，当缓存区满了的时候才会写入到硬盘上。但是默认的缓存区大小为8192个字节，当然你也可以指定缓存区的大小。显然刚才输入的字符串并没有装满。

指定缓存区的大小：包装流的构造器

如 BufferedWriter(Writer out, int size) 创建一个使用给定大小输出缓冲区的新缓冲字符输出流。size为缓存区的大小

所以就需要我们手动的去刷新缓冲区。调用包装流的 flush() 方法。这个方法的作用就是将缓存区的内容写入到硬盘上并清空缓存区。

当然能不能不写 flush（）方法也让它刷新呢？

答：可以，调用 close（) 方法关闭流即可。当你调用 close（）方法时它会先刷新缓存区。这就是刚才上面所说要记得用完之后要关闭流。不过最好有使用到包装流的时候两个方法都记得写上。



```
void write(char ch);//写入单个字符。
void write(char []cbuf,int off,int len)//写入字符数据的某一部分。
void write(String s,int off,int len)//写入字符串的某一部分。
void newLine()//写入一个行分隔符。
void flush();//刷新该流中的缓冲。将缓冲数据写到目的文件中去。
void close();//关闭此流，再关闭前会先刷新他。
```

### 6、Writer/Reader字符流

#### a、Reader抽象类

用于读取字符流的抽象类。子类必须实现的方法只有 read(char[], int, int) 和 close()。

子类简单介绍：

| 名称              | 构造器                                | 解释                                                         |
| :---------------- | ------------------------------------- | ------------------------------------------------------------ |
| BufferReader      | `BufferReader(Reader r);`             | 接受Reader对象作为参数，并对其添加字符缓冲器，使用readline()方法可以读取一行。 |
| CharArrayReader   | `CharArrayReader(char[], int, int)`   | 与ByteArrayInputStream对应                                   |
| StringReader      | `StringReader(String s);`             | 与StringBufferInputStream对应                                |
| InputStreamReader | ` inputstreamReader(inputstream is);` | 从输入流读取字节，再将它们转换成字符                         |
| FilterReader      | ` filterReader(Reader r);`            | 允许过滤字符流                                               |

主要方法：

| method                                         | 解释                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| int read()                                     | 读取一个字符，返回值为读取的字符                             |
| int read(char cbuf[])                          | 读取一系列字符到数组cbuf[]中，返回值为实际读取的字符的数量   |
| abstract int read(char cbuf[],int off,int len) | 读取len个字符，从数组cbuf[]的下标off处开始存放，返回值为实际读取的字符数量，该方法必须由子类实现 |

#### b、 Writer抽象类

写入字符流的抽象类。子类必须实现的方法仅有 write(char[], int, int)、flush() 和 close()。

主要方法：

| method                                  | 解释                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| void write(int c)                       | 将整型值c的低16位写入输出流                                  |
| void write(char cbuf[])                 | 将字符数组cbuf[]写入输出流                                   |
| void write(char cbuf[],int off,int len) | 将字符数组cbuf[]中的从索引为off的位置处开始的len个字符写入输出流 |
| void write(String str)                  | 将字符串str中的字符写入输出流                                |
| void write(String str,int off,int len)  | 将字符串str 中从索引off开始处的len个字符写入输出流           |
| void  flush( )                          | 刷空输出流，并输出所有被缓存的字节。                         |
| void close()                            | 关闭流                                                       |

### 7、PrintWriter 和 PrintStream（打印流）

#### a、PrintStream

是一个字节打印流，System.out对应的类型就是PrintStream。

| 构造器                                                       | 解释                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| PrintStream(File file)                                       | 创建具有指定文件的新打印流。                                 |
| PrintStream(File file, String csn)                           | 创建具有指定文件名称和字符集的新打印流。                     |
| PrintStream(OutputStream out)                                | 创建新的打印流                                               |
| PrintStream(OutputStream out, boolean autoFlush)             | 创建新的打印流。 autoFlush为是否自动刷新，默认为true，只在有包装流的时候才起作用 |
| PrintStream(OutputStream out, boolean autoFlush, String encoding) | 创建新的打印流。autoFlush为是否自动刷新，默认为true，只在有包装流的时候才起作用 |
| PrintStream(String fileName)                                 | 创建具有指定文件名称的新打印流。                             |
| PrintStream(String fileName, String csn)                     | 创建具有指定文件名称和字符集的新打印流。                     |

#### b、PrintWriter

是一个字符打印流。构造函数可以接收四种类型的值。

>1，字符串路径。
>
>2，File对象。
>
>3，OutputStream
>
>4，Writer

对于1，2类型的数据，还可以指定编码表。也就是字符集。

对于3，4类型的数据，可以指定自动刷新。

**注意：该自动刷新值为true时，只有三个方法可以用：println,printf,format.默认为 false ，需要手动刷新，对一般流和包装流都起作用。**

### 8、InputStreamWriter 和 OutputStreamReader（转换流）

#### a、InputStreamReader

InputStreamReader 将字节流转换为字符流。**是字节流通向字符流的桥梁**。如果不指定字符集编码，该解码过程将使用平台默认的字符编码，如：GBK。

构造方法：

| 构造器                                                | 解释                   |
| ----------------------------------------------------- | ---------------------- |
| InputStreamReader(InputStream in)                     | 默认字符集             |
| InputStreamReader(InputStream in, Charset cs)         | 使用给定字符集         |
| InputStreamReader(InputStream in, CharsetDecoder dec) | 使用给定字符集解码器的 |
| InputStreamReader(InputStream in, String charsetName) | 使用指定字符集         |

每次调用 InputStreamReader 中的一个 read() 方法都会导致从底层输入流读取一个或多个字节。

要启用从字节到字符的有效转换，可以提前从底层流读取更多的字节，使其超过满足当前读取操作所需的字节。

为了达到最高效率，可以考虑在 BufferedReader 内包装 InputStreamReader。

例如：

`BufferedReader in = new BufferedReader(new InputStreamReader(System.in));`

#### b、OutputStreamWriter

OutputStreamWriter 将字节流转换为字符流。是字节流通向字符流的桥梁。如果不指定字符集编码，该解码过程将使用平台默认的字符编码，如：GBK。

每次调用 write() 方法都会导致在给定字符（或字符集）上调用编码转换器。

在写入底层输出流之前，得到的这些字节将在缓冲区中累积。可以指定此缓冲区的大小，不过，默认的缓冲区对多数用途来说已足够大。

**注意**，传递给 write() 方法的字符没有缓冲。

为了获得最高效率，可考虑将 OutputStreamWriter 包装到 BufferedWriter 中，以避免频繁调用转换器。例如：

`Writer out = new BufferedWriter(new OutputStreamWriter(System.out));`

在转换流中是可以指定编码表的。

默认情况下，都是本机默认的码表GBK.

常见码表：

ascii:美国标准信息交换码。使用的是1个字节的7位来表示该表中的字符。

ISO8859-1:拉丁码表。使用1个字节来表示。

GB2312:简体中文码表。

GBK：简体中文码表，比GB2312融入更多的中文文件和符号。

unicode:国际标准码表。都用两个字节表示一个字符。

UTF-8：对unicode进行优化，每一个字节都加入了标识头。

编码转换：

字符串 –>字节数组 ：编码。通过getBytes(charset);

字节数组–>字符串： 解码。通过String类的构造函数完成。String(byte[],charset);

### 9、PipedInputStream 和 PipedOutputStream（管道流）

管道流，用于在应用程序中创建管道通信。一个线程的PipedInputStream对象从另外一个线程的PipedOutputStream对象读取输入。要使管道流有用，必须同时构造管道输入流和管道输出流。

连接：

```
1，通过两个流对象的构造函数。
2，通过两个对象的connect方法。
```

例子：

```java
public class Producer extends Thread{

    private PipedOutputStream pos;
    public Producer(PipedOutputStream pos) {
        this.pos = pos;
    }
    @Override
    public void run() {
        try {
            pos.write("世界你好".getBytes());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class Consumer extends Thread{

    private PipedInputStream pis;
    public Consumer(PipedInputStream pis) {
        this.pis = pis;
    }
    @Override
    public void run() {
        byte[] b = new byte[100];   //将读取的数据保存在这个字节数组中
        try {
            int len = pis.read(b);//返回数组的实际长度
            System.out.println(new String(b, 0, len));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class PipedStreamTest {

    public static void main(String[] args) throws IOException {

        PipedInputStream pis = new PipedInputStream();
        PipedOutputStream pos = new PipedOutputStream();
        pis.connect(pos);//连接管道
        new Producer(pos).start();
        new Consumer(pis).start();
    }
}
```

可以看到两线程之间进行了通信。

通常两个流在使用时，需要加入多线程技术，也就是让读写同时运行。

**注意**：对于read方法。该方法是阻塞式的，也就是没有数据的情况，该方法会等待。

### 10、SequenceInputStream（合并流）

有没有发现这个流只有一个，其他的流都是成对的。有些情况下，当我们需要从多个输入流中向程序读入数据。此时，可以使用合并流，将多个输入流合并成一个SequenceInputStream流对象。

其可接收枚举类所封闭的多个字节流对象。

特点：可以将多个读取流合并成一个流。这样操作起来很方便。

原理：其实就是将每一个读取流对象存储到一个集合中。它从输入流的有序集合开始，并从第一个输入流开始读取，直到到达文件末尾，接着从第二个输入流读取，依次类推，直到到达包含的最后一个输入流的文件末尾为止。

```java
public class SequenceInputStreamTest {

    public static void main(String[] args) {
        doSequence();
    }
    private static void doSequence() {
        //创建一个合并的流对象
        SequenceInputStream sis = null;

        //创建输出流
        BufferedOutputStream bos = null;
        try {
            //构建流集合
            Vector<InputStream> vector = new Vector<>();
            vector.addElement(new FileInputStream("a.txt"));
            vector.addElement(new FileInputStream("a1.txt"));
            vector.addElement(new FileInputStream("a2.txt"));
            Enumeration<InputStream> e = vector.elements();
            sis = new SequenceInputStream(e);
            bos = new BufferedOutputStream(new FileOutputStream("aa.txt"));

            //读写数据
            byte[] b = new byte[1024];
            int len;
            while ((len = sis.read(b)) != -1) {
                bos.write(b, 0, len);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e1) {
            e1.printStackTrace();
        } finally {
            if (sis != null) {
                try {
                    sis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (bos != null) {
                try {
                    bos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

### 11、DataInputStream 和 DataOutputStream（数据操作流）

DataInputStream和DataOutputStream二者分别实现了DataInput/DataOutput接口

DataInputStream能以一种与机器无关（当前操作系统等）的方式，直接从地从字节输入流读取JAVA基本类型和String类型的数据，常用于网络传输等（网络传输数据要求与平台无关）

DataOutputStream则能够直接将JAVA基本类型和String类型数据写入到其他的字节输入流。

例子：

```java
public class DataStreamTest {

    public static void main(String[] args) {

        FileOutputStream fos;
        try {
            fos = new FileOutputStream("b.data");
            DataOutputStream dos=new DataOutputStream(fos);  
            dos.writeInt(100);  
            dos.writeUTF("DataOutputStream Test");  
            dos.close();  

            FileInputStream fis=new FileInputStream("b.data");  
            DataInputStream dis=new DataInputStream(fis);  
            System.out.println("int:"+dis.readInt());  
            System.out.println("UTF:"+dis.readUTF());  
            dis.close();  
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }  
    }
}
```

结果：

产生一个b.data文件（此时已经不是文本文件，此时编码为JAVA虚拟机通用格式，即UTF-8）

**注**：当要求输入输出流必须遵循平台无关时，可以使用这两个类

### 12、ObjectInputStream 和 ObjectOutputStream（序列化和反序列化流）

保存并读取一个对象

#### a、ObjectOutputStream

是将一个对象的所有相关属性、信息（不包括方法）写入到底层流中、而DataOutputStream一次写入的只是一个java基础类型的数据。

```java
常用构造方法：
ObjectOutputStream oos = new ObjectOutputStream(OutputStream out);
//创建一个写入指定OutputStream的ObjectOutputStream对象.
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("F:\\obj.object"));

主要方法：
void writeObject(Object obj);
//将指定的对象写入ObjectOutputStream.对对象进行序列化。

```

它虽然不是实现了FilterOutputStream装饰类，但是实现了ObjectOut，而此接口实现了DataOut接口，并且对这个接口进行了扩展，使得ObjectOut在具有DataOut中定义的各种方法，同时也具有将对象、数组、字符串写入到底层字节流中的功能，这样也就意味着ObjectOutputStream同样具有DataOutputStream功能的同时也具有将对象、数组字符串写入到底层字节输出流中的功能。

当然ObjectOuputStream同样还实现了别的接口、因为他写入一个对象的时候、不仅仅写入的是标示这个Object的所有属性、同时还有额外的一些信息、比如版本号、作者等、但是这些对我们是透明的、具体的写入方法由JDK说了算。

**注意：被写入的对象必须实现Serializable接口或Externalizable接口。**

#### b、ObjectInputStream

一次读取一个对象、不必关心对象每个属性的写入顺序、而DataOutputStream读取时要严格按照写入时的顺序读取（当然、在使用skip方法时还要考虑字节数）

```java
常用构造方法：
ObjectInputStream ois = new ObjectInputStream(InputStream in);
//创建从指定 InputStream 读取的 ObjectInputStream。
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("F:\\obj.object"));

主要方法：
Object readObject();
//从ObjectInputStream读取对象。由于其返回的是Object类型 需要对其进行强转。
```

### 13、序列化

> **序列化**可以将一个java对象以二进制流的方式在网络中传输并且可以被持久化到数据库、文件系统中；
>
> **反序列化**则是可以把之前持久化在数据库或文件系统中的二进制数据以流的方式读取出来重新构造成一个和之前相同内容的java对象。

序列化的实现：

1)、需要序列化的类需要实现Serializable接口，该接口没有任何方法，只是标示该类对象可被序列化。

2）、序列化过程：使用一个输出流(如：FileOutputStream)来构造一个ObjectOutputStream(对象流)对象，接着，使用ObjectOutputStream对象的writeObject(Object obj)方法就可以将参数为obj的对象写出(即保存其状态)

3)、反序列化过程：使用一个输入流(如：FileInputStream)来构造一个ObjectInputStream(对象流)对象，接着，使用ObjectInputStream对象的readObject(Object obj)方法就可以将参数为obj的对象读出(即获取其状态)

例子：

Student.java

```java
/**
 * 继承Serializable接口用于给该标示一个ID号。该id号由虚拟机根据该类运算得出。 
 * 用于判断该类和obj对象文件是否为同一个版本。当不为同一个版本时将会抛出无效类异常。 
 * @author Administrator
 */
public class Student implements Serializable{
    /**
     * 由于在定义对象时需要定义一个静态final的long字段serialVersionUID。 
     * 如果不定义的话系统将根据该类各个方面进行计算该值，极容易导致读取异常。
     */
    private static final long serialVersionUID = -6205013588337175827L;
    //static 和 transient修饰符修饰的都不会被写入到对象文件中去。  
    //非瞬态和非静态字段的值都将被写入  
    private  String name;
    private  int age;
    public Student(String name, int age) {
        super();
        this.name = name;
        this.age = age;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    @Override
    public String toString() {
        return "Student [name=" + name + ", age=" + age + "]";
    }
}
```

ObjectStreamDemo.java

```java
public class ObjectStreamDemo {

    public static void main(String[] args) throws FileNotFoundException, IOException, ClassNotFoundException {
        write();
        read();
    }
    private static void read() throws FileNotFoundException, IOException, ClassNotFoundException {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("obj.object"));
        Object object = ois.readObject();
        System.out.println(object.toString());
        object = ois.readObject();
        System.out.println(object.toString());
        object = ois.readObject();
        System.out.println(object.toString());
    }
    private static void write() throws FileNotFoundException, IOException {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("obj.object"));
        //对象序列化
        oos.writeObject(new Student("周杰论", 10));
        oos.writeObject(new Student("wuli韬韬", 30));
        oos.writeObject(new Student("TF-BOYS", 15));
        oos.close();
    }
}
```

> `serialVersionUID`作用：
>
> 如果没有设置这个值，你在序列化一个对象之后，改动了该类的字段或者方法名之类的，那如果你再反序列化想取出之前的那个对象时就可能会抛出异常，
>
> 因为你改动了类中间的信息，serialVersionUID是根据类名、接口名、成员方法及属性等来生成一个64位的哈希字段。
>
> 当修改后的类去反序列化的时候发现该类的serialVersionUID值和之前保存的serialVersionUID值不一致，所以就会抛出异常。  
>
> 而显示的设置serialVersionUID值就可以保证版本的兼容性，如果你在类中写上了这个值，就算类变动了，它反序列化的时候也能和文件中的原值匹配上。
>
> 而新增的值则会设置成null，删除的值则不会显示。

总结：

1）、如果一个类可被序列化，其子类也可以，如果该类有父类，则根据父类是否实现Serializable接口，实现了则父类对象字段可以序列化，没实现，则父类对象字段不能被序列化。

2）、声明为transient类型的成员数据不能被序列化。transient代表对象的临时数据；

3）、当一个对象的实例变量引用其他对象，序列化该对象时也把引用对象进行序列化；

### 14、输入流和输出流的应用

**包装流更加高效**

![img](http://img.blog.csdn.net/20160807023459947)

这两个从硬盘上读取内容的时间应该差不多，但时间就差在写的时间上了，缓存区一般在内存，内存的速度是很快的，将数据写入到硬盘的速度是很慢的，所以我们只需要减少写入硬盘的次数即可。包装流的默认缓存区大小为8192字节，包装流读取8次才写一次，所以包装流的效率是大大增加了。

## 三、Java IO 的一般使用原则

一）按数据来源（去向）分类：

> .1 、是文件： FileInputStream, FileOutputStream, ( 字节流 )FileReader, FileWriter( 字符 )
>
> 2 、是 byte[] ： ByteArrayInputStream, ByteArrayOutputStream( 字节流 )
>
> 3 、是 Char[]: CharArrayReader, CharArrayWriter( 字符流 )
>
> 4 、是 String: StringBufferInputStream, StringBufferOuputStream ( 字节流 )StringReader, StringWriter( 字符流 )
>
> 5 、网络数据流： InputStream, OutputStream,( 字节流 ) Reader, Writer( 字符流 )
>

二）按是否格式化输出分：

>1 、要格式化输出： PrintStream, PrintWriter

三）按是否要缓冲分：

>1 、要缓冲： BufferedInputStream, BufferedOutputStream,( 字节流 ) BufferedReader, BufferedWriter( 字符流 )

四）按数据格式分：

>1 、二进制格式（只要不能确定是纯文本的，比如图片、音频、视频） : InputStream, OutputStream 及其所有带 Stream 结尾的子类
>
>2 、纯文本格式（含纯英文与汉字或其他编码方式）； Reader, Writer 及其所有带 Reader, Writer 的子类
>

五）按输入输出分：

>1 、输入： Reader, InputStream 类型的子类
>
>2 、输出： Writer, OutputStream 类型的子类
>

六）特殊需要：

>1 、从 Stream 到 Reader,Writer 的转换类： InputStreamReader, OutputStreamWriter
>
>2 、对象输入输出： ObjectInputStream, ObjectOutputStream
>
>3 、进程间通信： PipeInputStream, PipeOutputStream, PipeReader, PipeWriter
>
>4 、合并输入： SequenceInputStream
>
>5 、更特殊的需要： PushbackInputStream, PushbackReader, LineNumberInputStream, LineNumberReader

