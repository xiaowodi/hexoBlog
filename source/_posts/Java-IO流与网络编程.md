





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

**数据流**：

- DataInputStream：数据输入流
- DataOutputStream：数据输出流

```java
@Test
public void dataStreamDemoWrite() throws IOException{
    // 数据流 可以读取特定类型的数据，可以写特定的数据类型的数据

    // 输出
    DataOutputStream dos = new DataOutputStream(new FileOutputStream(new File("Files\\data.txt")));
    dos.writeChar('c');
    dos.writeBoolean(true);
    dos.writeDouble(2.0798654);
    dos.writeLong(123456789L);
    dos.flush();
}
@Test
public void dataStreamDemoReade() throws IOException{
    DataInputStream dis = new DataInputStream(new FileInputStream(new File("Files\\data.txt")));
    char c = dis.readChar();
    boolean b = dis.readBoolean();
    double v = dis.readDouble();
    long l = dis.readLong();
    System.out.println(c+"\t"+b+"\t"+v+"\t"+l+"\t");
}
```





## 1.7 对象流

### 1.7.1 对象序列化机制的理解

- `ObjectInputStream`和`ObjectOutputStream`
- 用于存储和读取基本数据类型数据或对象的处理流。它的强大之处就是把Java中的对象写入到数据源中，也能把对象从数据源中还原回来。
- 序列化：用`ObjectInputStream`类读取基本类型数据或对象的机制
- 反序列化：用`ObjectInputStream`类读取基本类型数据或对象的机制
- `ObjectOutputStream`和`ObjectInputStream`不能序列化`static`和`transient`修饰的成员变量
- 对象序列化机制允许把内存中的Java对象转换成平台无关的二进制流，从而允许把这种二进制流持久地保存在磁盘上，或通过网络将这种二进制流传输到另一个网络节点。当其他程序获取这种二进制流，就可以恢复原来的Java对象。
- 序列化的好处在于可将任何实现了`Serializable`接口的对象转化为字节数据，使其在保存和传输时刻被还原
- 序列化是RMI（Remote Method Invoke-远程方法调用）过程的参数和返回值都必须实现的机制，而RMI是JavaEE的基础。因此序列化机制是JavaEE平台的基础
- 如果需要让某个对象支持序列化机制，则必须让对象所属的类及其属性是可序列化的，为了让某个类是可序列化的，该类必须实现如下两个接口之一。否则，会抛出`NotSerializableException`异常
  - `Serializable`
  - `Externalizable`



### 1.7.2 对象流序列化与反序列化字符串操作

定义一个Person类，并实现`Serializable`接口，实现系列化功能，

通过`ObjectOutputStream`将Person对象写入到`person.dat`文件中，然后通过`ObjectInputStream`将person读回内存。

```java
class Person implements Serializable{
    private String name;
    private int age;
    public Person(String name, int age){
        this.name=name;
        this.age=age;
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
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

/**
     * 序列化与反序列化
     *  实现Serializable接口，即可将对象通过ObjectOutputStream存储到磁盘中
     *                              通过ObjectInputStream读取到内存中
     */
@Test
public void serializableDemo() throws IOException, ClassNotFoundException {
    ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(new File("Files\\person.dat")));
    Person person = new Person("yu", 25);
    oos.writeObject(person);

    ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File("Files\\person.data")));
    Person person1 = (Person)ois.readObject();
    System.out.println(person1);

}
```

### 1.7.3 serialVersionUID的理解

- 凡是实现`Serializable`接口的类都有一个表示序列化版本标识符的静态变量
  - `private static final long serialVersionUID`
  - `serialVersionUID`用来表明类的不同版本间的兼容性。简言之，其目的是以序列化对象进行版本控制，有关各版本反序列化时是否兼容
  - 如果类没有显示定义这个静态常量，它的值是Java运行时环境根据类的内部细节自动生成的。若类的实力变量做了修改，`serialVersionUID`可能发生变化，故建议显示声明
- 简单来说，Java的序列化机制是通过在运行时判断类的`serialVersionUID`来验证版本一致的。在进行反序列化时，JVM会把传来的字节流中的`serialVersionUID`与本地相应实体类的`serialVersionUID`进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常（InvalidCastException）



