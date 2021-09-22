# 1、科普 Predicate
这就是lambda表达式的那四件套：`Function`、`Supplier`、`Consumer`、`Predicate` （四大函数式接口）之一。

主要是提供一个`断言函数`，好像用于测试比较多。

示例代码：
```
import java.util.function.Predicate;


public class TestDemo3 {
    public static void main(String[] args) {
        Predicate<Integer> predicate = new Predicate<Integer>() {
            @Override
            public boolean test(Integer integer) {
                if (integer > 10) {
                    return true;
                }
                return false;
            }
        };

        System.out.println(predicate.test(11));
        System.out.println(predicate.test(10));
        System.out.println(predicate.test(9));
    }
}

```
结果：
```
true
false
false
```
# 2、那为什么需要 Predicate

这个和`Consumer`一样，给国哥的感觉就是： 更贴合lambda表达式，更有`函数式编程那味儿`。
