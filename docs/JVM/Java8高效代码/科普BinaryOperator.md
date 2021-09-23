# 1、科普 BinaryOperator
我们知道，四大函数式接口：`Function`、`Supplier`、`Consumer`、`Predicate`。

但今天不说它们，今天看的是 `BinaryOperator`，就是一个封装好的，两个参数之间的lambda表达式。

使用demo：
```
import java.util.Comparator;
import java.util.function.BinaryOperator;

public class TestDemo {
    public static void main(String[] args) {
        BinaryOperator<Integer> add = (n1, n2) -> n1 + n2;
        System.out.println(add.apply(3, 4));

        BinaryOperator<String> addStr = (n1, n2) -> n1 + "====" + n2;
        System.out.println(addStr.apply("3", "4"));

        BinaryOperator<Integer> compare = BinaryOperator.minBy(Comparator.naturalOrder());
        System.out.println(compare.apply(3, 4));

        BinaryOperator<String> compare2 = BinaryOperator.minBy(Comparator.naturalOrder());
        System.out.println(compare2.apply("3", "4"));
    }
}

```
结果：
```
7
3====4
3
3
```

# 2、那么BinaryOperator和BiFunction的区别

国哥接触一个新东西的时候，会下意识去想，这个东西和之前的东西是否有什么关联。

比如，明眼人都看出来了，有了`BiFunction`，完全可以不需要`BinaryOperator`啊。

```java

import java.util.function.BiFunction;
import java.util.function.BinaryOperator;
import java.util.function.Predicate;


public class TestDemo3 {
    public static void main(String[] args) {
        // 完全等效
        BinaryOperator<Integer> f1 = (a, b) -> a * a + b * b;
        BiFunction<Integer, Integer, Integer> f2 = (a, b) -> a * a + b * b;

        System.out.println(f1.apply(3, 4));
        System.out.println(f2.apply(3, 4));

        Predicate<Integer> predicate = integer -> {
            return integer == 9 + 16;
        };

        System.out.println(predicate.test(f1.apply(3, 4)));
        System.out.println(predicate.test(f2.apply(3, 4)));

        // 报错代码
        BinaryOperator<String> f3 = (a, b) -> a * b + " ### ";
        // 有效代码
        BiFunction<Integer, Integer, String> f4 = (a, b) -> a * b + " ### ";
    }
}

```

从实践结果来看， `BinaryOperator` 只能适配那些少数场景，更具体点就是，T\U\R三者类型一致 ：
```
BiFunction<T, U, R> 可以等价 BinaryOperator<T>   // T\U\R三者类型一致

BiFunction<Integer, Integer, Integer> 可以等价 BinaryOperator<Integer>
```
但是以下场景是不合法的：
```
BiFunction<Integer, Integer, String> 不能替换成 BinaryOperator<String>
```
那么`为什么有了BiFunction还要BinaryOperator`的答案就出来了(在入参和返回值类型一致的情况下)：
* BinaryOperator简洁，编码好看
* 可读性更强
* 开发者可以更为懒惰

# 3、参考
* [StackOverflow: difference-between-bifunctionx-x-and-binaryoperatorx](https://stackoverflow.com/questions/52993929/difference-between-bifunctionx-x-and-binaryoperatorx)