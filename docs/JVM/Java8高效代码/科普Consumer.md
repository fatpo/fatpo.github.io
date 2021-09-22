# 1、科普 Consumer
这就是lambda表达式的那四件套：`Function`、`Supplier`、`Consumer`、`Predicate` （四大函数式接口）之一。

主要是提供一个`消费者函数`。

示例代码：
```
import java.util.Arrays;
import java.util.List;
import java.util.function.Consumer;


public class TestDemo3 {
    public static void main(String[] args) {
        Consumer<String> consumer = v -> System.out.println(v);

        List<String> ans = Arrays.asList("a", "b", "c", "d", "e", "f");
        ans.forEach(consumer);
    }
}

```
结果：
```
a
b
c
d
e
f
```
# 2、那为什么需要 Consumer

我在SO上面搜一下`When we should use Consumer in Java 8?`，期待能出现类似`When we should use Supplier in Java 8?`这样的好回答。

很不幸，并不能找到问题。

看来大家都没这个疑惑，其实国哥想了下，我用下面的代码也行：
```
Consumer<String> consumer = v -> System.out.println(v);

List<String> ans = Arrays.asList("a", "b", "c", "d", "e", "f");
ans.forEach(consumer);
```
用下面的代码也行：
```
List<String> ans = Arrays.asList("a", "b", "c", "d", "e", "f");
for (String i : ans){
    System.out.println(i);
}
```
无非是第一种更贴合lambda表达式，更有`函数式编程那味儿`。
