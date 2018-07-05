### list

##### list接口
![](https://github.com/TrueOr/Java/raw/master/Collection/picture/list结构.png)<br>
List是一个有序的集合，和set不同的是，List允许存储项的值为空，也允许存储相等值的存储项。

List是继承于Collection接口，除了Collection通用的方法以外，扩展了部分只属于List的方法。
![](https://github.com/TrueOr/Java/raw/master/Collection/picture/list方法.png)<br>
从上图可以发现，List比Collection主要多了几个add（…）方法和remove(…)方法的重载，还有几个index(…)， get(…)方法。
而AbstractList也只是实现了List接口部分的方法，和AbstractCollection是一个思路。

##### ArraryList
ArrayList是一个数组实现的列表，由于数据是存入数组中的，所以它的特点也和数组一样，查询很快，但是中间部分的插入和删除很慢。<br>
源代码分析：
* ArrayList的类关系和成员变量:
```
//ArrayList继承了Serializable并且申明了serialVersionUID，表示ArrayList是一个可序列化的对象，可以用Bundle传递
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == EMPTY_ELEMENTDATA will be expanded to
     * DEFAULT_CAPACITY when the first element is added.
     */
     //从这里可以看到，ArrayList的底层是由数组实现的，并且默认数组的默认大小是10
    private transient Object[] elementData;

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;
```
* ArrayList构造函数:
```
//ArrayList有2个构造函数，一个是默认无参的，一个是传入数组大小的
//在JavaEffect书中明确提到，如果预先能知道或者估计所需数据项个数的，需要传入initialCapacity
//因为如果使用无参的构造函数，会首先把EMPTY_ELEMENTDATA赋值给elementData
//然后根据插入个数于当前数组size比较，不停调用Arrays.copyOf()方法，扩展数组大小
//造成性能浪费
/**
     * Constructs an empty list with the specified initial capacity.
     *
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public ArrayList(int initialCapacity) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
    }

    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        super();
        this.elementData = EMPTY_ELEMENTDATA;
    }
```
* ArrayList的add()：
```
//首先看到，不过是指定index执行add操作，还是在尾部执行add操作，都会先确认当前的数组空间是否够插入数据
//并且从
//int oldCapacity = elementData.length;
//int newCapacity = oldCapacity + (oldCapacity >> 1);
//if (newCapacity - minCapacity < 0)
//            newCapacity = minCapacity;
//看出，ArrayList默认每次都是自增50%的大小再和minCapacity比较，如果还是不够，就把当的
//size扩充至minCapacity
//然后，如果是队尾插入，也简单，就把数组向后移动一位，然后赋值
//如果是在中间插入，需要用到System.arraycopy，把index开始所有数据向后移动一位
//再进行插入
/**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    /**
     * Inserts the specified element at the specified position in this
     * list. Shifts the element currently at that position (if any) and
     * any subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }

     private void ensureCapacityInternal(int minCapacity) {
            if (elementData == EMPTY_ELEMENTDATA) {
                minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
            }

            ensureExplicitCapacity(minCapacity);
        }

        private void ensureExplicitCapacity(int minCapacity) {
            modCount++;

            // overflow-conscious code
            if (minCapacity - elementData.length > 0)
                grow(minCapacity);
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
```
* Arraylist的remove()
```
//首先来看remove(int index)
//先进行边界确认，传入的index是否超过了当前数组的大小，如果是抛出异常
//如果在数组范围内，就把index之后的数据整体向前移动一位，最后一位值清空
//如果是remove(Object o)，传入的是一个对象，就会进行一次indexOf的操作，去当前数组中寻找
//判断是否存在，这里的代码就十分冗余了，就是把indexOf的代码拷贝了一次，完全可以调用indexOf方法
//根据返回值是否为-1来判断该值是否存在，如果存在就调用fastRemove方法
//fastRemove(int index)和remove(int index)方法除了边界检查一模一样
//完全可以在remove调用完rangeCheck(index)后调用fastRemove就可以了
//这里不是很明白设计者的意图
/**
     * Removes the element at the specified position in this list.
     * Shifts any subsequent elements to the left (subtracts one from their
     * indices).
     *
     * @param index the index of the element to be removed
     * @return the element that was removed from the list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

    /**
     * Removes the first occurrence of the specified element from this list,
     * if it is present.  If the list does not contain the element, it is
     * unchanged.  More formally, removes the element with the lowest index
     * <tt>i</tt> such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>
     * (if such an element exists).  Returns <tt>true</tt> if this list
     * contained the specified element (or equivalently, if this list
     * changed as a result of the call).
     *
     * @param o element to be removed from this list, if present
     * @return <tt>true</tt> if this list contained the specified element
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    /*
     * Private remove method that skips bounds checking and does not
     * return the value removed.
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```
* ArrayList的indexof()
```
//这里就和上面remove寻找是一模一样的
public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```
例子，帮助大家理解下:
```
List list = new ArrayList();
        list.add("zero");
        list.add(null);
        list.add("two");
        list.add(null);
        list.add("three");
        Log.i("test", "index : " + list.indexOf(null));

Answer:I/test: index : 1
```

从这个例子可以看出三点List的特征<br>
1.是按顺序查找<br>
2.允许存储项为空<br>
3.允许多个存储项的值相等<br>


##### Vector
* Vector就是ArrayList的线程安全版，它的方法前都加了synchronized锁，其他实现逻辑都相同。
如果对线程安全要求不高的话，可以选择ArrayList，毕竟synchronized也很耗性能。

##### LinkedList
故名思意就是链表，和我们大学在数据结构里学的链表是一会事，LinkedList还是一个双向链表。<br>
![](https://github.com/TrueOr/Java/raw/master/Collection/picture/linkedlist1.png)<br>

LinkedList继承于AbstractSequentialList,和ArrayList一个套路。内部维护了3个成员变量，一个是当前链表的头节点，一个是尾部节点，还有是链表长度。然后我们在来看下Node这个数据结构。<br>
![](https://github.com/TrueOr/Java/raw/master/Collection/picture/linkedlist2.png)<br>
和C语言实现方式差不多，由于是双向链表，所以记录了next和prev，只不过把C语言里的指针换成了对象。<br>

* LinkedList的add(E e)
```
  //首先来看下void linkLast(E e)，尾部插入
    //就是把newNode的前面节点执行现在的尾部节点，newNode的后面节点执行null,因为是在尾部嘛
    //然后把现在的尾部节点的后面节点指向newNode,因为现在的尾部节点不是最后一个了

    //然后再来看下中间插入
    //也是一个套路。假设现在在3号位插入一个newNode
    //就是通过现在的3号Node的prev找到2号节点，然后修改2号节点的next，指向nowNode
    //然后nowNode的prev指向2号节点，next指向3号节点
    //最后3号节点的prev变成了nowNode,next不变
    //这样就完成了一次中间插入  

    /**
     * Inserts the specified element at the specified position in this list.
     * Shifts the element currently at that position (if any) and any
     * subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }

    /**
     * Appends the specified element to the end of this list.
     *
     * <p>This method is equivalent to {@link #addLast}.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        linkLast(e);
        return true;
    }

    /**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }

    /**
     * Inserts element e before non-null Node succ.
     */
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```
* linklist的void linkLast(E e)
```
/indexOf操作非常简单，就是从头开始遍历整个链表，如果没有就反-1，有就返回当前下标
/**
     * Returns the index of the first occurrence of the specified element
     * in this list, or -1 if this list does not contain the element.
     * More formally, returns the lowest index {@code i} such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
     * or -1 if there is no such index.
     *
     * @param o element to search for
     * @return the index of the first occurrence of the specified element in
     *         this list, or -1 if this list does not contain the element
     */
    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```
* LinkedList的remove()
```
 //如果直接调无参的remove(),就会默认删除头节点
    //删除头节点非常简单，就是把头节点的值清空，next清空
    //然后把nextNode只为头节点，然后清空next的prev
    //最后size减1
    //如果是删除中间节点，调用remove(int index)
    //首先判断Index对应的节点是否为头节点，即index是否为0
    //如果不是中间节点，就是x的prev指向x的next
    public E remove() {
        return removeFirst();
    }

    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }

     public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }

    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;
        if (prev == null)
            first = null;
        else
            prev.next = null;
        size--;
        modCount++;
        return element;
    }

    /**
     * Unlinks non-null node x.
     */
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }

    /**
     * Unlinks non-null first node f.
     */
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
```
### 综述
通过上面对ArrayList和LinkedList的分析，可以理解List的3个特性<br>
* 1.是按顺序查找
* 2.允许存储项为空
* 3.允许多个存储项的值相等

然后对比LinkedList和ArrayList的实现方式不同，可以在不同的场景下使用不同的List<br>
ArrayList是由数组实现的，方便查找，返回数组下标对应的值即可，适用于多查找的场景<br>
LinkedList由链表实现，插入和删除方便，适用于多次数据替换的场景<br>

