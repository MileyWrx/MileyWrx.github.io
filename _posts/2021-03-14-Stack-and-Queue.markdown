---
layout:     post
title:      "01 Stack and Queue"
subtitle:   "Chapter 2 of _Algorithm(I)_, Princeton Univercity"
date:       2021-03-14
author:     "Miley"
header-img: "img/post-bg-rwd.jpg"
tags:
    - Data Structures
    - Java
---



## Contents

1.  [Stack: Linked List Implementation](#利用链表实现栈)
2.  [Stack: Resizing Array Implementation](#利用扩容数组实现栈)
3.  [Queue: Linked List Implementation](#利用链表实现队列)
4.  [Application:Arithmetic expression calculations](#算数表达式求值)

[本章节作业代码](https://github.com/MileyWrx/Homework_PrincetonAlgorithm/tree/master/week2)

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
#### 1.1 内存占用
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
#### 1.2 静态类与非静态类
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
增加了resize函数:  
当集合大小增加至数组大小时，将集合移动到另一个原数组两倍大的数组中；当集合大小减小至小于数组的四分之一时，将集合移动到另一个原数组一半大的数组中。
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
#### 2.2 Recursive
example:
```js
static int gcd(int p, int q) {
 if (q == 0) return p;
 else return gcd(q, p % q);
 }
```

#### 2.3 Iterator in Java
1. 如果一个类Implements Iterable,那么这个类需要有iterator()函数，返回这个类的Itertor。eg: 作业中 public Dequeqe<Item> implements Iterable<Item>  
2. 如果一个类Implements Iterator,那么这个类需要有:
 * boolean hasNext(): 判断current是不是最后一个元素
 * void next(): 返回当前的元素，并把指针向后移动一个
3. 自己实现一个Iterator：
 * 将这个Iterator设置为你要Iterate的类的Inner Class
 * 由于内部类可以访问外部类的内部变量，在iterator中设置一个current，一般指向外部类需要iterate的的集合的头指针  
 
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
Iterator代表这个类表示拥有一系列元素的集合，可以被便历。使用Iterator就可以使用Java中的foreach loop：
```js
for(Item item: stack){
    ···
}
```
## 利用链表实现队列

> RequireJS is a JavaScript file and module loader. It is optimized for in-browser use, but it can be used in other JavaScript environments

```js
public class Queue<Item>
{
    private Node first;
    private Node last;
    private int N;
    
    private class Node{
        Item item;
        Node next;
    }
    
    public boolean isEmpty() { return first == null; }
    public int size() { return N; }
    public void enqueue(Item item){
        Node oldLast = last;
        last = new Node();
        last.item = item;
        // 注意：check队列是否为空！
        if (isEmpty()) {
            first = last;
        } else {
            oldLast.next = last;
        }
        N++;
    }
    
    public Item dequeue(){
        Item item = first.item;
        first = first.next;
        if (size() == 1) { last = null; }
        N--;
        return item;
    }
}
```
 
## 算数表达式求值
例如：( 1 + ( ( 2 + 3 ) * ( 4 + 5 ) ) )  在这里我们采用E.W.Dijkstra发明的算法，
 * value: push onto the value stack
 * operator: push onto the operator stack
 * Left parenthesis: ignore
 * Right parenthesis: pop operator and two values, push the result of calculation onto the oprand (value) stack  
  
```js
// Dijkstra 双栈算术表达式求值算法:
import edu.princeton.cs.algs4.StdIn;
import edu.princeton.cs.algs4.StdOut;
import edu.princeton.cs.algs4.Stack;

public class Evaluate
{
    public static void main(String[] args)
    {
        Stack<String> ops = new Stack<String>();
        Stack<Double> vals = new Stack<Double>();
        while (!StdIn.isEmpty())
        {
            String s = StdIn.readString();
            if (s.equals("(")) ;
            else if (s.equals("+")) ops.push(s);
            else if (s.equals("-")) ops.push(s);
            else if (s.equals("*")) ops.push(s);
            else if (s.equals("/")) ops.push(s);
            else if (s.equals("sqrt")) ops.push(s);
            else if (s.equals(")"))
            {
                String op = ops.pop();
                double v = vals.pop();
                if (op.equals("+")) v = vals.pop() + v;
                else if (op.equals("-")) v = vals.pop() - v;
                else if (op.equals("*")) v = vals.pop() * v;
                else if (op.equals("/")) v = vals.pop() / v;
                else if (op.equals("sqrt")) v = Math.sqrt(v);
                vals.push(v);
            }
            else vals.push(Double.parseDouble(s));
        }
        StdOut.println(vals.pop());
    }
}

```
#### Java的数组不支持Generic泛型


## References
1. [Princeton《算法》(上) 笔记 - CSDN](https://blog.csdn.net/littleorange6/article/details/84301824)   
2. [普林斯顿算法: 背包、队列和栈](https://libhappy.com/2016/03/algs-1.0/)
3. [TO THE NEW: Why is generic array creation not allowed in java?](https://www.tothenew.com/blog/why-is-generic-array-creation-not-allowed-in-java/)
