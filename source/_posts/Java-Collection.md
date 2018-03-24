----------------------
title: Java Collection
----------------------
date: 2018-02-05 10:26:40
tags: collection
categories: JAVA

### Description
&nbsp;&nbsp;In programming, we often need to store data. Although the array is a good choice, but the premise is known to save the number of objects, once the length of the array initialization, can not be changed, and the collection is used to save the dynamic growth data (at compile time can not determine the specific number) of.   
&nbsp;&nbsp;Collection classes are also called container classes.   
- 1.Collection   
A set of elements subject to some kind of rule
   - List : Elements must be in a specific order
   - Set : Can not have duplicate elements
   - Queue : Maintain a queue (first in, first out) order   

- 2.Map   
A set of paired "key-value" objects

### 1.List
&nbsp;&nbsp;The List collection represents an ordered, repeatable collection of elements, each element of the collection has its corresponding sequential index.By default, the element's index is set in the order in which elements are added.   
### 1.1 ArrayList
&nbsp;&nbsp;In order to reduce the number of distribution, improve performance, ArrayList provides the following methods.
- 1.ensureCapacity(int minCapacity): Increment the Array [] array Object [] array length by minCapacity.
- 2.trimToSize(): Adjust ArrayList collection Object [] array length is the current number of elements.
   ```java
    import java.util.*;
    
    public class ListTest{
        public static void main(String[] args){
            List books = new ArrayList();
            // add three elements to the books collection
            books.add(new String("----- the first element -----"));
            books.add(new String("----- the second element -----"));
            books.add(new String("-----the third element -----"));
            System.out.println(books);
    
            // insert the new string object in the second position
            books.add(1 , new String("----- the added element -----"));
            for (int i = 0 ; i < books.size() ; i++ ){
                System.out.println(books.get(i));
            }
    
            // delete the third element
            books.remove(2);
            System.out.println(books);
    
            // judge the specified element in the List collection position: output 1, indicating that in the second place
            System.out.println(books.indexOf(new String("----- the added element -----")));
            // Replace the second element with a new string object
            books.set(1, new String("----- the new element -----"));
            System.out.println(books);
    
            // the second element of the books collection (including) to the third element (not included) is intercepted into subsets
            System.out.println(books.subList(1 , 2));
        }
    }
   ```
### 1.2 Vector
&nbsp;&nbsp;An ancient collection almost identical to the ArrayList usage.Stack is a subclass provided by Vector for simulating the "stack" data structure (LIFO).   
   ```java
    import java.util.*;
    
    public class VectorTest{
        public static void main(String[] args){
            Stack v = new Stack();
            // followed by the three elements into the stack
            v.push("---------- head ----------");
            v.push("---------- body ----------");
            v.push("---------- tail ----------");
    
            System.out.println(v);
    
            // visit the first element
            System.out.println(v.peek());  
    
            // pop the first element
            v.pop();            
            // only left with head and body
            System.out.println(v);
        }
    }
   ```
### 1.3 LinkedList
&nbsp;&nbsp;LinkedList implements List interface, it can be a queue operation, which can be based on the index random access to the elements in the collection. At the same time it also implements the Deque interface, which can use LinkedList as a double-ended queue. Naturally, it can also be used as a "stack."
   ```java
    import java.util.*;
    
    public class LinkedListTest{
        public static void main(String[] args) {          
            LinkedList books = new LinkedList();
            // add a string element to the end of the queue (double-ended queue)
            books.offer("queue rear");
    
            // add a string element to the top of the stack (double-ended queue)
            books.push("top of stack");
    
            // add a string element to the head of the queue (equivalent to the top of the stack)
            books.offerFirst("queue head");
    
            for (int i = 0; i < books.size(); i++ ){
                System.out.println(books.get(i));
            }
    
            // visit the top of the stack element
            System.out.println(books.peekFirst());
            
            // visit the last element of the queue
            System.out.println(books.peekLast());
            
            // the top of the stack element will pop up stack
            books.pop();
            System.out.println(books);
            
            // access and delete the last element of the queue
            System.out.println(books.pollLast());
            
            // only one element in the middle: "top of stack"
            System.out.println(books);
        }
    }
   ```
