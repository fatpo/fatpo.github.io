# 1、科普 UnaryOperator
我们知道，四大函数式接口：`Function`、`Supplier`、`Consumer`、`Predicate`。

我们也知道，二元操作函数 `BinaryOperator`，[传送门](https://fatpo.github.io/#/JVM/Java8高效代码/科普BinaryOperator)。

但今天不说它们，今天看的是 `UaryOperator`，就是一个封装好的，和`Function`几乎一样的lambda表达式。

其实看也能看出来，UnaryOperator 和 BinaryOperator 非常相似，Unary是`一元`，Binary是`二元`。


# 2、使用demo
```
import java.util.function.Function;
import java.util.function.UnaryOperator;


public class TestDemo3 {
    public static void main(String[] args) {
        UnaryOperator<Integer> f1 = value -> value * 10;
        Function<Integer, Integer> f2 = value -> value * 10;
        System.out.println(f1.apply(10));
        System.out.println(f2.apply(10));

        // 错误代码
        // UnaryOperator<Integer> f3 = value -> value * 10 + "____return";
        
        // 合理代码
        Function<Integer, String> f4 = value -> value * 10 + "____return";
    }
}
```


# 2、那么UnaryOperator和Function的区别

(⊙o⊙)… 这个其实国哥问过类似的问题：`那么BinaryOperator和BiFunction的区别？`

当初国哥给的答案是(在入参和返回值类型一致的情况下)：
* BinaryOperator简洁，编码好看
* 可读性更强
* 开发者可以更为懒惰

这次的`UnaryOperator和Function的区别`也是如此，不多说。

# 3、参考
* [fatpo: 科普BinaryOperator](https://fatpo.github.io/#/JVM/Java8高效代码/科普BinaryOperator)