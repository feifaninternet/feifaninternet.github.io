-----------------------
title: String 源码
tags: String
categories: Origin Code
date: 2018-06-07 12:31:11
-----------------------

### String 类
String 类被 final 所修饰，为不可变量，并发程序对不可变量非常友好，String 类实现了Serializable,Comparable<Stirng>,CharSequence接口。

### String 属性
String类中包含一个不可变的char[]数组来存放字符串，一个int型的hash用来存放计算后的哈希值。

   ```java
    /** The value is used for character storage. */
    private final char[] value;
    
    /** Cache the hash code for the string */
    private int hash; // Default to 0
    
    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -6849794470754667710L;
    
   ```

### String 构造函数

   ```java
    //不含参数的构造函数，一般没什么用，因为value是不可变量
    public String() {
        this.value = new char[0];
    }
    
    //参数为String类型
    public String(String original) {
        this.value = original.value;
        this.hash = original.hash;
    }
    
    //参数为char数组，使用java.utils包中的Arrays类复制
    public String(char value[]) {
        this.value = Arrays.copyOf(value, value.length);
    }
    
    //从bytes数组中的offset位置开始，将长度为length的字节，以charsetName格式编码，拷贝到value
    public String(byte bytes[], int offset, int length, String charsetName)
            throws UnsupportedEncodingException {
        if (charsetName == null)
            throw new NullPointerException("charsetName");
        checkBounds(bytes, offset, length);
        this.value = StringCoding.decode(charsetName, bytes, offset, length);
    }
    
    //调用public String(byte bytes[], int offset, int length, String charsetName)构造函数
    public String(byte bytes[], String charsetName)
            throws UnsupportedEncodingException {
        this(bytes, 0, bytes.length, charsetName);
    }
   ```
   
### String 常用方法
### 1.equals 方法

   ```java
    boolean equals(Object anObject)
    
    public boolean equals(Object anObject) {
        //如果引用的是同一个对象，直接返回真
        if (this == anObject) {
            return true;
        }
        //如果不是String类型的数据，返回假
        if (anObject instanceof String) {
            String anotherString = (String) anObject;
            int n = value.length;
            //如果char数组长度不相等，返回假
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                //从后往前单个字符判断，如果有不相等，返回假
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                            return false;
                    i++;
                }
                //每个字符都相等，返回真
                return true;
            }
        }
        return false;
    }
   ```
   
代码中可以看出是逐一比较字符的，对于超级长的字符串的比较还是非常费时间的。

### 2.compareTo方法

   ```java
    int compareTo(String anotherString)
    
    public int compareTo(String anotherString) {
        //自身对象字符串长度len1
        int len1 = value.length;
        //被比较对象字符串长度len2
        int len2 = anotherString.value.length;
        //取两个字符串长度的最小值lim
        int lim = Math.min(len1, len2);
        char v1[] = value;
        char v2[] = anotherString.value;
    
        int k = 0;
        //从value的第一个字符开始到最小长度lim处为止，如果字符不相等，返回自身（对象不相等处字符-被比较对象不相等字符）
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        //如果前面都相等，则返回（自身长度-被比较对象长度）
        return len1 - len2;
    }
   ```
   
### 3.hashCode 方法

   ```java
    int hashCode()
    
    public int hashCode() {
        int h = hash;
        //如果hash没有被计算过，并且字符串不为空，则进行hashCode计算
        if (h == 0 && value.length > 0) {
            char val[] = value;
    
            //计算过程
            //s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            //hash赋值
            hash = h;
        }
        return h;
    }
   ```
String类重写了hashCode方法，Object中的hashCode方法是一个Native调用。String类的hashCode是通过多项式计算来的，我们可以通过不同的字符串得到相同的hashCode，所以两个String对象的hashCode相同，并不代表两个String是一样的。
   