如果想要一个类的对象可序列化，除了实现`Serializable`接口之外，还必须保证其内部所有的属性也必须是可序列化的。（默认情况下，基本数据类型都可序列化）

一般情况下需要自定义`serialVersionUID`



> **序列化机制**
>
> 对象序列化机制允许把内存中的Java对象转换成平台无关的二进制流，从而允许把这种二进制流持久地保存在磁盘上，或通过网络将这种二进制流传输到另一个网络节点。当其他程序获取了这种二进制流，就可以恢复成原来的Java对象。



## 1.8 RandomAccessFile(随机存取文件流)的使用

- `RandomAccessFile`声明在`java.io`包下，但直接继承于`java.lang.Object`类。并且它实现了`DataInput、DataOutput`这两个接口，也就意味着这个类**既可以读也可以写**
- `RandomAccessFile`类支持“随机访问”的方式，程序可以直接跳到文件的任意地方来读、写文件
  - 支持只访问文件的部分内容
  - 可以向已存在的文件后追加内容
- `RandomAccessFile`对象包含一个记录指针，用以表示单签读写处的位置。`RandomAccessFile`类对象可以自己移动记录指针：
  - `long getFilePointer()`
  - `void seek(long pos)`
- 构造器
  - `public RandomAccessFile(File file, String mode)`
  - `public RandomAccessFile(String name, String mode)`
- 创建RandomAccessFile类实例需要指定一个mode参数，该参数指定RandomAccessFile的访问模式：
  - `r`：以只读方式打开
  - `rw`：打开以便读取和写入
  - `rwd`：打开以便读取和写入；同步文件内容的更新
  - `rws`：打开以便读取和写入；同步文件内容和元数据的更新
- 如果模式为只读`r`，则不会创建文件，而是会去读取一个已经存在的文件，如果读取的文件不存在则会出现异常。如果模式为`rw`读写，如果文件不存在则去创建文件，如果存在则不会创建。

```java
import org.junit.Test;

import java.io.File;
import java.io.IOException;
import java.io.RandomAccessFile;

/**
 * RandomAccessFile的使用
 * 1.RandomAccessFile直接继承于java.lang.Object类，实现了DataInput和DataOutput接口
 * 2.RandomAccessFile既可以作为一个输入流，又可以作为一个输出流
 * 3.如果RandomAccessFile作为输出流时，写出到的文件如果不存在，则在执行过程中自动创建。
 *   如果写出到的文件存在，则会对原有文件内容进行覆盖。（默认情况下，从头覆盖）
 */
public class RandomAccessFileTest {

    @Test
    public void test(){

        RandomAccessFile raf1 = null;
        RandomAccessFile raf2 = null;
        try {
            raf1 = new RandomAccessFile(new File("爱情与友情.jpg"),"r");
            raf2 = new RandomAccessFile(new File("爱情与友情1.jpg"),"rw");

            byte[] buffer = new byte[1024];
            int len;
            while((len = raf1.read(buffer)) != -1){
                raf2.write(buffer,0,len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(raf1 != null){
                try {
                    raf1.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
            if(raf2 != null){
                try {
                    raf2.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }

    @Test
    public void test2() throws IOException {

        RandomAccessFile raf1 = new RandomAccessFile("hello.txt","rw");

        raf1.write("xyz".getBytes());

        raf1.close();

    }

}
```

RandomAccessFile实现数据的插入（seek()方法会重置偏移量）

