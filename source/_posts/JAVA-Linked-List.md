-----------------------
title: JAVA Linked List
-----------------------
date: 2018-02-02 11:45:03
tags: linked list

### About

&nbsp;&nbsp;A linked list is a common underlying data structure that is a linear list but does not store data in a linear order, but a pointer to the next node in each node.So the list is inserted faster than the linear order table,however, the query speed is relatively slow.The use of linked list structure can overcome the shortcomings of the array linked list need to know the size of the data in advance, the linked list structure can take full advantage of computer memory space for flexible dynamic memory management.However,the list has the lost random array read the advantages of the same time as the list linked to increase the pointer field,the space overhead is relatively large.   
&nbsp;&nbsp;Linked list as a basic data structure can be used to generate other type of data structure.A linked list usually consists of a series of nodes,each containing arbitrary instance data and one or two links to point to the previous or next node,which allows inserting and removing nodes anywhere on the table,but not allow random access.There are many different types of lists : unidirectional lists,double linked lists and circular lists.   
&nbsp;&nbsp;Linked to the list as a unit, the data is not stored in continuous storage, in addition to the data, but also contains a pointer to the next node, the data always need to access the data from the beginning of the list.   

### Linked list
#### Create a node class   
   ```java
    class Node{
    
        /**
         * data : represents the value of the node
         * next : the reference to the next node
         */
        @Getter @Setter
        private int data;
        
        @Getter @Setter
        private Node next;
        
        /**
         * only one node
         */
        Node(int data){
            m_Data = data;
            m_Next = null;
        }
        
        /**
         * contains multiple nodes,pointers must be defined
         */
        Node(int data, Node next){
            m_Data = data;
            m_Next = next;
        }

    }
   ```
#### Create a linked list
   ```java
    class LinkList{
    
        @Getter @Setter
        private Node m_Node;
   
        /**
         * empty linked list
         */
        LinkList(){
            m_Node = null;
        }
        
        /**
         * create a linked list with only one node because no pointers are declared
         */
        LinkList(int data){
            m_Node = new Node(data);
        }
        
        /**
         * traverse all nodes
         */
        String viewAllNodes(){
            Node next = m_Node;
            String s;
            while (next != null){
                s += next.getData();
                next = next.getNext();              
            }
            return s;
        }
        
        /**
         * insert at the specified id location
         */
        void insertAfterId(int data, int id){
            Node next = m_Node;
            if(next == null){
                m_Node = new Node(data);
            }else{
                while(next.getNext() != null && next.getData() != id)
                next = next.getNext();
                next.setNext(new Node(data, next.getNext()));
             }
        }
        
        /**
         * remove the node at the specified id location
         */
        boolean removeAtId(int id){
            Node ahead = m_Node;
            Node follow = ahead;
            if(ahead == null){
                return false;
            }else if(ahead.getData() == id){
                m_Node = m_Node.getNext();
                return true;
            }else{
                ahead = ahead.getNext();
                while(ahead != null){
                    if(ahead.getData() == id){
                        follow.setNext(ahead.getNext());
                        return true;
                    }
                    follow = ahead;
                    ahead = ahead.getNext();
                }
                    return false;
            }
        }
        
        /**
         * remove all nodes
         */
        void removeAll(){
            m_Node = null;
        }
        
    }
   ```
### Some commonly used linked list operations

#### 1. List reversed 
   ```java
    public class LinkedListMethodDemo{
      
        public Node ReverseIteratively(Node head) {
            
            Node pReversedHead = head;
            Node pNode = head;
            Node pPrev = null;
            
            while (pNode != null) {
                Node pNext = pNode.next;
                if (pNext == null) {
                    pReversedHead = pNode;
                }
                pNode.next = pPrev;
                pPrev = pNode;
                pNode = pNext;
            }
            this.head = pReversedHead;
            return this.head;
        }
    }
   ```
#### 2.Find single-linked list of intermediate nodes

&nbsp;&nbsp;Use the fast and slow way to find the middle node of the single-linked list, the fast pointer takes two steps at a time, and the slow pointer takes one step at a time. When the fast pointer finishes, the slow pointer just reaches the intermediate node.
   ```java
    public class SearchMidNode{
       
        public Node searchMidNode(Node head) {
            
            Node p = this.head, 
            q = this.head;
            while (p != null && p.next != null && p.next.next != null) {
                p = p.next.next;
                q = q.next;
            }
            System.out.println("Mid:" + q.data);
            return q;
        }
    }
   ```
#### 3.Sort the linked list
   ```java
    public class SortLinkedList{
    
        public Node orderList() {
            
            Node nextNode;
            int tmp;
            Node curNode = head;
            
            while (curNode.next != null) {
                nextNode = curNode.next;
                while (nextNode != null) {
                    if (curNode.data > nextNode.data) {
                        tmp = curNode.data;
                        curNode.data = nextNode.data;
                        nextNode.data = tmp;
                    }
                nextNode = nextNode.next;
                }
                curNode = curNode.next;
            }
            return head;
        }
    }
   ```
#### 4.Delete duplicate nodes in the list
   ```java
    public class SortLinkedList{
    
        public void deleteDuplicate(Node head){
            Node p = head;
            while (p != null){
                Node q = p;
                while (q.next != null){
                    if(q.next == p.next){
                        q.next == q.next.next;
                    }else {
                        q = q.next;
                    }
                }
                p = p.next;
            }
        }
    }
   ```