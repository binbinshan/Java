### Java 自带了各种 Map 类，这些 Map 类可归为三种类型：
* 通用Map
> 用于在应用程序中管理映射，通常在 java.util 程序包中实现
> HashMap、Hashtable、Properties、LinkedHashMap、IdentityHashMap、TreeMap、WeakHashMap、ConcurrentHashMap
* 专用Map
> 通常我们不必亲自创建此类Map，而是通过某些其他类对其进行访问
> java.util.jar.Attributes、javax.print.attribute.standard.PrinterStateReasons、java.security.Provider、java.awt.RenderingHints、javax.swing.UIDefaults
* 自行实现Map
> 一个用于帮助我们实现自己的Map类的抽象类AbstractMap

### 常用Map
* HashMap
最常用的Map,它根据键的HashCode 值存储数据,根据键可以直接获取它的值，具有很快的访问速度。HashMap最多只允许一条记录的键为Null(多条会覆盖);允许多条记录的值为 Null。非同步的。
* TreeMap
能够把它保存的记录根据键(key)排序,默认是按升序排序，也可以指定排序的比较器，当用Iterator 遍历TreeMap时，得到的记录是排过序的。TreeMap不允许key的值为null。非同步的。
* Hashtable
与 HashMap类似,不同的是:key和value的值均不允许为null;它支持线程的同步，即任一时刻只有一个线程能写Hashtable,因此也导致了Hashtale在写入时会比较慢。
* LinkedHashMap
保存了记录的插入顺序，在用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的.在遍历的时候会比HashMap慢。key和value均允许为空，非同步的。

Map 初始化
```
Map<String, String> map = new HashMap<String, String>();
```
插入元素
```
map.put("key1", "value1");
```
获取元素
```
map.get("key1")
```
移除元素
```
map.remove("key1");
```
清空map
```
map.clear();
```
### 四种常用Map插入与读取性能比较
插入10次平均(ms)<br>
			 |1W | 10W | 100W
-------------|---|-----|------
HashMap      |56 | 261 | 3030
LinkedHashMap|25 | 229 | 3069
TreeMap 	 |29 | 295 | 4117
Hashtable 	 |24 | 234 | 3275

读取10次平均(ms)<br>
			 |1W | 10W | 100W
-------------|---|-----|------
HashMap      |2  | 21  | 220
LinkedHashMap|2  | 20  | 216
TreeMap 	 |5  | 102 | 1446
Hashtable 	 |2  | 22  | 259

