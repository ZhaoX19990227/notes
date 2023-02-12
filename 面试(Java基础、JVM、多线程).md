## 深拷贝和浅拷贝

`浅拷贝`：复制对象时只复制对象本身，包括基本数据类型的属性，但是不会复制引用数据类型属性指向的对象，即拷贝对象的与原对象的引用数据类型的属性指向同一个对象。浅拷贝没有达到完全复制，即原对象与克隆对象之间有关系，会相互影响
`深拷贝`：复制一个新的对象，引用数据类型指向对象会拷贝新的一份，不再指向原有引用对象的地址
深拷贝达到了完全复制的目的，即原对象与克隆对象之间不会相互影响

## IO流

按照`流的流向分`，可以分为输入流和输出流；
按照`操作单元划分`，可以划分为字节流和字符流；
按照`流的角色划分`，可分为节点流和处理流。

### 字节流可以处理一切文件，而字符流只能处理纯文本文件

**节点流**：直接操作数据读写的流类，比如 FileInputStream

**处理流**：对一个已存在的流的链接和封装，通过对数据进行处理为程序提

供功能强大、灵活的读写功能，例如 BufferedInputStream （缓冲字节

流）

处理流和节点流应用了Java的装饰者设计模式。

处理流是对节点流的封装，最终的数据处理还是由节点流完成的。

<img src="/Users/zhaoxiang/Desktop/java系统笔记/面试java2.assets/image-20220927152409709.png" alt="image-20220927152409709" style="zoom:50%;" />

普通流每次读写一个字节，而缓冲流在内存中设置一个缓存区，缓冲区先

存储足够的待操作数据后，再与内存或磁盘进行交互。这样，在总数据量

不变的情况下，通过提高每次交互的数据量，减少了交互次数

<img src="/Users/zhaoxiang/Desktop/java系统笔记/面试java2.assets/image-20220927152448436.png" alt="image-20220927152448436" style="zoom:50%;" />

## BIO、NIO、AIO

### BIO同步阻塞IO

传统的网络通讯模型，就是BIO，属于**同步阻塞**的IO模型。比如我们熟悉的Socket通信。

#### Socket服务端创建过程：

```markdown
服务端创建一个ServerSocket；
客户端用一个Socket去连接服务端的那个ServerSocket；
ServerSocket接收到了一个的连接请求，就创建一个Socket和一个线程去跟那个Socket进行通讯。
```

#### 客户端和服务端进行阻塞式通信过程

```markdown
客户端发送一个请求；
服务端Socket进行处理后返回响应；
在响应返回前，客户端那边就阻塞等待，任何事情都做不了。
```

### NIO同步非阻塞IO

```
基于Reactor模型来实现的，相当于，一个线程处理大量的客户端的请求，通过一个线程轮询大量channel，每次就获取一批有事件的channel，然后对每个请求启动一个线程处理即可。
这里的核心就是非阻塞，selector一个线程就可以不停轮询channel，所有客户端请求都不会阻塞，最多就是等待下一轮的轮询。
```

#### **NIO**--优化BIO的核心

```
一个客户端并不是时时刻刻都有数据进行交互，所以客户端选择了让线程歇一歇，只有客户端有相应的操作的时候才发起通知，再创建一个线程来处理请求。
```

#### Reactor模型

<img src="/Users/zhaoxiang/Desktop/java系统笔记/面试java2.assets/image-20220927152839569.png" alt="image-20220927152839569" style="zoom:40%;" />

### AIO异步非阻塞IO

AIO基于**Proactor**模型实现，分为**发送请求和读取数据**两个步骤：

```markdown
# 发送请求：处理每个连接发送过来的请求。

每个请求都会绑定一个Buffer；通知操作系统去完成异步的读(这个时间你就可以去做其他的事情)；
调用你的接口；
返回异步读完的数据。

# 读取数据：将数据往回写。
一个Buffer，让操作系统去完成写。
```

**发送请求和读取数据**的主要区别在于

```
将数据写入的缓冲区后，剩下的交给操作系统去完成；
操作系统写回数据也是一样，写到Buffer里面，完成后再通知客户端来进行读取数据。
```

#### AIO模型图

<img src="/Users/zhaoxiang/Desktop/java系统笔记/面试java2.assets/image-20220927153040522.png" alt="image-20220927153040522" style="zoom:50%;" />

#### 同步阻塞--为什么说BIO是同步阻塞的呢？

```
针对磁盘文件读写IO操作来说，因为用BIO的流读写文件，例如FileInputStrem，必须等着完成了这次IO才能返回。
```

#### 同步非阻塞--为什么说NIO为啥是同步非阻塞？

```
因为无论多少客户端都可以接入服务端，客户端接入并不会耗费一个线程，只会创建一个连接，然后注册到selector上去，一个selector线程不断的轮询所有的socket连接，发现有事件了就通知你，然后你就启动一个线程处理一个请求即可，这个过程的话就是非阻塞的。但是这个处理的过程中，你还是要先读取数据，处理，再返回的，这是个同步的过程。
```

#### 异步非阻塞--为什么说AIO是异步非阻塞？

```
当基于AIO的api去读写文件时，发起一个请求之后，等读写完成后， 操作系统会来回调你的接口， 告诉你操作完成。在这期间不需要等待， 也不需要去轮询判断操作系统完成的状态，你可以去干其他的事情。同步还得主动去轮询操作系统，异步就是操作系统反过来通知你。
所以说AIO就是异步非阻塞的。
```

## 内部类

定义在类内部的类就被称为内部类，内部类分为 `静态内部类`，`成员内部类`，`局部内部类`，`匿名内部类`

1. 定义在类内部的静态类，就是`静态内部类`

   1.静态内部类可以访问外部类所有的静态变量和方法，即使是private的也一样。 

   2.静态内部类和一般类一致，可以定义静态变量、方法，构造方法等。 

   3.其它类使用静态内部类需要使用`外部类.静态内部类`方式，

   如下所示：Out.Inner inner = new Out.Inner();     inner.print(); 

   4.Java集合类`HashMap`内部就`有一个静态内部类Entry`。Entry`是HashMap存放元素的抽象`，HashMap内部维护Entry数组用了存放元素，但是Entry对使用者是透明的。像这种和外部类关系密切的，且不依赖外部类实例的，都可以使用静态内部类。

2. `成员内部类`  定义在类内部的`非静态类`，就是成员内部类。

   成员内部类`不能定义静态方法和变量（final修饰的除外）`。这是因为成员内部类是非静态的，`类初始化的时候先初始化静态成员，`如果允许成员内部类定义静态变量，那么成员内部类的静态变量初始化顺序是有歧义的。

3. `局部内部类（定义在方法中的类）`

   定义在方法中的类，就是局部类。如果一个类`只在某个方法中使用`，则可以考虑使用局部类。

4. `匿名内部类`（要继承一个父类或者实现一个接口、直接使用new来生成一个对象的引用） 

   匿名内部类我们必须要`继承一个父类或者实现一个接口`，当然`也仅能只继承一个父类或者实现一个接口。`

   同时它也是`没有class关键字`，这是因为匿名内部类是`直接使用new来生成一个对象的引用。`

## Stream流

map 必须是一对一的，即每个元素都只能转换为1个新的元素
flatMap 可以是一对多的，即每个元素都可以转换为1个或者多个新的元素。现有一个句子列表，需要将句子中每个单词都提取出来得到一个所有单词列表。这种情况用map就搞不定了，需要flatMap上场了：

```java
public void stringToIntFlatmap() { 
  List<String> sentences = Arrays.asList("hello world","Jia Gou Wu Dao"); 
  // 使用流操作 
  List<String> results = sentences.stream() 
    .flatMap(sentence -> Arrays.stream(sentence.split(" "))) 
    .collect(Collectors.toList()); 
  System.out.println(results); 
} 
```

##### 📖：将一个List或者数组中的值拼接到一个字符串里并以逗号分隔开。

```java
//传统方法
public void testForJoinStrings() { 
  List<String> ids = Arrays.asList("205", "10", "308", "49", "627", "193", "111", "193"); 
  StringBuilder builder = new StringBuilder(); 
  for (String id : ids) { 
    builder.append(id).append(',');
  } 
  // 去掉末尾多拼接的逗号 
  builder.deleteCharAt(builder.length() - 1); 
  System.out.println("拼接后：" + builder.toString()); 
}
//使用Stream流的collect方法
public void testCollectJoinStrings() {
    List<String> ids = Arrays.asList("205", "10", "308", "49", "627", "193", "111", "193");
    String joinResult = ids.stream().collect(Collectors.joining(","));
    System.out.println("拼接后：" + joinResult);
}
```

## for 和 forEach 区别

- 区别：

  ​	for 循环就是按顺序遍历，随机访问元素

  ​	for each循环本质上是使用iterator迭代器遍历，顺序链表访问元素；

- 性能比对：

  ​	对于ArrayList底层为数组类型的结构，使用for循环遍历比使用foreach循环遍历稍快一些，但相差不大

  ​	对于LinkListed底层为单链表类型的结构，使用for循环每次都要从第一个元素开始遍历，速度非常慢；使用foreach可以直接读取当前结点，速度比for快很多

- 原理接释：

  ​	ArrayList数组类型结构对随机访问比较快，而for循环中的get()方法，采用的即是随机访问的方法，因此在ArrayList里，for循环较快

### **JSON** 字符串与**实体对象**互相转化

```java
// 字符串转对象 
Studen student = JSON.parseObject("{"name":"小明","age":18}", Student.class); 
// 对象转字符串 
String str = JSON.toJSONString(student);
```

### **JSON** 字符串与 **JSONObject** 互相转化

```java
// 字符串转JSONObject对象 
JSONObject jsonObject = JSONObject.parseObject("{"name":"小明","age":18}"); 
// JSONObject对象转字符串 
String str = jsonObject.toJSONString();
```

### **JSON** 字符串转化为 **集合类**

```java
// 定义解析字符串
String studentListStr = "[{"name":"小明","age":18},{"name":"小牛","age":24}]";
// 解析为 List<Student>
List<Student> studentList = JSON.parseArray(studentListStr, Student.class);
// 定义解析字符串
String studentMapStr = "{"name":"小明","age":18}";
// 解析为 Map<String,String>
Map<String, String> stringStringMap = 
JSONObject.parseObject(studentMapStr, new TypeReference<Map<String, String>>(){});
```

```java
// 需要在事务提交过后进行某一项或者某一系列的业务操作时候我们就可以使用TransactionSynchronizationManager 
// 通过spring的aop机制将需要进行后置业务处理的操作，提交给spring的处理机制，并且切入到事务处理的后面

// 例如：发消息设置预设值
TransactionSynchronizationManager.registerSynchronization(new TransactionSynchronizationAdapter() {
   @Override
   public void afterCommit() {
      PresetValueMsg presetValueMsg = new PresetValueMsg();
      presetValueMsg.setAgentId(agentForm.getId());
      List<Integer> dataTypes = new ArrayList<>();
      for (PresetValueEnum valueEnum : PresetValueEnum.values()) {
         dataTypes.add(valueEnum.getDataType());
      }
      presetValueMsg.setDataTypes(dataTypes);
      rabbitTemplate.convertAndSend(presetValueExchange,presetValueQueue,JSON.toJSONString(presetValueMsg));
   }
});
```

## 集合📦

## Collections常用方法

- sort(Collection)方法的使用(含义：对集合进行排序)。 

- shuffle(Collection)方法的使用(含义：对集合进行随机排序)。 

- binarySearch(Collection,Object)方法的使用(含义：查找指定集合中的元素，返回所查找元素的索引)。 

- replaceAll(List list,Object old,Object new)。全部替换

- reverse()方法的使用(含义：反转集合中元素的顺序)。

### **ArrayList** **和** LinkedList的区别

```
对于随机访问get和set，ArrayList要优于LinkedList，因为LinkedList要移动指针
ArrayList尾部插入删除还可以，其他部分插入删除效率低

在数组中间插入删除元素时，也是ArrayList高于LinkedList 

LinkedList占用内存多，因为是通过节点来存储数据的

ArrayList的 toArray 方法返回一个数组
ArrayList的 asList 方法返回一个列表
```

#### 求两个list的并集、交集、差集

- 并集：list1.addAll(list2);
- 交集：list1.retainAll(list2);
- 差集：list1.removeAll(list2);

## ArrayList 数组实现

### add方法扩容

```markdown
JDK8中 默认构造方法的大小为0 ，当插入第一个元素时，扩容到10。

存满10个后，再添加元素后，会进行1,5倍的扩容，也就是15。

再进行扩容，是根据15右移一位，15>>1=7，再加上15=22

JDK8以前，默认构造方法的大小就是10。

插入末尾时间复杂度O(1)，插入第i个位置时间复杂度O(n - i)。
```

### 如何删除ArrayList里面的元素

```markdown
# 对于ArrayList的Iterator迭代器的规则是 fail-fast
表示不允许在遍历的时候，被其他线程修改，会抛出异常 ConcurrentModificationException。

# fail-safe:表示在遍历时被其他线程进行修改时，会牺牲一致性来保证遍历成功执行。

例如CopyOnWriteArrayList的迭代器就是实现的fail-safe。相当于线程安全的

ArrayList，通过显式锁 ReentrantLock 实现线程安全。允许存储null值。

CopyOnWriteArraySet：相当于线程安全的HashSet，内部使用

CopyOnWriteArrayList 实现。允许存储null值。

# fail-fast的实现原理
记录了循环被修改前的modCount 和修改后的modCount 进行比较，如果不同则ConcurrentModificationException。
# fail-safe的实现原理
使用成员变量snapShot记录数组对象，add方法中实际上是复制了一个新的数组出来，并且让长度+1。也就是循环和add的数组不是同一个数组，读写分离，遍历时可以修改。
```

#### HashCode

```markdown
# 为什么要比较hashcode？
为了提高效率，如果hashcode不同就不用再比较equals 。保证同一个对象，如果重写了equals而没有重写hashcode，也可能会出现两个相同的对象拥有不同的hashcode
```

#### HashMap

```markdown
# 遍历map：
keySet()/values()/entrySet()/iterator()
```

```markdown
# JDK8之前通过key 的hashCode方法获取hash值，然后通过hash和数组长度进行(n - 1) & hash 与运算来获取下标，如果这个位置存在元素，则判断该元素与要存入元素的hash和key是否相同，如果相同则直接覆盖，否则通过拉链法解决冲突。通过hashcode方法是为了减少hash碰撞的几率，散列更均匀。
# JDK8后，当链表长度大于8，首先会进行调用treeifyBin方法根据hashmap数组来决定是否转换成红黑树，只有数组长度达到64，才会转化成红黑树，否则就只是进行resize方法对数组扩容。

# HashMap的扩容是有上限的，必须小于1<<30。如果超出阈值，则不再增长，阈值会被设置为Integer最大值。
```

##### HashMap在jdk8中相较于jdk7在底层实现方面的不同

（1）使用空参构造器创建HashMap对象时，底层没有直接创建一个长度为16的数组。
（2）jdk8底层存储数据的数组使用的是Node[ ]，而不是Entry[ ] 。
（3）jdk8在首次调用put()方法时，底层才会创建长度为16的数组。
（4）jdk7中底层存储数据的结构是：数组+链表。jdk8中底层存储数据的结构是：数组+链表+红黑树。

```markdown
# HashEntry用来封装具体的键值对，是个典型的四元组。与HashMap中的Entry类似，HashEntry也包括同样的四个域，分别是key、hash、value和next。不同的是，在HashEntry类中，key，hash和next域都被声明为final的，value域被volatile所修饰，因此HashEntry对象几乎是不可变的，这是ConcurrentHashmap读操作并不需要加锁的一个重要原因。由于value域被volatile修饰，所以其可以确保被读线程读到最新的值，这是ConcurrentHashmap读操作并不需要加锁的另一个重要原因。
```

#### JDK1.7和JDK1.8 put方法的区别

**相同点**

1. HashMap是懒惰创建的

2. 计算索引获取桶下标

3. 没有桶下标没被占用，则创建Node节点

4. 如果已经被占用

   1. 如果是TreeNode，表示已经树化了，根据树化的添加更新逻辑

   2. 如果是普通的Node节点，执行链表的添加更新逻辑

5. 返回前检查是否超出阈值，如果超过则扩容

**不同点**

1. JDK7是头插法，JDK8是尾插法

