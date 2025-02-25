# 典型回答


### extends
<? extends T> 表示类型的上界，表示参数化类型的可能是T 或是 T的子类



```java
// 定义一个泛型方法，接受任何继承自Number的类型
public <T extends Number> void processNumber(T number) {
    // 在这个方法中，可以安全地调用Number的方法
    double value = number.doubleValue();
    // 其他操作...
}
```



举个例子，假设我们有一个基本类 `Animal` 和两个子类 `Dog` 和 `Cat`：



```plain
class Animal {
    public void makeSound() {}
}

class Dog extends Animal {
    public void bark() {}
}

class Cat extends Animal {
    public void meow() {}
}
```



我们可以使用 `extends` 限定符来定义一个泛型方法，只允许传入 `Animal` 或其子类：



```plain
public class GenericExample {
    // 泛型方法，类型参数 T 必须是 Animal 或 Animal 的子类
    public <T extends Animal> void processAnimal(T animal) {
        animal.makeSound();
    }

    public static void main(String[] args) {
        GenericExample example = new GenericExample();
        
        Dog dog = new Dog();
        Cat cat = new Cat();

        example.processAnimal(dog); // 合法
        example.processAnimal(cat); // 合法
        // example.processAnimal(new String()); // 编译错误
    }
}
```



### super


<? super T> 表示类型下界（Java Core中叫超类型限定），表示参数化类型是此类型的超类型（父类型），直至Object



```java
// 定义一个泛型方法，接受任何类型的List，并向其中添加元素
public <T> void addElements(List<? super T> list, T element) {
    list.add(element);
    // 其他操作...
}
```



假设我们需要定义一个方法，向一个 `List` 中插入元素，这个 `List` 的泛型类型可以是某个类或该类的父类：



```plain
import java.util.List;

public class GenericExample {
    // 泛型方法，类型参数 T 必须是 Number 或 Number 的父类
    public <T> void addNumberToList(List<? super T> list, T number) {
        list.add(number);
    }

    public static void main(String[] args) {
    GenericExample example = new GenericExample();

    List<Number> numberList = new ArrayList<>();
    example.addNumberToList(numberList, 10); // 合法，Integer 是 Number 的子类
    example.addNumberToList(numberList, 10.5); // 合法，Double 是 Number 的子类

    example.addNumberToList(numberList, "Hello"); // 编译错误，String 不是 Number 的子类
    }
}
```



### PECS 原则


在使用 限定通配符的时候，需要遵守**PECS原则**，即<font style="color:rgb(0, 0, 0);">Producer Extends, Consumer Super；上界生产，下界消费。</font>

<font style="color:rgb(0, 0, 0);"></font>

如果要从集合中读取类型T的数据，并且不能写入，可以使用 ? extends 通配符；(Producer Extends)，如上面的processNumber方法。



如果要从集合中写入类型T的数据，并且不需要读取，可以使用 ? super 通配符；(Consumer Super)，如上面的addElements方法



> extend的时候是可读取不可写入，那为什么叫上界生产呢？
>
> 因为这个消费者/生产者描述的<集合>，当我们从集合读取的时候，集合是生产者。
>



如果既要存又要取，那么就不要使用任何通配符。



综合示例：

```plain
import java.util.ArrayList;
import java.util.List;

public class GenericBoundsExample {

    // 使用 extends 限定符来读取泛型类型
    public static <T extends Comparable<T>> T findMax(List<T> list) {
        T max = list.get(0);
        for (T element : list) {
            if (element.compareTo(max) > 0) {
                max = element;
            }
        }
        return max;
    }

    // 使用 super 限定符来写入泛型类型
    public static <T> void addElement(List<? super T> list, T element) {
        list.add(element);
    }

    public static void main(String[] args) {
        List<Integer> intList = new ArrayList<>();
        addElement(intList, 5); // Integer 是 Number 的子类

        List<Number> numberList = new ArrayList<>();
        addElement(numberList, 5); // Integer 是 Number 的子类
        addElement(numberList, 5.5); // Double 是 Number 的子类

        List<String> stringList = new ArrayList<>();
        // addElement(stringList, 5); // 编译错误，Integer 不是 String 的子类

        List<Integer> integers = new ArrayList<>();
        integers.add(3);
        integers.add(7);
        integers.add(2);

        Integer max = findMax(integers);
        System.out.println("Max: " + max); // Max: 7
    }
}

```

