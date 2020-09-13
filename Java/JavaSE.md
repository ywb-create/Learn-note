# Java SE

## 集合

### 1.集合类体系

- Collection接口：单列数据，定义了存储一组独享的方法的集合
  - List：元素有序，可重复的集合
  - Set：元素无序，不可重复的集合
- Map接口：双列数据，保存具有映射关系的”key-value“对

#### Collection API

```java
public class TestCollectionMethod {
    //测试contains()
    @Test
    public void test(){
        Collection collection=new ArrayList();
        collection.contains(new String("abc"));
        System.out.println(collection.size());
        Iterator iterator=collection.iterator();
        System.out.println(iterator.getClass());
    }

    @Test
    public void testIterator(){
        Collection collection=new ArrayList();
        collection.add("abc");
        collection.add("qwe");
        collection.add("123");
        //正确写法
        Iterator iterator = collection.iterator();
        while(iterator.hasNext()){
            Object object=iterator.next();
            if ("abc".equals(object)){
                iterator.remove();//remove() 删除当前指针指向的集合元素
            }
        }
        //错误写法
        while(collection.iterator().hasNext()){//每调用一次collection.iterator都会返回一个新的迭代器对象
            System.out.println(collection.iterator().next());
        }
    }


    @Test
    public void testForeach(){
        String[] arrs=new String[]{"aaa","bbb","ccc"};
        //foreach原理，内部仍然使用iterator。调用arrs的迭代器中的hasNext()和next(),把next的元素赋值给变量a，
        for (String a:arrs) {              
            a="aaa";
        }
        for (int i = 0; i < arrs.length; i++) {
            System.out.println(arrs[i]);
        }
    }


}
```

#### Collection和Collections

​	Collection是集合的父类，Collection是集合的工具类，里面封装着操作集合的方法

```java
//Collections操作list set map
public class TestCollections {

    @Test
    public void testMethod(){
        List list=new ArrayList();
        list.add(1);
        list.add(2);
        System.out.println(list);
        Collections.reverse(list);//反转list中的元素
        System.out.println(list);
        Collections.shuffle(list);//对list进行随机排序
        System.out.println(list);
        Collections.sort(list);//按照自然顺序升序排序（可传入comparator接口指定）
        System.out.println(list);
        Collections.swap(list,0,1);//对list中的第i个元素和第j个元素进行交换
    }

    //排序相关，均为static
    @Test
    public void testMethod2(){
        List list=new ArrayList();
        list.add(1);
        list.add(2);
        Comparable max = Collections.max(list);//返回集合中最大元素（可传入comparator接口指定）
        System.out.println(max);
        Comparable min = Collections.min(list);
        System.out.println(min);//返回集合中最小元素（可传入comparator接口指定）
        System.out.println(Collections.frequency(list, 1));//返回此元素在集合中的次数

        List dest= Arrays.asList(new Object[list.size()]);//底层要求dest的size()>=list的size()
        System.out.println(dest);//[null, null]
        Collections.copy(dest,list);//把list中的元素复制到新创建的元素中

        Collections.replaceAll(list,1,2);//把集合中的1全换成2
    }

    //同步控制
    @Test
    public void testMethod3(){
        List list=new ArrayList();
        list.add(1);
        list.add(2);
        List list1 = Collections.synchronizedList(list);//返回一个线程安全的list（同步代码块包裹）
//        Collections.synchronizedMap();
//        Collections.synchronizedSet();
    }
}
```

### 2.List

#### ArrayList、LinkedList、Vector

> ArrayList、LinkedList、Vector异同

- 同：都是List的实现类,容器内元素都有序可重复
- 异：ArrayList底层用数组存储，增删慢，查询快，线程不安全；LinkedList底层是双向链表，查询慢，增删快，线程不安全。Vector底层也是数组存储，但是是线程安全的，里面对元素的操作都加锁，效率慢

>ArrayList源码分析

```java
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;//EMPTY_ELEMENTDATA初始为空
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
      minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);//DEFAULT_CAPACITY 10
    }

    ensureExplicitCapacity(minCapacity);
}


private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
      newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
      newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

/*
1.new ArrayList 时，底层数组长度为0
2.当第一次add时，将数组的长度初始化为10，并将元素添加到第一个位置
3.当数组长度不够时，扩容（grow），扩容为原来的1.5倍
*/
```

>LinkedList源码分析（双向链表）

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

>Vector源码

​	新建为10，扩容为原来的2倍

