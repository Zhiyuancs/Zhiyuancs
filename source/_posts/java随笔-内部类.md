title: java随笔--内部类
tags:
  - JAVA随笔
categories:
  - 学习笔记
date: 2019-10-19 10:00:00
---

### 一、成员内部类

​		内部类可以用来实现多继承。成员内部类可以访问外部类的所有成员变量，包括private修饰的。

​		内部类跟外部类有一个很重要区别：内部类可以用private修饰，而外部类是不能使用private修饰的。如果内部类仅仅在类内部使用时，使用private修饰后，就可以更好的隐藏内部信息。

​		当需要使用到内部类的时候，还是推荐使用`getInnerInstance`的方式来获取，特别是当内部类只有无参构造器的时候

```java
public class Outer {
    private int num;
    private Inner inner;

    Outer(){
        num = 1;
        inner = new Inner();
    }

    public void print(){
        System.out.println("Outer.print()");
        System.out.println(inner.num);
        System.out.println(num);
    }
    //获取内部类对象
    public Inner getInnerInstance() {
        return new Inner();
    }

    class Inner{
        private int num;

        Inner(){
            num = 2;
        }

        public void print(){
            System.out.println("Inner.print()");
            System.out.println(this.num);
            System.out.println(Outer.this.num);
        }
    }
}
```



```java
public class Test {
    public static void main(String[] args) {
        Outer outer = new Outer();
        Outer.Inner inner = outer.new Inner();
        outer.print();
        inner.print();
    }
}
```

​		外部类访问内部类是先生成内部类实例，然后就能访问所有方法和属性，内部类访问外部类方法和属性则直接使用Outer.属性/方法名 即可。

### 二、局部内部类

局部内部类就是定义在代码块、方法体内、作用域（使用花括号“{}”括起来的一段代码）内的类。局部内部类有以下特性：

1. 局部内部类只能在代码代码块、方法体内和作用域中使用。
2. 局部内部类同样可以无限制调用外部类的方法和属性。
3. 可以使用abstract修饰，声明为抽象类。

```java
public class Outer {
    public static final int TOTAL_NUMBER = 5;
    public int id = 123;
    public void func() {
        final int age = 15;
        String str = "http://www.weixueyuan.net";
        class Inner {
            public void innerTest() {
                System.out.println(TOTAL_NUMBER);
                System.out.println(id);
                // System.out.println(str);不合法，只能访问本地方法的final变量
                System.out.println(age);
            }
        }
        new Inner().innerTest();
    }
    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.func();
    }
}
```

### 三、匿名内部类

```java
new 父类（参数列表）|实现接口（） {
//匿名内部类的内部定义
}
```

```java
public class AnonymousTest {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            public void run() {
                for (int i = 0; i < 10; i++) {
                    try {
                        sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(i);
                }
            }
        }).start();
    }
}
```

这里创建了一个继承于Thread的匿名内部类，覆盖了其中的 run方法，并创建了一个实例返回给了t，然后再调用run方法，可以看到，匿名内部类只能存在一个实例对象，因为new过一次就无法再创建了。匿名内部类不仅可以继承于类，也可以实现于接口：

```java
public class AnonymousTest {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            public void run() { …… }).start();
    }
}
```

当我们创建的类只需要一个实例的时候可以使用匿名内部类，比如说在多线程中，要使用多线程，一般先继承Thread类或者实现Runnable接口，然后再去调用它的方法，而每个任务一般都不一样，每次都新建一个类显然会很难管理，因为每个类只用一次就丢掉了，这个时候使用匿名内部类就很方便了，不仅不需要管理一堆一次性类，而且创建起来简单粗暴。

```
使用匿名内部类还是有很多限制的：
1、匿名内部类必须是继承一个类或者实现一个接口，但是两者不可兼得，同时也
只能继承一个类或者实现一个接口。
2、匿名内部类不能定义构造函数。
3、匿名内部类中不能存在任何的静态成员变量和静态方法。
4、匿名内部类是特殊的局部内部类，所以局部内部类的所有限制同样对匿名内部类生效。
5、匿名内部类不能是抽象的，它必须要实现继承的类或者实现的接口的所有抽象方法。
```

```java
//通过初始化块来实现匿名内部类的初始化
//匿名内部类只能定义在方法等内部
public class AnonymousTest{
	public static void main(String[] args){
		Human human = new Human(){
			private String name;
			{
				name="human";
			}
			@Override
			public void walk(){……}
		};
		human.walk();
	}
}
```

注意，如果匿名内部类需要使用外部的参数或者变量，那么必须使用final修饰，因为内部类使用的其实是参数的拷贝，并不是参数本身，为了更明显的表明参数不可变，编译器会要求使用final关键字来修饰需要使用的变量。

```java
public class AnonymousTest {
    public static void main(String[] args) {
        Human human = new AnonymousTest().getHumanInstance("Frank");
        human.walk();
    }

    public Human getHumanInstance(final String name){
        return new Human() {
            private String nameA;
            {
                nameA = name;
            }
            @Override
            public void walk() {
                System.out.println(nameA + " walk.");
            }
        };
    }
}
```

### 四、静态内部类

它的创建不依赖外部类，创建内部类的实例不需要像普通内部类一样先创建外部类实例才能创建。它不能无限制访问外部类的方法和成员变量，**只能访问静态成员变量和静态方法**。

```java
public class Caculate {

    //定义一个pair类来将两个数捆绑
    static class Pair{
        private int first;
        private int second;

        public Pair(int first, int second) {
            this.first = first;
            this.second = second;
        }

        public int getFirst() {
            return first;
        }

        public int getSecond() {
            return second;
        }
    }

    //获取一个int数组中的最大和最小值
    public static Pair getMaxMin(int[] values){
        int max = Integer.MIN_VALUE;
        int min = Integer.MAX_VALUE;
        for (int i:values){
            if (min > i) min = i;
            if (max < i) max = i;
        }
        return new Pair(min,max);
    }

    public static void main(String[] args){
        int[] list = {1,3,5,2,77,23,25};
        Caculate.Pair pair = Caculate.getMaxMin(list);
        System.out.println(pair.getFirst());
        System.out.println(pair.getSecond());
        System.out.println(pair.first);
        System.out.println(pair.second);
    }
}
```