### 2.Queue
&nbsp;&nbsp;Queue is used to simulate the "queue" this data structure.The head of the queue holds the oldest element in the queue, and the tail of the queue holds the element with the shortest storage time in the queue. The new element is offered at the end of the queue, the poll operation returns the element at the head of the queue, and the queue does not allow random access to the elements in the queue.
### 2.1 PriorityQueue
&nbsp;&nbsp;PriorityQueue is not a more standard queue implementation. PriorityQueue saves the order of queue elements not in the order they were queued, but in the queue elements.
   ```java
    import java.util.*;
    
    public class PriorityQueueTest{
        public static void main(String[] args) {
            PriorityQueue pq = new PriorityQueue();
            // add four elements
            pq.offer(6);
            pq.offer(-3);
            pq.offer(9);
            pq.offer(0);
    
            //  elements arranged in order of size, output额[-3, 0, 6, 9]
            System.out.println(pq);
           
            // access the first element of the queue, in fact, the smallest element in the queue: -3
            System.out.println(pq.poll());
        }
    }    
   ```
&nbsp;&nbsp;PriorityQueue does not allow null elements to be inserted, and its ordering is similar to that of a TreeSet.

### 2.2 Deque
&nbsp;&nbsp;Deque interface represents a "double-ended queue", the double-ended queue can add and delete elements from both ends at the same time, so the implementation of Deque class can be used as a queue can also be used as a stack.
### 2.2.1 ArrayDeque
&nbsp;&nbsp;An array-based, dual-ended queue, similar to an ArrayList, uses a dynamic, reallocatable Object [] array at the bottom to store collection elements that are reallocated at the bottom when the collection element exceeds the capacity of the array An Object [] array to hold the collection elements.
   ```java
    import java.util.*;
    
    public class ArrayDequeTest{
        public static void main(String[] args) {
            ArrayDeque stack = new ArrayDeque();
            // push three elements to stack
            stack.push("head");
            stack.push("body");
            stack.push("tail");
    
            // output: [head, body, tail]
            System.out.println(stack);
    
            // visit the first element, but do not pop it out of the "stack"
            System.out.println(stack.peek());
    
            //output: [head, body, tail]
            System.out.println(stack);
    
            // pop the first element，output：tail
            System.out.println(stack.pop());
    
            // output: [head, body]
            System.out.println(stack);
        }
    }
   ```
#### List in different scenarios of choice
1. List provided by java is a "linear table interface," ArrayList (linear array based on the array), LinkedList (linear list based on the chain) are two typical implementations of linear tables.
   
2. Queue represents the queue, Deque represents the deque (can be used as a queue, can also be used as a stack).
   
3. Because arrays hold all the array elements in a contiguous block of memory, arrays perform best at random accesses. Therefore, the internal array to achieve the underlying set of random access in the best performance.
   
4. Internal to the list as the underlying implementation of the collection in the implementation of insert, delete operation has a good performance.
   
5. When performing iterative operations, the collection performance achieved with the linked list as the bottom is better than the collection performance achieved with the array as the bottom.

### 3.Set
&nbsp;&nbsp;Set collection is similar to a jar, there is no obvious order between multiple objects thrown into the Set collection.When we add a new element, the Set will accept the new element if the new element has the same value as the object in the Set, and equals will return false. Otherwise, it will be rejected.

### 3.1 HashSet
&nbsp;&nbsp;HashSet is a typical implementation of the Set interface. HashSet uses the HASH algorithm to store the elements in the set, so it has good access and lookup performance. When an element is stored in the HashSet collection, the HashSet will call the object hashCode () method to get the hashCode value of the object, and then determine the storage location of the object in the HashSet according to the HashCode value.It is noteworthy that the HashSet collection to determine the equality of the two elements of the standard is equal to two objects through the equals () method, and two objects hashCode () method returns the same value
   ```java
    import java.util.*; 
    
    // the equals method of class A always returns true, but does not override its hashCode () method
    class A{
        public boolean equals(Object obj){
            return true;
        }
    }
    
    // the class B hashCode () method always returns 1, but does not override its equals () method
    class B{
        public int hashCode(){
            return 1;
        }
    }
    
    // the hashCode () method of class C always returns 2, and has overridden its equals () method
    class C{
        public int hashCode(){
            return 2;
        }
        public boolean equals(Object obj){
            return true;
        }
    }
    public class HashSetTest{
        public static void main(String[] args) {
            HashSet books = new HashSet();
            // add to books
            books.add(new A());
            books.add(new A());
    
            books.add(new B());
            books.add(new B());
    
            books.add(new C());
            books.add(new C());
            
            // result: [B@1, B@1, C@2, A@3bc257, A@785d65]
            System.out.println(books);
        }
    }
   ```