2. JDK7是大于等于阈值且没有空位时才扩容，而8是只要达到阈值就扩容

3. JDK8在扩容计算Node索引时，会进行优化。扩容时，会通过*hash&*旧容量，如果等于*0*，则扩容后位置保持不变，否则会移动，新位置下标 = 旧下表 + 旧容量

#### 负载因子为什么是0.75f？

```
在空间占用和查询时间之间取得较好的平衡
```

#### HashMap 怎样解决冲突？

```
HashMap中处理冲突的方法实际就是链地址法
```

#### JDK7多线程下HashMap添加数据有什么问题？

```
由于是头插法的方式，会导致扩容后重新计算桶下标后导致元素顺序与扩容之前不一致，在多线程环境下使用HashMap会导致死链。
```

#### JDK8多线程下HashMap添加数据有什么问题？

```
可能会出现一个线程还没有完成添加数据，此时是没有元素的，但是另一个线程也在添加元素，会导致第一个线程将第二个线程添加的值覆盖了，也就是丢失数据
```

#### String对象的hashcode()是如何设计的？为什么每次都是乘以31？

```
目的是达到更好的散列效果。
```

#### 扩展问题：抛开HashMap，hash **冲突有那些解决办法？**

```markdown
在产生hash冲突时,两个不相等的对象就会有相同的 hashcode 值. 
当hash冲突产生时,一般有以下几种方式来处理:
# 拉链法:每个哈希表节点都有一个next指针,多个哈希表节点可以用next指针构成一个单向链表，被分配到同一个索引上的多个节点可以用这个单向链表进行存储。

# 开放定址法:一旦发生了冲突,就去寻找下一个空的散列地址,只要散列表足够大,空的散列地址总能找到,并将记录存入List<> iniData = new ArrayList<>()

# 再哈希:又叫双哈希法,有多个不同的Hash函数.当发生冲突时,使用第二个,第三个….等哈希函数计算地址,直到无冲突
```

## **ConcurrentHashMap** 

*ConcurrentHashMap不允许key为null*

### ConcurrentHashMap的结构

<img src="/Users/zhaoxiang/Desktop/java系统笔记/面试java2.assets/image-20220927155846324.png" alt="image-20220927155846324" style="zoom:50%;" />

JDK7中，Segment是一种可重入锁（ReentrantLock），在ConcurrentHashMap中扮演锁的角色；HashEntry则用与存储键值对数据。一个ConcurrentHashMap里包含一个Segment数组。一个Segment包含一个HashEntry数组，每一个HashEntry是一个链表结构的元素，每个Segment守护着一个HashEntry数组里的元素，当对HashEntry数组的数据进行修改的时候，必须获取到与他对应的Segment锁。

#### **JDK7** **和** JDK8 ConcurrentHashMap区别

JDK7，是饿汉式初始化。ConcurrenHashMap使用的是Segment + 数组 + 链表的结构。超过容量的75%则扩容。

JDK8开始，成为懒汉式初始化。ConcurrenHashMap移除了Segment，而是使用和HashMap类似的结构，

数组+链表+红黑树，将数组的每个节点头作为锁，如果多个线程访问的头节点不同，则不会冲突。

达到75%则进行扩容。

初始容量并不是真正指定的大小，如果是16，那么真正的长度为32，表示要放16个元素，如果只有16容量大小，已经超过了扩容阈值，所以直接*2。如果是12，正好是16的75%，但是JDK8中达到了75%就扩容，所以容量变为32。

负载因子，只会在第一次扩容使用到，之后还是会默认使用75%。 

##### ConcurrentHashMap扩容 元素迁移原理

是从后往前进行元素迁移,处理完成的链表会成为forwardingNode，其他线程看到这个forwardingNode则不会重复处理。最后会用新数组替换原数组。			

##### 扩容时的get原理

将原HashEntry数组分为两部分，还没有迁移的部分和已经迁移完成的部分

没有迁移的部分，直接在原数组中进行get即可，可并发

如果是已经迁移成功地部分，则去新数组中get，可并发

##### 扩容时的put原理

将原HashEntry数组分为三部分，还没有迁移的部分、正在迁移的部分和已经迁移完成的部分

还没有迁移的部分，可以并发put

正在迁移的部分，会被阻塞put已经迁移完成的部分，会由其他线程帮忙扩容

#### HashMap **与** ConcurrentHashMap 的异同

1. 都是 key-value 形式的存储数据；

2. HashMap 是线程不安全的，ConcurrentHashMap 是 JUC 下的线程安全的；

3. HashMap 底层数据结构是数组 + 链表（JDK 1.8 之前）。JDK 1.8 之后是数组 + 链表 + 红黑树。当链表中元素个数达到 8 的时候，链表的查询速度不如红黑树快，链表会转为红黑树，红黑树查询速度快 时间复杂度为O(logn)； 

4. HashMap 初始数组大小为 16（默认），默认加载因子为0.75 说明如果表中已经填满了75%以上，就会自动再散列，新表的桶数是原来的两倍。

5. ConcurrentHashMap 在 JDK 1.8 之前是采用分段锁来现实的 Segment + HashEntry，Segment 数组大小默认是 16，2 的 n 次方；JDK 1.8 之后，采用 CAS + Synchronized 来保证并发安全进行实现。

## ConcurrentHashMap的get()方法

```java
public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
  			//spread方法确保返回的是正数
        int h = spread(key.hashCode());
        if ((tab = table) != null && (n = tab.length) > 0 &&
            //表示定位到桶下标对应的头节点是否不为空
            (e = tabAt(tab, (n - 1) & h)) != null) {
            //如果头节点是要找的key
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            //hash为负数表示该bin在扩容中或者是treebin，这时调用find方法来查找
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            //正常遍历链表 用equals比较
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```

### ConcurrentHashMap的put()方法

```java
public V put(K key, V value) {
  return putVal(key, value, false);
}
final V putVal(K key, V value, boolean onlyIfAbsent) {
  if (key == null || value == null) throw new NullPointerException();
  //spread会综合高位低位，具有更好的 hash 性
  int hash = spread(key.hashCode());
  int binCount = 0;
  for (Node<K,V>[] tab = table;;) {
    //f是链表头节点
    //fh是链表头节点的hash
    //i是链表在table中的下标
    Node<K,V> f; int n, i, fh;
    //要创建table
    if (tab == null || (n = tab.length) == 0)
      //初始化table，使用了cas 进行下一轮循环
      tab = initTable();
    //要创建链表头节点
    else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
      //添加链表头使用了cas
      if (casTabAt(tab, i, null,
                   new Node<K,V>(hash, key, value, null)))
        break;                   // no lock when adding to empty bin
    }
    //帮忙扩容
    else if ((fh = f.hash) == MOVED)
      //帮忙之后 进入下一轮循环
      tab = helpTransfer(tab, f);
    else {
      V oldVal = null;
      //锁住链表头节点
      synchronized (f) {
        //再次确认链表头节点没有被移动
        if (tabAt(tab, i) == f) {
          //链表
          if (fh >= 0) {
            binCount = 1;
            //遍历链表
            for (Node<K,V> e = f;; ++binCount) {
              K ek;
              //找到相同的key
              if (e.hash == hash &&
                  ((ek = e.key) == key ||
                   (ek != null && key.equals(ek)))) {
                oldVal = e.val;
                //更新
                if (!onlyIfAbsent)
                  e.val = value;
                break;
              }
              //已经是最后的节点了 ，新增Node节点，追加至链表尾
              Node<K,V> pred = e;
              if ((e = e.next) == null) {
                pred.next = new Node<K,V>(hash, key,
                                          value, null);
                break;
              }
            }
          }
          else if (f instanceof TreeBin) {
            Node<K,V> p;
            binCount = 2;
            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                  value)) != null) {
              oldVal = p.val;
              if (!onlyIfAbsent)
                p.val = value;
            }
          }
        }
      }
      if (binCount != 0) {
        if (binCount >= TREEIFY_THRESHOLD)
          treeifyBin(tab, i);
        if (oldVal != null)
          return oldVal;
        break;
      }
    }
  }
  addCount(1L, binCount);
  return null;
}

```

## 🎙️设计模式

####  适配器模式

分为类适配器和对象适配器

类适配器：既继承又实现。但是由于Java不支持多继承

对象适配器  将对象组合进来 作为行参进行初始化构造

![image-20220731221211655](/Users/zhaoxiang/Desktop/面试java2.assets/image-20220731221211655.png)

#### 装饰器模式

一般是对类的功能进行扩展，IO流中节点流和处理流 处理流是对节点流的封装  实际上最终的数据处理还是使用的节点流

#### 代理模式

spring的aop，类似用梯子上外网，类似经销商

#### 模板方法模式

在一个方法中定义一个算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。如AQS提供tryAcquire，tryAcquireShared等模板方法，给子类实现自定义的同步器。

### 单例模式

见设计模式.md

## 🪧线程

### 线程状态

```markdown
# 线程一共有六种状态，分别为New、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED。
```

##### NEW

```markdown
# 当线程被创建出来还没有被调用start()时候的状态。    
```

##### RUNNABLE

```markdown
# 当线程被调用了start()，且处于等待操作系统分配资源（如CPU）、等待IO连接、正在运行状态，即表示Running状态和Ready状态。

🎙️注：不一定被调用了start()立刻会改变状态，还有一些准备工作，这个时候的状态是不确定的。
```

##### BLOCKED

```markdown
# 当进入synchronized块/方法或者在调用wait()被唤醒/超时之后重新进入synchronized块/方法，锁被其它线程占有，这个时候被操作系统挂起，状态为阻塞状态。

🎙️注：阻塞状态的线程，即使调用interrupt()方法也不会改变其状态。
```

##### WAITING

```markdown
# 无条件等待，当线程调用wait()/join()/LockSupport.park()不加超时时间的方法之后所处的状态，如果没有被唤醒或等待的线程没有结束，那么将一直等待，当前状态的线程不会被分配CPU资源和持有锁。
```

##### TIMED_WAITING

```markdown
# 有条件的等待，当线程调用sleep(睡眠时间)/wait(等待时间)/join(等待时间)/ LockSupport.parkNanos(等待时间)/LockSupport.parkUntil(等待时间)方法之后所处的状态，在指定的时间没有被唤醒或者等待线程没有结束，会被系统自动唤醒，正常退出。
```

##### TERMINATED

```markdown
# 执行完了run()方法。其实这只是Java语言级别的一种状态，在操作系统内部可能已经注销了相应的线程，或者将它复用给其他需要使用线程的请求，而在Java语言级别只是通过Java代码看到的线程状态而已。
```



#### 🌺状态转换图


![img](/Users/zhaoxiang/Desktop/面试java2.assets/70.png)



## 🌃Future

```markdown
开启另外一个线程去异步处理较为繁杂的业务逻辑
```

**需求:多线程/有返回值/异步任务**

```markdown
# Thread类需要Runnable接口作为参数,但是Runnable接口没有返回值

# 因此可以使用RunnableFuture接口  FutureTask接口实现了RunnableFuture

# 而FutureTask又间接实现Future和Runnable接口 FutureTask接口支持callable类型的参数的构造注入 
```

<img src="/Users/zhaoxiang/Desktop/面试java2.assets/image-20220809193755404.png" alt="image-20220809193755404" style="zoom:50%;" />

#### Callable接口和Runnable接口的区别?

1. Runnable接口不能抛异常
2. Runnable接口没有返回值

```java
public class FutureTaskTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
//			 FutureTask需要Callable对象 
        FutureTask<String> futureTask = new FutureTask(new MyThread());
//       Thread需要Runnable类型对象
        Thread t1 = new Thread(futureTask,"t1");
        t1.start();
        String s = futureTask.get();
        System.out.println(s);
    }
}
class MyThread implements Callable<String> {
    @Override
    public String call() throws Exception {
        System.out.println("i am coming!");
        return "你好";
    }
}
```

```java
//FutureTask可以配合线程池使用获取返回结果 而不会因为频繁new线程而浪费资源
public class FutureThreadPoolTest {
  public static void main(String[] args) throws ExecutionException, InterruptedException {
    long begin = System.currentTimeMillis();

    ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);
    FutureTask<String> futureTask1 = new FutureTask(() -> {
      try {
        TimeUnit.MILLISECONDS.sleep(500);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      return "task1 over~";
    });
    //将FutureTask任务交给线程池执行  submit有返回值  execute没有
    fixedThreadPool.submit(futureTask1);

    FutureTask<String> futureTask2 = new FutureTask(() -> {
      try {
        TimeUnit.MILLISECONDS.sleep(300);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      return "task2 over~";
    });
    fixedThreadPool.submit(futureTask2);

    System.out.println(futureTask1.get());
    System.out.println(futureTask2.get());

    try {
      TimeUnit.MILLISECONDS.sleep(300);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    long end = System.currentTimeMillis();
    System.out.println("耗时：" + (end - begin));
    System.out.println(Thread.currentThread().getName() + "\t -----end");
    fixedThreadPool.shutdown();
  }
}
```

```java
//get()方法会阻塞其他线程继续运行,所以一般get放在最后调用执行
//同时get方法还有一个超时设置,超时就不会继续等待结果的返回而是直接抛出异常
public class FutureAPITest {
  public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
    FutureTask<String> task = new FutureTask<String>(()->{
      System.out.println(Thread.currentThread().getName()+"\t ----come in");
      try {
        TimeUnit.SECONDS.sleep(5);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      return "task over~";
    });
    Thread t1 = new Thread(task,"t1");
    t1.start();

    System.out.println(Thread.currentThread().getName()+"\t----忙其他任务");
    System.out.println(task.get(3,TimeUnit.SECONDS));
  }
}
```

```java
// 可以通过isDone()轮询获取结果 但是会一直耗费cpu的资源
while (true) {
  if (task.isDone()) {
    System.out.println(task.get());
    break;
  } else {
    try {
      TimeUnit.MILLISECONDS.sleep(500);
      System.out.println("正在处理中，不要再催了～");
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

##### 结论：Future虽然可以获取结果，但是实际上对于结果的获取并不是很好，要么阻塞get()，要么轮询。

### 👷CompletableFuture

提供了一种类似观察者模式的机制，可以让任务执行完成后通知监听的一方。

<img src="/Users/zhaoxiang/Desktop/面试java2.assets/image-20220811195429635.png" alt="image-20220811195429635" style="zoom:50%;" />

#### 四大核心静态方法来构造CompletableFuture  尽量不要通过new来构造

```java
//没有返回值的构造方法1 
public static CompletableFuture<Void> runAsync(Runnable runnable) {
  return asyncRunStage(asyncPool, runnable);
}
//没有返回值的构造方法2  搭配线程池使用 如果没有指定Executor的方法 默认使用ForkJoinPool.commonPool()作为指定的线程池执行异步代码
public static CompletableFuture<Void> runAsync(Runnable runnable,
                                               Executor executor) {
  return asyncRunStage(screenExecutor(executor), runnable);
}
//有返回值的构造方法1	
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
  return asyncSupplyStage(asyncPool, supplier);
}
//有返回值的构造方法2
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,
                                                   Executor executor) {
  return asyncSupplyStage(screenExecutor(executor), supplier);
}
```

#### 代码实操

```java
//无返回值
public class CompletableFutureTest {
  public static void main(String[] args) throws ExecutionException, InterruptedException {
    CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
      System.out.println(Thread.currentThread().getName());
      try {
        TimeUnit.SECONDS.sleep(1);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    });
    System.out.println(completableFuture.get());
  }
}

//如果没有指定Executor的方法 默认使用ForkJoinPool.commonPool()作为指定的线程池执行
ForkJoinPool.commonPool-worker-1   
  null
```

```java
ExecutorService executorService = Executors.newFixedThreadPool(3);
CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
  System.out.println(Thread.currentThread().getName());
  try {
    TimeUnit.SECONDS.sleep(1);
  } catch (InterruptedException e) {
    e.printStackTrace();
  }
  return "hello completableFuture!";
},executorService);
System.out.println(completableFuture.get());
//completableFuture还有一个方法join()和get()类似 区别在于join()不会出现编译时异常
pool-1-thread-1
  hello completableFuture!
