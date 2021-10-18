





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


**使用read()方法一个字符一个字符的读取**

```java
import org.junit.Test;

import java.io.File;
import java.io.FileReader;
import java.io.IOException;

public class IoDemo {

    @Test
    public void test1() throws IOException {
        File file = new File("Files\\file1.txt");
        FileReader fileReader = null;
        char[] chars = new char[1024];
        try{
            fileReader = new FileReader(file);
            int read = fileReader.read();
            while (read!=-1){
                System.out.print((char)read);
                read = fileReader.read();
            }
        }catch (IOException e){
            e.printStackTrace();
        }finally {
            fileReader.close();
        }

    }
}
// 输出结果 
asdfsdfasdfsdf
dsfsd
cvc
qwref
```

**使用read(char[] chars) 方法读入数据**

```java
@Test
public void readDemo2() throws IOException {
    File file = new File("Files\\file1.txt");
    FileReader fr = null;
    char[] chars = new char[20];
    try {
        fr = new FileReader(file);
        int len=0;
        while((len=fr.read(chars))!=-1){

            System.out.println(len);
            System.out.println(new String(chars,0,len));
        }
    }catch (Exception e)
    {
        e.printStackTrace();
    }finally {
        fr.close();
    }
}
```

### 1.3.2 FileWriter写出数据的操作

```java
@Test
public void writterDemo() throws IOException {
    File file = new File("Files\\fileOutput.txt");
    File file2 = new File("Files\\fileOutput2.txt");
    // 默认 每次 write 覆盖原有文件中的内容
    FileWriter fw = new FileWriter(file,false);
    // 指定为true时， 每次write 表示追加内容到文件中
    FileWriter fw2 = new FileWriter(file2,true);
    fw.write("hello!");
    fw2.write("hello2");
    fw.close();
    fw2.close();

    // 默认 每次 write 覆盖原有文件中的内容
    fw = new FileWriter(file,false);
    // 指定为true时， 每次write 表示追加内容到文件中
    fw2 = new FileWriter(file2,true);
    fw.write("hello!");
    fw2.write("hello2");
    fw.close();
    fw2.close();
}
```

![image-20211018141242195](source/Java-IO流与网络编程/image-20211018141242195-16345375636911.png)



### 1.3.3 FileReader 和 FileWriter实现文本复制

小需求：要求将文件`file1.txt`中的内容复制到另一个文件`file2.txt`中

```java
@Test
public void copyContextToOther() throws IOException{
    String filePath1 = "Files\\file1.txt";
    String filePath2 = "Files\\file2.txt";
    File src = new File(filePath1);
    File target = new File(filePath2);
    // 将 file1.txt 中的内容读到内存中
    FileReader fr = new FileReader(src);
    // 将读取的数据临时存放到buffer中
    char[] buffer = new char[5];
    // 将file1.txt 中的内容写到file2.txt中
    FileWriter fw = new FileWriter(target);
    int len = 0;
    while((len = fr.read(buffer))!=-1){
        fw.write(buffer,0,len);
    }
    fw.close();
    fr.close();
}
```

![image-20211018143034994](source/Java-IO流与网络编程/image-20211018143034994-16345386387512.png)

### 1.3.4 FileInputStream和FileOutputStream不能读取文本文件

**tip:**

**FileInputStream**和**FileOutputStream**不能操作像`.java`,`.cpp`,`.txt`,`.c`等文本文件，操作文本文件使用**FileReader**和**FileWriter**字符流；

如果是`.jpg`,`.mp4`,`.mp3`等文件使用**FileInputStream**和**FileOutputStream**字节流操作

<font color=green>使用字节流**FileInputStream**和**FileOutputStream**操作文本文件，有可能出现乱码的问题</font>

