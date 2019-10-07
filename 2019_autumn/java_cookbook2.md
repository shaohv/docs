## java知识精要（二）

---

集合、泛型、异常、注解

[java知识精要（一）](https://www.cnblogs.com/holidays/p/java_cookbook1.html)

### 集合

1. Iterable v.s. Iterator

   两者都是接口，在Collection继承的是Iterable。
   Iterable表达了集合具备迭代访问的能力，而Iterator表示实现，可以从小到大也可以从大到小。
   https://zhuanlan.zhihu.com/p/52366312

2. Comparable v.s. Comparator
   https://www.cnblogs.com/skywang12345/p/3324788.html

3. 集合
   [集合需多用用](https://www.cnblogs.com/LittleHann/p/3690187.html)
   [优先队列用法](https://blog.csdn.net/easylovecsdn/article/details/87882243)

### 泛型
1. 应该将List&#60;E&#62;看做一个具体类型，不真实存在的逻辑类型

   ```java
   List<String> l1 = new ArrayList<String>();
   List<Integer> l2 = new ArrayList<Integer>();
   System.out.println(l1.getClass() == l2.getClass() );
   ```

   结果？

   因此，类的static成员是不可以使用类型参数的。

   ``` java
   public class R<T>{
       static T info; //错误
       static void test(T msg); // 错误
   }
   ```

2. ?, ? extends type, ? super type
   a. 通配符与继承关系
   List&#60;Object&#62;与 List&#60;String&#62;之间无继承关系。
   void test(List&#60;Object&#62; l)传入List&#60;String&#62;将编译报错

   通配符、通配符上下限都可以理解为对继承关系的补充。
   void test(List&#60;?&#62; l)函数可以接收List&#60;Integer&#62;, List&#60;String&#62;等
   void test(List&#60;? extends Numbers&#62; l)函数可以接收List&#60;Integer&#62;, List&#60;Double&#62;等。

   b. 理解下列代码，其中Rect为Shape子类

   ```java
   void addRect(List<?  extends Shape> shapes){
   shapes.add(0, new Rect());
   }
   ```

   

### 异常
异常设计目的是让业务逻辑与异常处理逻辑分离。
​java程序运行过程中出现异常时，系统自动生成一个异常对象，该对象被提交给java运行时；
​Java运行时收到异常对象后，寻找合适的catch块，如果未找到，Java程序退出。

1. try...catch...finally
   finally代码块一定会被执行，除非try or catch中调用了System.exit
   try or catch中的return, throw将在finally代码块之后执行
2. 异常体系
   Throwable
   -> Error
     -> Exception
       -> RuntimeException
       -> others... ：checked异常必须显示处理