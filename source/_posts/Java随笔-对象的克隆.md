title: Java随笔--对象的克隆
tags:
  - JAVA随笔
categories:
  - 学习笔记
date: 2019-10-17 15:32:00
---

java的引用类型（包括类、接口、数组等复杂类型）是无法通过等号直接赋值的。

```java
public class GoodsTest {
    public static void main(String[] args){
        Goods goodsA = new Goods("GoodsA",20);
        Goods goodsB = goodsA;
        System.out.println("Before Change:");
        goodsA.print();
        goodsB.print();

        goodsB.setTitle("GoodsB");
        goodsB.setPrice(50);
        System.out.println("After Change:");
        goodsA.print();
        goodsB.print();
    }
}
//修改goodsB,goodsA也会发生变化，两个变量指向同一个地址
```

如果要实现对象的复制，需要重写clone方法，所有类都继承自Object类，该类有一个protected的clone方法。

```java
//要使用克隆方法需要实现Cloneable接口
public class Goods implements Cloneable{
    private String title;
    private double price;

    public Goods(String aTitle, double aPrice){
        title = aTitle;
        price = aPrice;
    }
    
    //这里重载了接口的clone方法
    @Override
    protected Object clone(){
        Goods g = null;
        try{
            //使用super来引用父类的成分，用this来引用当前对象
            g = (Goods)super.clone();
        }catch (CloneNotSupportedException e){
            System.out.println(e.toString());
        }
        return g;
    }
```

如果一个类的成员变量包含另一个类的对象，使用上述方法只能克隆该类的非引用类型，引用类型仍然指向同一个地址。解决这个问题需要使用深克隆（序列化、反序列化）。下面例子中Cart和Goods类都需要实现Serializable接口。

```java
import java.io.*;

public class Cart implements Serializable{
    //实例域
    Goods goodsList = new Goods("",0);//简单起见，这里只放了一个商品
    double budget = 0.0;//预算

    //构造函数
    public Cart(double aBudget){
        budget = aBudget;
    }

    public Object deepClone() throws IOException, OptionalDataException,ClassNotFoundException {
        // 将对象写到流里
        ByteArrayOutputStream bo = new ByteArrayOutputStream();
        ObjectOutputStream oo = new ObjectOutputStream(bo);
        oo.writeObject(this);
        // 从流里读出来
        ByteArrayInputStream bi = new ByteArrayInputStream(bo.toByteArray());
        ObjectInputStream oi = new ObjectInputStream(bi);
        return (oi.readObject());
    }
}

public class Goods implements Serializable{……}
```