```



```java
CompletableFuture.supplyAsync(() -> {
  System.out.println(Thread.currentThread().getName());
  int result = ThreadLocalRandom.current().nextInt(10);
  try {
    TimeUnit.SECONDS.sleep(1);
  } catch (Exception e) {
    e.printStackTrace();
  }
  System.out.println("result:" + result);
  return result;
}).whenComplete((v, e) -> {
  if (e == null) {
    System.out.println("----计算完成"+v);
  }
}).exceptionally(e->{
  e.printStackTrace();
  System.out.println("异常情况："+e.getMessage());
  return null;
});
System.out.println(Thread.currentThread().getName()+"处理其他事～");

//main线程结束，整个程序就结束了。因为没有指定线程池的时候，默认的ForkJoin是守护线程，main一结束，ForkJoin也会结束。
ForkJoinPool.commonPool-worker-1
  main处理其他事～

  //可以选择让main线程sleep一会。 
  System.out.println(Thread.currentThread().getName()+"处理其他事～");

try {
  TimeUnit.SECONDS.sleep(3);
} catch (InterruptedException e) {
  e.printStackTrace();
}
```

### 🔒Synchronized实现原理

#### 管程

是一种程序结构，结构内的多个子程序（对象或模块）形成的多个工作线程互斥访问共享资源

通过`java -c 类路径` 可以查看到代码反汇编的内容 ` -v`可以看到详细的信息

monitorenter和monitorexist一一配对，但是会有额外的一次monitorexit。是为了防止在同步代码块中出现异常的导致锁没有能够正常释放的问题，所以会有第二次的monitorexit保证锁的正常释放。

#### 为什么所有的对象都可以被作为锁？

之所以每个对象都可以被作为锁，是因为在Java中，最大的父类就是Object类，然后Object底层C++ObjectMonitor与之关联，因此每一个对象都有一个对象监视器，每一个被锁住的对象都会和Monitor进行关联起来。

在C++管程对象ObjectMonitor初始化的时候，会有一系列的参数。

其中，_owner:指向当前获取ObjectMonitor对象的线程；

​			  _WaitSet:存放处于wait状态的线程队列

​			  _EntryList:存放处于等待锁阻塞状态的线程队列

​			  _recursions:锁的重入次数

​			 _count:记录线程获取锁的次数

​			

<img src="/Users/zhaoxiang/Desktop/面试java2.assets/image-20220814144750433.png" alt="image-20220814144750433" style="zoom:30%;" />

### 可重入锁  又名 递归锁

```markdown
# 同一个线程获取外部的锁后，再进入内部方法会自动获取锁（前提：锁对象是同一个对象），不会因为之前已经获取但是没有释放而阻塞。
```

### 🔒加锁流程

1. monitorenter  
2. 判断count==0
   - 如果等于0，表示当前锁对象没有被获取  则可进行加锁操作  count + 1；owner指向当前线程；recursions + 1；记录重入代码块的次数。 执行代码
   - 如果不等于0，说明当前锁对象已经被其他线程获取 
     - 判断owner指向的线程对象是不是自己
       - 否：表示已经被其他线程获取了锁  进入EntryList阻塞
       - 是：获取锁 正常执行代码

### 释放锁流程

1. monitorexit
2. count  - 1；recursions  - 1
3. 判断recursions==0
   - 如果等于0 正常释放 擦除owner
   - 如果不等于0 说明重入没有完全结束 owner保持不变

<img src="/Users/zhaoxiang/Desktop/java系统笔记/面试java2.assets/image-20220919192125732.png" alt="image-20220919192125732" style="zoom:50%;" />

```markdown
# static的方法属于类方法，它属于这个Class（注意：这里的Class不是指Class的某个具体对象），那么static获取到的锁，是属于类的锁。而非static方法获取到的锁，是属于当前对象的锁。所以，他们之间不会产生互斥。加了synchronized 且有static 的方法称为类锁，没有static 的方法称为对象锁。
```

### 排查死锁💀

- jps -l 查看Java相关进程   jstack 进程号    即可看到是否发生死锁

<img src="/Users/zhaoxiang/Desktop/面试java2.assets/image-20220814152137825.png" alt="image-20220814152137825" style="zoom:30%;" />

- jconsole  图形化

  - /usr/libexec/java_home -V
  - cd /Library/Java/JavaVirtualMachines/zulu-8.jdk/Contents/Home
  - jconsole
  - <img src="/Users/zhaoxiang/Desktop/面试java2.assets/image-20220814151928570.png" alt="image-20220814151928570" style="zoom:30%;" />

  

### 中断💔

```markdown
# 一个线程不该由其他线程来强制中断或停止，而是应该自己的线程由自己自行停止。

# 其次，Java没有办法立刻停止一个线程，然而对于一个很耗时的行为，中断很有必要。

# 因此Java提供了一种用于停止线程的协商机制------中断。

# 中断只是一种协商，中断的过程需要自己实现，调用线程的interrupt()，也只是将中断标识设为true。
```

#### 中断API

1. void interrupt(); **实例方法，仅仅设置线程中断状态为true，发起中断线程的协商而不会立刻停止线程。并且中断不活动的线程不会产生影响，也就是如果一个线程结束了，那么在去调用他的interrupt()方法，将会自动恢复它的中断标识位。**
2. static boolean interrupted();   **判断线程是否被中断并清除当前中断状态。该方法做了两件事：**
   1. **返回当前线程的中断状态，测试当前线程是否已经被中断**
   2. **将当前线程的中断状态清除并重新设为false，清除线程的中断状态。**
3. boolean isInterrupted();   **判断当前线程是否被中断（通过检查中断标志位）不会清除中断标记位。**

#### 💡：如何中断一个线程？

- 使用volatile关键字修饰共享变量

```java
public class InterruptThreadTest {
  private static volatile boolean flag = false;

  public static void main(String[] args) {
    new Thread(() -> {
      while (true) {
        if (flag) {
          System.out.println(Thread.currentThread().getName() + "状态被修改，停止循环！");
          break;
        }
        System.out.println("正常运行......");
      }
    }, "t1").start();
    try {
      TimeUnit.MILLISECONDS.sleep(20);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    new Thread(() -> {
      flag = true;
    }, "t2").start();
  }
}
```

- 使用原子类 AtomicBoolean 


```java
public class InterruptThreadTest {
  static AtomicBoolean atomicBoolean = new AtomicBoolean(false);
  public static void main(String[] args) {
    new Thread(() -> {
      while (true) {
        if (atomicBoolean.get()) {//true
          System.out.println(Thread.currentThread().getName() + "atomicBoolean的状态被修改");
          break;
        }
        System.out.println("atomicBoolean正常运行中.....");
      }
    }, "t1").start();
    try {
      TimeUnit.MILLISECONDS.sleep(20);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }

    new Thread( () -> {
      atomicBoolean.set(true);
    },"t2").start();
  }
}
```

- 使用interrupt API


```java
public class InterruptApiTest {
  public static void main(String[] args) {
    Thread t1 = new Thread(() -> {
      while(true) {
        if(Thread.currentThread().isInterrupted()) {  //如果当前线程中断标记位为true
          System.out.println("中断！");
          break;
        }
        System.out.println("====t1正常运行中.....");
      }
    }, "t1");
    t1.start();
    try {
      TimeUnit.MILLISECONDS.sleep(2);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }

    new Thread( () -> {
      t1.interrupt();  //向t1发送中断协商
    },"t2").start();
    /*new Thread(() -> {
            t1.interrupt();
        }, "t2").start();*/
    t1.interrupt();  //也可以自行中断
  }
}
```

#### Interrupt()源码分析

```java
//1、将该线程的中断标记位改为true
//2、可以将自己线程的中断标记位改为true，否则会调用该线程的checkAccess方法，这可能会导致抛出SecurityException 。

//3、如果此线程处于wait join sleep被阻塞状态, 那么在别的线程中调用这个线程对象的interrupt()方法，则会清除中断状态并退出被阻塞的状态，它会收到一个InterruptedException 。
public void interrupt() {
  if (this != Thread.currentThread())
    checkAccess();

  synchronized (blockerLock) {
    Interruptible b = blocker;
    if (b != null) {
      //这个方法是native修饰的  调用操作系统相关的方法
      interrupt0();           // Just to set the interrupt flag
      b.interrupt(this);
      return;
    }
  }
  interrupt0();  
}
```

#### isInterrupted()源码分析

```java
//返回当前线程的中断标记位
public boolean isInterrupted() {
  return isInterrupted(false);//实际上还是调用的操作系统层面的方法  false表示不清除中断标记位
}
private native boolean isInterrupted(boolean ClearInterrupted);
```

#### Interrupt()深入分析

```java
public class InterruptDeepTest {
  public static void main(String[] args) {
    Thread t1 = new Thread(() -> {
      while(true) {
        if (Thread.currentThread().isInterrupted()) {
          System.out.println(Thread.currentThread().getName() + "    " +
                             "中断标记位："+Thread.currentThread().isInterrupted());
          break;
        }
        try {
          TimeUnit.MILLISECONDS.sleep(2);
        } catch (InterruptedException e) {
          //Thread.currentThread.interrupt();  加上这行代码会直接抛出异常并且终止死循环
          e.printStackTrace();
        }
        System.out.println("hello InterruptDeepTest~");
      }
    }, "t1");
    t1.start();

    try {
      TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    new Thread(()->{
      t1.interrupt();
    },"t2").start();
  }
}
//报错 sleep interrupted 并且出现死循环
/*
	1、t1线程的中断标志位一开始是false，每200ms执行一次输出
	2、t2线程在睡眠一秒后运行，将t1的中断标志位改为了true
	3、正常来说，t1会break掉  但是由于t1   catch (InterruptedException e)  捕获了这个异常  则会抛出异常 然后清除中断标记位，又变成了false，就顺理成章的继续死循环。
	4、因此，需要在捕获到异常后  再次将中断标志位改成true，那么直接会在异常后下次直接break终止
*/
```

#### static boolean interrupted源码分析

💡**判断当前线程是否被中断，并且清除中断标记位** 

```java
public class InterruptedTest {
  public static void main(String[] args) {
    System.out.println(Thread.interrupted());
    System.out.println(Thread.interrupted());
    System.out.println(1);
    Thread.currentThread().interrupt();
    System.out.println(Thread.interrupted());
    System.out.println(Thread.interrupted());
  }
}
false
  false
  1
  true
  false  
  源码：
  public static boolean interrupted() {
  //清除中断标志位
  return currentThread().isInterrupted(true);
}
原因：中断标记位默认是false，发出中断协商后，变成true，interrupted方法会返回中断状态并且清除中断标志位，所以第三次输出是true，输出完后会清除标志位，恢复成false，所以最后一次输出是false。
```

```java
public boolean isInterrupted() {
  return isInterrupted(false);
}
private native boolean isInterrupted(boolean ClearInterrupted);
```

⚠️发现：静态方法interrupted和实例方法isInterrupted其实都是调用的同一个方法 只是传的boolean值相反

### LockSupport🥰

无论是先唤醒还是先等待，LockSupport都支持

👉：用来创建锁和其他同步类的基本线程阻塞原语

- park():没有许可证，阻塞当前线程

- unpark():解除被阻塞的线程

⚠️：unpark()实际上是调用了Unsafe的unpark()方法

⚠️：许可证最多只有一个

###  内存屏障🛡️

```markdown
# 内存屏障之前的所有写操作到要会刷新到主内存

# 内存屏障之后的所有读操作都能获得内存屏障之前的所有写操作的最新结果（实现了可见性）
```

- 读屏障 在读指令之前插入读屏障，让工作内存或缓存中的缓存数据失效，重新回到主内存中获取最新数据
- 写屏障 在写指令之后插入写屏障，强制把写缓冲区的数据刷回到主内存中 

```markdown
# volatile在多线程环境下无法保证原子性：若主内存volatiel修饰变量发生修改之后，线程工作内存中的操作将会作废去读主内存最新值，操作出现写丢失问题，即各线程私有内存和主内存中变量不同步，进而导致数据不一致。由此可见volatile解决的事变量读时的可见性问题，但无法保证原子性。例如A线程对主内存的值+1，线程B也是同样，但是A速度很快，把最新值刷回了主内存，线程去看到主内存的值已经是自己的结果了，就会作废一次刷新。
```

-  volatile写之前的操作，都禁止重排序到volatile之后
- volatile读之后的操作，都禁止重排序到volatile之前
- volatile写之后的volatile读，禁止重排序

### volatile是如何解决可见性的？

```java
当代码进行循环达到一定的阈值的时候，JIT即时编译器会将代码块视为热点代码，会进行优化，不再一次次的从物理硬盘中读取数据，而是私自将值进行改变。这样一来，其他线程还是去物理硬盘中修改数据，然后这
时候第一个线程根本不会再去读取物理硬盘中被修改过的数据了，从而导致线程之间不可见。当使用了volatile关键字的时候，JIT会放弃对代码块进行优化，从而保证了线程的可见性。
```

## CAS🔒

*CAS必须借助volatile才能读到共享变量的最新值来实现[比较并交换]的效果。线程数少于cpu核心数的时候，用cas很合适，如果多于则意味着没有跑道了，效率也高不起来。*

🧰工具类：JUC下的atomic类

- i++原子实现：atomicInteger.getAndIncrement();

底层是通过unsafe类来实现的。

❓：`为什么atomicInteger.getAndIncrement();`是原子的

📝：因为通过调用Unsafe的CAS方法，JVM会帮我们实现出CAS的会变指令。有若干条指令组成，从而完成某个功能的实现，并且原语的执行必须是连续的，在执行过程中不允许被中断，也就是CAS是一条CPU的原子指令，不会造成数据不一致的问题。

Unsafe类中所有的方法都是被native修饰的，也就是所有的方法都是直接调用操作系统底层资源执行相应的任务。

⚠️：如果是JDK8，推荐使用LongAdder对象，比AtomicLong性能更高（减少乐观锁的重试次数）

### CAS案例

```java
package com.zx.cas;

import java.util.ArrayList;
import java.util.List;

interface Account {
    // 获取余额
    Integer getBalance();

    // 取款
    void withdraw(Integer amount);

    /**
     *
     * 方法内会启动 1000 个线程，每个线程做 -10 元 的操作
     * 如果初始余额为 10000 那么正确的结果应当是 0
     */
    static void demo(Account account) {
        List<Thread> ts = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            ts.add(new Thread(() -> {
                account.withdraw(10);
            }));
        }
        long start = System.nanoTime();
        ts.forEach(Thread::start);
        ts.forEach(t -> {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        long end = System.nanoTime();
        System.out.println(account.getBalance()
                + " cost: " + (end - start) / 1000_000 + " ms");
    }
}
```

> CAS保证线程安全

```java
public class CASAccount implements Account{

    private AtomicInteger balance;
	/**
     * 构造方法 10000赋值给原子变量 balance
     * @param balance
     */
    public CASAccount(int balance) {
        this.balance = new AtomicInteger(balance);
    }

    @Override
    public Integer getBalance() {
        return balance.get();
    }

    @Override
    public void withdraw(Integer amount) {
        while (true) {
            Integer prev = balance.get();
            int next = prev - amount;
            if (balance.compareAndSet(prev,next)) {
                break;
            }
        }
    }
}
```

> synchronized保证线程安全

```java
class AccountUnsafe implements Account {

    private Integer balance;

    public AccountUnsafe(Integer balance) {
        this.balance = balance;
    }

    @Override
    public Integer getBalance() {
        return this.balance;
    }

    @Override
    public void withdraw(Integer amount) {
        synchronized(this) {
              this.balance -= amount;
        }
    }
}
```

> 测试

```java
public class UnsafeTest {
    public static void main(String[] args) {
        Account account = new CASAccount(10000);
        Account.demo(account);
    }
}
```

### 为什么无锁的效率高？

> 无锁情况下，即使重试失败，线程始终在高速运行，没有停歇，而 `synchronized `会让线程在没有获得锁的时候，发生上下文切换，进入阻塞。
>
> 但是CAS在多核CPU下才能发挥作用，而且线程数最好不要超过CPU数，否则也会发生上下文切换，影响效率。

### LongAddr 累加器

```java
public static void main(String[] args) {
    LongAdder longAdder = new LongAdder();
    longAdder.increment();
    longAdder.increment();
    longAdder.increment();
    System.out.println(longAdder.sum()); //3 只能从0开始，只能累加
}
```

#### LongAddr源码分析

❓：为什么LongAddr那么快？

📝：因为在LongAddr底层有一个base变量，有一个单元个数组Cell[]

- base变量：低并发 直接累加到该变量上
- Cell[]数组：高并发 累加进各个线程自己的槽Cell[i]中
  - 最终结果就是base+Cell[0~i]的和

🟰：1、最初无竞争时只更新base；2、如果更新base失败后，首次创建一个长度为2的Cell数组；3、当多个线程竞争同一个Cell比较激烈的时候，对Cell进行扩容（2的次方）

<img src="/Users/zhaoxiang/Desktop/面试java2.assets/image-20220818172002921.png" alt="image-20220818172002921" style="zoom:67%;" />

### 🙅‍♂️sleep👉CountDownLatch

```java
package atomic;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicTest {

    public static final int SIZE = 50;

    public static void main(String[] args) {

        MyNumber myNumber = new MyNumber();
        for (int i = 0; i < SIZE; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    myNumber.addPlusPlus();
                }
            }, String.valueOf(i)).start();
        }
        try {
          //由于main线程可能在循环还没结束就获取了值，导致数据不是理想的，并且sleep很不优雅
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "result:" + myNumber.atomicInteger.get());

    }
}

