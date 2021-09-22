# 1、科普 Supplier
这就是lambda表达式的那四件套：`Function`、`Supplier`、`Consumer`、`Predicate` （四大函数式接口）之一。

主要是提供一个不是`new 对象`的`创建对象`的方式：
```
import java.util.function.Supplier;

class Student {
    public String name;
    public Integer age;
}

public class TestDemo2 {
    public static void main(String[] args) {
        Supplier<Student> supplier = Student::new;
        System.out.println("######## first invoke ########");
        System.out.println(supplier.get().hashCode());
        System.out.println("######## second invoke ########");
        System.out.println(supplier.get().hashCode());
    }
}

```
# 2、那为什么需要Supplier
明明有了`new Object()`为啥还需要 `Supplier`呢？

很幸运，在SO上面看到了合心意的答案：`delayed execution`，延迟执行，多么简洁的四个字!

而且 在生产环境，一般都没什么问题，但是我们要搞测试的时候，用Supplier很方便。
```
方式1：Supplier<LocalDate> currentDate = ()-> LocalDate.now();

方式2：LocalDate currentDate = LocalDate.now();

测试的时候，我们想模拟过了一天的日期，用方式2无法很优雅得解决问题。
```
这里国哥就照搬SO的代码了，这里面有`美的感觉`，欣赏就好：
```java
class Person {
    final String name;
    private final LocalDate dateOfBirth;
    private final Supplier<LocalDate> currentDate;

    // Used by regular production code
    Person(String name, LocalDate dateOfBirth) {
        this(name, dateOfBirth, ()-> LocalDate.now());
    }

    // Visible for test
    Person(String name, LocalDate dateOfBirth, Supplier<LocalDate> currentDate) {
        this.name = name;
        this.dateOfBirth = dateOfBirth;
        this.currentDate = currentDate;
    }

    long getAge() {
        return ChronoUnit.YEARS.between(dateOfBirth, currentDate.get());
    }

    public static void main(String... args) throws InterruptedException {
        // current date 2016-02-11
        Person person = new Person("John Doe", LocalDate.parse("2010-02-12"));
        printAge(person);
        TimeUnit.DAYS.sleep(1);
        printAge(person);
    }

    private static void printAge(Person person) {
        System.out.println(person.name + " is " + person.getAge());
    }
}
```
The output would correctly be:
```
John Doe is 5
John Doe is 6
```
Our unit test can inject the "now" date like this:
```java
@Test
void testGetAge() {
    Supplier<LocalDate> injectedNow = ()-> LocalDate.parse("2016-12-01");
    Person person = new Person("John Doe", LocalDate.parse("2004-12-01"), injectedNow);
    assertEquals(12, person.getAge());
}
```
想起了胡弦老师一句很经典的话：`不好测的代码是不是没写好啊？`。

按照上面的做法，就很好测，很好写。

# 3、参考
* [StackOverflow: when-we-should-use-supplier-in-java-8](https://stackoverflow.com/questions/40244571/when-we-should-use-supplier-in-java-8)