```java
// 
import org.junit.Test;

import java.io.File;
import java.io.IOException;
import java.io.RandomAccessFile;

/**
 * RandomAccessFile的使用
 * 1.RandomAccessFile直接继承于java.lang.Object类，实现了DataInput和DataOutput接口
 * 2.RandomAccessFile既可以作为一个输入流，又可以作为一个输出流
 * 3.如果RandomAccessFile作为输出流时，写出到的文件如果不存在，则在执行过程中自动创建。
 *   如果写出到的文件存在，则会对原有文件内容进行覆盖。（默认情况下，从头覆盖）
 *
 * 4.可以通过相关的操作，实现RandomAccessFile“插入”数据的效果
 */
public class RandomAccessFileTest {

    /**
     * 使用RandomAccessFile实现数据的插入效果
     */
    @Test
    public void test3() throws IOException {
        RandomAccessFile raf1 = new RandomAccessFile("hello.txt","rw");

        raf1.seek(3);//将指针调到角标为3的位置
        //保存指针3后面的所有数据到StringBuilder中
        StringBuilder builder = new StringBuilder((int) new File("hello.txt").length());
        byte[] buffer = new byte[20];
        int len;
        while((len = raf1.read(buffer)) != -1){
            builder.append(new String(buffer,0,len)) ;
        }
        //调回指针，写入“xyz”
        raf1.seek(3);
        raf1.write("xyz".getBytes());

        //将StringBuilder中的数据写入到文件中
        raf1.write(builder.toString().getBytes());

        raf1.close();

        //思考：将StringBuilder替换为ByteArrayOutputStream
    }

}
```



## 1.9 Path、Paths、Files的使用











# 2. 网络编程

### 2.1 网络编程概述

- Java是Internet上的语言，它从语言级上提供了对网络应用程序的支持，程序员能够很容易开发常见的网络应用程序。
- Java提供的网络类库，可以实现无痛的网络连接，联网的底层细节被隐藏在Java的本机安装系统中，有JVM进行控制。并且Java实现了一个跨平台的网络库，**程序面对的是一个统一的网络编程环境**。
- 网络编程的目的：直接或间接地通过网络协议与其它计算机实现数据交换，进行通讯
- 网络编程中有两个主要的问题：
  - 如果准确地定位网络上一台或多台主机；定位主机上的特定的应用
  - 找到主机后如何可靠高效地进行数据传输

### 2.2 网络通信要素概述

- 通信双方地址
  - IP
  - 端口号
- 一定的规则（即：网络通信协议。有两套参考模型）
  - OSI参考模型：模型过于理想化，未能在因特网上进行广泛推广
  - TCP/IP参考模型（或TCP/IP协议）：实事上的国际标准
- 网络通信协议

