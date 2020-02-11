title: java随笔--正则表达式
author: Zhiyuan
tags:
  - JAVA随笔
categories:
  - 学习笔记
date: 2019-09-22 17:00:00

---

题目地址：https://leetcode-cn.com/problems/string-to-integer-atoi/

```java
import java.util.regex.*;
class Solution {
       public static int myAtoi(String str) {
       str=str.trim();
        
        String pattern="^[\\+\\-\\d]\\d*";//正则表达式，表示以正号或负号或数字开头，且后面是0个或多个数字
        Pattern p=Pattern.compile(pattern);
        Matcher m=p.matcher(str);
        
        String res="";
        if(m.find()){//能匹配到
            res=str.substring(m.start(),m.end());
        }else{//不能匹配到
            return 0;
        }
        
        //能匹配到但只有一个+-号，也返回0
        if(res.length()==1&&(res.charAt(0)=='+'||res.charAt(0)=='-')){
            return 0;
        }
        
        try{
            int r=Integer.parseInt(res);
            return r;
        }catch(Exception e){
            return res.charAt(0)=='-'?Integer.MIN_VALUE:Integer.MAX_VALUE;
        }
	}
}
```

在使用java正则表达式时要注意：java中"\\\\"表示"\\","\\\\+"表示"+",因为"+"需要转义，

### lookingAt()

lookingAt()对前面的字符串进行匹配,只有匹配到的字符串在最前面才返回true 

```java
Pattern p=Pattern.compile("\\d+"); 
Matcher m=p.matcher("22bb23"); 
m.lookingAt();//返回true,因为\d+匹配到了前面的22 
Matcher m2=p.matcher("aa2223"); 
m2.lookingAt();//返回false,因为\d+不能匹配前面的aa 
```

### **Mathcer.start()/ Matcher.end()/ Matcher.group()**

start()返回匹配到的子字符串在字符串中的索引位置. 
end()返回匹配到的子字符串的最后一个字符在字符串中的索引位置. 
group()返回匹配到的子字符串 

start(),end(),group()均有一个重载方法它们是start(int i),end(int i),group(int i)专用于分组操作,Mathcer类还有一个groupCount()用于返回有多少组. 

```java
Pattern p=Pattern.compile("\\d+"); 
Matcher m=p.matcher("我的QQ是:456456 我的电话是:0532214 我的邮箱是:aaa123@aaa.com"); 
while(m.find()) { 
     System.out.println(m.group()); 
} 
输出：
456456 
0532214 
123 
 //或者   
while(m.find()) { 
     System.out.println(m.group()); 
     System.out.print("start:"+m.start()); 
     System.out.println(" end:"+m.end()); 
} 
输出：
456456 
start:6 end:12 
0532214 
start:19 end:26 
123 
start:36 end:39 
```

可以看出，每执行一次find()，matcher对应的分组都会自动加一。start(),end(),group()三个方法的值都会改变,匹配到的子字符串的信息,以及它们的重载方法,也会改变成相应的信息. 

后面遇到有关正则表达式问题再进行补充。