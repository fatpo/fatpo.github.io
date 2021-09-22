# 1、科普Function
这就是lambda表达式的那四件套：`Function`、`Supplier`、`Consumer`、`Predicate`之一（四大函数式接口）。

Function是一个入参，一个返回值：
```
    private static void compute1() {
        Function<Integer, Integer> f1 = value -> value * value;
        System.out.println(f1.apply(10));
    }

```
# 2、科普BiFunction
BiFunction仅仅是比Function多一个入参：双入参，一个返回值。
```
    private static void compute2() {
        BiFunction<Integer, Integer, Integer> f2 = (value1, value2) -> value1 * value2;
        System.out.println(f2.apply(10, 20));
    }
```

它就是函数，感觉有点像是废话。

函数也是一个对象，可以作为入参的，有python那味儿。

给一个经典使用场景，可能看官您会更加明白我的废话：
```
public static void main(String[] args) {
    compute4(1, 2, (a, b) -> a * b);
    compute4(1, 2, (a, b) -> a / b);
    compute4(1, 2, (a, b) -> a + b);
    compute4(1, 2, (a, b) -> a - b);
}

private static void compute4(Integer t1, Integer t2, BiFunction<Integer, Integer, Integer> function) {
    int res = function.apply(t1, t2);
    System.out.println("compute4: " + res);
}
```
结果：
```
compute4: 2
compute4: 0
compute4: 3
compute4: -1
```




为了更好地研究怎么写一个TriFunction，我们看看BiFunction的代码：
```java
package java.util.function;

import java.util.Objects;

/**
 * Represents a function that accepts two arguments and produces a result.
 * This is the two-arity specialization of {@link Function}.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #apply(Object, Object)}.
 *
 * @param <T> the type of the first argument to the function
 * @param <U> the type of the second argument to the function
 * @param <R> the type of the result of the function
 *
 * @see Function
 * @since 1.8
 */
@FunctionalInterface
public interface BiFunction<T, U, R> {

    /**
     * Applies this function to the given arguments.
     *
     * @param t the first function argument
     * @param u the second function argument
     * @return the function result
     */
    R apply(T t, U u);

    /**
     * Returns a composed function that first applies this function to
     * its input, and then applies the {@code after} function to the result.
     * If evaluation of either function throws an exception, it is relayed to
     * the caller of the composed function.
     *
     * @param <V> the type of output of the {@code after} function, and of the
     *           composed function
     * @param after the function to apply after this function is applied
     * @return a composed function that first applies this function and then
     * applies the {@code after} function
     * @throws NullPointerException if after is null
     */
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t, U u) -> after.apply(apply(t, u));
    }
}
```
感觉不难，我研究下怎么写Tri。

# 3、创造TriFunction
TriFunction.java: 
```
public interface TriFunction<T, U, P, R> {
    R apply(T t, U u, P p);
}
```
试一下，结果验证：
```
public class Demo{
    public static void main(String[] args) {
        compute1();
        compute2();
        compute3();
    }

    private static void compute1() {
        Function<Integer, Integer> f1 = value -> value * value;
        System.out.println(f1.apply(10));
    }

    private static void compute2() {
        BiFunction<Integer, Integer, Integer> f2 = (value1, value2) -> value1 * value2;
        System.out.println(f2.apply(10, 20));
    }


    private static void compute3() {
        TriFunction<Integer, Integer, Integer, Integer> f3 = (value1, value2, value3) -> value1 * value2 * value3;
        System.out.println(f3.apply(10, 20, 30));
    }
}
```

输出：
```
100
200
6000
```
结果符合预期，稍微润色下TriFunction，我们也学BiFunction加一个andThen():
```
import com.sun.istack.internal.NotNull;

import java.util.Objects;
import java.util.function.Function;

public interface TriFunction<T, U, P, R> {
    R apply(T t, U u, P p);

    default <V> TriFunction<T, U, P, V> andThen(@NotNull Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t, U u, P p) -> after.apply(apply(t, u, p));

    }
}

```