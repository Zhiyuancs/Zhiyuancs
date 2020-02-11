---
title: java随笔--一些注意点和tricks
date: 2019-11-05 18:31:49
tags:
	JAVA随笔
categories:
	学习笔记
---

### split()

String类下的split()方法在根据正则表达式对字符串进行分割时会产生空字符串，在处理时需要进行判断。

```java
//1、空字符串不会被解析
public class test {
	public static void main(String[] args) {
		String str = "1,2,3,4,,,";
		String[] arr = str.split(",");
		for (String string : arr) {
			System.out.println("str"+string);
		}
		System.out.println(arr.length);
	}
}
//输出：str1,str2,str3,str4    4

//2、最后一个分隔符被分的字符串不为空时，其余空字符串可被解析。
public class test {
	public static void main(String[] args) {
		String str = "1,2,3,4,,,5";
		String[] arr = str.split(",");
		for (String string : arr) {
			System.out.println("str"+string);
		}
		System.out.println(arr.length);
	}
}
//输出：str1,str2……str4,str,str,str5     7

//3、limit参数为整数时，只会截取前几个；为0时，正常截取；
//	 为负数时，即使是第一种情况，也会输出空字符串
String[] s = str.split(",",-1);
```

### s.equals(s1)

判断字符串s是否等于s1，应该使用equals方法，不可以通过“==”进行判断。

### HashSet如何检查重复

当你把对象加入`HashSet`时，HashSet会先计算对象的`hashcode`值来判断对象加入的位置，同时也会与其他加入的对象的hashcode值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用`equals()`方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让加入操作成功。

所以Set中存储自己定义的对象时，需要重写hashCode()和equals()方法。

```java
//Person类(name,age两个属性)
//重写hashCode
public int hashCode(){
        final int prime = 31;
        int result = 1;
        result = prime * result + ((age == null) ? 0 : age.hashCode());
        result = prime * result + ((name == null) ? 0 : name.hashCode());
        return result;
    }

//重写equals
public boolean equals(Person obj){
	
}
```

### hashCode方法

如果重写equals()方法，就必须重新定义hashCode方法，以便用户可以将对象插入到散列表(自定义对象的HashMap)。hashCode 方法应该返回一个整型数值（也可以是负数)，并合理地组合实例域的散列码,以便能够让各个不同的对象产生的散列码更加均匀。下面是一个重写hashCode的例子。

```java
public class Employee{
	public int hashCode(){
		return 7*name.hashCode()
			+ 11*new Double(salary).hashCode()
			+ 13*hireDay.hashCode();
	}
}
//如果参数为null,安全起见使用下面的方法
public int hashCode(){
    return 7*Objects.hashCode(name)
        + 11*Double.hashCode(salary)
        + 13*Objects.hashCode(hireDay);
}
//组合多个散列值时可以采用更好的方法
public int hashCode(){
    return Objects.hash(name,salary,hireDay);
}
```

Equals与hashCode的定义必须一致：如果x.equals(y) 返回true, 那么x.hashCode( ) 就必
须与y.hashCode( ) 具有相同的值。例如， 如果用定义的Employee.equals 比较雇员的ID， 那
么hashCode 方法就需要散列ID，而不是雇员的姓名或存储地址。

### 静态导入

import语句不仅可以导入类，还可以导入静态方法和静态域，如：

```java
import static java.lang.System.*
//可以使用System类的静态方法和静态域，而不必加类名前缀
out.println("hello");  //省略了System.out
exit(0);
```

