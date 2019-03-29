-----------------
title: Java Queue
tags: JAVA
categories: JAVA
-----------------
date: 2018-02-02 17:01:03

### The definition of the queue
&nbsp;&nbsp;Is a linear table with limited operations. Is a first-in,first-out(FIFO) linear table.   
- front : The end that is allowed to be deleted is called the head of the team.
- rear  : The end that allows insertion is called the tail.   

### Order queue

#### The basic operation of the order queue
&nbsp;&nbsp;Their initial value should be set to 0 at queue initialization
- When entering the team : Insert the new element into the rear of the position pointed by.
- Departure : delete the front of the elements, and then add 1 front and return deleted elements.   

#### Sequence table overflow phenomenon
- Underflow: When the queue is empty, make a spillover operation caused by the phenomenon. Underflow is a normal phenomenon and is commonly used as a condition of program transfer control.
- Really overflow: When the queue is full, into the stack operation space overflow phenomenon. True overflow is a mistake, should try to avoid.
- False overflow : Due to the entry and exit of the operation, only increase the head and tail pointer does not reduce, resulting in deleted elements of the space will never be reused, and thus can not be enqueued.