class MyNumber {
    AtomicInteger atomicInteger = new AtomicInteger();

    public void addPlusPlus() {
        atomicInteger.getAndIncrement();
    }
}
```

```java
MyNumber myNumber = new MyNumber();
CountDownLatch countDownLatch = new CountDownLatch(SIZE);
for (int i = 0; i < SIZE; i++) {
  new Thread(() -> {
    try {
      for (int j = 0; j < 1000; j++) {
        myNumber.addPlusPlus();
      }
    } finally {
      //每次减1
      countDownLatch.countDown();
    }
  }, String.valueOf(i)).start();
}
//阻塞主线程，countDownLatch计数器没达到0，都会一直阻塞在这里。
countDownLatch.await();
System.out.println(Thread.currentThread().getName() + "result:" + myNumber.atomicInteger.get());
}
```

#### CAS----Unsafe可能因为do while循环自旋消耗性能，或者ABA问题

解决方法：

- ​	AtomicStampedReference 版本戳 + 1  解决修改过多少次的问题
- ​    AtmoicMarkableReference 状态戳   true｜false    解决是否被修改过的问题

### ThreadLocal详解

<img src="https://img-blog.csdnimg.cn/2021041614544691.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTMzMzUwOQ==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:67%;" />

![image-20220722210947841](/Users/zhaoxiang/Desktop/面试java2.assets/image-20220722210947841.png)



每一个Thread对象都有一个属性ThreadLocalMap，ThreadLocalMap是ThreadLocal中的静态内部类。   ThreadLocalMap的Entry继承了弱引用。

⚠️：**所以，ThreadLocalMap就是一个以ThreadLocal对象为Key，任意对象为value的Entry对象。**

<img src="/Users/zhaoxiang/Desktop/面试java2.assets/image-20220818201822163.png" alt="image-20220818201822163" style="zoom:50%;" />



#### 为什么将ThreadLocalMap的keyThreadLocal设为弱引用？

📝：key设置为弱引用是为了防止threadlocal=null时，还无法回收的问题。

#### 「内存泄漏」

```markdown
# 由于ThreadLocal对象是弱引用，如果外部没有强引用指向它，它就会被GC回收，导致Entry的Key为null，但是由于Entry对象还在强引用value，导致value无法被回收，这时「内存泄漏」就发生了，value成了一个永远也无法被访问，但是又无法被回收的对象。ThreadLocal对象同时被弱引用和强引用两个引用着，只要强应用还指向ThreadLocal对象，gc时就不会被回收，如果强引用不指向ThreadLocal对象了，只有key指向，gc时就会被回收。

# 使用ThreadLocal时，一般建议将其声明为static final的，避免频繁创建ThreadLocal实例。
```

### 👦对象的内存布局

在HotSpot虚拟机中，对象在堆内存中的存储布局分为三部分：对象头、实例数据和对齐填充。

⚠️：对齐填充要求对象起始地址必须是8的倍数

#### 对象头

对象头由三部分组成：

1. Mark Word：记录对象和锁的有关信息。当一个对象被 synchronized 关键字加锁之后，围绕锁的操作就都会和MarkWord有关联。MarkWord在64位系统中占8个字节。会保存一些分代年龄、无锁状态下对象的HashCode、偏向锁的线程ID、轻量级锁指向栈中锁记录的指针、指向重量级锁的指针、锁的标志位等内容。
2. 指向类的指针：大小也通常为8个字节，它主要指向类的数据，也就是指向方法区中的位置。
3. 数组长度：只有数组对象才有。

##### Mark Word

⚠️：new一个空的Object，会占有 8 + 8 = 16个字节。也就是对象头默认大小就是16个字节。

因此，Mark Word64bit的存储结构：

<img src="/Users/zhaoxiang/Desktop/java系统笔记/面试java2.assets/image-20220819104738509.png" alt="image-20220819104738509" style="zoom:50%;" />

<img src="/Users/zhaoxiang/Desktop/java系统笔记/面试java2.assets/image-20220819095519571.png" alt="image-20220819095519571" style="zoom:50%;" />

##### 类元信息  也叫类型指针

Customer c1 = new Customer();

`c1就是在栈里的引用，new出来的就是在堆中的对象，那么类元信息就是Customer这个模版在方法区中`

#### 实例数据

```java
class Customer {
  int num；
	boolean flag；
}
只有一个对象头的实例对象  忽略压缩指针的影响
8字节的Mark Word + 8字节的类型指针 + 4个字节 + boolean 1个字节 = 21个字节
由于对齐填充要求必须是8的倍数，所以是 24 个字节
```

<img src="/Users/zhaoxiang/Desktop/java系统笔记/面试java2.assets/image-20220819103723770.png" alt="image-20220819103723770" style="zoom:50%;" />

![image-20220819104237018](/Users/zhaoxiang/Desktop/java系统笔记/面试java2.assets/image-20220819104237018.png)

![image-20220819104352761](/Users/zhaoxiang/Desktop/java系统笔记/面试java2.assets/image-20220819104352761.png)

❓：为什么这里类型指针的是4个字节而不是上面说的8个字节？

为了节约内存空间，所以默认打开了压缩指针。`-XX:+UseCompressedClassPointers`

因此，默认开启压缩指针，8+4 = 12，然后对齐填充到了16个字节。

如果关闭了压缩指针  就是8+8=16

### Synchronized锁升级

JDK5之前，用户态和内核态之间的转换，会消耗大量的系统资源。

在JDK6之后偏向锁是默认开启的，但是启动时间也是有延迟的，默认是4s

`-XX:+UseBiasedLocking 开启偏向锁`

` -XX:BiasedLockingStartupDelay=0  启动时则启动偏向锁`

`-XX:-UseBiasedLocking	关闭偏向锁 关闭之后会默认直接进入轻量级锁状态` 

*无锁----->偏向锁---->轻量级锁---->重量级锁*

hashcode 是 懒加载 只有在调用hashcode方法的时候才会在Mark Word中记录下来

无锁：001

偏向锁：101	MarkWord存储的是偏向的线程ID

轻量级锁：000	MarkWord存储的是指向线程栈中Lock Record的指针

重量级锁：010	MarkWord存储的是指向堆中的monitor对象的指针

>偏向锁会偏向第一个访问锁的线程，如果在接下来的运行过程中，该锁没有被其他的线程访问，那么持有偏向锁的线程后续进入和退出同步快将永远不需要触发同步（不需要再加锁和释放锁，而是去MarkWord中有没有自己的线程ID），也就是偏向锁在资源没有竞争的情况下消除了同步语句，哪怕是CAS也没有，提高程序性能。
>
>如果偏向锁去MarkWord中检查线程ID和自己不想等，则说明发生了竞争，这个时候会尝试使用轻量级锁CAS尝试替换MarkWord中的线程ID为新线程的ID。
>
>- 竞争成功：表示之前的线程不在了，MarkWord中的线程ID为新线程的ID，锁不会升级，因为现在还是一个线程，仍然是偏向锁。
>- 竞争失败：可能需要升级为轻量级锁，才能保证线程间的公平竞争

## Lock接口

```java
public interface Lock {
	//获取锁，调用该方法将会获取锁，当锁获取后，从该方法返回
	void lock();
	//可中断地获取锁，和lock()方法的不同之处在于该方法会响应中断，即在锁的获取过程中可以中断当前线程 
	void lockInterruptibly() throws InterruptedException;
	//尝试非阻塞的获取锁，调用该方法后会立刻返回，如果能够获取则返回true,否则返回false 
	boolean tryLock();
	//超时地获取锁 1、当前线程在超时时间内成功获取锁。2、当前线程在超时时间内被中断。3、超时时间结束返回false。 
	boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
	//释放锁 
	void unlock();
	//获取等待通知组件 
	Condition newCondition();
}
```

## ReentrantLock的解锁解读

#### unlock( )主要目的：就是获取permit，唤醒阻塞线程。

会调用如下方法：release | tryRelease | unparkSuccessor(h);

#### 1）release

##### ①. 主要作用：

- 解锁方法的入口是AQS的release方法，首先会调用tryRelease方法，这个是AQS实现类自己实现的方法，去CAS改变state状态，如果解锁成功，则会进入if里的代码，判断head节点的waitStatus!=0，如果等于0代表没有后置节点需要去唤醒。之后调用unparkSuccessor方法。
- ![image-20221202212907685](/Users/zhaoxiang/Desktop/java系统笔记/面试(Java基础、JVM、多线程).assets/image-20221202212907685.png)

##### 2）tryRelease()

###### ①. 主要作用

- tryRelease方法主要就是让银行受理窗口空出来，通过CAS改变state状态、受理窗口的用户线程置为空，**这样就可以让其他线程进来办业务了**。
- ![image-20221202212928240](/Users/zhaoxiang/Desktop/java系统笔记/面试(Java基础、JVM、多线程).assets/image-20221202212928240.png)

##### 3）unparkSuccessor

###### ①. 主要作用：

- 1.将哨兵节点状态改为0；
- 2.从尾部节点开始扫描，找到距离head最近的一个waitStatus<=0的节点
- 3.最近的这个线程节点waitStatus<=0，就直接唤醒这个线程即可（给他一个许可证）

###### ②. 源码分析

1. 要先知道：waitStatus>0时，代表为CANCELLED = 1状态，即线程取消排队。
2. 如果waitStatus<0，先将头结点的waitStatus状态设为初始值0，之后查看后置节点的状态，如果==>0==代表后置节点取消了排队，不需要唤醒。
3. 但是当前节点需要去唤醒后续的节点让后续节点再去执行，所以会从尾结点开始寻找找到离当前线程最近的一个且waitStatus<0的去唤醒。唤醒操作LockSupport.unpark(s.thread)；取消最近的那个节点的挂起，让他恢复执行能力。

```java
private void unparkSuccessor(Node node) { 
	int ws = node.waitStatus;//获得head节点的状态
  
	if (ws < 0){
    	compareAndSetWaitStatus(node， ws， 0);// 设置head节点状态为0 
     } 	
  
	Node s = node.next;//得到head节点的下一个节点 
  
	if (s == null || s.waitStatus > 0) { //如果下一个节点为null或者status>0表示cancelled状态. 
		//通过从尾部节点开始扫描，找到距离head最近的一个waitStatus<=0的节点 
		s = null; 
		for (Node t = tail; t != null && t != node; t =	t.prev) 
			if (t.waitStatus <= 0) 
			s = t; 
	} 
 
	if (s != null) //next节点不为空，直接唤醒这个线程即可（总之离头节点最近的一个可唤醒节点）
	LockSupport.unpark(s.thread); 
}
```

###### ③. unparkSuccessor()方法中寻找要唤醒的下一个节点时，为什么从后往前遍历？

- 因为enq()方法中，当发生上下文切换的时候，可能导致next=null，就会出现后续节点被漏掉的情况。
- 但是prev是不会出现等于null的情况，所以采用从后往前利用prev来遍历，就不会出现漏掉现象。

<img src="/Users/zhaoxiang/Desktop/java系统笔记/面试(Java基础、JVM、多线程).assets/image-20221202213016177.png" alt="image-20221202213016177" style="zoom:67%;" />



#### 例子说明

##### 情况1：此时同步队列的数据，当t0线程执行完成业务后，进行解锁操作，此时所有等待的线程都没有取消等待。则t0线程会唤醒t1线程。

![在这里插入图片描述](/Users/zhaoxiang/Desktop/java系统笔记/面试(Java基础、JVM、多线程).assets/efdb9e163fd34f9b8630d226304002ee.png)

##### 情况2：如果t1和t3线程取消的排队时，t0线程会唤醒t2，从后往前找离head最近的一个没有取消派对的节点。

![在这里插入图片描述](/Users/zhaoxiang/Desktop/java系统笔记/面试(Java基础、JVM、多线程).assets/c28036deec5c4a0e8154d2771ce787b8.png)

### 流程总结

1. 线程执行到parkAndCheckInterrupt方法时被挂起。
2. 当被头节点唤醒后的线程会继续执行，设置interrupted=true，表示被中断，会继续执行for循环逻辑。
3. 到现在一个正常的获取锁失败（或成功）——>加入同步队列——>挂起——>被唤醒继续执行的流程已经整体走了一遍。

### 举例总结

①. 业务场景，比如说我们有三个线程A、B、C去银行办理业务了，A线程最先抢到执行权开始办理业务，那么B、C两个线程就在CLH队列里面排队如图所示，注意傀儡结点和B结点的状态都会改为-1。（C还是0，因为后面没有节点需要被唤醒了）

![image-20221202213052747](/Users/zhaoxiang/Desktop/java系统笔记/面试(Java基础、JVM、多线程).assets/image-20221202213052747.png)

②. 当A线程办理好业务，离开的时候，会把傀儡结点的waitStatus从-1改为0 | 将status从1改为0，将当前线程置为null

③. 这个时候如果B上位，首先将status从0改为1(表示占用)，把thread置为线程B ，会执行如下图的①②③④，会触发GC，然后就把第一个灰色的傀儡结点给清除掉了，这个时候原来的B结点重新成为傀儡结点。

![image-20221202213109868](/Users/zhaoxiang/Desktop/java系统笔记/面试(Java基础、JVM、多线程).assets/image-20221202213109868.png)

# AQS

**AQS同步器是用来实现锁或者其他同步器组件的公共基础部分的抽象实现，整体就是一个抽象的FIFO队列来完成资源获取线程的排队工作，并通过一个int类型的变量表示持有锁的状态**

**AQS中源码画的是单向链表，其实是CLH变体的虚拟双向队列FIFO**

##### AQS为什么是JUC内容中最重要的基石？

- ReentrantLock | CountDownLatch | ReentrantReadWriteLock | Semaphore。
- 他们内部都有一个抽象内部类Sync，并且继承了AbstractQueuedSynchronizer。

### AQS的执行流程：	

1. 当线程获取锁失败时，会加入到同步队列中，在同步队列中的线程会按照从头至尾的顺序依次再去尝试获取锁执行。
2. 当线程获取锁后如果调用了condition.await()方法，那么线程就加入到等待队列排队，当被唤醒(signal()，signalAll())时再从等待队列中按照从头至尾的顺序加入到同步队列中，然后再按照同步队列的执行流程去获取锁。
3. 所以AQS最核心的数据结构其实就两个队列，同步队列和等待队列，然后再加上一个获取锁的同步状态。

```java
public abstract class AbstractQueuedSynchronizer{
  
  /**
     * 1.【同步队列】的头节点
     */
    private transient volatile Node head;

    /**
     * 2.【同步队列】的尾节点
     */
    private transient volatile Node tail;
  
    /**
     * 3.【同步状态】
     */
    private volatile int state;
  
     /**
     * 4.【等待队列】
     */
    public class ConditionObject implements Condition， java.io.Serializable {

          /** 等待队列的头节点 */
          private transient Node firstWaiter;
      
