





# 1. IO流

## 1.1 File类的使用

### File类的理解

1. File类的一个对象，代表一个文件或一个目录（俗称：文件夹）

2. File类声明在`java.io`包下

3. File类中涉及到关于文件或文件目录的创建，删除、重命名、修改时间、文件大小等方法

   并未涉及到写入或读取文件内容的操作。如果需要读取或写入文件内容，必须使用IO流来完成

4. 后续File类的对象常会作为参数传递到流的构造器中，指明读取或写入的“终点”。

### File的实例化

#### 构造器

![image-20211012095630962](source/Java-IO%E6%B5%81%E4%B8%8E%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B/image-20211012095630962.png)



```java
File file = new File(String filePath);
File file = new File(String parentPath, String childPath);
File file = new File(File parentFile, String childPath);
```



#### 常用方法

```java
public boolean isDirectory()：判断是否是文件目录
public boolean isFile() ：判断是否是文件
public boolean exists() ：判断是否存在
public boolean canRead() ：判断是否可读
public boolean canWrite() ：判断是否可写
public boolean isHidden() ：判断是否隐藏
public boolean createNewFile() ：创建文件。若文件存在，则不创建，返回false
public boolean mkdirs() ：创建文件目录。如果此文件目录存在，就不创建了。如果词文件目录的上层目录不存在， 也不创建。
public boolean mkdir() ：创建文件目录，如果上层文件目录不存在，一并创建
public boolean delete() ：删除文件或者文件夹（直接删除，不走回收站）
//  分隔符常量
File.separator  用于路径的分隔符  /* unix：/   windows： \\ */
```





## 1.2 IO流概述

- I/O是Input/Output的缩写，I/O技术是非常实用的技术，用于处理设备之间的数据传输。如读/写文件、网络通讯等。

- Java程序中，对于数据的输入/输出操作以“流Stream”的方式进行。
- java.io包下提供了各种“流”类和接口，用以获取不同种类的数据，并通过标准的方法输入或输出数据。
- 输入Input：读取外部数据（磁盘，光盘等存储设备的数据）到程序（内存）中
- 输出output：将程序（内存）数据输出到磁盘、光盘等存储设备中



**流的分类**

- 按操作数据单位不同分为：**字节流（8bit）**、**字符流（16bit）**
- 按数据流的流向不同分为：输入流、输出流
- 按数据流的角色不同分为：节点流、处理流

| 抽象基类 | 字节流       | 字符流                     |
| -------- | ------------ | -------------------------- |
| 输入流   | InputStream  | Reader（往内存中读取数据） |
| 输出流   | OutputStream | Writer（将内存中数据写出） |

Java的IO流涉及40多个类，实际上非常规则，都是从上述4个抽象基类派生的；由这四个类派生出来的子类名称都是其父类名作为子类名后缀。



**IO流的体系**

| 分类                 | 字节输入流           | 字节输出流            | 字符输入流        | 字符输出流         |
| -------------------- | -------------------- | --------------------- | ----------------- | ------------------ |
| 抽象基类             | InputStream          | OutputStream          | Reader            | Writer             |
| 访问文件（节点流）   | FileInputStream      | FileOutputStream      | FileReader        | FileWriter         |
| 访问数组             | ByteArrayInputStream | ByteArrayOutputStream | CharArrayReader   | CharArrayWriter    |
| 访问管道             | PipedInputStream     | PipedOutputStream     | PipedReader       | PipedWriter        |
| 访问字符串           |                      |                       | StringReader      | StringWriter       |
| 缓冲流(处理流的一种) | BufferedInputStream  | BufferedOutputStream  | BufferedReader    | BufferedWriter     |
| 转换流               |                      |                       | InputStreamReader | OutputStreamWriter |
| 对象流               | ObjectInputStream    | ObjectOutputStream    |                   |                    |
|                      | FilterInputStream    | FilterOutputStream    | FilterReader      | FilterWriter       |
| 打印流               |                      | PrintStream           |                   | PrintWriter        |
| 推回输入流           | PushbackinputStream  |                       | PushbackReader    |                    |
| 特殊流               | DataInputStream      | DataOutputStream      |                   |                    |







## 1.3 文件流

### 1.3.1 FileReader读入数据的基本操作

**四步骤：**

1. 建立一个流对象，将已存在的一个文件加载进流

   - ```java
     FileReader fr = new FileReader(new File("Test.txt"));
     ```

2. 创建一个临时存放数据的数组

   - ```java
     char[] ch = new char[1024];
     ```

3. 调用流对象对象的读取方法将流中的数据读入到数组中

   - ```java
     fr.read(ch);
     ```

4. 关闭资源

   - ```java
     fr.close();
     ```

     

```java




```















## 1.4 缓冲流的使用





## 1.5 转换流的使用







## 1.6 其他的流的使用





## 1.7 对象流的使用





## 1.8 RandomAccessFile的使用





## 1.9 Path、Paths、Files的使用











# 2. 网络编程







