## 一、Java概述
### 1、Java语言特性
| | Java |
| --- | --- |
| 跨平台 | 平台无关 |
| 内存管理 | 自动 |
| 参数传递方式 | 值传递 |
| 多继承 | 不支持 |
| 系统资源的控制能力 | 弱 |
| 适合领域 | 企业级Web应用开发 |


## 二、变量
### 1、基本类型


### 2、拆装箱


## 3、关键字


### 4、常见的字符编码有哪些


### 5、String、StringBuilder和StringBuffer


### 6、Java的求值策略
Java中只有值传递，只不过传递的内容是对象的引用



## 三、面向对象
### 1、三大基本特征
封装、继承、多态

**封装** 就是把现实世界中的客观事物抽象成一个Java类，然后在类中存放**属性**和**方法**。

**继承** 目的是为了复用。子类可以继承父类，这样就可以把父类的属性和方法继承过来。

**多态 **是指在父类中定义的方法被子类继承之后，可以通过重写，使得父类和子类具有不同的实现，这使得同一个方法在父类及其各子类中具有不同含义。

### 2、面向对象的五大原则
+ 单一职责原则
    - 内容：一个类最好只做一件事
    - **提高可维护性**
    - **提升稳定性**
+ 开放封闭原则
    - 内容：对扩展开放，对修改封闭
    - **促进可扩展性**
    - **降低风险**
+ 里氏替换原则
    - 内容：子类必须能够替换其基类
    - **提高代码的可互换性**
    - **增加代码的可重用性**
+ 依赖倒置原则
    - 内容：程序要依赖于抽象接口，而不是具体的实现
    - **减少系统耦合**
    - **提高代码的可测试性**
+ 接口隔离原则
    - 内容：使用多个小的专门的接口，而不要使用一个大的总接口
    - **减少系统耦合**
    - **提升灵活性和稳定性**

### 3、接口和抽象类
用来实现抽象层

```java
public interface PayService{
    public void pay(PayRequest payRequest);
}
```

```java
public abstract class Test{
    public String a;
    public String b;
    String c;
    protected String d;
}
```



| | 接口 | 抽象类 |
| --- | --- | --- |
| 方法定义 | | |
| 修饰符 | | |
| 构造器 | | |
| 继承和实现 | | |
| 单继承、多实现 | | |
| 职责 | | |




## 四、枚举和注解
### 1、枚举定义




## 五、异常
### 1、异常分类


## 六、泛型
### 1、泛型定义


### 2、类型擦除




## 七、IO流
### 1、IO分类


2、

## 八、反射
### 1、反射定义


## 九、序列化与反序列化
### 1、定义


### 2、Java序列化原理


## 十、内置工具类
### 1、`Timer`实现定时调度


### 2、`euqals`和`hashCode`


### 3、Lambda表达式