测试代码<br>
```

public class Test {
    static int hashMapW = 0;
    static int hashMapR = 0;
    static int linkMapW = 0;
    static int linkMapR = 0;
    static int treeMapW = 0;
    static int treeMapR = 0;
    static int hashTableW = 0;
    static int hashTableR = 0;

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            Test test = new Test();
            test.test(100 * 10000);
            System.out.println();
        }
        System.out.println("hashMapW = " + hashMapW / 10);
        System.out.println("hashMapR = " + hashMapR / 10);
        System.out.println("linkMapW = " + linkMapW / 10);
        System.out.println("linkMapR = " + linkMapR / 10);
        System.out.println("treeMapW = " + treeMapW / 10);
        System.out.println("treeMapR = " + treeMapR / 10);
        System.out.println("hashTableW = " + hashTableW / 10);
        System.out.println("hashTableR = " + hashTableR / 10);
    }

    public void test(int size) {
        int index;
        Random random = new Random();
        String[] key = new String[size];
        // HashMap 插入
        Map<String, String> map = new HashMap<String, String>();
        long start = System.currentTimeMillis();
        for (int i = 0; i < size; i++) {
            key[i] = UUID.randomUUID().toString();
            map.put(key[i], UUID.randomUUID().toString());
        }
        long end = System.currentTimeMillis();
        hashMapW += (end - start);
        System.out.println("HashMap插入耗时 = " + (end - start) + " ms");
        // HashMap 读取
        start = System.currentTimeMillis();
        for (int i = 0; i < size; i++) {
            index = random.nextInt(size);
            map.get(key[index]);
        }
        end = System.currentTimeMillis();
        hashMapR += (end - start);
        System.out.println("HashMap读取耗时 = " + (end - start) + " ms");
        // LinkedHashMap 插入
        map = new LinkedHashMap<String, String>();
        start = System.currentTimeMillis();
        for (int i = 0; i < size; i++) {
            key[i] = UUID.randomUUID().toString();
            map.put(key[i], UUID.randomUUID().toString());
        }
        end = System.currentTimeMillis();
        linkMapW += (end - start);
        System.out.println("LinkedHashMap插入耗时 = " + (end - start) + " ms");
        // LinkedHashMap 读取
        start = System.currentTimeMillis();
        for (int i = 0; i < size; i++) {
            index = random.nextInt(size);
            map.get(key[index]);
        }
        end = System.currentTimeMillis();
        linkMapR += (end - start);
        System.out.println("LinkedHashMap读取耗时 = " + (end - start) + " ms");
        // TreeMap 插入
        key = new String[size];
        map = new TreeMap<String, String>();
        start = System.currentTimeMillis();
        for (int i = 0; i < size; i++) {
            key[i] = UUID.randomUUID().toString();
            map.put(key[i], UUID.randomUUID().toString());
        }
        end = System.currentTimeMillis();
        treeMapW += (end - start);
        System.out.println("TreeMap插入耗时 = " + (end - start) + " ms");
        // TreeMap 读取
        start = System.currentTimeMillis();
        for (int i = 0; i < size; i++) {
            index = random.nextInt(size);
            map.get(key[index]);
        }
        end = System.currentTimeMillis();
        treeMapR += (end - start);
        System.out.println("TreeMap读取耗时 = " + (end - start) + " ms");
        // Hashtable 插入
        key = new String[size];
        map = new Hashtable<String, String>();
        start = System.currentTimeMillis();
        for (int i = 0; i < size; i++) {
            key[i] = UUID.randomUUID().toString();
            map.put(key[i], UUID.randomUUID().toString());
        }
        end = System.currentTimeMillis();
        hashTableW += (end - start);
        System.out.println("Hashtable插入耗时 = " + (end - start) + " ms");
        // Hashtable 读取
        start = System.currentTimeMillis();
        for (int i = 0; i < size; i++) {
            index = random.nextInt(size);
            map.get(key[index]);
        }
        end = System.currentTimeMillis();
        hashTableR += (end - start);
        System.out.println("Hashtable读取耗时 = " + (end - start) + " ms");
    }
}
```
### Map 遍历
初始化数据<br>
```
Map<String, String> map = new HashMap<String, String>();
map.put("key1", "value1");
map.put("key2", "value2");
```
* 增强for循环遍历
使用keySet()遍历
```
for (String key : map.keySet()) {
    System.out.println(key + " ：" + map.get(key));
}
```
使用entrySet()遍历
```
for (Map.Entry<String, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " ：" + entry.getValue());
}
```
*迭代器遍历
使用keySet()遍历
```
Iterator<String> iterator = map.keySet().iterator();
while (iterator.hasNext()) {
    String key = iterator.next();
    System.out.println(key + "　：" + map.get(key));
}
```
使用entrySet()遍历
```
Iterator<Map.Entry<String, String>> iterator = map.entrySet().iterator();
while (iterator.hasNext()) {
    Map.Entry<String, String> entry = iterator.next();
    System.out.println(entry.getKey() + "　：" + entry.getValue());
}
```
### HashMap四种遍历方式性能比较
比较方式,分别对四种遍历方式进行10W次迭代，比较用时。<br>
```
public class TestMap {
    public static void main(String[] args) {
        // 初始化，10W次赋值
        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
        for (int i = 0; i < 100000; i++)
            map.put(i, i);

        /** 增强for循环，keySet迭代 **/
        long start = System.currentTimeMillis();
        for (Integer key : map.keySet()) {
            map.get(key);
        }
        long end = System.currentTimeMillis();
        System.out.println("增强for循环，keySet迭代 -> " + (end - start) + " ms");

        /** 增强for循环，entrySet迭代 */
        start = System.currentTimeMillis();
        for (Entry<Integer, Integer> entry : map.entrySet()) {
            entry.getKey();
            entry.getValue();
        }
        end = System.currentTimeMillis();
        System.out.println("增强for循环，entrySet迭代 -> " + (end - start) + " ms");

        /** 迭代器，keySet迭代 **/
        start = System.currentTimeMillis();
        Iterator<Integer> iterator = map.keySet().iterator();
        Integer key;
        while (iterator.hasNext()) {
            key = iterator.next();
            map.get(key);
        }
        end = System.currentTimeMillis();
        System.out.println("迭代器，keySet迭代 -> " + (end - start) + " ms");

        /** 迭代器，entrySet迭代 **/
        start = System.currentTimeMillis();
        Iterator<Map.Entry<Integer, Integer>> iterator1 = map.entrySet().iterator();
        Map.Entry<Integer, Integer> entry;
        while (iterator1.hasNext()) {
            entry = iterator1.next();
            entry.getKey();
            entry.getValue();
        }
        end = System.currentTimeMillis();
        System.out.println("迭代器，entrySet迭代 -> " + (end - start) + " ms");
    }
}
```
运行三次，比较结果

