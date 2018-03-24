---------------
title: JAVA IO
---------------
date: 2018-03-20 14:53:55
tags: IO 流
categories: JAVA

### Description
JAVA IO 也成为 IO 流，它的核心就是对文件的操作，对于字节，字符类型的输入和输出流。   

主要类或者接口：
1. File -- 文件夹
2. RandomAccessFile -- 随机存储文件类
3. InputStream -- 字节输入流
4. OutputStream -- 字节输出流
5. Reader -- 字符输入流
6. Writer -- 字符输出流   

结构图：   
![java流类结构图](/picture/java-io.jpg)

#### 流的概念和作用
流是一组有顺序的，有起点和终点的字节集合，是对数据传输的总称或抽象。数据在设备间进行传输称之为流。流的本质是数据传输，根据传输的特性将流分为各种类，方便更直观的进行数据操作。   

#### IO 流的分类
- 根据数据处理的不同类型分为：字节流和字符流
- 根据数据流向不同分为：输入流和输出流

#### 字节流和字符流
字符流的由来：因为数据编码的不同，而有了对字符进行高效操作的流对象，本质上其实就是对于字节流的读取时，去查了指定的码表。
   
字节流和字符流的区别：
- 读写单位的不同：字节流以字节（8bit）为单位。字符流以字符为单位，根据码表映射字符，一次可能读多个字节。
- 处理对象不同：字节流可以处理任何类型的数据，如图片、avi等，而字符流只能处理字符类型的数据。   

#### 输入流和输出流
输入流只能进行读操作，输出流只能进行写操作。

### Java IO 流对象
#### 1.输入字节流 InputStream
1. InputStream是所有数据字节流的父类，它是一个抽象类。
2. ByteArrayInputStream、StringBufferInputStream、FileInputStream是三种基本的介质流，它们分别从Byte数组、StringBuffer、和本地文件中读取数据，PipedInputStream是从与其他线程共用的管道中读取数据。
3. ObjectInputStream和所有FileInputStream 的子类都是装饰流。   

示例：
   ```
    public static void main(String[] args) throws IOException {
        //内存中的字节数组
        byte[] bArr = new byte[]{1,2,3};
                      
        //字节输入流
        InputStream in = new ByteArrayInputStream(bArr);
                   
        byte[] bff = new byte[3];
        //从输入流中读取字节
        in.read(bff,0,3);
        System.out.println(bff[0] + "," + bff[1] + "," + bff[2]);  
        
        in.close();
    
    }
   ```
#### 2.输出字节流 OutputStream
1. OutputStream是所有输出字节流的父类，它是一个抽象类。
2. ByteArrayOutputStream、FIleOutputStream是两种基本的介质，它们分别向Byte 数组，和本地文件中写入数据。PipedOutputStream是从与其他线程共用的管道中写入数据。
3. ObjectOutputStream和所有FileOutputStream的子类都是装饰流。

示例：
   ```
    public static void main(String[] args) throws IOException {
        //内存中的字节数组
        byte[] bArr = new byte[]{1,2,3};
                      
        //字节输出流
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
                   
        byte[] bff = new byte[3];
        //往字节输出流中写入字节数组
        bos.write(bff);
        //从输出流中获取字节数组
        byte[] bArryFromOs = bos.toByteArray();
        System.out.println(bArryFromOs[0] + "," + bArryFromOs[1] + "," + bArryFromOs[2]); 
        
        bos.close();
    
    }
   ```
#### 3.字符输入流 Reader
1. Reader是所有的输入字符流的父类，它是一个抽象类。
2. CharReader、StringReader 是两种基本的介质流，它们分别将Char数组、String中读取数据。PipedInputReader 是从与其他线程共用的管道中读取数据。
3. BufferedReader 很明显的是一个装饰器，它和其子类复制装饰其他Reader对象。
4. InputStreamReader 是一个连接字节流和字符流的桥梁，它将字节流转变为字符流。

示例：
   ```
    public static void main(String[] args) throws IOException  {
            //读取本地文件，得到字符流
            Reader reader = new FileReader("C:\\test.txt");
            //读取一个字符并打印
            System.out.println((char)reader.read());
            //关闭流
            reader.close();
        }
   ```
#### 4.字符输出流 Writer
1. Writer 是所有输出字符流的父类，它是一个抽象类。
2. CharArrayWriter、StringWriter 是两种基本的介质流，它们分别向Char 数组、String 中写入数据。PipedInputWriter 是从与其他线程共用的管道中读取数据。
3. BufferedWriter 很明显是一个装饰器，他和其子类复制装饰其他Reader对象。
4. FilterWriter 和PrintStream 及其类似，功能和使用也非常相似。
5. OutputStreamWriter 是OutputStream 到Writer 转换到桥梁，它的子类FileWriter 其实就是一个实现此功能的具体类,功能和使用OutputStream极其类。

示例：
   ```
    public class WriterDemo{
        public static void main(String args[]) throws Exception{  
            // 第1步、使用File类找到一个文件
            File f = new File("d:" + File.separator + "test.txt") ;    
            // 第2步、通过子类实例化父类对象
            Writer out = null ; 
            //实例化 
            out = new FileWriter(f)  ;      
            // 第3步、进行写操作
            String str = "Hello World!!!" ;  
            // 将内容输出，保存文件
            out.write(str) ;                        
            // 第4步、关闭输出流
            out.close() ;                      
        }
    };
   ```