![img](https://img-blog.csdnimg.cn/img_convert/09bdce6b569fe3cf80206cc704dbf7cb.png)

![img](source/Java-IO流与网络编程/e769997523201f9fa4f83e501931e6e1.png)

> 网络编程中有两个主要的问题：
>
> 1. 如何准确地定位网络上一台或多台主机；定位主机上的特定的应用
> 2. 找到主机后如何可靠高效地进行数据传输
>
> 网络编程中的两个要素：
>
> 1. 对应问题1：IP和端口号
> 2. 对应问题2：提供网络通信协议：TCP/IP参考模型（应用层、传输层、网络层、物理+数据链路层）

### 2.3 通信要素1：IP和端口号

#### 2.3.1 IP的理解与InetAddress类的实例化

- IP地址：`InetAddress`
  - 唯一的表示Internet上的计算机（通信实体）
  - 本地回环地址（hostAddress）：127.0.01 主机名（hostName）：localhost
  - IP地址分类方式1：IPV4和IPV6
    - IPV4：由4个字节组成
    - IPV6：由16个字节组成
  - IP地址分类方式2：公网地址（万维网使用）和私有地址（局域网使用）。`192.168. . `开头的就是私有地址，范围即为`192.168.0.0 ~ 192.168.255.255`，专门为组织内部使用。
- Internet上的主机有两种方式表示：
  - 域名（hostName）：www.baidu.com
  - IP地址（hostAddress）：220.181.38.149

```java
@Test
public void test1() throws UnknownHostException {
    InetAddress inetAddress = InetAddress.getByName("www.baidu.com");
    System.out.println(inetAddress);
    // 获取本地ip
    System.out.println(InetAddress.getLocalHost());
    // 获取主机名
    System.out.println(inetAddress.getHostName());
}

//输出
www.baidu.com/220.181.38.149
DESKTOP-AQE9VH2/192.168.70.1
www.baidu.com
```

#### 2.3.2 端口号的理解

- 端口号表示正在计算机上运行的进程（程序）
  - 不同的进程有不同的端口号
  - 被规定为一个16位的整数`0~65535`
  - 端口分类：
    - 公认端口：`0~1023`.被预先定义的服务通信占用（如：HTTP占用端口80，FTP占用端口21，Telnet占用端口23）
    - 注册端口：`1024~49151`.分配给用户进程或应用程序。（如：Tomcat占用端口8080，MySQL占用端口3306，Oracle占用端口1521等）。
    - 动态/私有端口：49152~65535.
- 端口号与IP地址的组合得出一个网络套接字：`Socket`

![img](https://img-blog.csdnimg.cn/img_convert/cdbda7a9127a6319dcd57cc16be48849.png)



### 2.4 通信要素2：网络协议

- **网络通信协议**

  计算机网络中实现通信必须有一些约定，即通信协议，对速率，传输代码，代码结构，传输控制步骤，出错控制等制定标准。

- **问题：网络协议太复杂**

  计算机网络通信设计内容很多，比如指定源地址和目标地址，加密解密，压缩解压缩，差错控制，流量控制，路由控制等

- **通信协议分层的思想**

  在制定协议时，把复杂问题分解成一些简单的成分，再将他们复合起来。最常用的复合方式使层次方式，即同层间可以通信，上一层可以调用下一层，而与再下一层不发生关系。各层互不影响，利于系统的开发和扩展。



#### 2.4.1 TCP和UDP网络通信协议的对比

- 传输层协议中有两个非常重要的协议：
  - 传输控制协议TCP
  - 用户数据报协议UDP
- TCP/IP以其两个主要协议：传输控制协议（TCP）和网络互联协议（IP）而得名，实际上是一组协议，包括多个具有不同功能互为关联的协议。
- IP协议是网络层的主要协议，支持网络间互连的数据通信
- TCP/IP协议模型从更实用的角度出发，形成了高效的四层体系结构，即物理层、数据链路层、IP层、传输层T、应用层。
- TCP协议：
  - 使用TCP协议前，需先建立TCP连接，形传输数据通道
  - 传输前，采用“三次握手”方式，点对点通信，是可靠的
  - TCP协议进行通信的两个应用进程：客户端、服务端
  - 在连接中进行大数据量的传输，传输完毕，需释放已建立的连接，效率低
- UDP协议：
  - 将数据、源、目的封装成数据包，不需要建立连接
  - 每个数据报的大小限制在64K内
  - 发送不管对方是否准备好，接收方收到也不确认，故是不可靠的
  - 可以广播发送
  - 发送数据结束时无需释放资源，开销小，速度快。

![img](https://img-blog.csdnimg.cn/img_convert/e028a11c223c41376e42a0cecd7dc505.png)

![img](https://img-blog.csdnimg.cn/img_convert/3125b3b4a4431912fa787bbeaea342b6.png)

### 2.5 TCP网络编程

```java
// Client端 向 服务器端 发送数据
@Test
public void client() throws IOException {
    // client 端 发送消息
    Socket socket = null;
    OutputStream os = null;
    try{
        // 1. 创建Socket对象，指明服务器端的ip和端口
        InetAddress inet = InetAddress.getByName("192.168.137.237");
        socket = new Socket(inet, 8899);
        // 2. 获取一个输出流，用于输出数据
        os = socket.getOutputStream();
        // 3. 写出数据
        os.write("hello world !".getBytes());
    }catch(IOException e){
        e.printStackTrace();
    }finally {
        if(os!=null){
            os.close();
        }
        if(socket!=null){
            socket.close();
        }
    }
}

// Server端接收Client端发送来的数据
@Test
public void server() throws IOException {
    // server 端 接收消息
    ServerSocket ss = null;
    Socket socket = null;
    InputStream is = null;
    ByteArrayOutputStream baos = null;
    // 1. 创建服务器端的ServerSocket，指明自己的端口号
    ss = new ServerSocket(8899);
    //2. 调用accept()表示接收来自客户端的socket
    socket = ss.accept();
    // 3. 获取输入流    获取输出流的时候，会阻塞，等待Client端 给服务器端发送数据
    is = socket.getInputStream();
    // 4. 读取输入流中的数据
    baos = new ByteArrayOutputStream();
    byte[] buffer = new byte[5];
    int len=0;
    while((len=is.read(buffer))!=-1){
        baos.write(buffer,0,len);
    }
    System.out.println("收到来自："+socket.getInetAddress().getHostAddress()+"的数据");
    System.out.println(baos.toString());
}
```



TCP网络编程：从客户端发送文件给服务端，服务端保存到本地，并返回“发送成功”给客户端。并关闭相应的连接。

```java
@Test
public void client2() throws IOException {
    // 1. 客户端向服务端发送一张图片， 等待服务端接收完毕后返回 接收成功！ 信息
    Socket socket = new Socket("127.0.0.1", 9090);
    OutputStream os = socket.getOutputStream();
    FileInputStream fis = new FileInputStream(new File("Files\\卡通 - 4.jpg"));
    byte[] buffer = new byte[5];
    int len = 0;
    while ((len=fis.read(buffer))!=-1){
        // 从文件中读取的数据写入到到socket输出流中
        os.write(buffer,0,len);
    }
    // 关闭socket数据的输出
    socket.shutdownOutput();
    // 接收服务器端的数据，并显示到控制台上
    InputStream is = socket.getInputStream();
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    buffer = new byte[5];
    len = 0;
    while ((len=is.read(buffer))!=-1){
        baos.write(buffer,0,len);
    }
    System.out.println(baos.toString());
    fis.close();
    os.close();
    socket.close();
}

@Test
public void server2() throws IOException{
    ServerSocket ss = new ServerSocket(9090);
    Socket socket = ss.accept();
    InputStream is = socket.getInputStream();
    FileOutputStream fos = new FileOutputStream(new File("Files\\图片1.jpg"));
    byte[] buffer = new byte[1024];
    int len=0;
    while((len=is.read(buffer))!=-1){
        fos.write(buffer,0,len);
    }
    System.out.println("服务器端接收数据完毕");

    // 给客户端进行反馈
    OutputStream os = socket.getOutputStream();
    os.write("照片服务器端已经接收完毕".getBytes());
    os.close();
    fos.close();
    is.close();
    socket.close();
    ss.close();
}
```



### 2.6 UDP网络编程

- 类**DatagramSocket**和**DatagramPacket**实现了基于UDP协议网络程序。
- UDP数据报通过数据报套接字DatagramSocket发送和接收，系统不保证UDP数据包一定能够安全送到目的地，也不能确定什么时候可以抵达
- DatagramPacket对象封装了UDP数据报，在数据报中包含了发送端的IP地址和端口号以及接收端的IP地址和端口号。
- UDP协议中每个数据报都给出了完整的地址信息，因此无需建立发送方和接收方的连接。如同发快递包裹一样。
- 流程
  1. DatagramSocket与DatagramPacket
  2. 建立发送端，接收端
  3. 建立数据包
  4. 调用Socket的发送、接收方法
  5. 关闭Socket

> 发送端与接收端是两个独立的运行程序

```java
@Test
public void udpSend() throws IOException{
    DatagramSocket socket = new DatagramSocket();
    String str = "我是UDP发送端";
    byte[] data = str.getBytes();
    InetAddress inetAddress = InetAddress.getLocalHost();
    DatagramPacket packet = new DatagramPacket(data,0,data.length,inetAddress,9090);
    socket.send(packet);
    socket.close();
}
@Test
public void udpReceive() throws IOException{
    DatagramSocket socket = new DatagramSocket(9090);
    byte[] buffer = new byte[1024];
    DatagramPacket packet = new DatagramPacket(buffer,0,buffer.length);
    socket.receive(packet);
    System.out.println(new String(packet.getData(),0,packet.getLength()));
    socket.close();
}
```

### 2.7 URL编程

URL网络编程实现数据下载

```java
@Test
public void urlDemo() throws IOException {
    HttpURLConnection urlConnection = null;
    InputStream is = null;
    FileOutputStream fos = null;
    URL url = new URL("https://pic2.zhimg.com/80/v2-576df7c2088825cc388a4a8f74f8b2ed_720w.jpg");
    urlConnection = (HttpURLConnection) url.openConnection();
    urlConnection.connect();
    is = urlConnection.getInputStream(); // 得到输入流
    fos = new FileOutputStream(new File("Files\\spark.jpg"));
    byte[] buffer = new byte[5];
    int len = 0;
    while ((len = is.read(buffer)) != -1) {
        fos.write(buffer, 0, len);
    }
    System.out.println("图片下载完成");
    fos.close();
    is.close();
    urlConnection.disconnect();
}
```



# 3. NIO

## 3.1 Java NIO 概述

### 阻塞IO

通常在进行同步I/O操作时，如果读取数据，代码会阻塞直至有可提供读取的数据。同样，写入调用将会阻塞直至数据能够写入。传统的Server/Client模式会基于（TPR：Thread per Request），服务器会为每个客户端请求建立一个线程，由该线程单独负责处理一个客户请求。这种模式带来的一个问题就是线程数量的剧增，大量的线程会增大服务器的开销。大多数的实现为了避免这个问题，都采用了线程池模型，并设置线程池线程的最大数量，这又带来了新的问题，如果线程池中有100个线程，而有100个用户都在进行大文件下载，会导致第101个用户的请求无法及时处理，即便第101个用户只想请求一个几kb大小的页面。传统的Server/Client模式如图所示：

![image-20211020210801147](source/Java-IO流与网络编程/image-20211020210801147-16347352859511.png)





### 非阻塞IO（NIO）

NIO中非阻塞I/O采用了基于Reactor模式的工作方式，I/O调用不会被阻塞，相反是注册感兴趣的特定I/O事件，如可读数据到达，新的套接字连接等等，在发生特定事件时，系统再通知我们。NIO中实现非阻塞I/O的核心对象就是Selector，Selector就是注册各种I/O事件地方，而且当我们感兴趣的事件发生时，就是这个对象告诉我们所发生的时间。

![image-20211020211145095](source/Java-IO流与网络编程/image-20211020211145095-16347355066182.png)

当有读或写等任何注册的时间发生时，可以从Selector中获得相应的SelectionKey，同时SelectionKey中可以找到发生的事件和该事件所发生的具体的SelectableChannel，以获得客户端发送过来的数据。

非阻塞指的是IO时间本身不阻塞，但是获取IO事件的select()方法是需要阻塞等待的。区别是阻塞的IO会阻塞IO操作上，NIO阻塞在事件获取上，没有事件就没有IO，从高层次看IO就不阻塞了。也就是说只有IO已经发生那么我们才评估IO是否阻塞，但是select()阻塞的时候IO还没有发生，何谈IO的阻塞呢？NIO的本质是延迟IO操作到真正发生IO的时候，而不是以前的只要IO流打开了就一直等到IO操作。

| IO                      | NIO                           |
| ----------------------- | ----------------------------- |
| 面向流(Stream Oriented) | 面向缓冲区（Buffer Oriented） |
| 阻塞IO（Blocking IO）   | 非阻塞IO（Non Blocking IO）   |
| 无                      | 选择器（Selectors）           |

Java的NIO由以下几个核心部分组成：

- Channels

  - > NIO中的Channel和 IO中的Stream(流)是差不多一个等级的。只不过Stream是单向的，譬如：**InputStream**，**OutputStream**。而**Channel**是双向的，**即可以用来进行读操作，又可以用来进行写操作**。
    >
    > NIO中的Channel的主要实现有：FileChannel，DatagramChannel，SocketChannel和ServerSocketChannel，分别对应文件IO，UDP和TCP（Server和Client）

- Buffers

  - > NIO中的关键实现有：ByteBuffer，CharBuffer，DoubleBuffer，FloatBuffer，IntBuffer，LongBuffer，ShortBuffer，分别对应基本数据类型：byte，char，double，float，int，long，short。

- Selectors

  - > Selector运行单线程处理多个Channel，如果你的应用打开了多个通道，但每个连接的流量都很低，使用Selector就会很方便。例如在一个聊天服务器中。要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法就会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子就如新的连接进行，数据接收等。

虽然Java NIO中除此之外还有很多类和组件，但Channel，Buffer和Selector构成了核心的API。其他组件，如Pipe和FileLock，只不过是与三个核心组件共同使用的工具类。

## 3.2 Channel





## 3.3  Buffer









## 3.4 Selector











## 3.5 Pipe和FileLock









## 3.6 其他
