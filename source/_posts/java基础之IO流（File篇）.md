---
title: IO里的老熟人File类
tags: [Interview,java,IO]
categories: Java IO
copyright: true
---
File类描述的是一个文件或文件夹。（文件夹也可以称为目录），该类的出现是对文件系统的中的文件以及文件夹进行对象的封装。可以通过对象的思想来操作文件以及文件夹，通过该对象的方法得到文件或文件夹的信息方便了对文件与文件夹的属性信息进行操作。<!--more-->文件包含很多的信息:如文件名、创建修改时间、大小、可读可写属性等。
##### 1.基本API
（1）.通过将给定路径来创建一个新File实例。
```java
new File(String pathname);
```
（2）.根据parent路径名字符串和child路径名创建一个新File实例。parent是指上级目录的路径，完整的路径为parent+child.
```java
new File(String parent, String child);
```
（3）.根据parent抽象路径名和child路径名创建一个新File实例。 parent是指上级目录的路径，完整的路径为parent.getPath()+child.
说明：如果指定的路径不存在（没有这个文件或是文件夹），不会抛异常，这时file.exists()返回false。
```java
new File(File parent, String child);
```
***说明：如果指定的路径不存在（没有这个文件或是文件夹），不会抛异常，这时file.exists()返回false。***

（4）创建：
```java
createNewFile()   //在指定位置创建一个空文件，成功就返回true，如果已存在就不创建然后返回false

mkdir()           //在指定位置创建目录，这只会创建最后一级目录，如果上级目录不存在就抛异常。

mkdirs()          //在指定位置创建目录，这会创建路径中所有不存在的目录。

renameTo(File dest)  //重命名文件或文件夹，也可以操作非空的文件夹，文件不同时相当于文件的剪切,剪切时候不能操作非空的文件夹。
                     //移动/重命名成功则返回true，失败则返回false。
```
（5）.删除：
```java
delete()
//删除文件或一个空文件夹，如果是文件夹且不为空，则不能删除，成功返回true，失败返回false。

deleteOnExit()
//在虚拟机终止时，请求删除此抽象路径名表示的文件或目录，保证程序异常时创建的临时文件也可以被删除
```
（6）.判断：
```java
exists()         //文件或文件夹是否存在。

isFile()          //是否是一个文件，如果不存在，则始终为false。

isDirectory()     //是否是一个目录，如果不存在，则始终为false。

isHidden()        //是否是一个隐藏的文件或是否是隐藏的目录。

isAbsolute()      //测试此抽象路径名是否为绝对路径名。
```
（7）.获取：
```java
getName()    //获取文件或文件夹的名称，不包含上级路径。

getPath()    //返回绝对路径，可以是相对路径，但是目录要指定

getAbsolutePath()    //获取文件的绝对路径，与文件是否存在没关系

length()    // 获取文件的大小（字节数），如果文件不存在则返回0L，如果是文件夹也返回0L。

getParent()    //返回此抽象路径名父目录的路径名字符串；如果此路径名没有指定父目录，则返回null。

lastModified()    //获取最后一次被修改的时间。



staic File[] listRoots()    //列出所有的根目录（Window中就是所有系统的盘符）

list()    //返回目录下的文件或者目录名，包含隐藏文件。对于文件这样操作会返回null。

list(FilenameFilter filter)    //返回指定当前目录中符合过滤条件的子文件或子目录。对于文件这样操作会返回null。

listFiles()    //返回目录下的文件或者目录对象（File类实例），包含隐藏文件。对于文件这样操作会返回null。

listFiles(FilenameFilter filter)    //返回指定当前目录中符合过滤条件的子文件或子目录。对于文件这样操作会返回null。
```

>路径问题
>- 对于UNIX平台，绝对路径名的前缀是"/"。相对路径名没有前缀。
>- 对于Windows平台，绝对路径名的前缀由驱动器号和一个":"组成，例"c:\\..."。相对路径没有盘符前缀。
更专业的做法是使用**File.separatorChar**或者**File.separator**（前者为char，后者为String），这个值就会根据系统得到的相应的分割符。

#### 2.常见File类面试题
1、File类型中定义了什么方法来创建一级目录
```java
mkdirs() //在指定位置创建目录，这会创建路径中所有不存在的目录。
```
2、File类型中定义了什么方法来判断一个文件是否存在
```java
exists()         //文件或文件夹是否存在。
```
3、如何用Java代码列出一个目录下所有的文件？
```java
//只查询当前目录
public class Main {

    public static void main(String[] args) {
        File f = new File("d:");
        for(File temp : f.listFiles()) {
            if(temp.isFile()) {
                System.out.println(temp.getName());
            }
        }
    }
}

// 如果需要对文件夹继续展开
    public static void main(String[] args) {
        showDirectory(new File("d:\\file"));
    }

    public static void showDirectory(File f) {
        _walkDirectory(f, 0);
    }

    private static void _walkDirectory(File f, int level) {
        if(f.isDirectory()) {
            for(File temp : f.listFiles()) {
                _walkDirectory(temp, level + 1);
            }
        }
        else {
            for(int i = 0; i < level - 1; i++) {
                System.out.print("\t");
            }
            System.out.println(f.getName());
        }
    }
```
在Java 7中可以使用NIO.2的API来做同样的事情，代码如下所示：
```java
 public static void main(String[] args) throws IOException {
        Path initPath = Paths.get("/Users/Hao/Downloads");
        Files.walkFileTree(initPath, new SimpleFileVisitor<Path>() {

            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs)
                    throws IOException {
                System.out.println(file.getFileName().toString());
                return FileVisitResult.CONTINUE;
            }

        });
    }
```