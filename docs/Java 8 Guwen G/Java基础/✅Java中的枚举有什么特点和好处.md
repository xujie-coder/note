枚举类型是指由一组固定的常量组成合法的类型。Java中由关键字enum来定义一个枚举类型



```java
public enum Season {
    SPRING, SUMMER, AUTUMN, WINER;
}
```



Java中枚举的好处如下：



### 1、枚举的`valueOf`可以自动对入参进行非法参数的校验
****

Java 枚举提供了 `valueOf` 方法，它可以根据字符串值返回相应的枚举常量。如果传入的字符串不匹配任何枚举常量，`valueOf` 会抛出 `IllegalArgumentException` 异常，从而自动进行非法参数的校验。

****

```plain
public class EnumExample {
    public static void main(String[] args) {
        try {
            Season ss = Season.valueOf("SPRING"); // 合法参数
            System.out.println(ss); // 输出 SPRING

            Season invalidSeason = Season.valueOf("HOLLIS"); // 非法参数，会抛出异常
        } catch (IllegalArgumentException e) {
            System.out.println("Invalid Season: " + e.getMessage()); // 输出 Invalid Season: HOLLIS
        }
    }
}

```



### 2、可以调用枚举中的方法，相对于普通的常量来说操作性更强


枚举可以定义方法，这使得每个枚举常量都可以具备独特的行为。与普通常量相比，枚举更具操作性和灵活性。



```plain
public enum Operation {
    ADD {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    SUBTRACT {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    MULTIPLY {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };

    public abstract double apply(double x, double y);
}

public class EnumExample {
    public static void main(String[] args) {
        double x = 10;
        double y = 5;

        System.out.println("Addition: " + Operation.ADD.apply(x, y)); // 输出 Addition: 15.0
        System.out.println("Subtraction: " + Operation.SUBTRACT.apply(x, y)); // 输出 Subtraction: 5.0
    }
}

```



### 3、枚举实现接口的话，可以很容易的实现策略模式


通过实现接口，每个枚举常量可以具有不同的实现，从而实现策略模式，参考 ：



[✅Spring在业务中常见的使用方式](https://www.yuque.com/hollis666/qyhor6/xn5f5v)





### 4、枚举可以自带属性，扩展性更强


枚举可以包含字段、构造函数和方法，使得每个枚举常量可以具有不同的属性和行为。



```plain
public enum Color {
    RED("#FF0000"),
    GREEN("#00FF00"),
    BLUE("#0000FF");

    private final String hexCode;

    Color(String hexCode) {
        this.hexCode = hexCode;
    }

    public String getHexCode() {
        return hexCode;
    }

    @Override
    public String toString() {
        return name() + " (" + hexCode + ")";
    }
}

public class EnumExample {
    public static void main(String[] args) {
        for (Color color : Color.values()) {
            System.out.println(color); // 输出 RED (#FF0000), GREEN (#00FF00), BLUE (#0000FF)
        }
    }
}

```



### 5、天生就是个单例，线程安全且不怕被破坏


[✅为什么说枚举是实现单例最好的方式？](https://www.yuque.com/hollis666/qyhor6/dt4dp5iq77akg00u)







# 扩展知识
## 枚举如何实现的？


如果我们使用反编译，对一个枚举进行反编译的话，就能大致了解他的实现方式，如上面的Season枚举，反编译后内容如下：



```java
public final class T extends Enum
{
    private T(String s, int i)
    {
        super(s, i);
    }
    public static T[] values()
    {
        T at[];
        int i;
        T at1[];
        System.arraycopy(at = ENUM$VALUES, 0, at1 = new T[i = at.length], 0, i);
        return at1;
    }

    public static T valueOf(String s)
    {
        return (T)Enum.valueOf(demo/T, s);
    }

    public static final T SPRING;
    public static final T SUMMER;
    public static final T AUTUMN;
    public static final T WINTER;
    private static final T ENUM$VALUES[];
    static
    {
        SPRING = new T("SPRING", 0);
        SUMMER = new T("SUMMER", 1);
        AUTUMN = new T("AUTUMN", 2);
        WINTER = new T("WINTER", 3);
        ENUM$VALUES = (new T[] {
            SPRING, SUMMER, AUTUMN, WINTER
        });
    }
}
```



通过反编译后代码我们可以看到，**public final class T extends Enum，说明，该类是继承了Enum类的，同时final关键字告诉我们，这个类也是不能被继承的。当我们使用enum来定义一个枚举类型的时候，编译器会自动帮我们创建一个final类型的类继承Enum类,所以枚举类型不能被继承，我们看到这个类中有几个属性和方法。**



## 枚举如何比较
枚举的equals方法底层用的还是==，所以两者都可以



## 枚举单例的好处


[✅为什么说枚举是实现单例最好的方式？](https://www.yuque.com/hollis666/qyhor6/dt4dp5iq77akg00u)