#### 2.List遍历

- foreach遍历时不可操作数据
- iteator()遍历时如使用list.remove()会抛出并发修改异常（ConcurrentModificationException），且效率低
- for()遍历时，删除数据后会改变集合中下标位置，正确的写法为remove(i--)或者倒序删除，效率高

```java
//测试arrayList的遍历
public class TestArrayList {

    public static void main(String[] args) {
        ArrayList list=new ArrayList();
        list.add("123q");
        list.add("123w");
        list.add("123");
        list.add("123");
        //方式一
//        Iterator iterator = list.iterator();
//        while (iterator.hasNext()){
//            Object next = iterator.next();
//            if ("123q".equals(next))
//                iterator.remove();
////                list.remove(next);//产生并发修改异常：ConcurrentModificationException
//        }
//        System.out.println(list);


//        list.remove("123");       //只会remove第一个"123"元素
//        System.out.println(list);

        //方式二
        for (int i = 0; i < list.size(); i++) {
            if ("123q".equals(list.get(i)))
                list.remove(i);//删除后第二个数据下标会变成0
//                list.remove(i--);//删除全部数据的正确做法
        }
        System.out.println(list.indexOf("123w"));
        System.out.println(list);


        //方式三 foreach不能操作集合中数据
//        for (Object o :
//                list) {
//            if ("123q".equals(o))
////                list.remove("123q");//ConcurrentModificationException
//            System.out.println(o);
//        }

    }

    @Test
    public void test(){
        List list=new ArrayList();
        for (int i = 0; i < 100000; i++) {
            list.add(1);
        }
        long l = System.currentTimeMillis();
        for (int i = 0; i < list.size(); i++) {
            list.remove(i--);
        }
        long l1 = System.currentTimeMillis();
        System.out.println(l1-l);//948ms
    }

    @Test
    public void test2(){
        List list=new ArrayList();
        for (int i = 0; i < 100000; i++) {
            list.add(1);
        }
        long l = System.currentTimeMillis();

        Iterator iterator=list.iterator();
        while (iterator.hasNext()){
            iterator.next();
            iterator.remove();
        }
        long l1 = System.currentTimeMillis();
        System.out.println(l1-l);//969ms
    }

    @Test
    public void test3(){
        ArrayList<Integer> list=new ArrayList<>();
        list.add(3);
        list.add(3);
        list.add(2);
        for(int i=0;i<list.size();i++){
            if(list.get(i)==2)
                list.set(i,3);
        }

        System.out.println(list);
    }
    @Test
    public void test4(){
        Object[] objects=new Object[10];
        String[] strings=new String[]{"1","2"};
        for(Object o:strings){
            if(o instanceof String){
                objects[0]=o;
            }
        }
        System.out.println(objects[0]);
    }
}
```

### 3.Set

>Set存储无序不可重复的数据,主要用于过滤重复数据

#### Set的性质

- 无序性：存储的数据在底层数组中并非按照数组索引的顺序添加，而是根据数据的hash值

- 不可重复性：保证添加的元素按照equals()判断时，不能返回相同的元素，只能添加一个

#### 2.要求

- 向Set中添加的数据所在的类一定要重写hashCode()和equals()。
- 重写的hashCode()和equals()尽量保持一致性，则在equals()使用的属性都会用来计算hash值

#### 3.HashSet

##### 添加元素流程

​	先计算元素的哈希值，然后根据散列算法计算此hash值对应的元素下标，再比较此集合下标下的hash值是否与该元素相同，若不同，直接添加到Set中，若相同，则equals判断内容是否相同，相同则不添加，不相同则添加到Set中。

##### 扩容流程

​	初始化数组为16，若使用率大于0.75，则扩容为原来的2倍。

##### 面试题

