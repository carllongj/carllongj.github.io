## t-c-f作用
* t-c-f(try-catch-finally),是用于处理异常的流程控制
  * 当try块中出现异常,会根据具体的异常选择对应的catch块来执行
  * finally则是在try和catch块执行完成后执行语句,一般会在try或者catch后一定执行,也会有一些例外.
## finally一定会执行吗
* 在方法中,可能会存在多个finally语句,要让对应finally执行必须满足两点
    1. 进入finally对应的try块中
    2. 在try块不能存在让JVM进程终止的语句或者条件(比如执行System.exit()方法或者在try块中终止进程)
## t-c-f的实现
* 在Java中,`t-c-f`是比较重要的流程控制语句,它在Java程序指令中是通过`异常表`(Exceptions)来实现的,具体的结构如下
   ```
           类型                  名称                数量 
            u2                  start_pc(from)         1
            u2                  end_pc(to)             1
            u2                  handler_pc(target)     1
            u2                  catch_type             1
   ```
    * 其中
      1. u2是表示两个字节长度大小
      2. start_pc 表示开始位置
      3. end_pc 表示结束位置
      4. handler_pc 表示跳到下一步处理的起始位置
      5. catch_type 表示捕获的异常类型
   * 通过一段代码来看看异常表的数据,源码如下
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
   * 方法的字节码数据,如下
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
    * Exception table表明了当前t-c-f的执行流程,第一行说明:
      1. 若在 0 - 17条之间之间出现了java.lang.Exception类异常则跳转到第28条指令.
      2. 若在 0 - 17条指令之间出现了其它非java.lang.Exception类异常则跳转到第48条指令
      3. 若在 28 - 37条指令之间出现了任何异常,则跳转到48条指令
   
    * 流程控制将会如下:
      1. 如果是正常流程,则会从 0 - 25 跳转到第59条指令结束,其中finally指令在10 - 22 段
      2. 如果在 0 - 17 行指令间出现Exception异常,则会跳转到第28条指令,继续执行,也就是执行catch块代码
      3. 如果在 0 - 17 条指令间出现非Exception异常,则会跳转到第48条指令,也就是执行finally块代码.
      4.若在28 - 37条指令间出现了任何异常,则会跳转到第48条指令,也就是在catch块中出现任何异常,则会跳转到48条指令.
	  * 可以看出,不管是任何的流程,最终都会执行finally中的语句.
## 参考
* [关于 Java 中 finally 语句块的深度辨析](http://www.ibm.com/developerworks/cn/java/j-lo-finally/index.html)
* **《深入理解Java虚拟机》**