          /** 等待队列的尾节点 */
          private transient Node lastWaiter;
    }
}
```

### Node

```java
static final class Node {    
	static final Node SHARED = new Node();  // 共享模式下的节点
    static final Node EXCLUSIVE = null; // 独占模式下的节点
    static final int CANCELLED =  1; // 被取消了的状态
    static final int SIGNAL    = -1; // 处于park() 状态等待被unpark()唤醒的状态
    static final int CONDITION = -2; // 处于等待 condition 的状态
    static final int PROPAGATE = -3; // 共享模式下需要继续传播的状态
    volatile int waitStatus; // 当前节点的状态
    volatile Node prev; // 前驱节点指针
    volatile Node next; // 后继节点指针
    volatile Thread thread; // 抢占锁的线程
    Node nextWaiter; // 指向下一个也处于等待的节点
}
```

- 1.**AQS里面有个变量叫State，它的值有几种？**：答案是3个状态：没占用是0，占用了是1，大于1是可重入锁。
- 2.**如果AB两个线程进来了以后，请问队列中共有多少个Node节点？**：答案是3个，其中队列的第一个是傀儡节点(哨兵节点)。

##### acquire( )：源码和3大流程走向

```java
//成功直接返回，失败则将节点以独占锁模式加入队列
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

- **从源码中看出执行流程为：**tryAcquire——>addWaiter——>acquireQueued。

  - ##### tryAcquire()：公平和非公平

    - tryAcquire实现具体加锁逻辑，当加锁失败时返回false，则会执行addWaiter(Node.EXCLUSIVE)，将线程加入到同步队列中。

      Node.EXCLUSIVE为独占锁的模式，即同时只能有一个线程获取锁去执行。

      > 公平和非公平tryAcquire内部的区别：
      > (1) 公平锁与非公平锁的lock()方法唯一的区别：就在于公平锁在获取同步状态时多了一个限制条件：hasQueuedPredecessors()。
      > (2) hasQueuedPredecessors是公平锁加锁时：判断同步队列中是否存在有效节点的方法。

  - addWaiter(Node mode )

    - 首先会初始化一个node节点，将当前线程设置到node节点中。然后判断head和tail节点是否为空，head和tail节点是懒加载的，当AQS初始化时为null，则第一次进来时if (pred != null) 条件不成立，执行enq方法。

    - 如果再有D，D线程这时获取到了cpu的执行权，此时head节点已经初始化，则进入条件中的代码，其实也是通过CAS操作将D节点加入到同步队列尾部，后面会调用acquireQueued。
    - ![image-20221202210607158](/Users/zhaoxiang/Desktop/java系统笔记/面试(Java基础、JVM、多线程).assets/image-20221202210607158.png)

  - addWaiter中的enq(node);	
    - 此时可能多个线程会同时调用enq方法，所以该方法中也使用CAS保证线程安全。for (;;)是个死循环，【第一次循环会CAS操作初始化head(哨兵)节点】，要知道head节点是个空节点，没有设置线程。
    - 然后第二次循环时通过CAS操作将B节点设置为尾部节点，并将B的前置节点指向head，之后会跳出循环，返回生成的Node节点到addWaiter，从源码可以看到addWaiter方法后面没有逻辑，之后会调用acquireQueued。
    - ![image-20221202210549171](/Users/zhaoxiang/Desktop/java系统笔记/面试(Java基础、JVM、多线程).assets/image-20221202210549171.png)

  - acquireQueued(addWaiter(), arg)

    - 这个方法有两个逻辑，首先如果该节点的前置节点是head会走第一个if，再次去尝试获取锁。

      **逻辑1：**获取锁成功，则将头节点设置为自己，并返回到acquire方法，此时acquire方法执行完，代表获取锁成功，线程可以执行自己的逻辑了。这里有下面几个注意点：

      (1) p.next = null；// 将哨兵节点的后置节点为null。
      (2) setHead方法将t1节点设置为头节点，因为头节点是个空节点，所以设置t1线程节点线程为null，设置t1前置节点为null，此时旧的head节点已经没有任何指向和关联，可以被gc回收。

      **逻辑2：**获取锁失败或者前置节点不是头节点，都会走第二个if逻辑，首先会判断当前线程是否需要挂起，如果需要则执行线程挂起。


    - 1.前置节点为头结点&&获取锁成功就**走第一个if逻辑**。
    
    - 2.当获取锁失败 或者 前置节点不是头节点都会**走第二个if逻辑**。


```java
final boolean acquireQueued(final Node node， int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            //如果前置节点是头结点并且成功获取锁
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            //获取锁失败 或者 前置节点不是头结点
            if (shouldParkAfterFailedAcquire(p， node) &&
                parkAndCheckInterrupt())//线程被挂起阻塞
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

#### addWaiter()整体流程为：

将当前线程封装成一个Node对象，然后判断【等待队列】的尾节点是否为空，不为空的话，则将新创建的Node执行入队操作，修改链表指向；如果尾节点为空，则调用enq()执行入队，enq()内部是一个自旋操作，直到入队成功。 

```java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```

#### 队列尾部添加一个节点  返回的是前一个节点

```java
private Node enq(final Node node) {
        //通过死循环来保证节点的正确添加
        for (;;) {
            /**
             * 第一次循环：
             *  tail赋值给临时变量t， 注意此时tail仍然是null, 进入if块
             * 调用compareAndSetHead(new Node()) 创建了一个新的节点 ，这个节点的结构如下：
             *         Node() {    // Used to establish initial head or SHARED marker
             *         }
             *   该节点中的线程为 NULL，也就是我们下文所称的哑节点
             *   此时队列里有了第一个Node
             */
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                /**
                 * 第二次循环：在第一次循环时head赋给了tail,此时tail 不为空，
                 * 进入else块，将线程T1入队，也就是维护链表关系
                 */
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }

```

1. 注意这是一个 for 循环，在第一次执行的时候，由于 tail 为 NULL，因此进入 if 块创建了一个空节点，空节点的意思是该节点中没有包含线程，如图所示。

   ![image-20221202210801273](/Users/zhaoxiang/Desktop/java系统笔记/面试(Java基础、JVM、多线程).assets/image-20221202210801273.png)

2. 第二次循环时，tail 即指向没有线程的那个Node，以下称哑节点，此时 tail 不为空，进入 else 逻辑。
3. 首先更新 T2 Node 的 prev 指针指向哑节点，然后通过 CAS 设置 T2 Node 为尾节点，因为当前尾节点指向的就是哑节点 t，因此 compareAndSetTail(t, node) 可以成功执行，执行成功后，尾节点指针指向 T2 Node。
4. t.next = node 将哑节点的 next 指针指向 T2。

下图是 T2 入队成功后队列图：

![image-20221202210906251](/Users/zhaoxiang/Desktop/java系统笔记/面试(Java基础、JVM、多线程).assets/image-20221202210906251.png)

#### shouldParkAfterFailedAcquire   获取锁失败后是否阻塞

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;//获取前驱节点的等待状态
    if (ws == Node.SIGNAL)
    //SIGNAL状态：前驱节点释放同步状态或者被取消，将会通知后继节点。因此，可以放心的阻塞当前线程，返回true。
        return true;
    if (ws > 0) {//前驱节点被取消了，跳过前驱节点并重试
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {//独占模式下，一般情况下这里指前驱节点等待状态为SIGNAL
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);//设置当前节点等待状态为SIGNAL
    }
    return false;
}
```

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);//阻塞当前线程
    return Thread.interrupted();
}
```

通过调用同步器的release(int arg)方法可以释放同步状态，该方法在释放了同步状态之后，会"唤醒"其后继节点（进而使后继节点重新尝试获取同步状态）。首先释放成功锁后，使用了h指针指向了当前队列的头部，判断一下队列中是否有等待的元素，注意对头元素waitStatus不能是0，如果是0，说明队列只有一个空节点，队列中没有等待元素。因为入队元素后会将头结点的waitStatus改成-1，SIGNAL。

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {//释放同步状态
        Node h = head;
        if (h != null && h.waitStatus != 0)//独占模式下这里表示SIGNAL
            unparkSuccessor(h);//唤醒后继节点
        return true;
    }
    return false;
}
```

接着进入了unpartSuccessor方法。从名字看就是恢复在h节点之后挂起的线程。node就是入参h，head节点，首先把head节点waitStatus从-1改为0。

```java
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;//获取当前节点等待状态
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);//更新等待状态
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {//找到第一个没有被取消的后继节点（等待状态为SIGNAL）
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);//唤醒后继线程
}
```

总结：在获取同步状态时，同步器维护一个同步队列，获取状态失败的线程都会被加入到队列中并在队列中进行自旋；移出队列

（或停止自旋）的条件是前驱节点为头节点且成功获取了同步状态。在释放同步状态时，同步器调用tryRelease(int arg)方法释放同步状态，然后唤醒头节点的后继节点。

### 公平锁和非公平锁

```java
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;
    @ReservedStackAccess
    final void lock() {
//      CAS判断当前是否没有线程占领，如果没有就将state改为1，自己设为owner线程
//			如果已经有线程占有了，那么就和公平锁一样去调用acquire方法
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }
  
static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }
```

#### 公平锁

```java
/**
 * tryAcquire 的公平版本。除非递归调用或没有服务员或者是第一个，否则不要授予访问权限。
 */
protected final boolean tryAcquire(int acquires) {
    // 获取当前线程
    final Thread current = Thread.currentThread();
    // 获取AQS当前state值
    int c = getState();
    // 判断state是否为0，为0则代表当前没有线程持有锁
    if (c == 0) {
        // 首先判断是否有线程在排队，如果有，tryAcquie()方法直接返回false
        // 如果没有，则尝试获取锁资源
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 如果state != 0，则代表有线程持有锁资源
    // 判断占有锁的线程是不是当前线程，如果是，则进行可重入操作
    else if (current == getExclusiveOwnerThread()) {
        // 可重入
        int nextc = c + acquires;
        // 检查锁重入是否超过最大值，二进制第一位表示符号 
        // 01111111 11111111 11111111 11111111
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

#### 非公平锁

```java
//执行不公平的 tryLock。 tryAcquire 在子类中实现，但两者都需要对 trylock 方法进行非公平尝试。
final boolean nonfairTryAcquire(int acquires) {
    // 获取当前线程
    final Thread current = Thread.currentThread();
    // 获取AQS当前state值
    int c = getState();
    // 如果state == 0，说明没有线程占用着当前的锁资源
    if (c == 0) {
        // CAS直接尝试获取锁资源，直接抢锁，不管有没有线程在队列中
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        // 检查锁重入是否超过最大值，二进制第一位表示符号 
        // 01111111 11111111 11111111 11111111
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        // 修改state当前值
        setState(nextc);
        return true;
    }
    return false;
}
```

- 非公平锁获取锁时比公平锁获取时少了一个判断`!hasQueuedPredecessors()`，hasQueuedPredecessors表示判断前面是否还有其他节点，是否需要排队。

```java
public final boolean hasQueuedPredecessors() {
    Node t = tail; 
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

### Condition的await()方法

```java
 public final void await() throws InterruptedException {
            if (Thread.interrupted())  throw new InterruptedException();
     		//添加一个节点到条件变量等待队列中
            Node node = addConditionWaiter();
     		//释放许可，让其他线程正常工作，返回当前节点的状态
            int savedState = fullyRelease(node);
            int interruptMode = 0;
     		//判断当前节点是否已经加入到同步队列中
            while (!isOnSyncQueue(node)) {
                //如果没有加入则直接park挂起
                LockSupport.park(this);
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
   //到这里表示已经被唤醒signal，加入到同步队列中了。则开始使用acquireQueued尝试获取许可
   //从await中唤醒的线程，必须一定要再次获取许可才可以，之前释放了几个就要获取几个许可
     				//否则lock和unLock对不上号了
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
            if (node.nextWaiter != null) // clean up if cancelled
                unlinkCancelledWaiters();
            if (interruptMode != 0)
                reportInterruptAfterWait(interruptMode);
        }
```

#### Condition对象的signal()通知

signal()通知的时候，是在条件等待队列中，按照FIFO进行，首先从第一个节点下手：

```java
//将等待时间最长的线程（如果存在）从该条件的等待队列移动到拥有锁的等待队列。
public final void signal() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}
    /*
    删除并转移节点，直到命中未取消的一或空。
    参数：
    first - （非空）条件队列中的第一个节点
    */
 private void doSignal(Node first) {
            do {
                if ( (firstWaiter = first.nextWaiter) == null)
                    lastWaiter = null;
                first.nextWaiter = null;
//将条件等待队列中的元素移动到同步等待队列的队尾，这样当前面有许可可以使用的时候，就可以自动被唤醒，对于移动过程中，如果是一个被CANCEL的节点，那么也会被直接唤醒。
            } while (!transferForSignal(first) &&
                     (first = firstWaiter) != null);
        }
```

#### 排它锁的释放

```java
public final boolean release(int arg) {
    //tryRelease()是一个抽象方法，在子类中有具体实现和tryAcquire()一样
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            // 唤醒节点的后继者（如果存在）。（遇到CANCEL的直接跳过）
            unparkSuccessor(h);
            return true;
    }
    return false;
}
```

### 共享锁  例如：信号量  读写锁的读锁

#### 获得共享锁使用acquireShared()：

```java
public final void acquireShared(int arg) {
    //tryAcquireShared是一个抽象方法，需要在子类中实现
    //表示获取arg个共享许可 如果返回负数 则表示失败
  	//返回0表示成功，但是已经没有多余的许可可以使用
    //返回正数表示获取成功，并且后续的获取许可的请求也可以成功
    if (tryAcquireShared(arg) < 0)
        //申请失败，进入同步等待队列
        doAcquireShared(arg);
}
private void doAcquireShared(int arg) {
    //加入到同步等待队列，设置节点类型为共享
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                //返回前一个节点
                final Node p = node.predecessor();
                if (p == head) {
                    //如果前一个节点是是头结点，意味着只有第二个节点才有资格申请许可
                    //因为是一个FIFO队列，第一个已经获取了许可
                    //因此第二个节点就是需要获取许可的第一个节点
                    
                    //尝试获取
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                     //如果获取成功，就把自己设置为头部节点并根据条件判断是否要唤醒后续线程
                        //如果条件允许，就会[传播]这个唤醒到后继节点
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                //如果没有申请许可，就只能park住等待
                //如果将来被唤醒了，就会从这里开始执行，又会回到上面的tryAcquireShared()和setHeadAndPropagate()去尝试获取许可和传播唤醒后继节点。
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

#### setHeadAndPropagate()

设置队列头，并检查后继者是否可能在共享模式下等待，如果是这样，如果设置了传播 > 0 或传播状态，则传播。

```java
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // Record old head for check below
    setHead(node);
 
    //propagate > 0表示后续的许可申请释放可以成功  
    //由propagate 和 waitStatus来判断是否可以唤醒后续节点 如果只有propagate来判断，在并发环境中可能会出现线程不能唤醒的情况
    //waitStatus小于0表示已初始但是没有取消的节点
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        //获取下一个节点
        Node s = node.next;
        if (s == null || s.isShared())
            //共享模式的释放操作——发出后继信号并确保传播。 （注：独占模式下，如果需要信号，释放就相当于调用head的unparkSuccessor。）唤醒下一个线程，或者设置传播状态。
            //被唤醒的线程，又会尝试tryAcquireShared()和setHeadAndPropagate()
            doReleaseShared();
    }
}
```

#### doReleaseShared()   唤醒下一个线程，或者设置传播状态。

```java
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                //如果需要唤醒后续线程，则唤醒并将状态设为0 ---  初始化
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            //设置PROPAGATE状态，保证唤醒可以传播下去
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        //如果在上述过程中，被别的线程干扰，则会重试一次。
        //如果没有别的线程干扰，则正常退出。
        if (h == head)                   // loop if head changed
            break;
    }
}
```

### 锁降级

遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级为读锁。

- 同一个线程获取了写锁，在没有释放写锁的情况下，还可以获取读锁，这就是写锁的降级。
- 如果释放了写锁，那么就完全转换为读锁 。
- 锁降级的目的：为了让当前线程感知到数据的变化，保证数据可见性

不可以锁升级 也就是不可以从读锁升级到写锁。

### 比读写锁更快的锁  邮戳锁 StampedLock

stamp（long类型）代表了锁的状态，当stamp返回0的时候， 表示线程获取锁失败。并且，当释放锁或者转换锁的时候，都要传入最初获取的stamp值。

- 所有的获取锁的方法，都返回一个邮戳（Stamp），Stamp为0表示获取失败，其余都表示获取成功。
- 所有释放锁的方法，都需要一个邮戳（Stamp），这个Stamp必须和成功获取锁时得到的Stamp一致
- StampedLock是不可重入的，如果一个线程已经获取了写锁，再去获取写锁的话就会造成死锁。

#### 三种访问模式：

1⃣️Reading（悲观读）：和ReentrantReadWriteLock的读锁类似

2⃣️Writing（悲观写）：和ReentrantReadWriteLock的写锁类似

3⃣️Optmistic reading（乐观读）：无锁机制，支持读写并发，假如被修改再升级为悲观读模式

## 线程池

Java里面线程池的顶级接口是Executor，但是严格意义上讲Executor并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是ExecutorService。 

- newCachedThreadPool

  ```java
  public static ExecutorService newCachedThreadPool() {
      return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
  }
  ```

  - 核心线程数为0，最大线程数为Integer的最大值，救急线程的空闲生存时间为60s

    全部都是救急线程（60s后回收），来一个创建一个救急线程

    救急线程可以无限创建

    队列采用的是SynchronousQueue，特点是没有容量，没有线程来取的时

    候，是放不进去的。（一手交钱一手交货）

- newFixedThreadPool

  ```java
  public static ExecutorService newFixedThreadPool(int nThreads) {
      return new ThreadPoolExecutor(nThreads, nThreads,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>());
  }
  ```

  - 该线程池重用在共享无界队列上运行的固定数量的线程。固定的线程数，核心线程数就是等于最大线程数。就不会存在救急线程，所以执行了两个任务后，第三个任务就会在阻塞队列中等待，直到有线程空闲后再执行。

- newScheduledThreadPool

  ```java
  public ScheduledThreadPoolExecutor(int corePoolSize) {
      super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
            new DelayedWorkQueue());
  }
  ```

  - 要求线程最大为Intger最大值，采用的是DelayedWorkQueue作为等待队列。这个延时队列是无界的，所以不存在救急线程。
  - 可安排在给定延迟后运行命令或者定期地执行。

- newSingleThreadExecutor 

  ```java
  public static ExecutorService newSingleThreadExecutor() {
      return new FinalizableDelegatedExecutorService
          (new ThreadPoolExecutor(1, 1,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>()));
  }
  ```

  - 希望多个任务派排队执行，线程数固定为1，任务数多于1时，会放入无界队列排队。任务执行完成，这唯一的线程也不会被释放。
  - Executors.newSingleThreadExecutor()返回一个线程池（这个线程池只有一个线程）,这个线程池可以在线程死后（或发生异常时）重新启动一个线程来替代原来的线程继续执行下去。

### 七个参数：

1. corePoolSize：核心线程数目（最多保留的线程）
2. maximumPoolSize：最大线程数目    救济线程数目=maximumPoolSize - corePoolSize 
3. keepAliveTime：生存时间-针对救济线程
4. unit：时间单位-针对救济线程
5. workQueue：工作队列
6. threadFactory：线程工厂-可以为线程起名字
7. handler-拒绝策略
   - AbortPolicy ：让调用者抛出RejectedExecutionExeception异常，这是默认策略
   - CallerRunsPolicy：让调用者执行任务
   - DiscardPolicy：放弃本次任务 不会抛出异常
   - DiscardOldestPolicy：放弃队列中最早的任务，本任务取代他

```java
public class ThreadPool {
    public static void main(String[] args) throws InterruptedException {
        //corePoolSize：2 maximumPoolSize：4 KeepAliveTIme：3 阻塞队列 拒绝策略
        ThreadPoolExecutor executor = new ThreadPoolExecutor(2, 4, 3, TimeUnit.SECONDS, new SynchronousQueue<>(), new ThreadPoolExecutor.DiscardOldestPolicy());
        for (int i = 0; i < 7; i++) {
            int finalI = i;
            executor.execute(() -> {
                System.out.println("prepare to execute thread i:"+ finalI );
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException exception) {
                    exception.printStackTrace();
                }
                System.out.println("finish to execute thread " + finalI );
            });
        }