```java
public class  InterviewHashSet {

    public static void main(String[] args) {
        HashSet set=new HashSet();
        Person p1 = new Person(1001, "AA");
        Person p2= new Person(1002,"BB");
        set.add(p1);
        set.add(p2);
        System.out.println(set);//[Person{id=1002, name='BB'}, Person{id=1001, name='AA'}]
        p1.setName("CC");

        //因为p1的name发生了改变，所以hash值改变，remove()时找的hash值与原来add()时不同，所以没有成功删除
        set.remove(p1);
        System.out.println(set);//[Person{id=1002, name='BB'}, Person{id=1001, name='CC'}]

        //新添加的hash值与前两个都不相同，在数组中的索引位置不同，所以并没有进行equals判断就添加了
        set.add(new Person(1001,"CC"));
        System.out.println(set);//[Person{id=1002, name='BB'}, Person{id=1001, name='CC'}, Person{id=1001, name='CC'}]


        //此时添加的元素的hash值与p1相同，然后进行equals判断，但内容不一样，也可以成功添加
        set.add(new Person(1001,"AA"));
        System.out.println(set);//[Person{id=1002, name='BB'}, Person{id=1001, name='CC'}, Person{id=1001, name='CC'}, Person{id=1001, name='AA'}]

    }

}

class Person{
    int id;
    String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return id == person.id &&
                name.equals(person.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }

    public Person(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

#### 4.LinkedHashSet

​	此类在添加数据的同事,每个数据还维护了两个地址，记录此数据的前一个数据和后一个数据

​	优点：对于频繁的遍历操作，此类的效率要高于HashSet

#### 5.TreeSet

​	TreeSet底层的数据结构为红黑树，向TreeSet中添加的数据要求时相同类的对象，TreeSet中存储的的对象要**实现Comparable或Cmparator接口。**自然排序中，判断两个对象是否相同的标准为compateTo()的返回值不为0，不再是equals。定制排序中，判断的标准为Compare的返回值不为0。

### 4.Map

#### 1.理解

​	Map的key：无序，不可重复，使用Set存储

​	Map的Value：无序，可重复，使用Collection存储

​	Map中的Entry：无序不可重复，使用 Set存储（一个Key-Value对构成一个Entry对象）

#### 2.HashMap

> HashMap底层是数组+链表+红黑树的结构

##### 1.API

```JAVA
public class TestHashMapMethod {
    public static void main(String[] args) {
        Map<Integer,Integer> map=new HashMap<>(4);//求索引位置时是当前元素的hash值 按位与 数组的长度  tab[i = (n - 1) & hash])
        Object[] objects = map.values().toArray();
        int n=4;
        n |= n >>> 1;
        System.out.println(n);
    }

    //添加，删除，修改方法
    @Test
    public void testMethod(){
        Map map=new HashMap();
        //添加
        map.put("AA",1);
        map.put("BB",2);
        //删除
        map.remove("BB");
        map.put("CC",3);
        map.put("DD",4);
        //修改
        map.put("DD",5);
        System.out.println(map);//{AA=1, CC=3, DD=5}

        Map map1=new HashMap();
        map1.putAll(map);//添加所有
        System.out.println(map1);//{AA=1, CC=3, DD=5}
        map1.clear();//清空
        System.out.println(map1);//{}

    }

    //元素查询
    @Test
    public void testMethod2(){
        Map map=new HashMap();
        map.put("AA", 1);
        map.put("BB",2);
        Object aa = map.put("AA", 2);

        //根据key查值
        System.out.println(map.get("AA"));//1
        //当前map中是否有当前key
        System.out.println(map.containsKey("AA"));//true
        //当前map中是否有当前value
        System.out.println(map.containsValue(1));//true
        //当前元素数量
        System.out.println(map.size());//2
        //判断当前map是否有元素
        System.out.println(map.isEmpty());//false
    }

    //map的遍历
    @Test
    public void testMethod3() {
        Map map = new HashMap();
        map.put("AA", 1);
        map.put("BB", 2);
        //方式1（还要在get(key),效率略低）
        Set set = map.keySet();
        Iterator iterator = set.iterator();
        while (iterator.hasNext()) {
            Object next = iterator.next();
            System.out.println("键：" + next + "  值：" + map.get(next));
        }

        //方式2(底层使用方式3)
        Collection values = map.values();
        for (Object o :
                values) {
            System.out.println(o);
        }

        //方式3（建议使用，同时取出key和value，效率高）

        Iterator<Map.Entry<Integer,String>> iterator2 = map.entrySet().iterator();
        while(iterator2.hasNext()){
            Map.Entry<Integer,String> entry = (Map.Entry<Integer, String>) iterator.next();
            System.out.println(entry.getKey()+" "+entry.getValue());
        }

        //底层使用方式2
        map.forEach((key,value)->{
            System.out.println(key+" "+value);
        });

    }

}
```

#### 3.HashMap底层原理

##### 1.put()

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;								//1.初始化Node数组，resize的结果为16
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);		//2.计算hash值后，map中无相同的hash值 则添加元素
    else {																					//3.有相同的hash值
        Node<K,V> e; K k;
        if (p.hash == hash &&												//3.1	equals(key)判断，相同则覆盖value
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;	
        else if (p instanceof TreeNode)							//3.2 判断是否为红黑树的节点
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);//3.2.1 是红黑树节点，进行红黑树的比对
        else {
            for (int binCount = 0; ; ++binCount) {	//3.2.2 不是红黑树，进行链表遍历
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);	//3.2.2.1 遍历后key均不相同
                    if (binCount >= TREEIFY_THRESHOLD - 1) 		//判断链表长度是否大于8，不是则插入元素到链表尾部；是则判断
                        treeifyBin(tab, hash);								//桶长度是否大于64，大于则变红黑树，不大于则扩容桶
                    break;
                }
                if (e.hash == hash &&													//3.2.2.2 有相同的key值，进行覆盖
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { 														//4.1 判断是否为Value被覆盖，是则返回旧值
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)													//4.2 判断新加元素后元素个数是否大于负载因子*桶长度
        resize();																		//大于则扩容为当前size两倍
    afterNodeInsertion(evict);
    return null;
}
```