&nbsp;&nbsp;equals() determines whether HashSet can be added, and where hashCode () determines where to place, both of them must be satisfied at the same time to allow a new element to join HashSet.
### 3.1.1 LinkedHashSet
&nbsp;&nbsp;LinkedHashSet uses lists to maintain the order of the elements so that the elements appear to be saved in the order they were inserted.LinkedHashSet will iterate through the elements in the collection as it traverses the elements in the LinkedHashSet collection. LinkedHashSet needs to maintain the insertion order of the elements, so the performance is slightly lower than the performance of the HashSet, but will have good performance when iterating through all the elements in the Set (traversal).
   ```java
    import java.util.*; 
    
    public class LinkedHashSetTest{
        public static void main(String[] args) {
            LinkedHashSet books = new LinkedHashSet();
            books.add("head");
            books.add("tail");
            System.out.println(books);
    
            // delete element head
            books.remove("head");
            // add head again
            books.add("head");
            
            // result: tail, head
            System.out.println(books);
        }
    }
   ```
&nbsp;&nbsp;The order of the elements is always the same as the order of the additions, and the collection elements are not allowed to repeat.

### 3.2 SortedSet
&nbsp;&nbsp;This interface is mainly used for sorting operations.
### 3.2.1 TreeSet
&nbsp;&nbsp;TreeSet is an implementation of the SortedSet interface, and TreeSet ensures that the collection elements are sorted.
   ```java
    import java.util.*;
    
    public class TreeSetTest{
        public static void main(String[] args) {
            TreeSet treeSet = new TreeSet();
            // add four Integer Object to treeSet
            treeSet.add(5);
            treeSet.add(2);
            treeSet.add(10);
            treeSet.add(-9);
    
            // result: the collection element is already in sort order
            System.out.println(treeSet);
    
            // output the first element in the collection
            System.out.println(treeSet.first());
    
            // output the late element in the collection
            System.out.println(treeSet.last());
    
            // returns a subset of less than 4, not including 4
            System.out.println(treeSet.headSet(4));
    
            // returns a subset greater than 5, if the Set contains 5, the subset also contains 5
            System.out.println(treeSet.tailSet(5));
    
            // returns a subset of greater than or equal to -3 and less than 4
            System.out.println(treeSet.subSet(-3 , 4));
        }
    }
   ```
&nbsp;&nbsp;TreeSet uses a red-black tree data structure to store the collection elements. TreeSet supports two sorts: natural sort, custom sort.
- 1.Natural sort: TreeSet will call the collection element compareTo method to compare the size of the relationship between the elements, and then set the elements sorted in ascending order.If compareTo returns 0, that is, if two objects compare with each other, the new object can not be added.If you try to add an object to the TreeSet, the object's class must implement the Comparable interface, or the program will throw an exception.
- 2.Custom sort: Comparator interface to achieve.
   ```java  
    import java.util.*;
    
    class M{
        int age;
        
        public M(int age){
            this.age = age;
        }
        
        public String toString(){
            return "M[age:" + age + "]";
        }
    }
    
    public class TreeSetTest4{
        public static void main(String[] args) {
            TreeSet ts = new TreeSet(
            new Comparator(){
                // according to the M object age attribute to determine the size
                public int compare(Object o1, Object o2){
                    M m1 = (M)o1;
                    M m2 = (M)o2;
                    return m1.age > m2.age ? -1
                        : m1.age < m2.age ? 1 : 0;
                }
            });    
            ts.add(new M(5));
            ts.add(new M(-3));
            ts.add(new M(9));
            System.out.println(ts);
        }
    }
   ```

