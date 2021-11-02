## classpath选项

1. **classpath选项是什么**  
    * **classpath** 选项是在执行javac 命令和java 命令都可能用到,这个选项的作用是将一些jar包或者目录添加到classpath下,程序查找类或者资源文件时就会搜索classpath路径下.
2. **classpath选项和cp选项区别**  
    * 没有区别,-cp只是-classpath的简写
3. **如何使用**  
    * `java -classpath .:lib/other.jar Test`
        1. 这个命令可以认为是一个简单的java程序执行的命令,其中,当前目录下或者other.jar包中必须存在一个Test.class文件,应该当前程序认为Test是程序入口.
        2. 优先级,首先是当前目录下会被优先搜索,若当前目录下存在一个Test.class文件,那么当前目录下的Test类将会被执行
        3. -classpath 参数指定了两个路径(实际上还有很多是默认会指定的,比如java核心库),即当前目录下和当前目录下的lib目录里面的other.jar.
4. **需要注意的点**  
    * classpath 对于资源文件来说是一个目录,不能精确到某一个文件名.jar包是一个压缩包,对于jar包来说,可以看做是一个特殊的目录.
        * 添加jar包到classpath中必须是jar包名称,对于其他的则是到某一个目录

## Java程序的执行方式

* Java程序通常会使用两种方式来执行    
    1. 是class文件,通过java mainClass 来执行(省略其他参数)
    2. 是jar文件,使用jar -jar Execute.jar 来执行
* jar文件执行也可以分成两种,一种是使用 -jar选项 直接执行,还有一种是通过classpath将jar引入,然后通过mainClass来执行(到达第一种方式的效果).
    * 区别:
        1. 通过-jar选项执行的,jar包中必须包含META-INF目录,并且其下有MANIFEST.MF文件.而不使用-jar选项的则可以没有.
        2. 通过-jar选项执行的,jar包中的MANIFEST.MF文件必须列有 Main-Class 属性.不使用-jar选项,在执行命令的时候必须指定mainClass.
        3. 通过-jar选项执行的,jar包的classpath搜索路径会优先于其他的jar包依赖,不使用的则可以自定义顺序.
        4. 通过-jar选项执行的,不能再通过classpath来指定额外的classpath,对于应用程序类加载器来说,它就只关注当前jar包中的资源.

## 作用

* 通过classpath来引入外部资源.一般而言,java程序通常会被打包成jar包(如果都是class文件,不好管理),如果要修改配置文件是一个比较麻烦的事情.所以可以通过classpath来引入jar包外部的资源文件.
* 说说打包方式,一般打包方式为两种:
    1. 为了简单方便而言就是打一个整包,所有的依赖和配置文件都在一个jar包中.
    2. 将依赖和打包的工程分开,只打包工程本身,依赖则通过在执行的时候通过classpath引入.
	* 第一种方式的优缺点:
        * 优点: 打包简单(我只会通过开发工具打包工程本身和依赖包,命令行没有用过).执行简单,只需要执行java -jar 命令即可.
        * 缺点: 由于目前大部分java程序都是部署在linux远程服务器上,在开发的过程中避免不了会将包上传到远程服务器上运行.如依赖包很多,则每一次上传花费的时间很多.不利于进行开发测试
    * 第二种方式的优缺点:
	    * 优点: 上传jar包到远程服务器快,并且依赖包可以一次上传,多次使用.
        * 缺点: 运行程序会比较复杂,需要将依赖拼接到classpath里面.
    * 由于项目构建工具存在maven,打包的问题都能解决,所以对于java程序来说,打包都通过maven来执行,所以两种方式的打包和执行的缺点都能解决.对于第一种方式来说,可以在远程服务器上打整包,也不需要上传.所以这两种方式都可以使用了.   
## 说说程序执行

1. 程序执行时,需要用到的class类资源和文件资源都会在classpath路径下去查找.
2. 在执行程序的时候可以方便的通过classpath选项来将外部资源文件来替换jar包中的资源文件(classpath存在优先级).
3. 上面提到过使用通过-jar选项执行的,不能再通过classpath来指定额外的classpath,但是可以在jar包中的META-INF/MANIFEST.MF文件中通过Class-Path属性来增加classpath
   
## 总结:

* 在选择只打包当前工程代码的情况下:
    1. 可以不使用-jar方式执行,但是最好通过脚本生成执行java程序的命令,主要是通过-classpath选项增加依赖.
    2. 如果使用-jar方式执行,必须在META-INF/MANIFEST.MF文件中指定Class-Path选项来指定依赖包地址.
    3. 使用 -jar方式并且需要外部资源文件,一种是不将资源文件打包到jar包中,另一种则是通过 -Xbootclasspath/p选项来将外部资源文件添加到当前jar包前,提高外部资源程序的优先级.
   
## 四丶个人建议

1. 不建议将工程以及其依赖打成一个jar包,不利于工程模块化.
2. 对于一些可能常变的配置文件可以将其放入到外部资源文件中,通过classpath引入.
3. 工程打包分情况,如果稳定版本,则打一个方便执行的项目包(如tomcat的安装包,内部分成多个目录).
4. 对于开发测试过程中的打包,则只打当前工程的jar包,方便开发以及测试.
5. 对于java程序执行,打成jar包但是不使用 `-jar`选项去执行,使用 `java -classpath lib mainClass` 去执行程序.
 
## 五丶参考

* [maven打包时,将所有依赖包拷贝到一个目录](https://blog.csdn.net/qq_23367291/article/details/84577048)
* [maven打包时,不拷贝资源文件到jar包中](https://blog.csdn.net/javajxz008/article/details/81629011)
* [maven打包时,指定外部配置文件](https://segmentfault.com/a/1190000003698765)
* [java的Xbootclasspath的使用](https://www.cnblogs.com/duanxz/p/3482311.html)
* [java命令执行jar包的方式](http://www.cnblogs.com/zpbolgs/p/7267384.html)