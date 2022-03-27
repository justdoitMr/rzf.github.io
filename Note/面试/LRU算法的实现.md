## LRU算法的实现

### 基于HashMap和LinkedList实现

~~~java
class LRUCache{
//    缓存的容量
    int capacity;
//   判断缓存中是否存在元素，时间复杂度是o(1)
    Map<String, Integer> map;
//    存储元素的key值，因为需要从map中删除
    LinkedList<String> list;

    /**
     * 构造函数
     * @param capacity
     */
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new HashMap<>();
        this.list = new LinkedList<>();
    }

    /**
     * 向缓存中添加元素
     * 1 首先判断缓存中是否存在当前的元素
     *  1.1 如果村子啊，那么就从链表中删除，然后在添加到哪链表的末尾
     *  1.2 如果不存在，首先判断缓存的容量是否已经满了
     *      1.2.1 如果没有满，直接添加到链表末尾，然后更新map
     *      1.2.2 如果满了，那么就删除链表头部的元素，然后添加到链表的末尾
     * @param key
     * @param val
     */
    public void put(String key,Integer val){

//        首先判断缓存中是否存在当前的元素
        Integer value = map.get(key);
        if(value != null){
            list.remove(key);
//            将元素添加到链表末尾
            list.addLast(key);
            map.put(key,value);
            return;
        }else {
            //        判断链表是否满
            if(list.size() < capacity){
//            如果链表没有满，那么就将元素添加到链表末尾
                list.addLast(key);
                map.put(key,val);
            }else {
//            链表满了，将链表首部元素溢删除
//            首先获取链表中的key
                String deleteKey = list.removeFirst();
//            从map中删除元素
                map.remove(deleteKey);
//            添加新的元素到链表
                list.addLast(key);
                map.put(key,val);
            }
        }

    }

    /**
     * 从缓存中获取元素
     * @param key
     * @return
     */
    public Integer get(String key){
//        首先从map中判断元素是否存在，时间复杂度是o(1)
        Integer isExistsValue = map.get(key);
        if(isExistsValue != null){
//            说明缓存中存在,首先进行删除
            list.remove(key);
//            然后在添加到链表的末尾
            list.addLast(key);
//            返回元素值
            return isExistsValue;
        }
//        不存在就返回Null
        return null;
    }
}
~~~

基于LinkedList实现，删除元素的时间复杂度是o(n)，但是查找一个元素的时间复杂度是O(1)

### 基于LinkedHashMap实现

~~~java
class LRUCache {
    LinkedHashMap<Integer,Integer>cache;
    int cap;

    public LRUCache(int capacity) {

        // 初始化缓存的大小
        cache=new LinkedHashMap<>();
        cap= capacity;
    }
    
    public int get(int key) {
        // 首先查看缓存中是否存在当前的元素
        if(!cache.containsKey(key)){
            return -1;
        }
        // 将当前元素设置为最近使用
        makeRecently(key);
        return cache.get(key);

    }
    
    public void put(int key, int value) {
        // 首先查看当前key是否存在
        if(cache.containsKey(key)){
            // 变更为其他的值
            cache.put(key, value);
            // 设置为最近访问
            makeRecently(key);
            return ;
        }
        // 查看当前容量是否达到上限
        if(cache.size() == this.cap){
            // 删除最久未使用的元素，删除链表头部的元素
            int k=cache.keySet().iterator().next();
            cache.remove(k);
        }
        // 插入新的元素到链表的末尾
        cache.put(key, value);

    }
    // 增加一个方法，是当前的元素成为最近使用的
    void makeRecently(int key){

        // 首先从链表中删除该元素
        int val=cache.get(key);
        // 将该元素从链表中删除
        cache.remove(key);
        cache.put(key, val);
    }
}
~~~