### 3.3 EnumSet
&nbsp;&nbsp;A collection designed specifically for enumeration classes.All elements in the EnumSet must be enumerated values ​​of the specified enumeration type explicitly or implicitly specified when creating the EnumSet. EnumSet collection elements are also ordered, they enumerate values ​​in the Enum class in the order of definition to determine the order of the elements of the collection.
   ```java
    import java.util.*;

    enum Season{
        SPRING,SUMMER,FALL,WINTER
    }
    
    public class EnumSetTest{
        public static void main(String[] args) {
            // create an EnumSet collection, the collection element is the enumeration value of the Season enumeration class
            EnumSet es1 = EnumSet.allOf(Season.class);
            // output [SPRING,SUMMER,FALL,WINTER]
            System.out.println(es1);

            // create an empty EnumSet collection
            EnumSet es2 = EnumSet.noneOf(Season.class); 
            //output []
            System.out.println(es2); 
            
             // manually add two elements
            es2.add(Season.WINTER);
            es2.add(Season.SPRING);
            // output [SPRING,WINTER]
            System.out.println(es2);

            // create an EnumSet collection with the specified enumeration value
            EnumSet es3 = EnumSet.of(Season.SUMMER , Season.WINTER); 
            // output [SUMMER,WINTER]
            System.out.println(es3);

            EnumSet es4 = EnumSet.range(Season.SUMMER , Season.WINTER); 
            // output [SUMMER,FALL,WINTER]
            System.out.println(es4);
            
            EnumSet es5 = EnumSet.complementOf(es4); 
            // output [SPRING]
            System.out.println(es5);
        }
    }
   ```
#### Set in different scenarios of choice
1. HashSet always better than the performance of the TreeSet (especially the most commonly used to add, query elements and other operations), because TreeSet requires additional red-black tree algorithm to maintain the order of the elements of the collection. Only when you need to maintain a sorted set, you should use the TreeSet, otherwise you should use HashSet.
   
2. For normal insert and delete operations, LinkedHashSet is a little slower than HashSet, which is caused by the overhead of maintaining the linked list. However, with the existence of the linked list, traversing LinkedHashSet will be faster.
   
3. EnumSet is the best performance in all Set implementation class, but it can only save the enumeration value of the same enumeration class as a collection element.   

4. HashSet, TreeSet, EnumSet are "thread-insecure" and can usually be "wrapped" by the synchronizedSortedSet method of the Collections utility.   
&nbsp;&nbsp;SortedSet s = Collections.synchronizedSortedSet(new TreeSet(...));

### 4.Map
&nbsp;&nbsp;Map is used to save the data with "mappings",So the Map collection holds two sets of values, one for the key in the Map and the other for the value in the Map. Both key and value can be any reference type of data. Map's key does not allow duplication.

### 4.1 HashMap
&nbsp;&nbsp;HashMap can not guarantee the order of the key-value pairs, and the criterion for judging whether two keys are equal or not is that the two keys return true through the equals () method, and the hashCode values ​​of the two keys must also be equal.
### 4.1.1 LinkedHashMap
&nbsp;&nbsp;LinkedHashMap also maintains a sequence of key-value pairs using doubly linked lists that maintain the order in which the Map iterates, in the same order as key-value pairs are inserted.
   ```java
    import java.util.*;
    
    public class LinkedHashMapTest{
        public static void main(String[] args) {
            LinkedHashMap scores = new LinkedHashMap();
            scores.put("Chinese" , 80);
            scores.put("English" , 82);
            scores.put("Math" , 76);
            // traverse
            for (Object key : scores.keySet()){
                System.out.println(key + "------>" + scores.get(key));
            }
        }
    }
   ```

### 4.2 HashTable
### 4.2.1 Properties
&nbsp;&nbsp;The Properties object is particularly handy when working with properties files. The Properties class can associate a Map object with a properties file so that the key-value pairs in the Map object can be written to the properties file. The property name - attribute value "is loaded into the Map object.
   ```java
    import java.util.*;
    import java.io.*;
    
    public class PropertiesTest{
        public static void main(String[] args) throws Exception{
            Properties props = new Properties();
            // add elements to Properties
            props.setProperty("username" , "xfan");
            props.setProperty("password" , "0920");
    
            // save the key-value pair in Properties to the a.ini file
            props.store(new FileOutputStream("a.ini"), "Contents"); 
    
            // create an new Properties Object
            Properties props2 = new Properties();
            // add elements to Properties
            props2.setProperty("loadPro" , "loadProV");
    
            // append the key-value pair in the a.ini file to props2
            props2.load(new FileInputStream("a.ini") );   
            System.out.println(props2);
        }
    }
   ```

