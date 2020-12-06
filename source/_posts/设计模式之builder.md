---
title: 设计模式之builder
date: 2020-12-06 21:17:33
tags: 设计模式专栏
category: 设计模式专栏
---

# builder 应用场景
```dtd
构建复杂对象
```
举例：
```dtd
一个 class Person 有 30 个属性

构造方法：要传入 30 个入参，太累了

采用 builder 模式

Person.build外形().build家境().build性格().build社会关系()

这样子，初始化一个 Person，可以只创建关心的属性
```

代码如下：
```dtd
https://github.com/fatpo/oyDesignPatterns/tree/main/src/main/java/design/builder
```

# builder 经典模式

背景：
```dtd
去电脑城配电脑

你可以和导购说: 我要一台低端电脑

也可以和导购说：我要一台高端电脑

电脑组成：CPU，内存，显示器，键盘等
```

UML 图：
{% asset_img 设计模式之builder.png This is an image %}

示例代码：
```java
package design.builder.v1;

import lombok.Data;

public class Main {
    public static void main(String[] args) {

        // 告诉导购你要高端机器
        Director director = new Director(new HighComputer());
        director.makeComputer();
        System.out.println(director.getComputer());

        // 告诉导购你要低端机器
        Director director2 = new Director(new LowComputer());
        director2.makeComputer();
        System.out.println(director2.getComputer());
    }
}

class Director {
    private BuildComputer buildComputer;

    public Director(BuildComputer buildComputer) {
        this.buildComputer = buildComputer;
    }

    public void makeComputer() {
        this.buildComputer.buildCpu();
        this.buildComputer.buildDisplay();
        this.buildComputer.buildKeyboard();
        this.buildComputer.buildMemory();
    }

    public Computer getComputer() {
        return this.buildComputer.getComputer();
    }
}

@Data
class Computer {
    private String cpu;
    private String memory;
    private String display;
    private String keyboard;
}

abstract interface BuildComputer {
    abstract public void buildCpu();

    abstract public void buildMemory();

    abstract public void buildDisplay();

    abstract public void buildKeyboard();

    Computer getComputer();
}

class HighComputer implements BuildComputer {
    Computer computer;

    public HighComputer() {
        computer = new Computer();
    }

    @Override
    public void buildCpu() {
        computer.setCpu("I5");
    }

    @Override
    public void buildMemory() {
        computer.setMemory("8G");
    }

    @Override
    public void buildDisplay() {
        computer.setDisplay("三星");
    }

    @Override
    public void buildKeyboard() {
        computer.setKeyboard("高端键盘");
    }

    @Override
    public Computer getComputer() {
        return this.computer;
    }
}


class LowComputer implements BuildComputer {
    Computer computer;

    public LowComputer() {
        computer = new Computer();
    }

    @Override
    public void buildCpu() {
        computer.setCpu("I3");
    }

    @Override
    public void buildMemory() {
        computer.setMemory("2G");
    }

    @Override
    public void buildDisplay() {
        computer.setDisplay("杂牌显示器");
    }

    @Override
    public void buildKeyboard() {
        computer.setKeyboard("免费键盘");
    }

    @Override
    public Computer getComputer() {
        return computer;
    }
}
```

听说这种模式用的很少，日常开发遇不到，我也确实没遇到过，有趣。

说是用的多是变种模式。

# 变种 builder 模式
背景：
```dtd
有一个 Person 类，假设里面有 30 个属性

全部都是不可变属性，都得加 final

并且 name 和 sex 两个属性是必须的，其他 28 个属性不必须

请设计该 Person
```

设计如下：
```java
package design.builder.v2;

public class Main {
    public static void main(String[] args) {
        Person person = new Person("fatpo", "man");
        System.out.println(person);
    }
}


/**
 * 不可变对象，Person
 */
class Person {
    /**
     * 不可变，必须
     */
    private final String name;

    /**
     * 不可变，必须
     */
    private final String sex;

    /**
     * 不可变，非必须
     */
    private final Integer age;

    /**
     * 不可变，非必须
     */
    private final String shoes;

    /**
     * 不可变，非必须
     */
    private final String clothes;

    /**
     * 不可变，非必须
     */
    private final String money;

    /**
     * 不可变，非必须
     */
    private final String car;

    /**
     * 不可变，非必须
     */
    private final String house;

    /**
     * 不可变，非必须
     */
    private final String career;

    Person(String name, String sex, Integer age, String shoes, String clothes, String money, String car, String house, String career) {
        this.name = name;
        this.sex = sex;
        this.age = age;
        this.shoes = shoes;
        this.clothes = clothes;
        this.money = money;
        this.car = car;
        this.house = house;
        this.career = career;
    }

    Person(String name, String sex) {
        this(name, sex, null, null, null, null, null, null, null);
    }
}
```

追加需求：
```dtd
如果偶尔要传入：car 和 house，构造方法调用更简单，而不是如下调用：

        Person person2 = new Person("fatpo", "man", null,null,null,null,"bmw", "shenzen", null);
        System.out.println(perso2);

不仅麻烦，还容易传错了！
```

设计 2：

增加一个 Person 的静态类 builder，它负责隔离必须和非必须属性，对所需的属性进行定制即可，编码简单，美观大方。

有一说一，这个变种的 builder 模式，在日常开发中，确实有遇到过（比如推荐项目的 recommendContext，各种 build），还挺实用的。

```java
package design.builder.v3;

public class Main {
    public static void main(String[] args) {
        Person person = new Person.Builder("fatpo", "man")
                .age("18")
                .money("10000000")
                .car("bmw")
                .career("IT")
                .build();

        System.out.println(person);
    }
}


/**
 * 不可变对象，Person
 */
class Person {
    /**
     * 不可变，必须
     */
    private final String name;

    /**
     * 不可变，必须
     */
    private final String sex;

    /**
     * 不可变，非必须
     */
    private final String age;

    /**
     * 不可变，非必须
     */
    private final String shoes;

    /**
     * 不可变，非必须
     */
    private final String clothes;

    /**
     * 不可变，非必须
     */
    private final String money;

    /**
     * 不可变，非必须
     */
    private final String car;

    /**
     * 不可变，非必须
     */
    private final String house;

    /**
     * 不可变，非必须
     */
    private final String career;


    private Person(Builder builder) {
        this.name = builder.name;
        this.sex = builder.sex;
        this.age = builder.age;
        this.shoes = builder.shoes;
        this.clothes = builder.clothes;
        this.money = builder.money;
        this.car = builder.car;
        this.house = builder.house;
        this.career = builder.career;
    }

    public static class Builder {
        private final String name;
        private final String sex;
        private String age;
        private String shoes;
        private String clothes;
        private String money;
        private String car;
        private String house;
        private String career;

        public Builder(String name, String sex) {
            this.name = name;
            this.sex = sex;
        }

        public Builder age(String age) {
            this.age = age;
            return this;
        }

        public Builder shoes(String shoes) {
            this.shoes = shoes;
            return this;
        }

        public Builder clothes(String clothes) {
            this.clothes = clothes;
            return this;
        }

        public Builder money(String money) {
            this.money = money;
            return this;
        }

        public Builder car(String car) {
            this.car = car;
            return this;
        }

        public Builder career(String career) {
            this.career = house;
            return this;
        }

        public Builder house(String house) {
            this.house = house;
            return this;
        }

        public Person build() {
            return new Person(this);
        }
    }

}
```