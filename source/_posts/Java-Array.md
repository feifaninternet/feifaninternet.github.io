-----------------
title: Java Array
tags: JAVA
categories: JAVA
date: 2018-02-01 18:06:38
-----------------

### Description

&nbsp;&nbsp;The same type of data collection.In fact, the array is a container.We can automatically number the elements in the array from 0 to facilitate the operation of these elements.Array is a way to store data.And array can hold any type of data.   

### The name of the array
   ```
    // type [] arrayName = new type[size]
    int[] arr = new int[5];
    
    // type [] arrayName = new type[]{element, element,...}
    int[] arr = new int[]{1,2,3,4};
   ```
### Initialization of the array
   ```
       int[] arr = { 1, 2, 3, 4, 5 };
       int[] arr2 = new int[] { 1, 2, 3, 4, 5 };
       
       int[] arr3 = new int[3];
       arr3[0] = 1;
       arr3[1] = 5;
       arr3[2] = 6;
   ```
### Array traversal
   ```
    public static void main(String[] args) {
    
    int[] x = { 1, 2, 3 };
    
    for (int y = 0; y < x.length; y++) {
    
        System.out.println(x[y]);
        
        } 
    }
   ```
### Array common exception
- java.lang.NullPointerException   
- java.lang.ArrayIndexOutOfBoundsException

### The way to sort array
### 1.Using arrays comes with methods
   ```java
    public class ArraySort{     
            public static int[] sortWithArrays( int[] args){            
                if(args == null|| args.length < 2){  
                        return args;  
                } 
                Arrays.sort(args); 
                return args; 
            }
     }    
   ```
### 2.Bubble sorting algorithm
   ```java
    public class BubbleSort{     
            public static int[] sortWithBubble(int[] args){  
                if(args == null|| args.length < 2){  
                    return args;  
                }
                for(int i = 0;i < args.length-1;i++){
                    for (int j = i+1;j < args.length;j++){
                        if(args[j] < args[i]){
                            int temp = args[i]; 
                            args[i] = args[j];
                            args[j] = temp;
                        }
                    }
                }    
                return args;
            }
     }    
   ```
### 3.Select sorting algorithm
   ```java
    public class SelectSort{     
            public static int[] sortWithSelect(int[] args){
                int minIndex;
                int temp;
                if(args == null || args.length < 2){
                    return;
                }
                for (int i = 0;i < args.length-1;i++){
                    minIndex = i;
                    for (int j = i+1;j < args.length;j++){
                        if (args[j] < args[minIndex]){
                           minIndex = j;                          
                        }          
                    }
                    if(minIndex != i){
                        temp = args[i];
                        args[i] = args[minIndex];
                        args[minIndex] = temp;
                    }    
                }
                return args;
            }
     }    
   ```
### 4.Insert sorting algorithm
   ```java
    public class InsertSort{     
            public static int[] sortWithInsert(int[] args){
                if(args == null || args.length < 2){
                    return;
                }
                for (int i = 1;i < args.length;i++){
                    for (int j = i;j > 0;j--){
                        if (args[j] < args[j-1]){
                            int temp = args[j];
                            args[j] = args[j-1];
                            args[j-1] = temp;
                        }      
                    }                   
                }
                return args;           
            }
     }    
   ```
  

