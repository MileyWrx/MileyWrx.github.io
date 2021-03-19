---
layout:     post
title:      "01 Stack and Queue"
subtitle:   "chapter 2 of Princeton Algorithm"
date:       2021-03-14
author:     "Miley"
header-img: "img/post-bg-rwd.jpg"
tags:
    - Data Structures
    - Java
---



## Catalog


1.  [Linked List Implementation](#利用链表实现栈)
2.  [Resizing Array Implementation](#利用扩容数组实现栈)
3.  [RequireJS & AMD](#requirejs--amd)
4.  [SeaJS & CMD](#seajs--cmd)


> Stack: Examine the item most reccently added ("last in forst out").

## 利用链表实现栈

```js
public class Stack<Item>
{
    private Node first;
    private int N;
    //inner class
    private class Node
    {
        Item item;
        Node next;
    }
    
    public boolean isEmpty() 
    {return first == null;}
    
    public int size()
    { return N; }
    
    public void push(Item item){
        Node oldFirst = first;
        first = new Node();
        first.item = item;
        first.next = oldFirst;
        N++;
    }
    
    public Item pop(){
        Item item = first.item;
        first = first.next;
        N--;
        return item;
    }
}
```
一个有N个元素的Stack,每个元素占用的内存：  
* 16 bytes for class overhead `Stack`  
* 8 bytes for inner class extra overhead. As the inner class `Node` is not static, so it has a reference to the outer class.  
* 8 bytes for the reference to String
* 8 bytesfor the reference to Node
Total: 40 bytes per stack node
ref: [Java:Size of inner class-Stackoverflow](https://stackoverflow.com/questions/12193116/java-size-of-inner-class)  

### More:
#### 1.1
new Object()将占用多少byte的内存空间？
```js
原生类型(primitive type)的内存占用
Primitive Type      Memory Required(bytes)
    boolean                      1
    byte                         1
    short                        2
    char                         2
    int                          4
    float                        4
    long                         8
    double                       8
```
#### 1.2
静态类与非静态类
In **static** method, The memory of a static method is fixed in the ram, for this reason we don’t need the object of a class in which the static method is defined to call the static method. To call the method we need to write the name of the method followed by the class name.   
```js
class GFG{
 public static void geek()
 { }
}
// calling
GFG.geek();

```
In **non-static** method, the memory of non-static method is not fixed in the ram, so we need class object to call a non-static method. To call the method we need to write the name of the method followed by the class object name.
```js
class GFG{
 public void geek()
 { }
}
// creating object
GFG g = new GFG();
// calling
g.geek();
```

## 利用扩容数组实现栈
增加了resize函数：
```js
public class Stack<Item>
{
    private Item[] s;
    private int N = 0;
    
    public Stack(int capacity)
    {s = (Item[]) new Object[capacity];}
    
    public boolean isEmpty() 
    {return N == 0;}
    
    public int size()
    {return N;}
    
    private void resize(int capacity){
        Item[] copy = (Item[]) new Object[capacity];
        for(int i = 0; i < s.length; i++){
            copy[i] = s[i];
        }
        s = copy;
    }
    
    public void push(Item item){
        if(N == s.length) resize(2 * s.length)
        s[N++] = item;
    }
   
    public Item pop(){
        Item item = s[--N];
        s[N] = null; // avoid loitering
        if(N > 0 && N == s.length/4) resize(s.length/2);
        return item;
    }
}
```
### More
#### 2.1 what is loitering?
> Loitering: Holding a reference to an object when it is no longer needed. -- _Algorithm_, Priceton Universily  

example:
```js
// loitering
public String pop()
{ return s[--N]; }

// avoid loitering
public String pop()
{
 String item = s[--N];
 s[N] = null;
 return item;
} 
```

#### 2.2 Iterator in Java
1.如果一个类Implements Iterable,那么这个类需要有iterator()函数，返回这个类的Itertor  
2.如果一个类Implements Iterator,那么这个类需要有:
 * boolean hasNext(), current是不是最后一个元素
 * void next(), 返回当前的元素，并把指针向后移动一个
Example:  
```js
// return an iterator over items in order from front to back
    public Iterator<Item> iterator(){
        return new DequeueIterator();
    }

    private class DequeueIterator implements Iterator <Item>{
        private Node current = first;
        
        // 判断集合中是否有元素，如果有元素可以迭代，就返回true
        public boolean hasNext()
        {return current != null;}

        //返回现在的数组，使指针指向下一个
        public Item next(){
            if(!hasNext()) throw new NoSuchElementException();
            Item ret = current.item;
            current = current.next;
            return ret;
        }
        
        public void remove()
        {throw new UnsupportedOperationException();}

    }
```


> RequireJS is a JavaScript file and module loader. It is optimized for in-browser use, but it can be used in other JavaScript environments
 

References
[Princeton《算法》笔记](https://blog.csdn.net/littleorange6/article/details/84301824) 是 RequireJS 在推广过程中对模块定义的规范化产出。