##### 2.hash

```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}

hash算法：first = tab[(n - 1) & hash]
```

##### 2.size为2的n次幂

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

​	初始化HashMap时，会强制将size（桶个数）设置为2的n次幂，主要是为了在计算桶的下标值时【 桶下标=hash & （size-1）】，2的n次幂-1正好二进制全为1，之后的与操作可以迅速算出桶的下标，能够更好的散列。

##### 4.负载因子

```java
/**
 * The load factor used when none specified in constructor.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

 * <p>As a general rule, the default load factor (.75) offers a good
 * tradeoff between time and space costs.  Higher values decrease the
 * space overhead but increase the lookup cost (reflected in most of
 * the operations of the <tt>HashMap</tt> class, including
 * <tt>get</tt> and <tt>put</tt>).  The expected number of entries in
 * the map and its load factor should be taken into account when
 * setting its initial capacity, so as to minimize the number of
 * rehash operations.  If the initial capacity is greater than the
 * maximum number of entries divided by the load factor, no rehash
 * operations will ever occur.
 *

负载因子为 0.75的原因：
  负载因子大则空间利用率提升，但是时间效率降低。  1时满了才扩容
  负载因子小则时间效率提升，但是空间利用率降低。	 0.5时一半就扩容
```

##### 5.链表长度为8

​	链表长度为8主要是一个数学问题，泊松分布。

```
Because TreeNodes are about twice the size of regular nodes, we
* use them only when bins contain enough nodes to warrant use
* (see TREEIFY_THRESHOLD). And when they become too small (due to
* removal or resizing) they are converted back to plain bins.  In
* usages with well-distributed user hashCodes, tree bins are
* rarely used.  Ideally, under random hashCodes, the frequency of
* nodes in bins follows a Poisson distribution
* (http://en.wikipedia.org/wiki/Poisson_distribution) with a
* parameter of about 0.5 on average for the default resizing
* threshold of 0.75, although with a large variance because of
* resizing granularity. Ignoring variance, the expected
* occurrences of list size k are (exp(-0.5) * pow(0.5, k) /
* factorial(k)). The first values are:
*
* 0:    0.60653066
* 1:    0.30326533
* 2:    0.07581633
* 3:    0.01263606
* 4:    0.00157952
* 5:    0.00015795
* 6:    0.00001316
* 7:    0.00000094
* 8:    0.00000006
* more: less than 1 in ten million
```

#### 4.LinkedHashMap

​	底层Node新增两个指针,分别指向上个元素和下个元素,对于频繁的遍历操作,此结构优于HashMap

#### 5.Hashtable

​	HashTable是线程安全的集合类,不可以存储null的健值,对比HashMap效率低。

### 5.CollectionOfNull

​	在集合中，TreeSet、TreeMap、Hashtable在存储null值时会抛出空指针异常