第一次
> 增强for循环，keySet迭代 -> 37 ms
> 增强for循环，entrySet迭代 -> 19 ms
> 迭代器，keySet迭代 -> 14 ms
> 迭代器，entrySet迭代 -> 9 ms

第二次
> 增强for循环，keySet迭代 -> 29 ms
> 增强for循环，entrySet迭代 -> 22 ms
> 迭代器，keySet迭代 -> 19 ms
> 迭代器，entrySet迭代 -> 12 ms

第三次
> 增强for循环，keySet迭代 -> 27 ms
> 增强for循环，entrySet迭代 -> 19 ms
> 迭代器，keySet迭代 -> 18 ms
> 迭代器，entrySet迭代 -> 10 ms

平均值
> 增强for循环，keySet迭代 -> 31 ms
> 增强for循环，entrySet迭代 -> 20 ms
> 迭代器，keySet迭代 -> 17 ms
> 迭代器，entrySet迭代 -> 10.33 ms

总结
	增强for循环使用方便，但性能较差，不适合处理超大量级的数据。<br>
	迭代器的遍历速度要比增强for循环快很多，是增强for循环的2倍左右。<br>
    使用entrySet遍历的速度要比keySet快很多，是keySet的1.5倍左右。<br>
	
### Map 排序
* HashMap、Hashtable、LinkedHashMap排序
注：TreeMap也可以使用此方法进行排序，但是更推荐下面的方法。
```
Map<String, String> map = new HashMap<String, String>();
map.put("b", "b");
map.put("a", "c");
map.put("c", "a");

// 通过ArrayList构造函数把map.entrySet()转换成list
List<Map.Entry<String, String>> list = new ArrayList<Map.Entry<String, String>>(map.entrySet());
// 通过比较器实现比较排序
Collections.sort(list, new Comparator<Map.Entry<String, String>>() {
    @Override
    public int compare(Map.Entry<String, String> mapping1, Map.Entry<String, String> mapping2) {
        return mapping1.getKey().compareTo(mapping2.getKey());
    }
});

for (Map.Entry<String, String> mapping : list) {
    System.out.println(mapping.getKey() + " ：" + mapping.getValue());
}
```
* TreeMap排序
TreeMap默认按key进行升序排序，如果想改变默认的顺序，可以使用比较器:
```
Map<String, String> map = new TreeMap<String, String>(new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
        // 降序排序
        return o1.compareTo(o2);
    }
});

map.put("b", "b");
map.put("a", "c");
map.put("c", "a");
for (String key : map.keySet()) {
    System.out.println(key + " ：" + map.get(key));
}
```
* 按value排序(通用)
```
Map<String, String> map = new TreeMap<String, String>();
map.put("b", "b");
map.put("a", "c");
map.put("c", "a");

// 通过ArrayList构造函数把map.entrySet()转换成list
List<Map.Entry<String, String>> list = new ArrayList<Map.Entry<String, String>>(map.entrySet());
// 通过比较器实现比较排序
Collections.sort(list, new Comparator<Map.Entry<String, String>>() {
    @Override
    public int compare(Map.Entry<String, String> mapping1, Map.Entry<String, String> mapping2) {
        return mapping1.getValue().compareTo(mapping2.getValue());
    }
});

for (String key : map.keySet()) {
    System.out.println(key + " ：" + map.get(key));
}
```
### 常用API 
			方法 		        | 					描述
------------------------------|------------------------------------------------------------------------------------------------
clear() 				      |     从 Map 中删除所有映射
remove(Object key) 		      |     从 Map 中删除键和关联的值
put(Object key, Object value) | 	将指定值与指定键相关联
putAll(Map t)                 | 	将指定 Map 中的所有映射复制到此 map
entrySet() 	                  |     返回 Map 中所包含映射的 Set 视图。Set 中的每个元素都是一个 Map.Entry 对象，可以使用 getKey() 和 getValue() 方法（还有一个 setValue() 方法）访问后者的键元素和值元素
keySet() 	                  |     返回 Map 中所包含键的 Set 视图。删除 Set 中的元素还将删除 Map 中相应的映射（键和值）
values ()                     | 	返回 map 中所包含值的 Collection 视图。删除 Collection 中的元素还将删除 Map 中相应的映射（键和值）
get(Object key)               | 	返回与指定键关联的值
containsKey(Object key)       | 	如果 Map 包含指定键的映射，则返回 true
containsValue(Object value)   |  	如果此 Map 将一个或多个键映射到指定值，则返回 true
isEmpty()                     | 	如果 Map 不包含键-值映射，则返回 true
size()                        | 	返回 Map 中的键-值映射的数目

