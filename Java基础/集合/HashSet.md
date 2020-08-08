# 简介

HashSe是一个无重复集合，它的底层使用HashMap来保存所有元素。它是非线程安全的，且允许null值。

```java
public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable {
	 private transient HashMap<E,Object> map;
    
    //定义一个虚拟的Object对象作为HashMap的value，将此对象定义为static final。  
     private static final Object PRESENT = new Object();
    //构造方法
    public HashSet() {
        map = new HashMap<>();
    }
    
    public HashSet(int initialCapacity, float loadFactor) {  
   		map = new HashMap<E,Object>(initialCapacity, loadFactor);  
   } 
    
   public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }
}
```



## add方法

将要插入的元素e和PRESENT作为键值对存进hashmap

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

public V put(K key, V value) {
     return putVal(hash(key), key, value, false, true);
}
```

**如何保证元素不重复？**

hashMap的put方法会转至putVal(int hash, K key, V value, boolean onlyIfAbsent,  boolean evict) 方法里。

该方法在添加元素时，会判断e的hash值（比较方式是==）和e（比较方式是==和equals）是否存在于map里：如果存在，则会替换旧掉value值（也就是PRESENT）。





## remove方法

```java
 public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
 }
```



## contains方法

```java
   public boolean contains(Object o) {
        return map.containsKey(o);
    }
```



## iterator方法

```java
public Iterator<E> iterator() {
    return map.keySet().iterator();
}
```