```java
public class TestCollectionOfNull {
    public static void main(String[] args) {

        ArrayList arrayList=new ArrayList();
        arrayList.add(null);
        System.out.println(arrayList);//[null]

        LinkedList list=new LinkedList();
        list.add(null);
        System.out.println(list);//[null]

        HashSet hashSet=new HashSet();
        hashSet.add(null);
        System.out.println(hashSet);//[null]

        LinkedHashSet linkedHashSet=new LinkedHashSet();
        linkedHashSet.add(null);
        System.out.println(linkedHashSet);//[null]

//        TreeSet treeSet=new TreeSet();
//        treeSet.add(null);
//        System.out.println(treeSet);//NullPointerException

        HashMap hashMap=new HashMap();
        hashMap.put(null,null);
        System.out.println(hashMap);//{null=null}

//        TreeMap treeMap=new TreeMap();
//        treeMap.put(null,null);
//        System.out.println(treeMap);//NullPointerException

//        Hashtable hashtable=new Hashtable();
//        hashtable.put(null,null);
//        System.out.println(hashtable);//NullPointerException

    }
}
```

### 6.线程安全的集合类

​	以上集合除了Vector和Hashtable，其他集合在高并发的环境下都可能会出现线程安全问题，但是Vector和Hashtable的的效率太低，不适合在高并发的场景中应用，除了上述两个集合外，java还提供了以下用于线程安全的集合类

#### Collections

```java
public static void testCollections(){

    ArrayList list=new ArrayList();
    List list1 = Collections.synchronizedList(list);
  //        Collections.synchronizedSet();
	//        Collections.synchronizedMap();
    for (int i = 0; i < 30; i++) {
        new Thread(()->{
            list1.add(UUID.randomUUID().toString().substring(0,8));
            System.out.println(list1);
        }).start();
    }

}
```

#### CopyOnWriteArrayList

```JAVA
//适合进行遍历操作，添加时要复制，效率低
public static void testList(){
    //List ist=new ArrayList();//ConcurrentModificationException
    //原理：写线程拿到锁之后修改集合，写完创建一个新集合（老集合size()+1）返回一个修改好的新集合 然后释放锁
    List<String> list=new CopyOnWriteArrayList();
    for (int i = 0; i < 30; i++) {
        new Thread(()->{
            list.add(UUID.randomUUID().toString().substring(0,8));
            System.out.println(list);
        }).start();
    }
}
```

#### CopyOnWriteArrayList

```JAVA
public static void testSet(){
    Set<String> set=new CopyOnWriteArraySet();
    for (int i = 0; i < 30; i++) {
        new Thread(()->{
            set.add(UUID.randomUUID().toString().substring(0,8));
            System.out.println(set);
        }).start();
    }
}
```

#### ConcurrentHashMap

```JAVA
public static void testMap(){
    // Map map=new HashMap();
    Map map=new ConcurrentHashMap();
    for (int i = 0; i < 30; i++) {
      new Thread(()->{
        map.put(Thread.currentThread().getName(),UUID.randomUUID().toString().substring(0,8));
        System.out.println(map);
      },String.valueOf(i)).start();
    }
  }
```

##### 1.7的ConcurrentHashMap

​	ConcurrentHashMap在1.7采用的是分段锁机制，用继承自ReetrantLock的Segment来维护桶。多个线程同时访问不同的Segment上的桶，并发更高，其中并发度与Segment的个数有关

##### 1.8的ConcurrentHashMap

