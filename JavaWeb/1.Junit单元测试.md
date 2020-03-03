# 介绍

Junit 是一个 Java 编程语言的单元测试框架。Junit 在测试驱动的开发方面有很重要的发展，是起源于 Junit 的一个统称为 xUnit 的单元测试框架之一。

# 用处

1. 可以书写一系列的测试方法，对项目所有的接口或者方法进行单元测试。
2. 启动后，自动化测试，并判断执行结果, 不需要人为的干预。
3. 只需要查看最后结果，就知道整个项目的方法接口是否通畅。
4. 每个单元测试用例相对独立，由Junit 启动，自动调用。不需要添加额外的调用语句。
5. 添加，删除，屏蔽测试方法，不影响其他的测试方法。 开源框架都对Junit 有相应的支持。

# 测试分类

1. 黑盒测试，看不到代码
2. 白盒测试，可以看到代码

Junit:白盒测试

步骤：

1. 定义一个测试类（也叫测试用例）
   - 测试类名：被测试的类名再加一个test
   - 包名：xxx.xxx.xxx.test

```java
package junit;
public class MathGame {
    public int Add(int a, int b) {
        return a+b;
    }
}
```

2. 定义测试方法，可以独立运行
   - 方法名：test+测试的方法名
   - 返回值：void
   - 参数列表：空参
3. 给方法加@Test注解
4. 导入Junit类的依赖

```java
import junit.MathGame;
import org.junit.Test;
public class MathGameTest {
    @Test
    public void testAdd(){
        MathGame mathGame=new MathGame();
        int result=mathGame.Add(1,2);
        System.out.println(result);
    }
}
```

运行这个测试类，在输出框就会输出结果3

注：在单元测试的时候一般不用输出方法，而是采用断言的方式

```java
 @Test
    public void testAdd(){
        MathGame mathGame=new MathGame();
        int result=mathGame.Add(1,2);
        Assert.assertEquals(3,result);
    }
//Assert.assertEquals(期望值,实际值);
```

 补充：

- @Before:修饰的方法会在测试方法之前自动执行
- @After:修饰的方法会在测试方法之后自动执行

```java
public class MathGameTest {
    @Test
    public void testAdd(){
        System.out.println("Add...");
        MathGame mathGame=new MathGame();
        int result=mathGame.Add(1,2);
        Assert.assertEquals(3,result);
    }
    //用于初始化一些资源的申请
    @Before
    public void init(){
        System.out.println("init...");
    }
    //用于程序运行后一些资源的释放
    @After
    public void close(){
        System.out.println("close...");
    }
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020011012021124.png)

就算是在程序运行过程中出现错误close()方法也会执行

