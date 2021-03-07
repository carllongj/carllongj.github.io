## t-c-f作用
   t-c-f(try-catch-finally),是用于处理异常的流程控制,当try块中出现异常,会根据具体的异常选择对应的catch块来执行,而finally则
   是在try和catch块执行完成后执行语句,一般会在try或者catch后一定执行,也会有一些例外.
## finally一定会执行吗
   在方法中,可能会存在多个finally语句,要让对应finally执行必须满足两点
      1. 进入finally对应的try块中
      2. 在try块不能存在让JVM进程终止的语句或者条件(比如执行System.exit()方法或者在try块中终止进程)
## t-c-f的实现
   在Java中,t-c-f是比较重要的流程控制语句,它在Java程序指令中是通过异常表(Exceptions)来实现的,具体的结构如下:
   ```
           类型                  名称                数量 
            u2                  start_pc(from)         1
            u2                  end_pc(to)             1
            u2                  handler_pc(target)     1
            u2                  catch_type             1
   ```
   其中
     1. u2是表示两个字节长度大小
     2. start_pc 表示开始位置
     3. end_pc 表示结束位置
     4. handler_pc 表示跳到下一步处理的起始位置
     5. catch_type 表示捕获的异常类型
   通过一段代码来看看异常表的数据,源码如下
   ```
    public class A{
        public static void main(String[] args) {
                try{
                     System.out.println("执行try块");
                     Integer i = null;
                     System.out.println(i);
                }catch(Exception e){
                     System.out.println("执行catch块");
                }finally {
                     System.out.println("执行finally块");
                }
        }
    }
   ```
   方法的字节码数据,如下
   ```
    Code:
      stack=2, locals=3, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String 执行try块
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: aconst_null
         9: astore_1
        10: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        13: aload_1
        14: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
        17: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        20: ldc           #6                  // String 执行finally块
        22: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        25: goto          59
        28: astore_1
        29: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        32: ldc           #8                  // String 执行catch块
        34: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        37: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        40: ldc           #6                  // String 执行finally块
        42: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        45: goto          59
        48: astore_2
        49: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        52: ldc           #6                  // String 执行finally块
        54: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        57: aload_2
        58: athrow
        59: return
      Exception table:
         from    to  target type
             0    17    28   Class java/lang/Exception
             0    17    48   any
            28    37    48   any

   ```
   Exception table表明了当前t-c-f的执行流程,第一行说明:
     1. 若在 0 - 17条之间之间出现了java.lang.Exception类异常则跳转到第28条指令.
     2. 若在 0 - 17条指令之间出现了其它非java.lang.Exception类异常则跳转到第48条指令
     3. 若在 28 - 37条指令之间出现了任何异常,则跳转到48条指令
   
   流程控制将会如下:
     1. 如果是正常流程,则会从 0 - 25 跳转到第59条指令结束,其中finally指令在10 - 22 段
     2. 如果在 0 - 17 行指令间出现Exception异常,则会跳转到第28条指令,继续执行,也就是执行catch块代码
     3. 如果在 0 - 17 条指令间出现非Exception异常,则会跳转到第48条指令,也就是执行finally块代码.
     4.若在28 - 37条指令间出现了任何异常,则会跳转到第48条指令,也就是在catch块中出现任何异常,则会跳转到48条指令.
	 从上面也可以看出来,不管是任何的流程,最终都会执行finally中的语句.