        TimeUnit.SECONDS.sleep(1);
        System.out.println("pool size:" + executor.getPoolSize());
        TimeUnit.SECONDS.sleep(5);
        System.out.println("pool size:" + executor.getPoolSize());

        executor.shutdownNow();
    }
}
```

> 如果在拒绝策略为DiscardOldestPolicy()的时候使用的是同步队列的话，就会栈溢出，因为会弹出最早进入队列的元素，然后进行死循环的调用执行线程池。

```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    if (!e.isShutdown()) {
        e.getQueue().poll();
        e.execute(r);
    }
}
```

### 自定义拒绝策略

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(2, 4, 3, TimeUnit.SECONDS, new SynchronousQueue<>(), (r,e)->{
    System.out.println("自定义拒绝策略");
    r.run(); //直接在当前的(调用提交方法所在)线程运行
});
```

### 如果线程池中的线程遇到异常会出现什么情况呢，会不会被销毁呢？ 

```java
public static void main(String[] args) throws InterruptedException {
    ThreadPoolExecutor executor = new ThreadPoolExecutor(1, 1, 0, TimeUnit.SECONDS, new LinkedBlockingDeque<>());
    executor.execute(() -> {
        System.out.println(Thread.currentThread().getName());
        throw new RuntimeException("error");
    });
    TimeUnit.SECONDS.sleep(1);
    executor.execute(() -> {
        System.out.println(Thread.currentThread().getName());
    });
}
```

> 两次输出的线程名字不同。看来线程池中抛异常的确实会被销毁

### 除了使用new一个ThreadPoolExecutor的方法，我们还可以使用Executors创建一个线程池。

```java
  public static void main(String[] args) throws InterruptedException{
        // 创建容量固定为10的线程池
        ExecutorService service = Executors.newFixedThreadPool(10); //ThreadPoolExecutor extends AbstractExecutorService, AbstractExecutorService implements ExecutorService
        service.execute(() -> {
            System.out.println("hello.word");
        });
        service.shutdown();
    }
```

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
FinalizableDelegatedExecutorService
    //重写了finalize，当线程池被gc时会调用shutdown方法。将传过来的ThreadPoolExecutor又传给了其父类DelegatedExecutorService.点 super(executor)看看里面做了什么？
     static class FinalizableDelegatedExecutorService
        extends DelegatedExecutorService {
        FinalizableDelegatedExecutorService(ExecutorService executor) {
            super(executor);
        }
        protected void finalize() {
            super.shutdown();
        }
    }

 static class DelegatedExecutorService extends AbstractExecutorService {
        private final ExecutorService e;
        DelegatedExecutorService(ExecutorService executor) { e = executor; }
        public void execute(Runnable command) { e.execute(command); }
        public void shutdown() { e.shutdown(); }
        public List<Runnable> shutdownNow() { return e.shutdownNow(); }
        public boolean isShutdown() { return e.isShutdown(); }
        public boolean isTerminated() { return e.isTerminated(); }
        ...
}
原来最后还是调用ExecutorService来实现的逻辑。有代理的感觉，包装的目的主要是为了安全性。看看将它创建的对象强转成ThreadPoolExecutor。
    public static void main(String[] args) throws InterruptedException{
   ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newSingleThreadExecutor();   
}
报错。这是因为它根本不是ThreadPoolExecutor类型的对象。
    如果是newFixedThreadPool强转运行就不会报错。
    public static void main(String[] args) throws InterruptedException{
        ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newFixedThreadPool(1);
}
强转以后可以干什么，当然是动态修改参数了。比如：
    public static void main(String[] args) throws InterruptedException{
        ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newFixedThreadPool(1);
                executor.setCorePoolSize(10);
    }
这样我们线程池的核心容量就改变了。想象要是newSingleThreadExecutor创建的对象也可以改变容量大小，它还可以称之为单线程线程池么？我们使用它来编码，不就有可能因为不知道其容量发生了变化出错吗？
    
看看源码，newCachedThreadPool是做什么的？
        public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>(),
                                      threadFactory);
}
可以看出来，其核心线程数是0，最大线程数是Integer.MAX_VALUE，空闲等待时间是60s，等待队列没有容量。由于其最大线程数几乎可以让你想创建多少线程就创建多少线程，并且空闲线程等待时间是很长的，有60s，因此可能存在许多隐患，建议谨慎使用。
```

## 返回执行结果的任务

```java
 <T> Future<T> submit(Callable<T> task);
```

可以设置按照一定的频率周期来执行任务

```java
 public static void main(String[] args) throws InterruptedException {
        ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(1);
        executor.scheduleAtFixedRate(() ->System.out.println("hello,world!"), 1,3, TimeUnit.SECONDS); //初始delay1s，之后每3s执行一次
        TimeUnit.SECONDS.sleep(20);
        executor.shutdown();
    }
```

还有一个和上面方法很相似的方法scheduleWithFixedDelay。它们甚至连参数都是一模一样，区别：

> scheduleAtFixedRate是按一定的频率去执行任务，不管任务有没有执行完，而scheduleWithFixedDelay是在任务执行结束的基础上再delay一定时间去执行任务。

```java
public static void main(String[] args) throws InterruptedException {
    ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(1);
    executor.scheduleWithFixedDelay(() ->{
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException exception) {
            exception.printStackTrace();
        }
        System.out.println("hello,world!");
        }, 1,3, TimeUnit.SECONDS);
    TimeUnit.SECONDS.sleep(20);
    executor.shutdown();
} 
```



### 提交任务

- void execute(Runnable command);	//没有返回值
- <T> Future<T> submit(Callable<T> task);   //有返回值 

![image-20220629211309716](/Users/zhaoxiang/Desktop/面试java2.assets/image-20220629211309716.png)

## Semaphore信号量

```
用来限制同时访问共享资源的线程数上限

有两个方法，分别是传入一个int类型的值表示线程数上限，还有一个参数。表示公平非公平

获取认证，acquire()，用一个少一个。

最终也需要release来释放
```

### Semaphore的应用

```
限流，让请求线程阻塞，高峰过去再释放许可，它只是限制线程数，而不是线程资源
```

## CountDownLatch

```
CountDownLatch的锁计数本质上就是AQS的资源数 state 。

用来进行线程同步协作，等待所有线程完成倒计时后再恢复运行。

await方法，让线程进入等待，等到其他所有线程完成任务后再恢复运行

countDown方法，让计数－1 用完就得创建新的CountDownLatch，不能修改值

如果释放资源后 state==0 ,说明已经到达latch，此时就可以调用

doReleaseShared 唤醒等待的线程。

CountDownLatch中的锁是响应中断的，如果线程在对锁进行操作期间发生中断，会直接抛出异常
```

<img src="/Users/zhaoxiang/Desktop/java系统笔记/面试java2.assets/image-20220927170022507.png" alt="image-20220927170022507" style="zoom:50%;" />

## CyclicBarrier循环珊栏

```
用来进行线程同步协作 等待线程满足某个计数

和CountDownLatch的区别在于：

CountDownLatch的作用是允许1或N个线程等待其他线程完成执行；
而 CyclicBarrier则是允许N个线程相互等待。

CountDownLatch的计数器无法被重置；CyclicBarrier的计数器可以被重置后使用，因此它被称为是循环的barrier。 

await方法 使计数－1。当计数为0时，便不会阻塞，继续向下运行。

还有一个参数，表示任务执行完后再执行的任务