​	ConcurrentHashMap在1.8采用的是CAS+synchronized+数组+链表+红黑树实现的，Node中的val和next用volatile修饰，保证其可见性。在进行put操作时，先进行CAS自旋，成功后加锁。

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,		//进行CAS
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            synchronized (f) {					//自旋成功后加锁
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
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

## 异常

### 1.概述

​	将程序中执行中发生的不正常情况称为异常，开发过程中的语法错误和逻辑错误不是异常。

### 2.分类

​	1.Error：Java虚拟机无法解决的严重问题

​	2.Exception：因编译错误或偶然的外在因素导致的一般性问题

### 3.Exception

#### 1.分类

​	编译期异常：如IOException、ClassNotFountException、CloneNotSupportedException 

​	运行期异常：RuntimeException（ClassNotCastException、IndexOfBoundsException、NullPointerException、NosuchElementException、IllegalStateException）

```java
public class TestRuntimeException {

    @Test
    public void test(){
        //java.lang.NullPointerException
        int args[]={1,2,3};
        args=null;
        System.out.println(args[2]);
    }

    @Test
    public void test2(){
        int args[]={1,2,3};
        System.out.println(args[3]);//ArrayIndexOutOfBoundsException
        String s="abc";
        System.out.println(s.charAt(3));//StringIndexOutOfBoundsException
    }

    @Test
    public void test3(){
        Object obj=new Date();//ClassCastException
        String s=(String) obj;
    }

    @Test
    public void test4(){
        String string="123a";
        Integer.parseInt(string);//NumberFormatException（数据格式异常）
    }

    @Test
    public void test5(){
        int a=1/0;//ArithmeticException

    }



}
```

#### 2.异常处理--抓抛模型

​	1.抛（throws）：一旦出现异常，在异常生成一个对应异常类的对象，并将此对象抛出，**抛出后，其后代码不再执行**，子类方法中抛出的异常要小于等于父类抛出的异常

```java
class Father{
    public void show() throws IOException{}
}

class Son extends Father{
    @Override
    public void show() throws FileNotFoundException{}
}

//子类重写的方法抛出的异常类型要小于等于父类抛出的异常类型
public class TestExceptionOfExtends {
    public static void main(String[] args) {

        Father father=new Son();//因为在多态时，捕捉的是父类方法抛出的异常类型，如果子类抛出的异常类型大于父类，那会无法捕捉
        try {
            father.show();
        } catch (IOException e) {
            e.printStackTrace();
        }

        Son son=new Son();
        try {
            son.show();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }

    }
}
```

​	2.抓（try-catch-finally）：此结构会将编译期出现的异常延迟到运行期出现

#### 3.try-catch-finally

​	1.try{}中声明的变量在出代码块后无法使用。

​	2.catch中要子类在上，父类在下

​	3.一旦出现异常，则会跳进catch中，抓到后进入finally代码块中或跳出try结构，**try中异常后的代码不再执行**。

​	4.若try-catch中有return语句，则会先把finally中的代码执行完再return，若finally中有return，则不会在执行try-catch中的return，此举可能会导致异常丢失，在实际开发中，不建议在finally中使用retrun语句。

```java
public static void main(String[] args) {
    int i;
    try{
        i=1/0;
    }catch(Exception e){
        e.printStackTrace();
    }
    System.out.println("测试捕捉到运行时异常后程序会继续运行");

    InputStream in=null;
    try {
        in=new FileInputStream("hello.txt");
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }finally {
        try {
            //如果new时未找到，跳进catch，最后执行finally时in为null，则会报nullPointerException
            if ( in != null)
                in.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    System.out.println("测试捕捉到编译时异常后程序会继续运行");
}
```

#### 4.throw

​	throw是手动产生一个异常对象。

## 反射

​	反射是JAVA被视为动态语言的关键，反射机制允许程序在运行期间借助Reflection API取得任何类的内部信息，并且直接操纵任意对象的内部实行及方法。

​	反射时Class类的一个实例，对应着加载到内存的一个运行时类。

### 获取Class的实例的四种方法

```java
//测试获取Class实例的四种方式（第三种最常用）
//获取的Class实例在ClassLoader中被加载后会被缓存一段时间，所以以下的四个clazz为同一对象，不过gc可以回收这些Class对象
public class TestGetClass {

    @Test
    public void test() throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        //1.通过  类.class
        Class<Person> clazz=Person.class;

        //2.通过 对象.getClass()
        Person person=new Person();
        Class<? extends Person> clazz1 = person.getClass();

        //3.通过Class的静态方法 Class.forName()        最常用
        Class clazz2 = Class.forName("java21.Person");

        //4.通过类加载器
        ClassLoader classLoader = TestGetClass.class.getClassLoader();
        Class<?> clazz3 = classLoader.loadClass("java21.Person");

        System.out.println(clazz==clazz1);//true
        System.out.println(clazz==clazz2);//true
        System.out.println(clazz==clazz3);//true
        
    }

}
```

### 创建运行时类的对象

```java
//测试创建运行时类
public class TestCreateInstance {

    @Test
    public void test() throws IllegalAccessException, InstantiationException {
        Class<Person> clazz = Person.class;
        Person person = clazz.newInstance();//被创建的类必须要有权限要够（不能为private修饰）的空参构造器
        System.out.println(person);
    }

}
```

### 获取运行时类的完整结构

```java
@myAnno("class")
public class Person {
    private String name;
    private int age;
    static int staticField;//静态属性有默认值

    public Person(){}

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    //构造器私有
    private Person(String name){
        this.name=name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @myAnno("method")
    public String show(String name,int age ){
        System.out.println(name+":"+age);
        return name;
    }

    private static int testStatic(){
        return staticField;
    }


}

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE,ElementType.METHOD})
@interface myAnno{
    String value() default "";
}

//测试调用当前运行时类的结构
public class TestGetRuntimeClassStruct {

    //操作运行时类的指定属性
    @Test
    public void test() throws Exception {
        //获取Class对象
        Class<Person> clazz = (Class<Person>) Class.forName("java21.Person");
        //创建当前Class对象的实例
        Person person = clazz.newInstance();
        //获取特定属性
        Field name = clazz.getDeclaredField("name");
        //保证属性可访问
        name.setAccessible(true);
        //设置属性值
        name.set(person,"小笨蛋");
        //获取属性值
        Object o = name.get(person);
        System.out.println(o);

        //获取类中的静态属性
        System.out.println("**************************");
        Field staticField = clazz.getDeclaredField("staticField");
        staticField.setAccessible(true);
        System.out.println(staticField.get(null));
        System.out.println(staticField.get(Person.class));
    }

    //操作运行时类的指定方法
    @Test
    public void test1() throws Exception{
        Class<Person> clazz = (Class<Person>) Class.forName("java21.Person");
        Person p = clazz.newInstance();
        Method method = clazz.getDeclaredMethod("show", String.class, int.class);
        method.setAccessible(true);

        // 第一个参数：方法的调用者，接下来的参数：方法的实参;invoke()的返回值即为此方法的返回值
        String str= (String) method.invoke(p,"小笨蛋",22);//执行方法
        System.out.println(str);

        System.out.println("*****调用类中的静态方法******");
        Method staticMethod = clazz.getDeclaredMethod("testStatic");
        staticMethod.setAccessible(true);
        Object o = staticMethod.invoke(null);
        Object o1 = staticMethod.invoke(Person.class);
        System.out.println(o+"  "+o1);
    }

    //操作运行时类的指定构造器(最常用的还是clazz.newInstance() 具有通用性)
    @Test
    public void test2() throws Exception{
        Class<Person> clazz = (Class<Person>) Class.forName("java21.Person");
        Constructor<Person> constructor = clazz.getDeclaredConstructor(String.class, int.class);
        constructor.setAccessible(true);
        Person person = constructor.newInstance("小笨蛋", 22);
        System.out.println(person);

    }

    //反射获取注解的值
    @Test
    public void test3() throws NoSuchMethodException {
        Class<Person> clazz=Person.class;
        Method method=clazz.getDeclaredMethod("show", String.class, int.class);
        method.setAccessible(true);
        Annotation[] declaredAnnotations = method.getDeclaredAnnotations();
        for (int i = 0; i < declaredAnnotations.length; i++) {
            Annotation declaredAnnotation = declaredAnnotations[i];//多态，需强转类型
            System.out.println(declaredAnnotation);
        }

        //反射获取方法上的注解值
        myAnno annotation = method.getAnnotation(myAnno.class);
        System.out.println(annotation);
        System.out.println(annotation.value());

        //反射获取类上的注解值
        myAnno declaredAnnotation = clazz.getDeclaredAnnotation(myAnno.class);
        System.out.println(declaredAnnotation.value());

    }

}
```

### 反射的应用-->动态代理

```java
//反射的应用-动态代理
public class TestProxy {

    public static void main(String[] args) throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        Superman superman=new Superman();
        Human proxyInstance = (Human) ProxyFactory.getProxyInstance(superman);
        proxyInstance.eat("番茄酱");
        String belief = proxyInstance.getBelief();
        System.out.println(belief);

    }

    @Test
    public void testLambda(){
        Superman superman=new Superman();
        Human instance = (Human) Proxy.newProxyInstance(superman.getClass().getClassLoader(), superman.getClass().getInterfaces(),
                (Object proxy, Method method, Object[] args) -> {
            System.out.println("lambde的动态代理");
            Object invoke = method.invoke(superman, args);
            invoke+="...";
            System.out.println("invoke="+invoke);//返回的是代理类加强过的被代理类的方法的返回值
            return invoke;
        });
        instance.eat("番茄酱");
        String belief = instance.getBelief();
        System.out.println(belief);
        instance.test();
    }

}

class ProxyFactory{


    public static Object getProxyInstance(Object obj){//obj:被代理对象
        MyInvocationHandler handler=new MyInvocationHandler();
        handler.setObj(obj);
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(),handler);
    }

}

class MyInvocationHandler implements InvocationHandler{

    private Object obj;//被代理对象
    public void setObj(Object obj) {
        this.obj = obj;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("进行了代理");
        Object invoke = method.invoke(obj, args);
        return invoke;
    }
}

//共同实现的接口
interface Human{
    String getBelief();
    void eat(String food);
    Object test();
}
//被代理类
class Superman implements Human{

    public Superman(){}

    @Override
    public String getBelief() {
        return "中国国产党";
    }

    @Override
    public void eat(String food) {
        System.out.println("我喜欢吃"+food);
    }

    public Object test(){
        return new Object();
    }
}
```

## 注解

### 元注解

```java
		@Retention：指定修饰的注解的生命周期
        RetentionPolicy
            SOURCE：源文件保存，编译后就不再保留
            CLASS：被编译后保存，运行后不保留（默认）
            RUNTIME：运行时依旧保存，可以通过反射读取信息

    @Target：指定修饰的注解可以修饰那种程序元素
        ElementType
            TYPE,               类，接口，注解，枚举
            FIELD,              域
            METHOD,             方法
            PARAMETER,          参数
            CONSTRUCTOR,        构造器
            LOCAL_VARIABLE,     局部变量
            PACKAGE,            包
    @Docunment：指被修饰的注解被javadoc工具提取成文档，如果定义此注解，则Retention必须定义为RUNTIME
    @Inherited：被修饰的注解经具有继承性，某类使用，其子类自动继承
```

### 注解原理

注解的实例是一个动态代理类（AnnotationInvocationHandler）的对象

```java

class AnnotationInvocationHandler implements InvocationHandler, Serializable {
    private static final long serialVersionUID = 6182022883658399397L;
    private final Class<? extends Annotation> type;
  	//键是我们注解属性名称，值就是该属性当初被赋上的值 。也就是说我们标记的值在这里可以找到。
    private final Map<String, Object> memberValues;
  	//是我们标记的方法集。
    private transient volatile Method[] memberMethods = null;

    AnnotationInvocationHandler(Class<? extends Annotation> var1, Map<String,Object> var2){
        Class[] var3 = var1.getInterfaces();
        if (var1.isAnnotation() && var3.length == 1 && var3[0] == Annotation.class) {
            this.type = var1;
            this.memberValues = var2;
        } else {
            throw new AnnotationFormatError();
        }
    }
  	//代理类中任何方法的调用都会被转到这里来。至此，自定义注解完成了调用并产生作用。
    public Object invoke(Object var1, Method var2, Object[] var3) {
        String var4 = var2.getName();
        Class[] var5 = var2.getParameterTypes();
        if (var4.equals("equals") && var5.length == 1 && var5[0] == Object.class) {
          return this.equalsImpl(var3[0]);
        } else if (var5.length != 0) {
          throw new AssertionError("Too many parameters for an annotation method");
        } else {
          byte var7 = -1;
          switch(var4.hashCode()) {
            case -1776922004:
              if (var4.equals("toString")) {
                var7 = 0;
              }
              break;
            case 147696667:
              if (var4.equals("hashCode")) {
                var7 = 1;
              }
              break;
            case 1444986633:
              if (var4.equals("annotationType")) {
                var7 = 2;
              }
          }

          switch(var7) {
            case 0:
              return this.toStringImpl();
            case 1:
              return this.hashCodeImpl();
            case 2:
              return this.type;
            default:
              //从map中找到我们在注解上的赋值
              Object var6 = this.memberValues.get(var4);
              if (var6 == null) {
                throw new IncompleteAnnotationException(this.type, var4);
              } else if (var6 instanceof ExceptionProxy) {
                throw ((ExceptionProxy)var6).generateException();
              } else {
                if (var6.getClass().isArray() && Array.getLength(var6) != 0) {
                  var6 = this.cloneArray(var6);
                }

                return var6;
              }
          }
       }
    }
}
```

所有注解都实现了下面的接口：

```java
public interface Annotation {
    boolean equals(Object obj);
    int hashCode();
    String toString();
    Class<? extends Annotation> annotationType();
}
```

​	通过反射获取注解时，返回的是Java运行时生成的动态代理对象$Proxy1

#### 总结

​     **注解本质是一个继承了Annotation的特殊接口，其具体实现类是Java运行时生成的动态代理类。通过代理对象调用自定义注解（接口）的方法，会最终调用AnnotationInvocationHandler的invoke方法。该方法会从memberValues这个Map中索引出在注解上赋予的对应的值**

#### 流程

> 注解->反射动态生成代理类->AnnotationInvocationHandler#invoke()->map(key)找到对应值->获取注解上的值