## 参考
[关于 Java 中 finally 语句块的深度辨析](http://www.ibm.com/developerworks/cn/java/j-lo-finally/index.html)
**《深入理解Java虚拟机》**','/articles/2019/04/26/1556280404475.html','1','1','0',1556968449863,1556968449863,0.08011374083835487,'1','1','','CodeMirror-Markdown'),('1556968598556','Spring获取方法参数名称的方式','','Java,反射,Spring,字节码','1547133780154',0,4,'![](https://img.hacpai.com/bing/20171113.jpg?imageView2/1/w/960/h/520/interlace/1/q/100) 

## 一丶背景
   在JDK8之前,想要获取Java方法的参数名通过反射是没有办法获取的,并且即使通过字节码获取参数也不是一定能获取到,能否获取到取决于编译时的编译参数.在JDK8之后,Java新增
   了 Parameter 类在运行时获取指定方法的参数名称.参数名称对于程序运行来说不是必需的,只需要能找到变量的内存位置即可.
## 二丶编译参数
   1.要获取到方法的参数名称,那么编译的时候必须要保留方法的参数名称在class文件中,在JDK8之前若要保留方法参数名则需要执行
   ```
   // -g保留所有的debug调试信息  -g:vars 表示保留参数名称信息
   javac -g 或者javac -g:vars 
   ```
   使用该编译参数也没有办法通过Java的反射库获取到方法的参数名称,只有通过字节码库来获取方法的参数名称,通过这种方式编译只是保证了在class文件的数据结构中包含了此数据
   
   2.如果是在java8版本下编译,可以使用以下编译参数
   ```
      javac -parameters
   ```
   使用java8版本则能保证通过反射可以获取到方法的参数名称
## 三丶编译后的class文件数据
   主要的点是,源代码自定义的数据名称(如类名称,方法名称,局部变量名称)这些名称都会被保存在class文件数据的常量池中,下面一段测试编译参数的源代码
   ```
     public class A{
          public void method(String parameter){}
     }
   ```
   1.使用javac编译(一般手动编译时使用),此时常量池中是没有参数名称 parameter 字符串定义的
   ```
   Constant pool:
    #1 = Methodref          #3.#12         // java/lang/Object."<init>":()V
    #2 = Class              #13            // A
    #3 = Class              #14            // java/lang/Object
    #4 = Utf8               <init>
    #5 = Utf8               ()V
    #6 = Utf8               Code
    #7 = Utf8               LineNumberTable
    #8 = Utf8               method
    #9 = Utf8               (Ljava/lang/String;)V
    #10 = Utf8               SourceFile
    #11 = Utf8               A.java
    #12 = NameAndType        #4:#5          // "<init>":()V
    #13 = Utf8               A
    #14 = Utf8               java/lang/Object
   ```
   2.使用javac -g编译(使用javac -g:vars同理),此时常量池中是包含了方法参数名称字符串 parameter 定义(常量池中的第13个常量定义).
   ```
   Constant pool:
    #1 = Methodref          #3.#17         // java/lang/Object."<init>":()V
    #2 = Class              #18            // A
    #3 = Class              #19            // java/lang/Object
    #4 = Utf8               <init>
    #5 = Utf8               ()V
    #6 = Utf8               Code
    #7 = Utf8               LineNumberTable
    #8 = Utf8               LocalVariableTable
    #9 = Utf8               this
    #10 = Utf8               LA;
    #11 = Utf8               method
    #12 = Utf8               (Ljava/lang/String;)V
    #13 = Utf8               parameter
    #14 = Utf8               Ljava/lang/String;
    #15 = Utf8               SourceFile
    #16 = Utf8               A.java
    #17 = NameAndType        #4:#5          // "<init>":()V
    #18 = Utf8               A
    #19 = Utf8               java/lang/Object

   ```
   3.使用javac -parameters 编译,该参数是jdk8版本新增的,此时常量池中也包含了方法参数名称 parameter 定义(常量池中第11个定义).
   ```
   Constant pool:
    #1 = Methodref          #3.#14         // java/lang/Object."<init>":()V
    #2 = Class              #15            // A
    #3 = Class              #16            // java/lang/Object
    #4 = Utf8               <init>
    #5 = Utf8               ()V
    #6 = Utf8               Code
    #7 = Utf8               LineNumberTable
    #8 = Utf8               method
    #9 = Utf8               (Ljava/lang/String;)V
    #10 = Utf8               MethodParameters
    #11 = Utf8               parameter
    #12 = Utf8               SourceFile
    #13 = Utf8               A.java
    #14 = NameAndType        #4:#5          // "<init>":()V
    #15 = Utf8               A
    #16 = Utf8               java/lang/Object

   ```
   
## 四丶Spring中获取变量名称的方式
   Spring中的注解 @RequestParam 就是指定了使用了参数绑定,如果没有给其属性 name 或者 value 设置值,就会使用方法的参数名称与Request中的请求参数名称做映射,将
   对应的请求参数的值映射到变量中,就达到了这个注解的目标,那么可以看Spring中是如何获取方法的参数名称的.Spring获取参数根接口的定义:
   ```
    public interface ParameterNameDiscoverer {
       String[] getParameterNames(Method method);
       String[] getParameterNames(Constructor ctor);
    }
   ```
   由于存在上述的两种情况,Spring有两个实现类来实现获取对应的参数名称获取
     1. StandardReflectionParameterNameDiscoverer  只有Java8的运行版本才会使用该实现类去获取参数名称
     2. LocalVariableTableParameterNameDiscoverer  通过字节码来获取参数名称的实现类
   然后就是具体的使用的类
     1.PrioritizedParameterNameDiscoverer 该类定义了一个优先级获取的规则,根据具体实现类获取的参数名称不为空则返回,否则使用下一个获取实现类尝试获取
     2.DefaultParameterNameDiscoverer    该类组装了一个优先级顺序,基本都是用该类的对象来获取参数名称
## 五丶总结
    1.能否获取到方法的参数名称取决于编译时是否保留了方法的参数名称数据到class数据中
    2.jdk8+ 可以通过反射来获取方法的参数名称,但能否获取到取决于编译时是否使用了 -parameters 参数
## 六丶参考
* [获取参数名称的方法](http://nullwy.me/2017/05/java-method-parameter/)
* [Java获取方法的参数名](https://blog.csdn.net/wthfeng/article/details/72112967)
* **《深入理解Java虚拟机》**


## 泛型概述
泛型是一种程序语言的设计设计风格,泛型允许程序员在强类型程序设计语言中编写代码时使用一些以后才指定的类型，在实例化时作为参数指明这些类型(摘自维基百科).

## Java中的泛型
Java中像集合框架(Collection,List,Map)这些库都是泛型类设计.
```
1. 对于一类操作来说,如果只是操作的内容不一样,泛型可以减少这部分代码的编写,如 List<String> 和 List<Object> 是不一样的类型,但是泛型避免了实现 List<String> 和 List<Object>两种类

2. 泛型减少了程序在运行时的类型转换异常,List<Object>对于存储方来说非常方便,但是一旦要获取列表中的数据,每一个元素的类型转换有极大的风险

3. Java中的泛型存在类型擦除,在运行的时候,泛型的类型会被擦除掉.
```