### 4. startsWith 与 endsWith 方法

   ```java
    boolean startsWith(String prefix,int toffset)
    
    public boolean startsWith(String prefix, int toffset) {
        char ta[] = value;
        int to = toffset;
        char pa[] = prefix.value;
        int po = 0;
        int pc = prefix.value.length;
        // Note: toffset might be near -1>>>1.
        //如果起始地址小于0或者（起始地址+所比较对象长度）大于自身对象长度，返回假
        if ((toffset < 0) || (toffset > value.length - pc)) {
            return false;
        }
        //从所比较对象的末尾开始比较
        while (--pc >= 0) {
            if (ta[to++] != pa[po++]) {
                return false;
            }
        }
        return true;
    }
    
    // startsWith 与 endsWith 已巧妙实现
    public boolean startsWith(String prefix) {
        return startsWith(prefix, 0);
    }
    
    public boolean endsWith(String suffix) {
        return startsWith(suffix, value.length - suffix.value.length);
    }
   ```
   
起始比较和末尾比较比较常用，例如在判断一个字符串是不是http协议或者初步判断一个文件是不是mp3文件。

### 5.concat 方法

   ```java
    String concat(String str)
    
    public String concat(String str) {
        int otherLen = str.length();
        //如果被添加的字符串为空，返回对象本身
        if (otherLen == 0) {
            return this;
        }
        int len = value.length;
        char buf[] = Arrays.copyOf(value, len + otherLen);
        str.getChars(buf, len);
        return new String(buf, true);
    }
   ```
   
字符串拼接，创建新字符串存储。

### 6.replace 方法

   ```java
    String replace(char oldChar,char newChar)
    
    public String replace(char oldChar, char newChar) {
        //新旧值先对比
        if (oldChar != newChar) {
            int len = value.length;
            int i = -1;
            char[] val = value; /* avoid getfield opcode */
    
            //找到旧值最开始出现的位置
            while (++i < len) {
                if (val[i] == oldChar) {
                    break;
                }
            }
            //从那个位置开始，直到末尾，用新值代替出现的旧值
            if (i < len) {
                char buf[] = new char[len];
                //复制从0到旧值最开始出现位置的值
                for (int j = 0; j < i; j++) {
                    buf[j] = val[j];
                }
                //替换从旧值最开始到字符串最后的所有符合条件的字符
                while (i < len) {
                    char c = val[i];
                    buf[i] = (c == oldChar) ? newChar : c;
                    i++;
                }
                return new String(buf, true);
            }
        }
        return this;
    }
   ```
   
### 7.trim 方法

   ```java
    String trim()
    
    public String trim() {
        int len = value.length;
        int st = 0;
        char[] val = value;    /* avoid getfield opcode */
    
        //找到字符串前段没有空格的位置
        while ((st < len) && (val[st] <= ' ')) {
            st++;
        }
        //找到字符串末尾没有空格的位置
        while ((st < len) && (val[len - 1] <= ' ')) {
            len--;
        }
        //如果前后都没有出现空格，返回字符串本身
        return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
    }
   ```
   
### String 中一些简单的常用方法

   ```java
        public int length() {
            return value.length;
        }
        public String toString() {
            return this;
        }
        public boolean isEmpty() {
            return value.length == 0;
        }
        public char charAt(int index) {
            if ((index < 0) || (index >= value.length)) {
                throw new StringIndexOutOfBoundsException(index);
            }
            return value[index];
        }
   ```
   
### 解释2个String对象内存地址相同

   ```java
    public void stringTest() {
        String a = "ab1";
        String b = "ab1";
        //结果是true
        System.out.println(a == b);
    }
   ```
   
原因如下：字符串a为常量，a所指向的地址来自常量池，b所指向的字符串常量会默认调用intern()方法(在方法区中的常量池里通过equals方法寻找等值的对象，如果没有找到则在常量池中开辟一片空间存放字符串并返回该对应String的引用，否则直接返回常量池中已存在String对象的引用。)显而易见，b所指向的地址直接是取的常量池中a的地址，所以会出现相同的情况。