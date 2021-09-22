# 1、科普 BinaryOperator
这就是lambda表达式的那四件套：`Function`、`Supplier`、`Consumer`、`Predicate`。

当然今天看的是 `BinaryOperator`，就是一个封装好的，两个参数之间的lambda表达式。

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