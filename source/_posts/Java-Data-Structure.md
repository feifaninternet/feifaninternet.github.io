------------------------
title: Java Data Structure
tags: JAVA
categories: JAVA
date: 2018-01-29 10:21:52
------------------------

### The class and interface of the structure

### Enumeration ( Interface )

&nbsp;&nbsp;Enumeration is widely used in data structure.The enumeration interface defines a way to retrieve consecutive elements from the data structure.This traditional interface has been replaced by an iterator,although rarely used,but it has not been abandoned.   
   ```java
    import java.util.Vector;
            import java.util.Enumeration;
            public class EnumerationTester {
               public static void main(String args[]) {
                  Enumeration<String> days;
                  Vector<String> dayNames = new Vector<String>();
                  dayNames.add("Sunday");
                  dayNames.add("Monday");
                  dayNames.add("Tuesday");
                  dayNames.add("Wednesday");
                  dayNames.add("Thursday");
                  dayNames.add("Friday");
                  dayNames.add("Saturday");
                  days = dayNames.elements();
                  while (days.hasMoreElements()){
                     System.out.println(days.nextElement()); 
                  }
               }
            }     
   ```
### BitSet
&nbsp;&nbsp;A BitSet class creates a special type of array to hold bit values,BitSet array size will increase with the need.   
   ```java
    import java.util.BitSet;
     
    public class BitSetDemo {
     
      public static void main(String args[]) {
         BitSet bits1 = new BitSet(16);
         BitSet bits2 = new BitSet(16);
          
         // set some bits
         for(int i=0; i<16; i++) {
            if((i%2) == 0) bits1.set(i);
            if((i%5) != 0) bits2.set(i);
         }
         System.out.println("Initial pattern in bits1: ");
         System.out.println(bits1);
         System.out.println("\nInitial pattern in bits2: ");
         System.out.println(bits2);
     
         // AND bits
         bits2.and(bits1);
         System.out.println("\nbits2 AND bits1: ");
         System.out.println(bits2);
     
         // OR bits
         bits2.or(bits1);
         System.out.println("\nbits2 OR bits1: ");
         System.out.println(bits2);
     
         // XOR bits
         bits2.xor(bits1);
         System.out.println("\nbits2 XOR bits1: ");
         System.out.println(bits2);
      }
    }
   ```
### Vector
&nbsp;&nbsp;The size of a Vector can be dynamically changed as needed.So when creating an object do not have to specify the size of it.The Vector class implements a dynamic array,and ArrayList and similar,but two are different.   
- Vector is accessed synchronously
- Vector contains many traditional methods,these methods do not belong to the collection framework   

&nbsp;&nbsp;Vector class supports four kinds of construction method.
       
1.Create a default vector,the default size is ten
   ```
    Vector()
   ```
2.Create a vector of the specified size
   ```
    Vector(int size)
   ```
3.Create a vector of the specified size,and specify the increment.Increments represent the number of elements each time the vector is incremented   
   ```
    Vector(int size,int incr)
   ```
4.Create a vector that contains the elements of the set
   ```
    Vector(Collection c)
   ```
### Stack
&nbsp;&nbsp;Stack implements a back-in-first-out data structure.You can understand a stack as a vertically-distributed stack of objects, and when you add a new one, you place the new one at the top of the other.Stack is a subclass of Vector.   
   ```java
    import java.util.*;
     
    public class StackDemo {
     
        // push elements into the top of the stack
        static void showPush(Stack<Integer> st, int a) {
            st.push(new Integer(a));
            System.out.println("push(" + a + ")");
            System.out.println("stack: " + st);
        }
     
        // remove the object at the top of the stack
        static void showPop(Stack<Integer> st) {
            System.out.print("pop -> ");
            Integer a = (Integer) st.pop();
            System.out.println(a);
            System.out.println("stack: " + st);
        }
     
        public static void main(String args[]) {
            Stack<Integer> st = new Stack<Integer>();
            System.out.println("stack: " + st);
            showpush(st, 42);
            showpush(st, 66);
            showpush(st, 99);
            showpop(st);
            showpop(st);
            showpop(st);
            try {
                showpop(st);
            } catch (EmptyStackException e) {
                System.out.println("empty stack");
            }
        }
    }
   ```
### HashTable
&nbsp;&nbsp;The HashTable class provides a means of organizing data based on user-defined key ( Hash key ) structures.The key is hashed, and the resulting hash code is used as an index for the values ​​stored in the table.   
   ```java
    import java.util.*;
    
    public class HashTableDemo {
    
       public static void main(String args[]) {
          // create a hash map
          Hashtable balance = new Hashtable();
          Enumeration names;
          String str;
          double bal;
          
          // specified the key and value
          balance.put("key1", new Double(3434.34));
          balance.put("key2", new Double(123.22));
          balance.put("key3", new Double(1378.00));
          balance.put("key4", new Double(99.22));
          balance.put("key5", new Double(-19.08));
    
          // show all balances in hash table.
          names = balance.keys();
          while(names.hasMoreElements()) {
             str = (String) names.nextElement();
             System.out.println(str + ": " +
             // get the value from this key
             balance.get(str));
          }
          System.out.println();
          // deposit 1,000 into key1`s account
          bal = ((Double)balance.get("key1")).doubleValue();
          balance.put("key1", new Double(bal+1000));
          System.out.println("key1's new balance: " +
          balance.get("key1"));
       }
    }
   ```
### Properties

&nbsp;&nbsp;Properties inherit from the HashTable.Properties class represents a durable set of properties.Each key in the property list and its corresponding value is a string.   
   ```java
    import java.util.*;
    
    public class PropDemo {
    
       public static void main(String args[]) {
          // create
          Properties capitals = new Properties();
          Set states;
          String str;
          
          // put elements
          capitals.put("Illinois", "Springfield");
          capitals.put("Missouri", "Jefferson City");
          capitals.put("Washington", "Olympia");
          capitals.put("California", "Sacramento");
          capitals.put("Indiana", "Indianapolis");
    
          // get set-view of keys
          states = capitals.keySet(); 
          Iterator itr = states.iterator();
          while(itr.hasNext()) {
             str = (String) itr.next();
             System.out.println("The capital of " +
                str + " is " + capitals.getProperty(str) + ".");
          }
          System.out.println();
    
          // look for state not in list -- specify default
          str = capitals.getProperty("Florida", "Not Found");
          System.out.println("The capital of Florida is "
              + str + ".");
       }
    }
   ```