```java
// 使用字节流操作文本文件
    //使用字节流FileInputStream处理文本文件，可能出现乱码。
    @Test
    public void testFileInputStream(){
        FileInputStream fis = null;
        try {
            //1.造文件
            File file = new File("Files\\file1.txt");

            //2.造流
            fis = new FileInputStream(file);

            //3.读数据
            byte[] buffer = new byte[5];
            int len;//记录每次读取的字节的个数
            while((len = fis.read(buffer)) != -1){
                String str = new String(buffer,0,len);
                System.out.print(str);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(fis != null) {
                //4.关闭资源
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

使用**FileInputStream**和**FileOutputStream**实现文件的复制

```java
@Test
public void copyByte() throws IOException{
    File file1 = new File("Files\\卡通 - 4.jpg");
    File file2 = new File("Files\\卡通 - 5.jpg");
    FileInputStream fis = new FileInputStream(file1);
    byte[] buffer = new byte[5];
    FileOutputStream fos = new FileOutputStream(file2);
    int len=0;
    while ((len=fis.read(buffer))!=-1){
        fos.write(buffer,0,len);
    }
    fos.close();
    fis.close();
}
```

![image-20211018145332931](source/Java-IO流与网络编程/image-20211018145332931-16345400144173.png)







## 1.4 缓冲流的使用

![img](source/Java-IO流与网络编程/7ce79819b9685ffc2bae685bec83b0f1.png)



- 缓冲流要`套接`在相应的节点流之上，根据数据操作单位可以把缓冲流分为：
  - **BufferedInputStream**和**BufferedOutputStream**
  - **BufferedReader**和**BufferedWriter**
- 当读取数据时，数据按块读入缓冲区，其后的读操作则直接访问缓冲区
- 当使用BufferedInputStream读取字节文件时，BufferedInputStream会一次性从文件中读取8192（8kb）,存在缓冲区中，直到缓冲区装满了，才重新从文件中读取下一个8kb的字节数组。
- 向流中写入字节时，不会直接写到文件，先写到缓冲区中直到缓冲区写满，BufferedOutputStream才会把缓冲区中的数据一次性写到文件中。使用`flush()`可以强制将缓冲区的内容全部写入到输出流中。
- 关闭流的顺序和打开流的顺序相反。只要关闭最外层流即可，关闭最外层流也会相应关闭内层节点流
- flush()方法的使用：手动将buffer中内容写入到文件
- 如果是带缓冲区的流对象的close()方法，不但会关闭流，还会在关闭流之间刷线缓冲区，关闭后不能再写出。





## 1.5 转换流的使用

### 1.5.1 转换流概述与InputStreamReader的使用

- 转换流提供了在字节流和字符流之间的转换
- Java API提供了两个转换流：
  - `InputStreamReader`：将InputStream转换成Reader
    - 实现将字节的输入流按指定字符集转换成字符的输入流
    - 需要和`InputStream`套接
    - 构造器
      - `public InputStreamReader(InpurStream in)`
      - `public InputStreamReader(InpurStream in, String charsetName)`
      - 例如：`Reader isr=new InpurStreamReader(System.in, "gbk")`
  - `OutputStreamWriter`：将Writer转换为OutputStream
    - 实现将字符的输出流按指定字符集转换为字节的输出流
    - 需要和`OutputStream`套接
    - 构造器
      - `public OutputStreamWriter(OutputStream out)`
      - `public OutputStreamWriter(OutputStream out, String charsetName)`
  - 字节流中的数据都是字符时，转换成字符流操作更高效
  - 很多时候我们使用转换流来处理文本乱码问题。实现编码和解码的功能。

![img](https://img-blog.csdnimg.cn/img_convert/4dfa85f71b54ed2d8e3a43371933b7fb.png)

### 1.5.2 转换流实现文件的读入和写出

**InputStreamReader**：将一个字节的输入流转换为字符的输入流

**OutputStreamWriter**：将一个字符的输出流转换为字节的输出流

```java
@Test
public void transformation() throws IOException {
    FileInputStream inputStream = new FileInputStream(new File("Files\\file1.txt"));
    FileOutputStream outputStream = new FileOutputStream(new File("Files\\file3.txt"));
    // 将字节流转换成字符流
    InputStreamReader isr = new InputStreamReader(inputStream);
    // 将字符流转换成字节输出流
    OutputStreamWriter osw = new OutputStreamWriter(outputStream);
    // 用字符数组承接字节流转换成字符流的数据
    char[] chars = new char[5];
    int len = 0;
    while ((len=isr.read(chars))!=-1){
        // 将字符流通过转换流写出到字节流FileOutputStream中
        osw.write(chars,0,len);
    }
    osw.close();
    isr.close();
}
```









## 1.6 其他的流的使用

### 1.6.1 标准输入、输出流

- System.in 和 System.out 分别代表了系统标准的输入和输出设备
- 默认输入设备是键盘；输出设备是：显示器
- System.in 的类型方式InputStream
- System.out的类型是PrintStream，其是OutputStream的子类FilterOutputStream的子类
- 重定向：通过System类的setln，setOut方法对默认设备进行改变。
  - `public static void setln(InputStream in)`
  - `public static void setOut(PrintStream out)`

### 1.6.2 打印流

- 实现将基本数据类型的数据格式转化为字符串输出
- 打印流：PrintStream和PrintWriter
  - 提供了一系列重载的print()和println()方法，用于多种数据类型的输出
  - PrintStream和PrintWriter的输出不会抛出IOException异常
  - PrintStream和PrintWriter有自动flush功能
  - PrintStream打印的所有字符都使用平台的默认字符编码转换为字节。在需要写入字符而不是写入字节的情况下，应该使用PrintWriter类。
  - System.out返回的是PrintStream的实例



### 1.6.3 数据流

- 为了方便地操作Java语言的基本数据类型和String的string数据，可以使用数据流
- 数据流有两个类：（用于读取和写出基本数据类型、String类的数据）
  - DataInputStream和DataOutputStream
  - 分别“套接”在InputStream和OutputStream













## 1.7 对象流的使用





## 1.8 RandomAccessFile的使用





## 1.9 Path、Paths、Files的使用











# 2. 网络编程