对象只需要创建一次，当计数为0时，下一次调用会重置为设置的计数
```

<img src="/Users/zhaoxiang/Desktop/java系统笔记/面试java2.assets/image-20220927170100854.png" alt="image-20220927170100854" style="zoom:50%;" />

# JVM

#### 程序计数器

唯一一个不可能发生OOM的区域，线程私有，每一个线程都有一个独立的程序计数器，为的是都能保证在线程切换后能恢复到正确的位置。如果线程正在执行的是一个方法，那么计数器记录的是正在执行的虚拟机字节码指令的地址，如果正在执行的是本地方法，那么计数器的值是underfined（空）。

#### 方法区

线程共享。主要负责常量池的回收与类型卸载。

#### 运行时常量池 

方法区的一部分，Class文件中除了有类的版本，字段，方法，接口等描述信息外，
 还有一项信息是 常量池表(Constant Pool Table)，用于存放 编译器生成的各种字面量和符号引用，这部分内容将在 类加载后存放到方法区的运行时常量池中。除了保存Class文件中描述的符号引用，还会把由符号引用翻译出来的直接引用也存储在运行时常量池中。

##### 出现OOM的情况：

- 堆内存耗尽，对象一直在使用，不能被回收
- 方法区内存耗尽，加载的类越来越多
- 虚拟机栈累计 线程越来越多得不到销毁  

##### 出现SOF情况：

- 虚拟机栈方法调用过多

### 对象的创建

1. 当 java 虚拟机遇到一条字节码 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

   在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需内存的大小在类加载完成后便可完全确定，为对象分配空间的任务实际上便等同于把一块确定大小的内存块从java堆中划分出来。假设java堆中内存是绝对规整的，所有被使用过的内存都放在一边，空闲的内存被放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间方向挪动一段与对象大小相等的距离。这种分配方式称为`指针碰撞(Bump The Pointer)。`

   但如果 java堆中的内存 并不是规整的，虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在分配的时候从列表中找到了一块足够大的空间划分给对象实例，并更新列表上的记录，这种分配方式称为`空闲列表(Free List)。`选择哪种分配方式由 java堆 是否规整决定，而 java堆 是否规整又由所采用的 垃圾收集器 是否带有 空间压缩整理(Compact)的能力决定。

   因此，当使用Serial、ParNew等带压缩整理过程的收集器时，系统采用的分配算法是指针碰撞，既简单又高效；而当使用	CMS这种 基于清除(Sweep)算法 的收集器时，理论上就只能采用较为复杂的空闲列表来分配内存。

2. 除如何划分可用空间外，还有一个需要考虑的问题：对象创建在虚拟机中是非常频繁的行为，即使仅仅修改一个指针所指向的位置，在 并发情况下也并不是线程安全的，可能出现正在给对象A分配内存，指针还没来得及修改，对象B又同时使用了原来的指针来分配内存的情况。解决这个问题有两种方案：

   ①对分配内存空间的动作进行同步处理---实际上虚拟机是采用 CAS,在JDK 5之前Java 是靠synchronized 保证同步的，synchronized是独占锁，独占锁是一种悲观锁，会导致其他线程挂起。乐观锁用到的机制就是 CAS 配上失败重试的方式保证更新操作的原子性。

### 内存溢出与泄漏

通过内存映像分析工具(如Eclipse Memory Analyzer)对dump出来的堆转存快照进行分析，重点是确认内存中的对象是否是必要的，先分清是因为内存泄漏(MemoryLeak)还是内存溢出(Memory Overflow)。
①如果是 内存泄漏，可通过`查看泄漏对象到GC Roots的引用链`，找到泄漏对象是通过怎样的引用路径、与哪些GC Roots相关联，才导致垃圾收集器无法回收他们。
②如果是 内存溢出，就是内存中的对象确实是必须存活的，那就应当检查JVM的堆参数(-Xms和-Xmx)设置，与机器的内存对比，看看是否还有向上调整的空间。再从代码上检查是否存在某些对象生命周期过长、持有状态时间过长、存储结构设计不合理等情况，尽量减少程序运行期的内存消耗。

#### String:intern()

String:intern()是一个本地方法，他的作用是：如果字符串常量池中已经包含一个等于此String对象的字符串，则返回该对象在常量池中的引用。否则，将会此String对象包含的字符串添加到常量池中，并且返回此String对象的引用。 在JDK6或者更早之前的HotSport中，常量池都是分配在 永久代 中。
可以通过  -XX:PermSize  和  -XX:MaxPermSize 限制 永久代 的大小。

**自JDK7起，原本存放在 永久代 的字符串常量池被移至 java堆 之中。**

*JDK6和JDK7的String:intern()区别*

```java
public static void main(String[] args) {
    String str1 = new StringBuilder("计算机").append("软件").toString();
    System.out.println(str1.intern() == str1);

    String str2 = new StringBuilder("ja").append("va").toString();
    System.out.println(str2.intern() == str2);
}
```

这段代码在JDK6中运行，结果是两个false，在JDK7中，会得到一个true和一个false。
因为在JDK6中，intern()方法会把首次遇到的字符串实例 复制到永久代的字符串常量池中存储，返回的也是永久代中的这个字符串实例的引用，而由StringBuilder创建的字符串对象实例实在java堆上。所以不是同一个引用。
在JDK7中的 intern() 方法实现不需要拷贝字符串的实例到永久代中了，既然字符串常量池已经移到了java堆中，那只需要在常量池里记录一下首次出现的实例引用即可，因此intern()返回的引用和由StringBuilder创建的那个字符串实例是同一个。而对 str2中的 Java 这个字符串在执行StrigBuilder.toString()之前就出现出现过了。字符串常量池中已经有它的引用，不符合intern()要求 首次遇到 的原则，“计算机软件”这个字符串是首次出现的，所以返回true。

#### 为什么用元空间代替永久代？

​	类的元数据信息（metadata）转移到Metaspace的原因是PermGen很难调整。PermGen中类的元数据信息在每次FullGC的时候可能会被收集。而且应该为PermGen分配多大的空间很难确定，因为PermSize的大小依赖于很多因素，比如JVM加载的class的总数，常量池的大小，方法的大小等。
​	由于**类的元数据可以在本地内存(native memory)**之外分配,所以其最大可利用空间是整个系统内存的可用空间。这样，你将不再会遇到OOM错误，溢出的内存会涌入到交换空间。最终用户可以为类元数据指定最大可利用的本地内存空间，JVM也可以增加本地内存空间来满足类元数据信息的存储。

#### 设置元空间的参数：

1. -XX:MaxMetaspaceSize:设置元空间最大值，默认是-1，即不限制，或者说只受限于本地内存大小。
2. -XX:MetaspaceSize:指定元空间的初始空间大小 字节为单位，达到该值就会触发垃圾收集进行类型卸载，同时收集器会对该值进行调整，如果释放了大量空间就适当降低该值，如果释放了少量空间，那么在不超过-XX:MetaspaceSize(如果设置了话)的情况下，适当提高该值。
3. -XX:MinMetaspaceFreeRatio：作用是在垃圾收集之后控制最小的元空间剩余容量的百分比，可减少因为元空间不足导致的垃圾收集的频率。类似的还有
4. -XX:MaxMetaspaceFreeRatio：用于控制最大的元空间剩余容量的百分比。

#### 垃圾收集相关参数                                                              

1. -XX:UseSerialGC:打开此开关后，使用Serial+Serial Old组合进行内存回收

2. -XX:UseParNewGC:使用ParNew和Serial Old组合进行垃圾回收，JDK9后弃用

3. -XX:UseConcMarkSweepGC:使用ParNew + CMS + Serial Old。Serial Old作为CMS出现“Concurrent Mode Failure” 失败后的后背收集器使用。

4. -Xloggc:log/gc.log   指定GC log的位置，以文件输出

5. -XX:NewRatio:新生代和老年代的比值

   例如 -XX:NewRatio=4,表示新生代：老年代=1:4，新生代占堆内存的 1/5

6. UseParallelOldGC:使用Parallel Scavenge + Parallel Old

7. SurvivorRatio：Eden和Survivor的比值 默认为8

8. -Xmn:设置新生代的大小

#### 引用计数算法   不是 Java 使用的判断对象是否已经死亡的算法

​	在对象中添加一个引用计数器，每当有一个地方引用他就加1，当引用失效时，就减1，任何时刻计数器为0的时候就说明对象不再是被使用的。但是当两个对象互相引用着彼此，导致计数器都不为0，所以无法被引用计数算法回收。

#### JAVA使用的是  可达性分析算法

​	思路是：通过一系列称为"GC Roots"的根对象作为起始节点集，从这些节点开始，根据引用关系向下搜索，搜索过程所走过的路径称为引用链，如果某个对象到GC Roots之间没有任何引用链相连，或者是GC Roots到这个对象不可达的时候则证明此对象不可能再被引用了。

#### 引用

引用分为 强引用、软引用、弱引用和虚引用。

1. 强引用：只要强引用关系还在，垃圾收集器就永远不会回收被引用的对象。

2. 软引用：首次垃圾回收不会回收该对象，如果内存还不足，会在第二次垃圾回收回收掉被软引用关联的对象。需要配合引用队列来释放。提供了SoftReference类实现了软引用。

3. 弱引用：当垃圾收集器开始工作，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。WeakReference类实现了软引用。需要配合引用队列来释放。

4. 虚引用：必须配合引用队列一起使用，当虚引用引用的对象被垃圾回收后，会将虚引用的对象加入队列，由Reference Handler线程释放其关联的外部资源。

#### 生存还是死亡  

​	如果对象在进行可达性分析算法后发现没有与GC Roots相连接的引用链，那他将会被第一次标记。随后进行一次筛选，筛选条件是此对象是否有必要执行 finalize()，假如对象没有覆盖finalize()，或者finalize()已经被虚拟机调用过，那么虚拟机将这两种情况都视为 没有必要执行。
如果这个对象被视为有必要执行finalize()，那么该对象会被放在一个叫做F-Queue队列中，并在之后由一条由虚拟机自动建立的、低调度优先级的Finalizer线程去执行它们的finalize()。如果某个对象的finalize()执行缓慢或者更极端的发生了死循环，将有可能导致F-Queue队列中的其他对象永久处于等待，甚至导致整个内存回收子系统崩溃。finalize()是对象逃脱死亡命运的最后机会，稍后收集器会对F-Queue中的对象进行第二次标记，如果对象在finalize()中拯救了自己[只要重新与引用链上的任何一个对象建立关联即可]则可以逃脱。

#### finalize方法的理解

将资源的释放和清理放在finalize方法中是非常不好的，非常影响性能，严重时甚至会引起OOM，从JDK9就已经不建议使用了。是由Finalizer线程调度执行的，finalize是守护线程，优先级为8。所以不能保证方法内的一定会被执行。重写了finalize方法的对象在第一次gc时并不能立刻释放它的内存，而是等Finalizer调用完finalize方法后，从第一个unfinalize队列中移除后，第二次GC才会真正释放内存，而移除动作是加锁了的，所以效率很低。

#### JAVA堆 区域

​	新生代（Young Generation）和老年代（Old Generation），在新生代中，每次垃圾收集都会发现大量对象死去，而每次回收后存活的少量对象，将会逐步晋升到老年代中存放。
  部分收集（Partial GC）：指目标不是完整收集整个java堆得垃圾收集，其中又分为：

1. 新生代收集（Minor GC/Young GC）:指目标只是新生代的垃圾收集  新生代分为伊甸园和幸存区

2. 老年代收集（Major GC/Old GC）:指目标只是老年代的垃圾收集

   目前只有CMS收集器会单独收集老年代的行为。

   【注意】Major GC这个词会有混淆，需要区分上下文区分到底是 老年代的收集还是整堆收集。

3. 混合收集（Mixed GC）：

   指目标是收集整个新生代以及部分老年代的垃圾收集，目前只有G1收集器会有这种行为。

4. 整堆收集（Full GC）：收集整个java堆和方法区的垃圾收集

##### Minor GC 触发条件

当Eden区满时，触发Minor GC。

##### Full GC 触发条件

1. System.gc()方法的调用
2. 老年代空间不足
3. 永生区空间不足（JVM规范中运行时数据区域中的方法区，在HotSpot虚拟机中又被习惯称为永生代或者永生区，Permanet Generation中存放的为一些class的信息、常量、静态变量等数据）
4. GC时出现promotion failed和concurrent mode failure
5. 统计得到的Minor GC晋升到旧老年代平均大小大于老年代剩余空间
6. 堆中分配很大的对象

##### 三色标记

- 黑色：已标记
- 灰色：标记中
- 白色：未标记

###### 漏标问题

并发过程中，用户在标记过程中修改了引用关系，导致失去引用，从而被认为应该被回收，但是又被其他已经被标记过的引用后，就不会被回收，但会一直是白色的，也就是漏标问题。

###### 解决

1. 增量更新

   只要发生赋值关系，被赋值的对象就会被记录

2. 原始快照

​		新增对象会被记录

​		被删除引用关系的对象也被记录

##### 新生代使用的是标记复制算法  老年代是标记整理

##### 标记-清除 算法（Mark-Sweep）

先标记需要回收的对象，再进行回收。或者先标记存活的对象，再回收未被标记的对象。
有两个缺点：

1. 执行效率不稳定，如果java堆中包含大量对象，其中大部分都会被回收，那么进行全局标记和清除的效率就会随着对象的数量增多而降低。
2. 内存碎片化的问题，标记，清除之后会产生大量的内存碎片，导致以后在程序运行的时候需要分配较大对象时无法找到足够大的连续的内存空间而不得不提前触发另一次垃圾收集的动作。

##### 标记-复制 算法（Mark-Copying）

​	"半区复制"：将可用内存一分为二，每次只使用其中一块，当这一块的内存用完了，就将还存活的对象复制到另一块上面，然后再把已使用的内存空间一次清理掉。
​	缺点：如果内存中是大量存活的对象，那么这种算法会产生大量的内存间复制的开销，并且可用内存缩小了一半，空间资源浪费严重。
​	"Appel式回收"：把新生代分为一块较大的Eden（伊甸园）空间和两块较小的Survivor空间，每次分配内存只使用Eden和其中一块Survivor空间上。
​	发生垃圾收集时，将Eden和Survivor中仍然存活的对象一次性复制到另外一块Survivor空间上，然后直接清理掉Eden和已使用过的Survivor空间。HotSport虚拟机默认Eden和Survivor的大小比例为8:1，也即每次新生代中可用内存空间为整个新生代容量的90%（Eden的80%+一块Survivor的10%），只有一个Survivor空间即10%的新生代是不会被浪费的。除此之外，还有一个“逃生门”的安全设计，当一个Survivor空间不足以放下一次Minor GC之后存活的对象，就需要依赖其他区域（大多就是老年代）进行分配担保。也就是如果另外一块Survivor空间没有足够空间存放上一次新生代收集下来的存活对象，这些对象便将通过分配担保机制直接进入老年代空间。

##### 标记-整理 算法（Mark-Compact）

​	标记过程和标记-清除一样，但是后续的步骤不是直接清除可回收对象，而是让所有存活下来的对象都向内存空间一端移动，然后直接清理掉边界以外的内存。
​	缺点：如果移动存活对象，尤其是老年代这种每次回收都有大量对象存活区域，更新所有引用这些对象就成为一种极为负重的操作，而且这种对象移动操作必须全程暂停用户应用程序才可以进行

[通常标记-清除算法 也是需要停顿用户线程来标记、清理可回收的对象，只是停顿时间相对而言要短]
HotSport虚拟机里关注吞吐量的Parallel Old收集器是基于标记-整理算法的，而关注延迟的CMS则是基于标记-清除算法的，并且在内存空间碎片过多的情况下，CMS收集器则会进行一次标记-整理算法收集一次。

##### Serial收集器  串行 单线程 新生代收集器   标记-复制

  不仅是他只会使用一个处理器或一条收集线程去完成垃圾收集工作，更重要的是强调在他进行垃圾收集时，必须暂停其他所有工作线程，直到他收集结束。也就是STW。有着优于其他收集器的地方那就是 简单而高效（与其他收集器的单线程相比），对于内存资源受限的环境，它是所有收集器里额外内存消耗最小的，对于单核处理器或处理器核心数较少的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率。所以，Serial收集器对于运行在客户端模式下的虚拟机来说是一个很好的选择。

**Serial Old收集器 是Serial的  老年代版本  单线程  标记-整理**

##### ParNew 收集器	多线程并行  新生代收集器 标记-复制

ParNew是Serial收集器的多线程并行版本。
除了Serial收集器外，只有ParNew收集器可以和CMS收集器配合使用工作。
	JDK5 出现了CMS收集器，首次实现了让垃圾收集线程和用户线程（基本上）同时工作
	但是，作为老年代收集器的CMS无法和在JDK1.4.0中的新生代收集器Parallel Scavenge配合工作，所以在JDK5中使用CMS收集老年代的时候，新生代只能选择ParNew或者Serial之一。ParNew收集器是激活CMS后的默认新生代收集器	也可以使用-XX:+/-UseParNewGC参数选项来强制指定或者禁用它。JDK9开始，取消了-XX:+/-UseParNewGC参数选项，这意味着ParNew合并入CMS，成为专门处理新生代的组成部分。

##### 为什么只有 ParNew能与CMS 收集器配合?

CMS作为老年代收集器，但却无法与JDK1.4已经存在的新生代收集器Parallel Scavenge配合工作；
因为Parallel Scavenge（以及G1都没有使用传统的GC收集器代码框架，而另外独立实现；而其余几种收集器则共用了部分的框架代码；

#### Parallel GC

eden内存不足 发生Minor GC 标记复制 STW

Old内存不足发生 Full GC 标记整理 STW

##### Parallel Scavenge收集器  多线程并行  新生代收集器  标记复制  

##### Parallel Old收集器 是Parallel Scavenge收集器的  老年代版本 多线程并发  标记整理  

*Parallel Scavenge收集器的目标则是达到一个可控制的吞吐量[CMS收集器更专注于停顿时间]*

###### 吞吐量就是 运行程序所消耗的时间 / (运行程序所消耗的时间 + 垃圾收集时间)

###### 高吞吐量的话用哪种gc算法，用哪种垃圾收集器

复制清除	parallel Scavenge

##### parallel Scavenge收集器提供了两个参数用于精准控制吞吐量：

*-XX:MaxGCPauseMillis: 控制最大垃圾收集停顿时间*
控制垃圾收集停顿时间缩短的话，其实是以牺牲吞吐量和新生代空间为代价的，这也直接导致了垃圾收集更频繁，吞吐量也下降。
-XX:GCTimeRatio:设置吞吐量大小-XX:GCTimeRatio:应设置为正整数，表示用户期望虚拟机消耗在GC上的时间不超过程序运行时间的1/（1+N），默认值为99，尽可能的保证应用程序执行的时间为收集器执行时间的99倍，即收集器的时间消耗不超过总运行时间的1%。
由于与吞吐量密切相关，所以Parallel Scavenge收集器也被称为吞吐量优先收集器
**-XX:+UserAdapativeSizePolicy:自适应调节策略。**
当这个参数被激活后，就不需要人工指定新生代对象大小（-Xmn）、Eden和Survivor区的比例（-XX:SurvivorRatio）、晋升老年代对象大小（-XX:PretenureSizeThreshold）等参数，虚拟机会自动根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最适合的停顿时间或者最大的吞吐量。只需要把基本的内存数据设置好(如-Xmx最大堆)，然后使用-XX:MaxGCPauseMillis或者
-XX:GCTimeRatio给虚拟机一个优化目标。这个自适应调节策略也是Parallel Scavenge收集器和ParNew收集器的一个重要特性。

##### 

##### CMS收集器（ConCurrent Mark Sweep）老年代  并发标记清除  并发低停顿  注意响应时间

是以获取最短回收时间为目标的收集器

1. 初始标记（CMS initial mark）   需要STW

2. 并发标记（CMS concurrent mark）

3. 重新标记（CMS remark）         需要STW

4. 并发清除（CMS concurrent sweep）

   初始标记和重新标记需要STW，初始标记仅仅是标记一下GC Roots能直接关联到的对象，速度很快；并发标记则是根据GC Roots的直接关联对象开始遍历整个对象图的过程，耗时较长但是不需要停顿用户线程；重新标记是为了修正在并行标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段停顿时间比初始标记长一些，但比并发标记短的多；并发清除，由于不需要移动对象，所以也可以和用户线程并发

   耗时最长的是并发标记和并发清除，但是垃圾收集器线程都可以和用户线程一起工作，所以CMS收集器的内存回收过程是与用户线程并发执行的。

   缺点：

   1. CMS收集器对处理器资源非常敏感
   2. 由于CMS无法处理浮动垃圾，有可能出现Concurrent Mode Failure失败而导致另一次完全STW的Full GC产生。在CMS的并发标记和并发清除阶段，用户线程还是在继续执行的，会产生新的垃圾对象，但是这些垃圾对象是在这次标记过程之后，CMS无法在本次垃圾收集中处理掉他们，只能等到下一次，这一部分就叫做“浮动垃圾”。
   3. 由于使用的是标记-清理算法，会产生内存碎片，会出现往往老年代还有很多空间，但是找不到一块足够大的内存空间来分配当前对象，而不得不提前触发一次Full GC的情况

##### Garbage First G1 面向服务端的 全功能的 用户可指定停顿时间  默认200ms

哪块内存中存放的垃圾数量最多，回收利益最大，这就是G1的Mixed GC模式
	G1把连续的java堆划分为多个大小相等的独立区域（Region），每一个Region都可以根据需要，扮演新生代的Eden空间、Survivor和老年代空间。
	Region中还有一类特殊的Humongous区域，专门用来存储大对象，G1认为只要大小超过了一个Region容量一半的对象即可判定为大对象。每个Region的大小可以通过参数
	-XX:G1HeapRegionSize设定，取值范围为1MB~32MB，且应为2的N次幂。而对于超过了整个Region容量的超级大对象，将会被存放在N个连续的Humongous Region中。
	G1大多数行为都把Humongous Region作为老年代的一部分来进行看待。
具体的思路就是让G1去跟踪各个Region里面的垃圾堆积的“价值”大小。价值即回收所获得的的空间大小及回收所需要时间的经验值，然后在后台维护一个优先级列表，每次根据用户设定允许的收集停顿时间`
	`(-XX:MaxGCPauseMillis)默认200毫秒，优先处理回收价值收益最大的那些Region。这种使用Region划分内存空间，以及具有优先级的区域回收方式，保证了G1收集器在有限的时间内获取尽可能高的收集效率。

1. 新生代回收   标记复制算法，复制到新的Region中   然后清除
2. 并发标记   老年代回收（达到老年代内存的45%）从GC Roots开始对堆中的对象进行可达性分析，递归扫描整个堆里的对象，找出要回收的对象，这个阶段耗时较长，但可以与用户程序并发执行。

  3.混合收集

​	负责更新Region的统计数据，对各个Region的回收价值和成本进行排序，根据用户所期望的停顿时间来制定回收计划，可以自由选择任意多个Region构成回收集，然后把决定回收的一部分Region的存活对象复制到空的Region中，再清理掉整个旧Region的全部空间，这里涉及到存活对象的移动，是必须暂停用户线程，由多条收集器线程并行完成的。

<img src="C:/Users/zx/Desktop/笔记/image-20220329215128863.png" alt="image-20220329215128863" style="zoom:50%;" />

### G1和CMS的区别

1. G1同时回收老年代和年轻代，而CMS只能回收老年代，需要配合一个年轻代收集器。另外G1的分代更多是逻辑上的概念，G1将内存分成多个等大小的region，Eden/ Survivor/Old分别是一部分region的逻辑集合，物理上内存地址并不连续。
2. CMS在old gc的时候会回收整个Old区，对G1来说没有old gc的概念，而是区分Full  young gc和Mixed gc，前者对应年轻代的垃圾回收，后者混合了年轻代和部分老年代的收集，因此每次收集肯定会回收年轻代，老年代根据内存情况可以不回收或者回收部分或者全部(这种情况应该是可能出现)。
3. G1在压缩空间方面有优势
4. G1通过将内存空间分成区域（Region）的方式避免内存碎片问题
5. Eden，Survivor，Old区不再固定、在内存使用效率上来说更灵活
6. G1可以通过设置预期停顿时间（Pause Time）来控制垃圾收集时间避免应用雪崩现象，可驾驭度，G1 是可以设定GC 暂停的 target 时间的，根据预测模型选取性价比收益更高，且一定数目的 Region 作为CSet，能回收多少便是多少。
7. G1在回收内存后会马上同时做，合并空闲内存的工作、而CMS默认是在STW（stop the world）的时候做
8. G1会在Young GC中使用，而CMS只能在Old区使用
9. SATB 算法在 remark 阶段延迟极低以及借助 RSet 的实现可以不做全堆扫描（G1 对大堆更友好）以外，最重要的是可驾驭度

### 衡量垃圾收集器的三个指标就是：内存占用，吞吐量，延迟

​	在CMS和G1之前的全部收集器，其工作的所有步骤都会产生STW式的停顿；CMS和G1分别使用增量更新和原始快照技术实现了标记阶段的并发，不会因管理的堆内存变大，要标记的对象变多而导致停顿的时间随之增长。但是对于标记阶段后的处理，仍未得到妥善解决。CMS使用标记-清除，虽然避免了整理阶段收集器带来的停顿，但是仍避免不了产生空间碎片。随着空间碎片不断淤积最终依然逃不过STW。G1虽然可以按更小的粒度进行回收，从而抑制整理阶段出现时间过长的停顿，但毕竟也还是要暂停的。

### JDK9的默认收集器 是G1

查看GC基本信息，JDK9之前使用-XX:+PrintGC  之后使用 -Xlog:gc
查看GC详细信息，JDK9之前使用-XX:+PrintGCDetails   之后用 -Xlog:gc*

1. 对象优先在Eden分配

2. 大对象直接进入老年代 

   ​	通过-XX:PretenureSizeThreshold来设置 大于该值即可晋升老年代，目的是避免在Eden区和两个Survivor区之间来回复制。这里要写字节  不能直接写MB单位。还有该参数只对Serial和ParNew两款新生代收集器生效。

3. 长期存活的对象进入老年代

   	对象通常在Eden中诞生，如果经过第一次Minor GC后仍然存活，并且能被Survivor容纳的话，该对象会被移入Survivor区域中，并将其年龄设为1岁，每熬过一次Minor GC就加一岁，当年龄达到一定程度时候（默认15岁）就会晋升到老年代中。对象晋升老年代的阈值，可以通过
   	-XX:MaxTenuringThreshold设置

#### GC分代年龄为什么最大为15？

因为Object Header采用4个bit位来保存年龄，4个bit位能表示的最大数就是15

#### 什么情况对象直接在老年代分配

1. 分配的对象大小  大于eden space。适合所有收集器。

2. eden剩余空间不足分配，且需要分配对象内存大小不小于eden space总空间的一半，直接分配到老年代，不触发Minor GC。适合-XX:+UseParallelGC、-XX:+UseParallelOldGC，即适合Parallel Scavenge。

3. 大对象直接进入老年代，使用-XX:PretenureSizeThreshold参数控制，适合-XX:+UseSerialGC、

   -XX:+UseParNewGC、-XX:+UseConcMarkSweepGC，即适合Serial和ParNew收集器。

4. 动态对象年龄判断

   并不是永远要求对象的年龄必须达到-XX:MaxTenuringThreshold才能晋升老年代

   如果在Survivor空间中低于或等于某年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到-XX:MaxTenuringThreshold中要求的年龄。

5. 空间分配担保

### 类加载

类加载过程分为三个阶段：

1. 将类的字节码文件放入方法区，并创建.class文件
2. 如果此类的父类没有加载，先加载父类
3. 加载是懒惰加载

**加载，验证，准备，解析，初始化，使用，卸载 七个阶段，其中验证，准备和解析统称为连接**
一、加载
根据查找路径找到相应的 class 文件然后导入
二、验证
检查加载的 class 文件的正确性
三、准备
给类中的静态变量分配内存空间
四、解析
是将常量池内的符号引用替换为直接引用的过程
五、初始化
对静态变量和静态代码块执行初始化工作。

### 双亲委派模型

​	站在虚拟机角度，只存在两种不同的类加载器。
1.启动类加载器（BootStrap ClassLoader）
负责加载存放在<·JAVA_HOME·>\lib目录，或者被-Xbootclasspath参数所指定的路径中存放的而且是虚拟机能够识别的类库加载到虚拟机的内存中。
2.其他所有的类加载器 独立存在于虚拟机之外，全部继承自抽象类 ClassLoader

- 启动类加载器(Bootstrap ClassLoader)用来加载java核心类库，无法被java程序直接引用。
- 扩展类加载器(extensions class loader):它用来加载 Java 的扩展库。Java 虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java 类。
- 系统类加载器（system class loader）：它根据 Java 应用的类路径（CLASSPATH）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader()来获取它。
- 用户自定义类加载器，通过继承 java.lang.ClassLoader类的方式实现。

#### 双亲委派模型要求：

​	除了顶层的启动类加载器外，其余的类加载器都应有自己的父类加载器。这些父子类之间不是继承，而是组合来复用父加载器的代码。
​	工作过程：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到最顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求的时候，才会由子加载器去尝试自己完成。

​	例如Object类，存在于rt.jar中，无论哪一个类加载器要加载这个类，最终都要委派给处于顶端的启动类加载器进行加载。因此Object类在程序的各种类加载器环境中都能够保证是同一个类。

##### 好处：java中的类随着他的类加载器一起具备了一种带有优先级的层次关系

#### 双亲委派的实现

用于实现双亲委派模型的代码十分简单，全部集中在java.lang.ClassLoader的loadClass()中

1. 首先 检查请求的类是否已经加载过了
2. 如果父类抛出ClassNotFoundException说明父类无法完成加载请求，再调用自身的findClass()来加载

#### 双亲委派作用：

1. 让上级类加载器中的类对下级共享（反之不行），即能让编写的类依赖到JDK的核心类
1. 让类的加载有优先次序，保证核心类先加载

#### 如何自定义类加载器？

自定义类加载器的方法：`继承 ClassLoader 类,重写 findClass()方法 `

#### 在什么情况下需要自定义类加载器呢?

1. 隔离加载类。 在某些框架内进行中间件与应用的模块隔离 ， 把类加载到不同的环境。比如，阿里内某容器框架通过自定义类加载器确保应用中依赖的 jar包不会影响到中间件运行时使用的 jar 包。
2. 修改类加载方式。 类的加载模型并非强制 ，除Bootstrap 外 ， 其他的加载并非定要引入，或者根据实际情况在某个时间点进行按需进行动态加载。
3. 扩展加载源。 比如从数据库、网络，甚至是电视机机顶盒进行加载。
4. 防止源码泄露。 Java代码容易被编译和篡改，可以进行编译加密。 那么类加载器也需要自定义，还原加密的字节码。

#### 破坏双亲委派模型

1. 继承ClassLoader覆盖loadClass方法
2. 使用线程上下文类加载

## JVM调优实战

![image-20221204180717311](/Users/zhaoxiang/Desktop/java系统笔记/面试(Java基础、JVM、多线程).assets/image-20221204180717311.png)

每秒钟产生75M的垃圾，第11秒则会进行一次新生代的垃圾回收，第11秒的75M便会进入到S1中，S1一共只有100M的内存，那么超过了百分之50，则会直接进入到老年代。

老年代的空间是2048M，也就是能够接受27次的11秒（新生代导致的垃圾进入到老年代），也就是300秒，五分钟一次的full Gc，这显然是不合理的。

## ⚠️重点

新建的对象在Eden中，经历一次Minor GC，Eden中的存活对象就会被移动到第一块survivor space S0，Eden被清空；等Eden区再满了，就再触发一次Minor GC，Eden和S0中的存活对象又会被复制送入第二块survivor space S1（这个过程非常重要，因为这种复制算法保证了S1中来自S0和Eden两部分的存活对象占用连续的内存空间，避免了碎片化的发生）。S0和Eden被清空，然后下一轮S0与S1交换角色，如此循环往复。如果对象的复制次数达到16次，该对象就会被送到老年代中。

### 调整JVM参数

![image-20221204182429109](/Users/zhaoxiang/Desktop/java系统笔记/面试(Java基础、JVM、多线程).assets/image-20221204182429109.png)

22秒会触发一次新生代的回收，22秒的垃圾会进入到S1中，但是并没有达到50%，也就不会进入到老年代中，再一次22秒后，S1和S2交换位置，22秒的垃圾和S1中的垃圾会被复制到S2中（避免碎片化的发生）。这样就不会那么频繁的垃圾进入到老年代从而出发Full GC了。

![image-20221204194324997](/Users/zhaoxiang/Desktop/java系统笔记/面试(Java基础、JVM、多线程).assets/image-20221204194324997.png)

## Redis《还有的笔记在windows电脑上》

#### String（字符串）

- 简介:String是Redis最基础的数据结构类型，它是二进制安全的，可以存储图片或者序列化的对象，值最大存储为512M
- 简单使用举例: `set key value`、`get key`等
- 应用场景：共享session、分布式锁，计数器、限流。

#### Hash（哈希）

- 简介：在Redis中，哈希类型是指v（值）本身又是一个键值对（k-v）结构
- 简单使用举例：`hset key field value` 、`hget key field`
- 应用场景：缓存用户信息等。
- **注意点**：如果开发使用hgetall，哈希元素比较多的话，可能导致Redis阻塞，可以使用hscan。而如果只是获取部分field，建议使用hmget。

#### List（列表）

- 简介：列表（list）类型是用来存储多个有序的字符串。
- 简单实用举例：`lpush key value [value ...]` 、`lrange key start end`
- 应用场景：消息队列，文章列表

#### Set(集合)

- 它是通过HashTable实现的，无序不可重复
- 简单使用举例：
  1. `[sadd 命令] [key 名称] [value 值]` 
  2. ` [smembers 命令] [key 名称]   返回集合中的所有的成员。` 
  3. `[scard 命令] [key 名称]  返回集合中元素的个数。`
  4. `[sismember 命令] [key 名称] [value 值]   判断指定的值是否是集合的成员，假如不是集合的成员，或 key 不存在，返回 0 。`
  5. `[srem 命令] [key 名称] [value 值]   用于移除集合元素中一个或者多个元素，假如要移除的元素不存在，默认不处理。`
- 应用场景：跟踪一些唯一性数据，比如访问某一博客的唯一IP地址信息。仅需在每次访问该博客时将访问者的IP存入Redis中，Set数据类型会自动保证IP地址的唯一性。

#### Zset(sorted set：有序集合)

- 每个元素都会关联一个double类型的分数。Redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的,但分数(score)却可以重复。
- 应用场景：排行榜 点赞

### 看门狗机制

如果你想让Redisson启动看门狗机制，你就不能自己在获取锁的时候，定义超时释放锁的时间，无论，你是通过lock() （void lock(long leaseTime, TimeUnit unit);）还是通过tryLock获取锁，只要在参数中，不传入releastime，就会开启看门狗机制，就是这两个方法不要用： boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException和void lock(long leaseTime, TimeUnit unit);因为它俩都传release但是，你传的leaseTime是-1，也是会开启看门狗机制的。加入一个线程拿到了锁设置了30s超时，在30s后这个线程还没有执行完毕，锁超时释放了，就会导致问题，Redisson给出了自己的答案，就是 watch dog 自动延期机制。其实，这个例子就很容易让人误导，这个30秒不是你传的leaseTime参数为30，而是你不传leaseTime或者传-1时，Redisson配置中默认给你的30秒。

### 为什么我们不传release超时释放锁时间，Redisson也会给我们默认传一个30秒的锁超时释放时间

当一个线程A在获取redis分布式锁的时候，没有设置超时时间，如果在释放锁的时候，出现了异常，那么锁就会常驻redis服务中，当另外一个线程B获取锁的时候，无论你是通过自定义的redis分布式锁setnx，还是通过Redisson实现的分布式锁的方式**if (redis.call(‘exists’, KEYS[1]) == 0) **，在获取锁之前，其实都有一个逻辑判断：如果该锁已经存在，就是key已经存在，就不往redis中写了，也就是获取锁失败那么线程B就永远不会获取到锁，自然就一直阻塞在获取锁的代码处，发生死锁。如果有了超时时间，异常发生了，超时的话，redis服务器自己就把key删除了，也就是锁释放了。这也就避免了并发下的死锁问题。

Redisson提供了一个监控锁的看门狗，它的作用是在Redisson实例被关闭前，不断的延长锁的有效期，也就是说，如果一个拿到锁的线程一直没有完成逻辑，那么看门狗会帮助线程不断的延长锁超时时间，锁不会因为超时而被释放。默认情况下，看门狗的续期时间是30s，也可以通过修改Config.lockWatchdogTimeout来另行指定。另外Redisson 还提供了可以指定leaseTime参数的加锁方法来指定加锁的时间。超过这个时间后锁便自动解开了，不会延长锁的有效期。

从源码中可以得知，如果不传release，默认会给个-1，如果release是-1的话，通过 if (leaseTime != -1) 判断就会开启看门狗机制，这也是为啥我说，无论你是tryLock还是Lock只要不传release，就会开启看门狗机制，所以，如果你想解决由于线程执行慢或者阻塞，造成锁超时释放的问题，就不要在两个方法中传release，实际上，通过传release参数来设置超时时间，风险是比较大的，你需要清楚的知道，线程执行业务的时间，设置的过小，redis服务器就自动给你释放了。

要使 watchLog机制生效 。只要不传leaseTime即可
watchlog的延时时间 可以由 lockWatchdogTimeout指定默认延时时间，但是不要设置太小。如100
watchdog 会每 lockWatchdogTimeout/3时间，去延时。
watchdog 通过 类似netty的 Future功能来实现异步延时
watchdog 最终还是通过 lua脚本来进行延时	
