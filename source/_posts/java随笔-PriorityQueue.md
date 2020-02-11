title: java随笔--PriorityQueue
tags:
  - JAVA随笔
categories:
  - 学习笔记
date: 2019-10-30 22:00:00
---

java的PriorityQueue是基于堆的优先队列。下面是其构造方法：

```java
//如果Collection已排序，则根据它的顺序进行排序
PriorityQueue(Collection<? extends E> c)

//自定义一个比较器，根据比较器进行优先级判断
PriorityQueue(Comparator<? super E> comparator)
```

第二种构造方法使用的较多，比较器的创建可以使用**匿名内部类**的方法指定。通过重写Comparator的compare方法来实现。下面通过例子来解释：

https://leetcode-cn.com/problems/top-k-frequent-elements/

```java
public List<Integer> topKFrequent(int[] nums, int k) {
        List<Integer> list=new ArrayList<>();
        if(nums.length==1){
            list.add(nums[0]);
            return list;
        }
        Map<Integer,Integer> hashmap=new HashMap<Integer,Integer>();
        for(int i:nums){
            if(!hashmap.containsKey(i)){
                hashmap.put(i,1);
            }else{
                int k1=hashmap.get(i)+1;
                hashmap.replace(i,k1);
            }
        }
        //new Comparator<T>() 实现接口的匿名内部类
        //泛型不可以省略
        PriorityQueue<Integer> queue=new PriorityQueue<Integer>(new Comparator<Integer>(){
        /*	public修饰符不可以省略
        	原因未知
        	这里是小根堆，使用自然排序（从小到大），当n1>n2，返回1
       	*/
       public int compare(Integer n1,Integer n2){
           if(hashmap.get(n1)>hashmap.get(n2))
               return 1;
           else if(hashmap.get(n1)==hashmap.get(n2))
               return 0;
           else
               return -1;
       } 
        });
        for(Integer num:hashmap.keySet()){
            queue.add(num);
            if(queue.size()>k){
                queue.poll();
            }
        }
        while(queue.size()>=1){
            list.add(queue.poll());
        }
    	//逆转
        Collections.reverse(list);
        return list;
    }
```

```java
//使用lambda表达式初始化
//(x, y) -> x – y  接受2个参数(数字),并返回他们的差值
PriorityQueue<Integer> heap =
         new PriorityQueue<Integer>((n1, n2) -> hashmap.get(n1) - hashmap.get(n2));
```