### 4.3 SortedMap
### 4.3.1 TreeMap
&nbsp;&nbsp;TreeMap is a red-black tree data structure, each key-value pair as a node of the red-black tree. When TreeMap stores the key-value pair (node), it is necessary to sort the nodes according to the key. TreeMap ensures that all key-value pairs are in order. Similarly, TreeMap also has two sorts of sorting: natural sorting, custom sorting.

### 4.4 WeakHashMap
&nbsp;&nbsp;WeakHashMap and HashMap usage is similar. The difference is that the key in the HashMap retains a "strong reference" to the actual object, and the object referenced in the HashMap will not be garbage collected as long as the object of the HashMap is not destroyed. However, the key in WeakHashMap only retains a "weak reference" to the actual object. If the object referenced by the key of the WeakHashMap object is not referenced by other strongly referenced variables, the objects referenced by these keys may be garbage collected. When the garbage After reclaiming the real object corresponding to the button, the key-value pairs corresponding to the keys may also be deleted automatically in the WeakHashMap.
   ```java
    import java.util.*;
    
    public class WeakHashMapTest{
        public static void main(String[] args){
            WeakHashMap whm = new WeakHashMap();
            // add three elements（no other references）
            whm.put(new String("Chinese") , new String("A"));
            whm.put(new String("Math") , new String("B"));
            whm.put(new String("English") , new String("C"));
    
            // add a strong reference object
            whm.put("java" , new String("B"));
            // output you can see four key-value
            System.out.println(whm);
            // notify the system immediately garbage collection
            System.gc();
            System.runFinalization();
            // normally only the key-value pairs that are strongly referenced are retained
            System.out.println(whm);
        }
    }
   ```

### 4.5 IdentityHashMap
&nbsp;&nbsp;The implementation mechanism of IdentityHashMap is basically similar to that of HashMap. In IdentityHashMap, IdentityHashMap considers the two keys equal if and only if the two keys are strictly equal (key1 == key2).
   ```java
    import java.util.*;
    
    public class IdentityHashMapTest{
        public static void main(String[] args) {
            IdentityHashMap ihm = new IdentityHashMap();
            // add two elements
            ihm.put(new String("Chinese") , 89);
            ihm.put(new String("Chinese") , 78);
    
            // that will only add one element
            ihm.put("java" , 93);
            ihm.put("java" , 98);
            System.out.println(ihm);
        }
    } 
   ```

### 4.6 EnumMap
&nbsp;&nbsp;EnumMap is a Map implementation used with enumeration classes. All keys in EnumMap must be enumerated values ​​of a single enumeration class. EnumMap must be created explicitly or implicitly specify its corresponding enumeration class. EnumMap based on the key natural order
   ```java
    import java.util.*;
    
    enum Season{
        SPRING,SUMMER,FALL,WINTER
    }
    public class EnumMapTest{
        public static void main(String[] args)  {
            // associate EnumMap with enum classes
            EnumMap enumMap = new EnumMap(Season.class);
            enumMap.put(Season.SUMMER , "Summer is hot");
            enumMap.put(Season.SPRING , "Spring is beautiful");
            System.out.println(enumMap);
        }
    }
   ```

#### Map in different scenarios of choice
1. The efficiency of HashMap and HashTable roughly the same, because their implementation mechanism is almost exactly the same. HashMap is usually faster than HashTable because HashTable requires additional thread synchronization control.   

2. TreeMap is usually slower than HashMap and HashTable (especially slower when inserting or deleting key-value pairs) because the underlying TreeMap uses a red-black tree to manage key-value pairs.   

3. One of the benefits of using TreeMap is that the key-value pairs in the TreeMap are always in an ordered state and do not need to be specifically ordered