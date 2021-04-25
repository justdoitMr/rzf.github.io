# 算法笔记

[TOC]

# 数据结构

## 数组

**基本概念**

- 定义：在**连续的内存空间**中，储存一组**相同类型**的元素
- 数组的访问: 通过索引访问元素。a[0]
- 数组的内存空间是连续的，增删需要移动后面的元素
- 二维数组的内存地址是连续的吗？
  - 二维数组实际是一个线性数组存放着其他数组的首地址

**操作复杂度分析**

- 访问:o(1)
- 搜索:o(n)
- 插入:o(n)
- 删除:o(n)

> 适合读多写少的情景

### 485

[最大连续1的个数](https://leetcode-cn.com/problems/max-consecutive-ones/)

**题目描述**

给定一个二进制数组， 计算其中最大连续 1 的个数。

**思路**

双指针法

**代码实现**

~~~ java
class Solution {
    public int findMaxConsecutiveOnes(int[] nums) {
         if(nums == null || nums.length==0)
            return 0;
        int count=0;
        int res=0;
        for(int i=0;i<nums.length;i++){
            if(nums[i] != 0)
            {
                count++;
            }else{
                res=Math.max(res,count);
                count=0;
            }
        }
        return Math.max(res,count);
    }
}
~~~

### 283

[移动零](https://leetcode-cn.com/problems/move-zeroes/)

**题目描述**

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

**代码实现**

~~~ java
class Solution {
    public void moveZeroes(int[] nums) {

        int i=0;
        for(int n=0;n<nums.length;n++){
            if(nums[n] != 0){
                nums[i]=nums[n];
                  i+=1;
            }
        }

        for(int m=i;m<nums.length;m++){
            nums[m]=0;
        }

    }
}
~~~

### 27

[移除元素](https://leetcode-cn.com/problems/remove-element/)

**题目描述**

给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度。

不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

**代码实现**

~~~ java
class Solution {
    public int removeElement(int[] nums, int val) {

    if(nums == null || nums.length ==0)
    return 0;
    int left=0,right=nums.length-1;
    int temp=0;

//边界判断
    while(left < right){
        // 从数组左边找等于val的元素
        while(left < right && nums[left]!= val)
            left++;
        // 从数组右边找等于val的元素
        while(left < right && nums[right]== val)
            right--;
        // 如果内层循环退出，说明找到，就交换两个元素的位置
        temp=nums[left];
        nums[left]=nums[right];
        nums[right]=temp;
    }
    if(nums[left] == val)
    return left;
    else
    return left+1;

    }
}
~~~

### 三数之和

[三数之和]()

使用排序法+双指针法

~~~ java
class Solution {
    public static List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>>res=new ArrayList();

// 使用排序方法+双指针法
// 首先判断数据是否是空数组
        if(nums == null || nums.length == 0 || nums.length < 3){
            return res;
        }
        // 对数组进行排序
        Arrays.sort(nums);
        // 遍历数组
        for(int k=0;k<nums.length-2;k++){
            // 首先判断当前元素值是否大于零
            if(nums[k] > 0 ){
                // 退出进行下一次循环
                continue;
            }
            if((k>0)&&(nums[k] == nums[k-1])){
                continue;
            }
            int left=k+1;
            int right=nums.length-1;
            while(left < right){
                int sum=nums[k] + nums[left] + nums[right];
                if( sum== 0){
                    // 添加元素到集合中
                    res.add(new ArrayList(Arrays.asList(nums[k],nums[left],nums[right])));
                    // 找下一个合适的点
                    while(left< right && nums[left] == nums[left+1])
                    left++;
                    while(left < right && nums[right] == nums[right-1])
                        right--;
                }
                if(sum < 0){
                    left++;
                }
                if(sum > 0){
                    right--;
                }
            }
        }
        return res;
    }
}
~~~

### 17

[电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

回溯法

~~~ java
class Solution {
	//一个映射表，第二个位置是"abc“,第三个位置是"def"。。。
	//这里也可以用map，用数组可以更节省点内存
	String[] letter_map = {" ","*","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
	public List<String> letterCombinations(String digits) {
		//注意边界条件
		if(digits==null || digits.length()==0) {
			return new ArrayList<>();
		}
		iterStr(digits, new StringBuilder(), 0);
		return res;
	}
	//最终输出结果的list
	List<String> res = new ArrayList<>();
	
	//递归函数
	void iterStr(String str, StringBuilder letter, int index) {
		//递归的终止条件，注意这里的终止条件看上去跟动态演示图有些不同，主要是做了点优化
		//动态图中是每次截取字符串的一部分，"234"，变成"23"，再变成"3"，最后变成""，这样性能不佳
		//而用index记录每次遍历到字符串的位置，这样性能更好
		if(index == str.length()) {
			res.add(letter.toString());
			return;
		}
		//获取index位置的字符，假设输入的字符是"234"
		//第一次递归时index为0所以c=2，第二次index为1所以c=3，第三次c=4
		//subString每次都会生成新的字符串，而index则是取当前的一个字符，所以效率更高一点
		char c = str.charAt(index);
		//map_string的下表是从0开始一直到9， c-'0'就可以取到相对的数组下标位置
		//比如c=2时候，2-'0'，获取下标为2,letter_map[2]就是"abc"
		int pos = c - '0';
		String map_string = letter_map[pos];
		//遍历字符串，比如第一次得到的是2，页就是遍历"abc"
		for(int i=0;i<map_string.length();i++) {
			//调用下一层递归，用文字很难描述，请配合动态图理解
            letter.append(map_string.charAt(i));
            //如果是String类型做拼接效率会比较低
			//iterStr(str, letter+map_string.charAt(i), index+1);
            iterStr(str, letter, index+1);
            letter.deleteCharAt(letter.length()-1);
		}
	}
}

~~~



## 链表

- 访问：o(n)
- 搜索：o(n)
- 插入：o(1)
- 删除：o(1)

> 写很快，读很慢，适合写多读少情况

### 203

[移除链表中的元素](https://leetcode-cn.com/problems/remove-linked-list-elements/)

**题目描述**

删除链表中等于给定值 **val** 的所有节点。

**代码实现**

~~~ java
class Solution {
    public ListNode removeElements(ListNode head, int val) {
        
        if(head == null)
        return null;
        //添加一个头结点，方便判断第一个元素
        ListNode dummy=new ListNode(0);
        dummy.next=head;

        ListNode pre=dummy;
      
        while(head != null ){
           if(head.val == val){
               pre.next=head.next;
           }else{
               pre=head;
           }
           head=head.next;
        }
        return dummy.next;
    }
}
~~~

### 206

[反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

**题目描述**

反转一个单链表。

**代码实现**

~~~ java 
class Solution {
    public ListNode reverseList(ListNode head) {

        if(head == null)
        return null;
        // 采用头插法建立单链表
       ListNode dummy=new ListNode(0);
       dummy.next=head;
       ListNode node=head.next;
       head.next=null;
       ListNode node1=head;
       ListNode temp=head;
       while(node != null){
           temp=dummy.next;
           dummy.next=node;
           node1=node.next;
           node.next=temp;
            node=node1;
       }
       return dummy.next;
    }
}
~~~

## 队列

- 访问：o(n)
- 搜索：o(n)
- 插入：o(1)
- 删除：o(1)

### 933

[最近的请求次数](https://leetcode-cn.com/problems/number-of-recent-calls/)

**题目描述**

写一个 `RecentCounter` 类来计算特定时间范围内最近的请求。

**代码实现**

~~~ java
class RecentCounter {

    // 使用队列实现功能,新的ping操作全部添加到队列的末尾，队列的头部是最先添加进来的ping操作
    Queue queue;

    public RecentCounter() {
       queue =new LinkedList();
    }
    //每一次都返回有多少次请求
    public int ping(int t) {
        //先把ping操作添加到队列的尾部
        queue.add(t);
        //队列不空就循环
        while(!queue.isEmpty()){
            //判断3000s内ping的次数
            if((t-(int)queue.peek())<=3000){
                return queue.size();
            }else{
                //如果超过300秒，那么就删除对头元素
                queue.remove();
            }
        }
        return 0;

    }
}
~~~

## 栈

- 访问：o(1)
- 搜索：o(n)
- 插入：o(1)
- 删除：o(1)

### 20 

[括号匹配](https://leetcode-cn.com/problems/valid-parentheses/)

**题目描述**

给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串 `s` ，判断字符串是否有效。

**代码实现**

~~~ java
class Solution {
    public boolean isValid(String s) {

    if(s == null)
        return true;
    Stack stack=new Stack();
    char chars[]=s.toCharArray();
    for(int i=0;i<chars.length;i++){
        if((chars[i] == '(')||(chars[i] == '{')|| (chars[i] == '['))
        {
            // 如果是左边的括号，直接入栈操作
            stack.push(chars[i]);
        }else{
            // 否则是右边的括号,或者其他符号
            if(!stack.isEmpty()){
                char c=(char)stack.pop();
                if((c == '(')&&(chars[i] == ')')){
                    continue;
                }else if((c == '{')&&(chars[i] == '}')){
                   continue;
                }else if((c == '[')&&(chars[i] == ']')){
                    continue;
                }else{
                    return false;
                }

            }else{
              // stack.push(chars[i]);
                return false;
            }
        }
    }
    return stack.isEmpty()?true:false;

    }
}
~~~

### 496

[下一个更大元素](https://leetcode-cn.com/problems/next-greater-element-i/)

**题目描述**

给你两个 没有重复元素 的数组 nums1 和 nums2 ，其中nums1 是 nums2 的子集。

请你找出 nums1 中每个元素在 nums2 中的下一个比其大的值。

nums1 中数字 x 的下一个更大元素是指 x 在 nums2 中对应位置的右边的第一个比 x 大的元素。如果不存在，对应位置输出 -1 。

**代码实现**

~~~ java
class Solution {
    public int[] nextGreaterElement(int[] nums1, int[] nums2) {

        //使用栈数据结构
        int []arr=new int[nums1.length];
        int m=0;
        Stack s1=new Stack();//保存数组1中的元素
        Stack s2=new Stack();//保存数组1中的元素
         for(int i=0;i<nums2.length;i++){
             s1.push(nums2[i]);
         }
       boolean isFound=false;
       int max=-1;
        // 第一个栈不空
        for(int j=0;j<nums1.length;j++){

                //循环重栈中找元素
            while(!s1.isEmpty()&& !isFound){
                // 抛出站定元素
                int top=(int)s1.pop();
                if(top>nums1[j]){
                    max=top;
                }else if(top == nums1[j]){
                    arr[m++]=max;
                    isFound=true;
                }
                s2.push(top);
            }
           while(!s2.isEmpty())
           s1.push(s2.pop());
             isFound=false;
             max=-1;
        }

    
         return arr;
    }
}
~~~

## 哈希表

- 搜索：o(1),如果发生碰撞，时间复杂度是o(k),k表示碰撞元素的个数
- 删除：o(1)
- 插入：o(1)

### 217

[存在重复元素](https://leetcode-cn.com/problems/contains-duplicate/)

**题目描述**

给定一个整数数组，判断是否存在重复元素。

如果存在一值在数组中出现至少两次，函数返回 `true` 。如果数组中每个元素都不相同，则返回 `false` 。

**代码实现**

~~~ java
class Solution {
    public boolean containsDuplicate(int[] nums) {
        // 使用哈希表，key存储当前的数组元素，value存储当前元素出现的次数

       Set<Integer> hashSet=new HashSet();

       for(Integer i:nums){
           if(!hashSet.add(i)){
               return true;
           }
       }
        return false;
    }
}
~~~

**使用HashMap实现**

~~~ java
class Solution {
    public boolean containsDuplicate(int[] nums) {
    HashMap<Integer,Integer> hashMap=new HashMap();

    for(int i=0;i< nums.length;i++){
        if(!hashMap.containsKey(nums[i])){
            hashMap.put(nums[i],1);
        }else{
            hashMap.put(nums[i],hashMap.get(nums[i])+1);
        }
    }
    for(int k:hashMap.keySet()){
        if(hashMap.get(k)>1){
            return true;
        }
    }
    return false;
    }
}
~~~

### 389

[找不同](https://leetcode-cn.com/problems/find-the-difference/)

**题目描述**

给定两个字符串 ***s*** 和 ***t***，它们只包含小写字母。

字符串 **t** 由字符串 **s** 随机重排，然后在随机位置添加一个字母。

请找出在 ***t*** 中被添加的字母。

**代码实现**

~~~ java
class Solution {
    public char findTheDifference(String s, String t) {
        // 遇到字符串，首先判断其长度是否为0，是否是Null

        if(s == null|| s.length() == 0){
            return t.charAt(0);
        }
// 使用数组模拟哈希表
        int hashTable[]=new int[26];
        int sSize=s.length();
        int tSize=t.length();

        for(int i=0;i<tSize;i++){
            if(i<sSize){
                hashTable[s.charAt(i)-'a']++;
            }
            hashTable[t.charAt(i)-'a']--;
        }
        for(int k=0;k<hashTable.length;k++){
            if(hashTable[k] != 0){
               return (char)('a'+k);
            }
        }
        return 'a';
    }
}
~~~

### 496

[下一个更大元素i](https://leetcode-cn.com/problems/next-greater-element-i/submissions/)

**题目描述**

给你两个 没有重复元素 的数组 nums1 和 nums2 ，其中nums1 是 nums2 的子集。

请你找出 nums1 中每个元素在 nums2 中的下一个比其大的值。

nums1 中数字 x 的下一个更大元素是指 x 在 nums2 中对应位置的右边的第一个比 x 大的元素。如果不存在，对应位置输出 -1 。

**思路说明**

遍历num2数组，对其中每一个元素，先入栈，判断当前元素是否大于栈顶的元素，如果大于当前栈顶的元素，那么就从栈顶抛出，然后入哈希表，表示当前出栈的元素的下一个最大元素是当前的元素，遍历完数组后，如果栈不空，那么抛出所有元素到哈希表中，并且设置当前元素下一个最大值是-1，最后以num1中元素为键值，从哈希表中获取当前元素下一个最大元素，注意，哈希表中的键存储当前元素，value存储比当前元素大的下一个元素值。

**代码实现**

~~~ java
class Solution {
    public int[] nextGreaterElement(int[] nums1, int[] nums2) {

    // 第二种解法

    Map<Integer,Integer>hashMap=new HashMap();
    Stack<Integer> stack=new Stack();

    int temp=0;

    for(int i=0;i<nums2.length;i++){
        // 如果栈不空，并且大于栈顶的元素
        while(!stack.isEmpty() && nums2[i] > stack.peek()){
            // 抛出栈顶的元素
            temp=stack.pop();
            hashMap.put(temp,nums2[i]);
        }
        stack.push(nums2[i]);
    }
    // 判断栈是否是空的，如果不空，全部抛出栈，并且添加到哈希表中
    while(!stack.isEmpty()){
        hashMap.put(stack.pop(),-1);
    }

    int []arr=new int[nums1.length];
    for(int j=0;j<nums1.length;j++){
        arr[j]=hashMap.get(nums1[j]);
    }
    return arr;
    }
}
~~~

## 集合

特点

- 无序不重复，所以可以用来检查某一个元素是否存在。

搜索：

- 无哈希冲突：o(1)
- 有哈希冲突：o(k),k代表冲突元素的个数

插入

- 无哈希冲突：o(1)
- 有哈希冲突：o(k),k代表冲突元素的个数

删除

- 无哈希冲突：o(1)
- 有哈希冲突：o(k)，k是冲突元素的个数

### 217

[存在重复元素](https://leetcode-cn.com/problems/contains-duplicate/)

**题目描述**

给定一个整数数组，判断是否存在重复元素。

如果存在一值在数组中出现至少两次，函数返回 `true` 。如果数组中每个元素都不相同，则返回 `false` 。

**代码实现**

~~~ java
class Solution {
    public boolean containsDuplicate(int[] nums) {
        // 使用哈希表，key存储当前的数组元素，value存储当前元素出现的次数

       Set<Integer> hashSet=new HashSet();

       for(Integer i:nums){
           if(!hashSet.add(i)){
               return true;
           }
       }
        return false;
    }
}
~~~

### 705

[设计一个哈希表](https://leetcode-cn.com/problems/design-hashset/)

**题目描述**

不使用任何内建的哈希表库设计一个哈希集合（HashSet）。

实现 MyHashSet 类：

    void add(key) 向哈希集合中插入值 key 。
    bool contains(key) 返回哈希集合中是否存在这个值 key 。
    void remove(key) 将给定值 key 从哈希集合中删除。如果哈希集合中没有这个值，什么也不做。

**代码实现**

~~~ java
class MyHashSet {
    private boolean []hashSet;

    /** Initialize your data structure here. */
    public MyHashSet() {
        hashSet=new boolean[10000000];
    }
    
    public void add(int key) {
        hashSet[key]=true;
    }
    
    public void remove(int key) {
        hashSet[key]=false;
    }
    
    /** Returns true if this set contains the specified element */
    public boolean contains(int key) {
        return hashSet[key];
    }
}
//使用空间换取时间
~~~

## 堆

- 搜索：o(1),一般只查看堆顶元素，查看堆内元素是o(n)
- 添加：o(logn)
- 删除：o(logn)

### 215

[数组中第k个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

**题目描述**

在未排序的数组中找到第 **k** 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。

**代码实现**

~~~ java
//使用大顶堆实现
class Solution {
    public int findKthLargest(int[] nums, int k) {

        if(nums==null || nums.length==0|| k==0)
        return Integer.MAX_VALUE;

        //创建一个最大堆
        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());

         for(int i=0;i<nums.length;i++){
            maxHeap.add(nums[i]);
         }

         while(k>1){
             maxHeap.poll();
             k--;
         }

        return maxHeap.peek();
    }
}
~~~

## 二叉树

### 101，对称二叉树

[对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/)

**递归实现**

思路：给定一棵对称的二叉树，镜对称，也就是说左子树和右子树结构一样，并且对应的节点里面的值也是一样的。所以我们要递归比较左子树和右子树。

- 记左子树的为left,右子树为right,如果left和right不相等的话，直接返回就可以了。
- 如果相等，那么接着比较left的左孩子和right的右孩子，left的右孩子和right的左孩子是否是一样的，
- 然后一直向下递归比较即可

从上面的过程，可以看到终止的条件

- 如果left 和right都是null的话，那么就返回true
- 如果left和right中有一个为空的话，那么不对称，返回false
- 如果left和right节点中的值不一样的话，那么说明不是对称的，返回false.

~~~ java
class Solution {
    public boolean isSymmetric(TreeNode root) {

        // 使用递归实现
        if(root == null){
            return false;
        }

        return isSimiliar(root.left,root.right);

    }

    public boolean isSimiliar(TreeNode left,TreeNode right){
        // 递归的终止条件
        // 两个节点都是空，或者两个节点中有一个为空，或者两个节点中的值不一样
        if(left == null && right == null){
            return true;
        }else if(left == null || right == null){
            return false;
        }else if(left.val != right.val){
            return false;
        }
        // 递归遍历两个子节点
        return isSimiliar(left.left,right.right)&& isSimiliar(left.right,right.left);
        
    }
}
~~~

**使用bfs方法**

~~~ java
class Solution {
    public boolean isSymmetric(TreeNode root) {

        // 使用递归实现
        if(root == null){
            return false;
        }
        // 使用广度优先遍历
        return bfs(root);
    }

    public  boolean bfs(TreeNode root){

        if(root == null){
            return false;
        }
        LinkedList<TreeNode>queue=new LinkedList();
        // 添加根节点到队列中
        queue.add(root);
        while(!queue.isEmpty()){
            // 记录每一层元素的个数
            int len=queue.size();
            // 抛出队列元素
            ArrayList<Integer>res=new ArrayList();
            while(len>0){
                TreeNode temp=queue.poll();
                len--;
            // 访问元素
               
                // 添加当前节点的左右孩子
                if(temp != null){
                     res.add(temp.val);
                    if(temp.left != null){
                        queue.add(temp.left);
                    }else{
                        queue.add(null);
                    }
                    if(temp.right != null){
                        queue.add(temp.right);
                    }else{
                        queue.add(null);
                    }
                }else{
                    // 如果当前元素为null值,向结果集中添加两个最值
                    res.add(-1);
                    res.add(-1);

                }
                

                //len--;
            }
             // 判断当前一层的元素是否是对称的
                for(int i=0,j=res.size()-1;i<=j;i++,j--){
                    if(res.get(i) != res.get(j)){
                        return false;
                    }
                }
            
        }
        return true;

    }
~~~

### 98，验证二叉搜索树

判断一棵二叉树是否是有效的二叉搜索树

**递归法**

思路：中序遍历二叉搜索树是一颗有序的树。

~~~ java
class Solution {
    public boolean isValidBST(TreeNode root) {

        // 儿茶排序树的中序遍历是有序的
        ArrayList<Integer>res=new ArrayList();
        if(root == null){
            return false;
        }
        order(root,res);
        // 判断结果集中的元素是否是有序的
        for(int i=1;i<res.size();i++){
            if(res.get(i)<=res.get(i-1)){
                return false;
            }
        }
        return true;
    }

    public void order(TreeNode root , ArrayList<Integer>res){
        // 使用中序遍历
        if(root == null){
            return ;
        }
        
        // 递归遍历左子树
        if(root.left != null){
            order(root.left,res);
        }
         // 添加当前节点的值到结果集中
        res.add(root.val);
        
        // 递归遍历右子树
        if(root.right != null)
        order(root.right,res);
    }
}
~~~



# 算法部分

## 双指针

- 普通双指针
  - 两个指针向同一个方向移动
- 对撞双指针
  - 两个指针面对面移动
- 快慢双指针
  - 慢指针+快指针

### 141

[环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

**题目描述**

给定一个链表，判断链表中是否有环。

**代码实现**

~~~ java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public boolean hasCycle(ListNode head) {

        if(head == null){
            return false;
        }

        ListNode fast=head;
        ListNode slow=head;

        while(fast != null && fast.next != null){
            slow = slow.next;
            fast = fast.next.next;
            if(slow == fast)
            return true;
        }
        return false;
        
    }
}
~~~

### 881

[救生艇](https://leetcode-cn.com/problems/boats-to-save-people/)

**题目描述**

第 i 个人的体重为 people[i]，每艘船可以承载的最大重量为 limit。

每艘船最多可同时载两人，但条件是这些人的重量之和最多为 limit。

返回载到每一个人所需的最小船数。(保证每个人都能被船载)。

**思路**

先对数组进行原地排序，然后使用对碰指针解决。

**代码实现**

~~~ java
class Solution {
    public int numRescueBoats(int[] people, int limit) {

        if(people == null || people.length == 0){
            return 0;
        }

        int first=0;
        int end=people.length-1;
        int num=0;
        // 首先对数组进行原地排序
        Arrays.sort(people);

        while(first <= end){
            if(people[first] + people[end]<=limit){
                num++;
                first++;
                end--;
            }
            else{
                num++;
                end--;
            }
        }

        return num;
    }
}
~~~

## 二分查找

### 704

[二分查找](https://leetcode-cn.com/problems/binary-search/)

**题目描述**

实现二分查找算法

**代码实现**

~~~ java
class Solution {
    public int search(int[] nums, int target) {
        // 二分查找算法

        int first=0;
        int last=nums.length-1;
        int mid=0;
        
        while(first <= last){
            mid=(last+first)/2;
            if(target < nums[mid]){
                last=mid-1;
            }else if(target > nums[mid]){
                first=mid+1;
            }else if(target == nums[mid]){
                return mid;
            }
        }
        return -1;
    }
}
~~~

### 35

[搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/)

**题目描述**

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

你可以假设数组中无重复元素。

**代码实现**

~~~ java
class Solution {
    public int searchInsert(int[] nums, int target) {

        // 二分插入

        int first=0;
        int last=nums.length-1;
        int mid=0;
        
        while(first < last){
            mid=(last+first)/2;
            if(target < nums[mid]){
                last=mid-1;
            }else if(target > nums[mid]){
                first=mid+1;
            }else if(target == nums[mid]){
                return mid;
            }
        }
        // 退出循环，说明没有找到该元素，使用插入法插入该元素
        // 此时first==last
        if(target <=nums[first])
        return first;
        else
        return first+1;
    }
}
~~~

### 162

[寻找峰值](https://leetcode-cn.com/problems/find-peak-element/)

**题目描述**

峰值元素是指其值大于左右相邻值的元素。

给你一个输入数组 nums，找到峰值元素并返回其索引。数组可能包含多个峰值，在这种情况下，返回 任何一个峰值 所在位置即可。

你可以假设 nums[-1] = nums[n] = -∞ 。

**思路解析**

由于数组的两边都达到负无穷，所以在数组的中间必定存在最值，一般情况下使用二分查找的话，需要数组是有序的，但是这一题目我们可以借助于二分法查找到最大的峰值，由于我们找的是相邻两个数之间的峰值，所以需要拿num[mid]和num[mid+1]进行比较，使用这种方法时间复杂度是o(logn)。

**代码实现**

~~~ java
class Solution {
    public int findPeakElement(int[] nums) {

        if(nums == null || nums.length == 0){
            return -1;
        }

        int l=0;
        int r=nums.length-1;
        int m=0;

        while(l < r){
            m=(r+l)/2;
            if(nums[m]> nums[m+1]){
                r=m;
            }else
            {
                l=m+1;
            }
        }

        return l;

    }
}
~~~

**方法二**

~~~ java
class Solution {
    public int findPeakElement(int[] nums) {

    // 方法二，直接遍历数组
    if(nums == null || nums.length == 0){
            return -1;
        }
        // 如果数组中只有一个元素或者只有两个元素，并且第一个元素大于第二个元素
        if(nums.length ==1 || nums[0]>nums[1]){
            return 0;
        }
    //    接下来处理数组中间的元素
        for(int i=1;i<nums.length-1;i++){
            if((nums[i] > nums[i-1])&&(nums[i]> nums[i+1]))
            return i;
        }
        // 判断最后两个元素
        if(nums[nums.length -1]> nums[nums.length-2]){
            return nums.length-1;
        }
        return -1;

    }
}
//遍历数组，时间复杂度是o(n),做题目要考虑边界
~~~

### 74

[搜索二维矩阵](https://leetcode-cn.com/problems/search-a-2d-matrix/)

**题目描述**

编写一个高效的算法来判断 m x n 矩阵中，是否存在一个目标值。该矩阵具有如下特性：

- 每行中的整数从左到右按升序排列。、
- 每行的第一个整数大于前一行的最后一个整数。

**思路**

因为二维数组行和列都有序，所以我们可以将二维数组转换为一维数组然后使用二分查找法进行查找，将二维数组转换为一维数组的方法：

- (x,y)转换为一维数组单位下表：x*y+y,y表示当前的列数
- pos为一维数组的下表，转换为二维数组中元素的下表：`max[pos/col][pos%col]`,col表示当前的列数

**代码实现**

~~~ java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {

        // 使用二分查找法，将二维数组转换为一位数组

        if(matrix == null || matrix.length == 0)
        return false;

        int row=matrix.length;
        int col=matrix[0].length;
        int l=0;
        int r=row*col-1;
        int m=0;
        int temp=0;
        // 下面使用二分查找法进行查找
        while(l<=r){
            m=(l+r)/2;
            temp=matrix[m/col][m%col];
            if(target == temp){
                return true;
            }else if(temp >target){
                r=m-1;
            }else if(temp <target){
                l=m+1;
            }
        }
        return false;

    }
}
~~~

## 滑动窗口

### ddd

[定长子串中元音字母最大个数](https://leetcode-cn.com/problems/maximum-number-of-vowels-in-a-substring-of-given-length/submissions/)

**题目描述**

给你字符串 s 和整数 k 。

请返回字符串 s 中长度为 k 的单个子字符串中可能包含的最大元音字母数。

英文中的 元音字母 为（a, e, i, o, u）。

**代码实现**

~~~ java
class Solution {
    public int maxVowels(String s, int k) {

        if(s == null || s.length() == 0 || s.length() < k){
            return 0;
        }

        Set hashSet=new HashSet();
        hashSet.add('a');
        hashSet.add('e');
        hashSet.add('i');
        hashSet.add('o');
        hashSet.add('u');

        int count=0;
        int max=0;
        for(int i=0;i<k;i++){
           if(hashSet.contains(s.charAt(i)))
                count++;
        }
        int j=k;
        max=count;

        while(j< s.length()){

            if(hashSet.contains(s.charAt(j-k))){
                count--;
            }
           
            if(j<s.length()&&hashSet.contains(s.charAt(j))){
                count++;
            }
            j++;
            if(max < count)
                max=count;
        }    

        return max;
    }
}
~~~

### 209

[长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)

**题目描述**

给定一个含有 n 个正整数的数组和一个正整数 target 。

找出该数组中满足其和 ≥ target 的长度最小的 连续子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。

**代码实现**

~~~ java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {

        if(nums == null || nums.length == 0){
            return 0;
        }

        int i=0;
        int j=0;
        int sum=0;
        int len=Integer.MAX_VALUE;
         while(j< nums.length){
            sum +=nums[j];
            // 如果累加和大于target
            while( sum >= target){
                if(j-i < len){
                    len=j-i+1;
                }
                sum -=nums[i];
                i++;
                
            }
            j++;
         }
         if(len==Integer.MAX_VALUE)
         return 0;
        return len;
    }
}
~~~

## 递归

递归算法的延伸：回溯，分治，DFS

**四要素**

- 参数
- 拆分
- 拆解
- 返回值

### 509

[斐波那契数列](https://leetcode-cn.com/problems/fibonacci-number/)

**题目描述**

斐波那契数，通常用 F(n) 表示，形成的序列称为 斐波那契数列 。该数列由 0 和 1 开始，后面的每一项数字都是前面两项数字的和。也就是：

F(0) = 0，F(1) = 1
F(n) = F(n - 1) + F(n - 2)，其中 n > 1

给你 n ，请计算 F(n) 。

**代码实现**

~~~ java
class Solution {
    public int fib(int n) {

        if(n<2)
        {
            return n == 1?1:0;
        }
        return fib(n-1)+fib(n-2);

    }
}
~~~

### 206

[反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

**题目描述**

反转一个单链表。

**代码实现**

使用递归方法

~~~ java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {

    // 使用递归法
    if(head == null || head.next == null){
        return head;
    }
    ListNode res=reverseList(head.next);
    head.next.next=head;
    head.next=null;
    return res;


    }
}
~~~

### 344

[反转字符串](https://leetcode-cn.com/problems/reverse-string/)

**题目描述**

编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 char[] 的形式给出。

不要给另外的数组分配额外的空间，你必须原地修改输入数组、使用 O(1) 的额外空间解决这一问题。

你可以假设数组中的所有字符都是 ASCII 码表中的可打印字符。

**代码实现**

~~~ java
class Solution {
    public void reverseString(char[] s) {

        if(s == null || s.length==0)
        return ;
        recursion(s,0,s.length-1);
    }

    private void recursion(char[] s,int left,int right){
        if(left >= right){
            return ;
        }
        recursion(s,left+1,right-1);
        char c=s[left];
        s[left]=s[right];
        s[right]=c;

    }
}
~~~

> 也可以使用双指针法解答

## 分治法

### 169

### 53

## 回溯法

### 回溯算法框架

~~~ java
public static List<List<Integer>> combine(int n, int k) {

        // 回溯法

        List<List<Integer>> res = new ArrayList();
        ArrayList<Integer> temp = new ArrayList();
        
//        一般都是从 1 开始递归回溯
        backTracking(res, 1, n, k, temp);

        return res;

    }

    /**
     * 回溯方法体，可以基于下面代码进行修改
     * @param res 返回的结果集
     * @param index 开始回溯的第一个节点
     * @param n
     * @param k
     * @param temp 临时结果集
     */
    private static void backTracking(List<List<Integer>> res, int index, int n, int k, ArrayList temp) {

        // 首先找到递归出口条件，
        if (temp.size() == k) {
            // 把temp集合添加到结果集中，然后返回
            res.add(new ArrayList(temp));
            return;
        }
        // 对每一个节点进行递归操作
        for (int i = index; i <= n; i++) {
            temp.add(i);
            backTracking(res, i + 1, n, k, temp);
            //做剪枝操作，剪枝减去的是最后面的元素
            temp.remove(temp.size() - 1);
        }
    }
}
~~~

### 77

算法框架举例说明

1. 路径：也就是已经做出的选择。

2. 选择列表：也就是你当前可以做的选择。

3. 结束条件：也就是到达决策树底层，无法再做选择的条件

**回溯法算法基本框架**

~~~ java
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return

    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
~~~

> **其核心就是 for 循环里面的递归，在递归调用之前「做选择」，在递归调用之后「撤销选择」**

**全排列问题**

比方说给三个数 `[1,2,3]`，你肯定不会无规律地乱穷举，一般是这样：

先固定第一位为 1，然后第二位可以是 2，那么第三位只能是 3；然后可以把第二位变成 3，第三位就只能是 2 了；然后就只能变化第一位，变成 2，然后再穷举后两位……

回溯法生成的递归树

![1618237168776](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1618237168776.png)

只要从根遍历这棵树，记录路径上的数字，其实就是所有的全排列。**我们不妨把这棵树称为回溯算法的「决策树」**。

**为啥说这是决策树呢，因为你在每个节点上其实都在做决策**。比如说你站在下图的红色节点上：

![1618237275796](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1618237275796.png)

你现在就在做决策，可以选择 1 那条树枝，也可以选择 3 那条树枝。为啥只能在 1 和 3 之中选择呢？因为 2 这个树枝在你身后，这个选择你之前做过了，而全排列是不允许重复使用数字的。

**现在可以解答开头的几个名词：[2] 就是「路径」，记录你已经做过的选择；[1,3] 就是「选择列表」，表示你当前可以做出的选择；「结束条件」就是遍历到树的底层，在这里就是选择列表为空的时候**。

如果明白了这几个名词，**可以把「路径」和「选择」列表作为决策树上每个节点的属性**，比如下图列出了几个节点的属性：

![1618237347454](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/12/222229-65151.png)

**我们定义的 backtrack 函数其实就像一个指针，在这棵树上游走，同时要正确维护每个节点的属性，每当走到树的底层，其「路径」就是一个全排列**。

再进一步，如何遍历一棵树？而多叉树的遍历框架就是这样：

~~~ java
void traverse(TreeNode root) {
    for (TreeNode child : root.childern)
        // 前序遍历需要的操作
        traverse(child);
        // 后序遍历需要的操作
}
~~~

而所谓的前序遍历和后序遍历，他们只是两个很有用的时间点，我给你画张图你就明白了：

![1618237432592](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1618237432592.png)

**前序遍历的代码在进入某一个节点之前的那个时间点执行，后序遍历代码在离开某个节点之后的那个时间点执行**。

回想我们刚才说的，「路径」和「选择」是每个节点的属性，函数在树上游走要正确维护节点的属性，那么就要在这两个特殊时间点搞点动作：

![1618237553557](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1618237553557.png)

~~~ java
for 选择 in 选择列表:
    # 做选择
    将该选择从选择列表移除
    路径.add(选择)
    backtrack(路径, 选择列表)
    # 撤销选择
    路径.remove(选择)
    将该选择再加入选择列表
~~~

**我们只要在递归之前做出选择，在递归之后撤销刚才的选择**，就能正确得到每个节点的选择列表和路径。

当然，这个算法解决全排列不是很高效，应为对链表使用 `contains` 方法需要 O(N) 的时间复杂度

但是必须说明的是，不管怎么优化，都符合回溯框架，而且时间复杂度都不可能低于 O(N!)，因为穷举整棵决策树是无法避免的。**这也是回溯算法的一个特点，不像动态规划存在重叠子问题可以优化，回溯算法就是纯暴力穷举，复杂度一般都很高**。

22 78 77 46

### 78

[子集问题](https://leetcode-cn.com/problems/subsets/)

**题目描述**

给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。

解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。

**回溯法**

~~~ java
public static List<List<Integer>> subsets(int[] nums) {

        // 使用回溯法解决
        List<List<Integer>> list=new ArrayList();
        // 首先添加一个空的集合到上面的集合中，空集也算是一个子集
        list.add(new ArrayList());
        // 循环遍历每一个元素，进行回溯,
        //i代表子集中元素的个数，因为空集就是0个，所以从i=1开始遍历，表示子集中有一个元素
        for(int i=1;i<=nums.length;i++){
            backTracking(nums,list,i,0,new ArrayList());
        }
        return list;
    }

    /*
    nums:代表是数组
    list:返回的结果，里面的元素也是数组
    len:子集的长度
    index:从哪一个索引的位置开始找子集
    subsub:代表子集
    */
    private static void backTracking(int []nums,List<List<Integer>> list,int len,int index,List<Integer> subList){

        // 递归结束条件,如果当前子集的长度等于len，那么就添加到list集合中，并且返回
        if(subList.size() == len){
            list.add(new ArrayList<>(subList));
            return;
        }
        // 否则从index位置开始遍历添加元素
        for(int i=index;i<nums.length;i++){
            // 添加一个元素到子集中
            subList.add(nums[i]);
            // 继续进行递归
            backTracking(nums,list,len,i+1,subList);
            // 这一步是回溯操作，回溯到上一次的位置
            subList.remove(subList.size()-1);
        }

    }

~~~

**DFS法**

~~~ java
public static List<List<Integer>> fun(int nums[]){
        List<List<Integer>> res=new ArrayList();
        dfs(nums,res,0,new ArrayList<>());

        return res;
    }

    private static void dfs(int []nums,List<List<Integer>> res,int index,List<Integer> subList){

        // 首先把元素添加到子集当中
        res.add(new ArrayList(subList));

        // 递归结束
        if(index == nums.length){
            return ;
        }
        // 然后从index位置开始遍历整个数组
        for(int i=index;i<nums.length;i++){
            subList.add(nums[i]);
            // 递归添加元素
            dfs(nums,res,i+1,subList);
            // 递归退出的话，需要删除子集中刚才添加的元素
            subList.remove(subList.size()-1);
        }

    }
~~~

### 22

[括号生成](https://leetcode-cn.com/problems/generate-parentheses/)

**题目描述**

数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。





### 17

[电话号码子集问题](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

~~~ java
public class Test17 {

    public static void main(String[] args) {
        List<String> strings = letterCombinations("23");
        System.out.println(strings.toString());

    }

    public static List<String> letterCombinations(String digits) {

        // 先使用哈希表存储对应的映射关系
        Map<Character,ArrayList<String>> map=new HashMap<Character,ArrayList<String>>();
        map.put('2',new ArrayList(Arrays.asList("a","b","c")));
        map.put('3',new ArrayList(Arrays.asList("d","e","f")));
        map.put('4',new ArrayList(Arrays.asList("g","h","i")));
        map.put('5',new ArrayList(Arrays.asList("j","k","l")));
        map.put('6',new ArrayList(Arrays.asList("m","n","o")));
        map.put('7',new ArrayList(Arrays.asList("p","q","r","s")));
        map.put('8',new ArrayList(Arrays.asList("t","u","v")));
        map.put('9',new ArrayList(Arrays.asList("w","x","y","z")));

        ArrayList temp=new ArrayList();
        for(int i=0;i<digits.length();i++){
            char c=digits.charAt(i);
            List list=map.get(c);
            temp.addAll(list);

        }

        // 存放结果的数组
        List<String> res=new ArrayList();

        // 下面对temp集合中的元素做排列操作
        int len=digits.length();
        String s="";
        //for(int j=0;j<temp.size();j++){
            backTracking(res,len,0,s,temp);
       // }


        return res;
    }
    public static void backTracking(List<String> list, int len, int index, String s, ArrayList temp){

        // 首先写出递归出口条件
        if(s.length() == len){
            // 添加结果到res
//            把StringBuilder转换为string
            String s1 = s.toString();
            list.add(s1);
            return;
        }

//        做选择
        for(int i=index;i<temp.size();i++){
            // 添加当前元素到字符串中
            s=s+temp.get(i);
            // 做递归操作
            backTracking(list,len,i+1,s,temp);
            // 做剪枝操作
            s=s.substring(0,s.length()-1);
        }
    }
}

~~~





















## DFS

注意回溯法和dfs方法的区分，可以这样理解，回溯法=dfs+剪枝，也就是说回溯法在递归遍历的过程中，如果找到满足条件的解，就不会再向下遍历递归，而dfs会一条路走到黑。

不使用递归的DFS算法可以使用栈去解决

### 938

[二叉搜索树的范围和](https://leetcode-cn.com/problems/range-sum-of-bst/)

**题目描述**

给定二叉搜索树的根结点 `root`，返回值位于范围 *[low, high]* 之间的所有结点的值的和。

**思路一**

使用递归的方法

~~~ java
class Solution {
    public int rangeSumBST(TreeNode root, int low, int high) {
        // 使用递归的方式解决
        if(root == null){
            return 0;
        }
        int leftsum=rangeSumBST(root.left,low,high);
        int rightsum=rangeSumBST(root.right,low,high);
        int res=leftsum+rightsum;
        // 判断树根的值是否在规定区间内
        if(root.val>=low && root.val <=high){
            res+=root.val;
        }

        return res;


    }
}
~~~

**思路二**

使用树的广度优先搜索

~~~ java
class Solution {
    public int rangeSumBST(TreeNode root, int low, int high) {
        int res=0;
        if(root == null){
            return 0;
        }
        // 广度优先搜索，借助队列对数树进行广度优先遍历
        TreeNode node=null;
        Queue <TreeNode>queue=new LinkedList<TreeNode>();
        queue.add(root);
        while(!queue.isEmpty()){
            // 移除栈顶的元素
            node=queue.poll();
            int temp=node.val;
            if(temp >=low && temp <= high){
                res+=temp;
            }
            if(node.left !=null){
                queue.add(node.left);
            }
            if(node.right != null){
                queue.add(node.right);
            }
        }

        return res;
    }
}
~~~

### 200

[岛屿数量](https://leetcode-cn.com/problems/number-of-islands/submissions/)

**题目描述**

给你一个由 '1'（陆地）和 '0'（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围。

**方法一：深度优先遍历**

~~~ java
class Solution {
    public int numIslands(char[][] grid) {

        // 使用深度优先搜索方法
        // 用来存储岛屿的个数
        int res=0;
        if(grid.length == 0 || grid == null){
            return 0;
        }

        for(int i=0;i<grid.length;i++){
            for(int j=0;j<grid[i].length;j++){
                if(grid[i][j] == '1'){
                    res+=1;
                    // 以当前节点开始深度遍历
                    dfs(grid,i,j,grid.length,grid[0].length);
                }
            }
        }
        return res;

    }

    private void dfs(char [][]arr,int x,int y,int row,int col){
        if(x < 0 || y<0 || x>=row|| y>=col || arr[x][y] =='0')
            return ;
        
        arr[x][y]='0';
        // 进行递归遍历
        dfs(arr,x+1,y,row,col);
        dfs(arr,x-1,y,row,col);
        dfs(arr,x,y+1,row,col);
        dfs(arr,x,y-1,row,col);
    }
}
~~~

**方法二：广度优先遍历**

~~~ java
class Solution {
    public int numIslands(char[][] grid) {

        // 广度优先遍历
        if(grid.length == 0 || grid == null){
            return 0;
        }
        int res=0;
        int row=grid.length;
        int col=grid[0].length;
        // 申请一个队列
        Queue<ArrayList<Integer>> queue=new LinkedList();
        for(int i=0;i<row;i++){
            for(int j=0;j<col;j++){
                if(grid[i][j]=='1'){
                    res+=1;
                    // 把当前元素的坐标添加到队列中
                  ArrayList l=new ArrayList();
                  l.add(i);
                  l.add(j);
                    queue.add(l);
                    grid[i][j]='0';

                }
                while(queue.size()>0){
                    // 抛出栈顶元素
                    ArrayList list=queue.poll();
                    int x=(Integer)list.get(0);
                    int y=(Integer)list.get(1);
                    // 下面判断当前元素的四周情况
                    if(x-1>=0 && grid[x-1][y] == '1'){
                        // 添加到队列
                        ArrayList l1=new ArrayList();
                        l1.add(x-1);
                        l1.add(y);
                        queue.add(l1);
                        grid[x-1][y]='0';
                    }
                    if(x+1<row && grid[x+1][y] == '1'){
                        // 添加到队列
                        ArrayList l2=new ArrayList();
                        l2.add(x+1);
                        l2.add(y);
                        queue.add(l2);
                        grid[x+1][y]='0';
                    }
                    if(y-1>=0 && grid[x][y-1] == '1'){
                        // 添加到队列
                        ArrayList l3=new ArrayList();
                        l3.add(x);
                        l3.add(y-1);
                        queue.add(l3);
                        grid[x][y-1]='0';
                    }
                    if(y+1<col && grid[x][y+1] == '1'){
                        // 添加到队列
                        ArrayList l4=new ArrayList();
                        l4.add(x);
                        l4.add(y+1);
                        queue.add(l4);
                        grid[x][y+1]='0';
                    }
                }
            }
        }
        return res;

    }
}
~~~

**方法三：并查集**

~~~ java


~~~

## BFS

常常需要借助队列实现

### 102

[二叉树层次遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

给你一个二叉树，请你返回其按 **层序遍历** 得到的节点值。 （即逐层地，从左到右访问所有节点）。

**代码实现**

~~~ java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {

  LinkedList<TreeNode> queue=new LinkedList();
        List<List<Integer>> list=new ArrayList();
        // 广度优先遍历，需要借助队列实现
        if(root == null){
            return list;
        }
      
        
        queue.add(root);
        while(queue.size()>0){
            int len=queue.size();
            ArrayList l=new ArrayList();
            while(len>0){
                TreeNode node=queue.poll();
                l.add(node.val);
                if(node.left != null){
                queue.add(node.left);
                }
                 if(node.right != null){
                    queue.add(node.right);
                 }
                len--;
            }
            list.add(l);
        
        }
    
     return list;
    }
}
~~~

**使用深度优先遍历**

~~~ java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
// 使用深度优先遍历算法
        List<List<Integer>> list=new ArrayList();
        if(root == null){
            return list;
        }
        // 否则进行深度优先遍历,0代表从第0层开始遍历
        dfs(root,list,0);
        return list;
    }

    private void dfs(TreeNode node,List<List<Integer>> list ,int leval){
// 递归出口
        if(node == null){
            return;
        }
        // 首先判断当前的层次是否超过数组的长度
        if(leval > list.size()-1){
            ArrayList l=new ArrayList();
            // l.add(node.val);
            list.add(l);
        }
        // 把当前节点的元素添加到集合中
        list.get(leval).add(node.val);
        // 对左边的子树进行递归
        dfs(node.left,list,leval+1);
        // 对右边的子树进行递归
        dfs(node.right,list,leval+1);
    }
}
~~~

### 107

[二叉树的层次遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/)

**方法一：使用BFS**

~~~ java
class Solution {
    public List<List<Integer>> levelOrderBottom(TreeNode root) {
        // // 可以使用广度优先遍历，但是插入元素使用头插法,借助于链表实现
        // List<List<Integer>> list=new LinkedList();
        // if(root == null){
        //     return list;
        // }
        // LinkedList<TreeNode> queue=new LinkedList();
        // // 先把根节点插入队列
        // queue.add(root);
        // while(queue.size()>0){
        //     // len就代表每一层有多少个元素
        //     int len=queue.size();
        //     ArrayList<Integer> l=new ArrayList();
        //     // 下面开始遍历一层的元素
        //     while(len > 0){
        //         TreeNode node=queue.poll();
        //         l.add(node.val);
        //         if(node.left != null){
        //             queue.add(node.left);
        //         }
        //         if(node.right != null){
        //             queue.add(node.right);
        //         }
        //         // 在这里使用头插法把数组插入链表中
               
        //         len--;
        //     }
        //      list.addFirst(l);
        // }
        // return list;

       List<List<Integer>> result = new ArrayList<>();
        if (root == null) {
            return result;
        }

        Queue<TreeNode> q = new LinkedList<>();
        q.add(root);
        // 先把结果存储到临时变量中
        LinkedList<ArrayList<Integer>> temp = new LinkedList<>();

        while (q.size() > 0) {
            int size = q.size();
            ArrayList<Integer> list = new ArrayList<>();
            while (size > 0) {
                TreeNode cur = q.poll();
                list.add(cur.val);
                if (cur.left != null) {
                    q.add(cur.left);
                }
                if (cur.right != null) {
                    q.add(cur.right);
                }
                size--;
            }
            temp.addFirst(new ArrayList<>(list));
        }
        result = new ArrayList<>(temp);
        
        return result;
    }
}
~~~

**深度优先遍历**

~~~ java
class Solution {
    public List<List<Integer>> levelOrderBottom(TreeNode root) {
        // // 可以使用广度优先遍历，但是插入元素使用头插法,借助于链表实现
        // List<List<Integer>> list=new LinkedList();
        // if(root == null){
        //     return list;
        // }
        // LinkedList<TreeNode> queue=new LinkedList();
        // // 先把根节点插入队列
        // queue.add(root);
        // while(queue.size()>0){
        //     // len就代表每一层有多少个元素
        //     int len=queue.size();
        //     ArrayList<Integer> l=new ArrayList();
        //     // 下面开始遍历一层的元素
        //     while(len > 0){
        //         TreeNode node=queue.poll();
        //         l.add(node.val);
        //         if(node.left != null){
        //             queue.add(node.left);
        //         }
        //         if(node.right != null){
        //             queue.add(node.right);
        //         }
        //         // 在这里使用头插法把数组插入链表中
               
        //         len--;
        //     }
        //      list.addFirst(l);
        // }
        // return list;

    //    List<List<Integer>> result = new ArrayList<>();
    //     if (root == null) {
    //         return result;
    //     }

    //     Queue<TreeNode> q = new LinkedList<>();
    //     q.add(root);
    //     // 先把结果存储到临时变量中
    //     LinkedList<ArrayList<Integer>> temp = new LinkedList<>();

    //     while (q.size() > 0) {
    //         int size = q.size();
    //         ArrayList<Integer> list = new ArrayList<>();
    //         while (size > 0) {
    //             TreeNode cur = q.poll();
    //             list.add(cur.val);
    //             if (cur.left != null) {
    //                 q.add(cur.left);
    //             }
    //             if (cur.right != null) {
    //                 q.add(cur.right);
    //             }
    //             size--;
    //         }
    //         temp.addFirst(new ArrayList<>(list));
    //     }
    //     result = new ArrayList<>(temp);
        
    //     return result;

        // 使用dfs方法解答

        List<List<Integer>>res=new ArrayList();
        if(root == null)
        {
            return res;
        }
        // 使用深度优先遍历
        dfs(root,res,0);
        // 对返回的结果进行逆转
        Collections.reverse(res);

        return res;
    }

    private void dfs(TreeNode root,List<List<Integer>> list,int leavl){

        // 递归函数的出口
        if(root == null){
            return;
        }
        
        if(leavl > list.size()-1){
            // 新的一层添加到数组中
            ArrayList <Integer>l=new ArrayList();
            list.add(l);
        }
        // 把当前的节点值添加到数组
        list.get(leavl).add(root.val);
        // 左边递归
        dfs(root.left,list,leavl+1);
        // 右边递归
        dfs(root.right,list,leavl+1);



    }
}
~~~

## 贪心算法

**思想**

贪心算法总是做出当前**看来最好的选择**，并不从整体最优考虑。所作的选择只是在某种意义上的**局部最优**
**选择。**

**贪心算法的缺点**

由于贪心算法不是从全局角度考虑，所以使用贪心算法得到的结果不一定是全局最优的解。

- 有的问题上面，可以得到整体的最优解。
- 有的问题上，无法得到全局最优解，但是可以得到近似的最优解。

**贪心算法特点**

- 是一步一步地进行，常以当前情况为基础根据某个优化测度作最优选择，而不考虑各种可能的整体情况，它省去了为找最优解要穷尽所有可能而必须耗费的大量时间
- 它采用自顶向下，以迭代的方法做出相继的贪心选择，每做一次贪心选择就将所求问题简化为一个规模更小的子问题，通过每一步贪心选择，可得到问题的一个最优解
- 虽然每一步上都要保证能获得局部最优解，但由此产生的全局解有时不一定是最优的，所以贪婪法不要回溯。

**两个性质**

- 最优子结构性质：**一个问题的最优解包含其子问题的最优解**，那么此问题具有最优子结构性质。
- 贪心选择性质：问题的整体最优解可以通过一些列局部最优的选择（贪心选择）来达到。因此贪心算法和分治法一样，都是自顶向下的思想。

**贪心算法和动态规划算法的区别**

- 贪心选择性质，指所求问题的整体最优解可以通过一系列的**局部最优**的选择取得，既贪心选择达到，这是贪心算法的一个基本要素，也是与动态规划算法的主要区别。
- 动态规划算法通常以**自底向上**的方式解各个子问题，而贪心算法通常以**自顶向下**的方式进行，以**迭代**的方式做出相继的贪心选择，每做一次贪心选择就是将问题化简为规模更小的子问题。

> 当一个问题的全局最优解包含子问题的最优解时候，就称此问题具有**最优子结构性质**，问题的最优子结构性质是问题可以使用动态规划算法或者贪心算法解决的关键特征。

322

1217

55



## 算法记忆化

思想是把计算过程中的中间结果保存到某一张表中，以后直接拿出来使用，不需要重新计算，递归算法中比较常用

509

322

## 动态规划

dp没有递归操作，因为使用数组存储之前的计算结果，用这些结果可以进行递推操作。一般数组的最后一个元素就是原问题的解。

**算法思想**

1. 动态规划算法与分治法类似，其基本思想也是将待求解问题分解成若干个子问题，但是动态规划分解后的子问题往往不是相互独立的，而且不同的子问题往往有很多的重叠性。
2. 但是适合于用动态规划求解的问题经分解得到的子问题往往不是互相独立的。不同子问题的数目常常很多。在用分治法求解时，有些子问题被重复计算了许多次。
3. 如果能够保存已解决的子问题的答案，而在需要时再找出已求得的答案，就可以避免大量重复计算，从而得到多项式时间算法。
4. 最优化原理（最优子结构性质） ：不论过去状态和决策如何，对前面的决策所形成的状态而言，余下的诸决策必须构成最优策略。简而言之，**一个最优化策略的子策略总是最优的**。一个问题满足最优化原理又称其具有最优子结构性质（一个优化策略的子策略也是最优的）

**策略**

1. 牺牲空间换取时间，让时间复杂度从指数级下降到多项式级别。
2. 用于解决具有多阶段策略的优化问题方法。

**和分治算法的区别**

1. 动态规划的子问题往往只求解一次，然后把结果存起来重复使用。
2. 分治法求解问题的时候往往把一个大问题划分为若干个小的问题求解，但是不会存储中间结果，重复的子问题往往会被多次计算，所以效率低。

**动态规划三要素**

- 初始状态
- 方程式
- 终止状态

**动态规划算法题解类型**

- 计数：有多少种方式或者方法
  - 机器人从左上角到右下角的路径
- 求最值：最大值或者最小值
  - 机器人从左到右最大路径之和
- 求存在性，是否存在某一种可能
  - 是否存在机器人从左到右的路径

### 93

[有效ip问题](https://leetcode-cn.com/problems/restore-ip-addresses/)

**思路**

|      |      | 1    | 9    | 2    | 1    | 6    | 8    | 0    | 1    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
| 1    | 0    | 1    | 1    | 1    | 0    | 0    | 0    | 0    | 0    |
| 2    | 0    | 0    | 1    | 2    | 2    | 2    | 3    | 2    | 2    |
| 3    | 0    | 0    |      |      |      |      |      |      |      |
| 4    | 0    | 0    |      |      |      |      |      |      |      |

`dp[i][j]`代表str的前j个字符组成多少个255以内数字

首先填写第一行，`dp[1][1]`=1表示字符串1组成0-255之间的数字有多少种，显然此时有一种，同样，一直到`dp[1][2]`一直是一种方法，后面1921字符串显然已经超出了范围，所以表格中填写0。

在看`dp[2][1]`，表示字符串中一个字符长度可以表示为2个255以内的数，但是显然这是不可能的，所以就填写0，所以`dp[3][1]`,`dp[4][1]`,`dp[3][2]`，`dp[3][2]`,`dp[4][2]`,`dp[4][3]`全部填写0，`dp[2][2]`表示，首先看9是否可以表示为一个0-255以内的数字，显然是可以的，所以`dp[1][2]`=1,所以在看其前面的数字组合是否可以表示255以内的数字，然后在看前两个字符是否可以表示为0-255以内的数字，显然19是可以的，再看19前面是0，不可以表示255以内数字，所以是1种

`dp[2][3]`表示前三个字符可以组成多少个255以内的数字，首先看2是否可以表示255以内的数字，显然`dp[1][3]`为1表示有一种，所以在添加2之前的数字，判断是否可以组成255以内的数字，显然92是可以的，然后继续添加9之前的数字，192也可以，所以一共是2种。

`dp[2][4]`首先判断1是否可以表示为255以内的数字，显然是可以的，因为`dp[1][3]`=1表示有一种，所以添加1前面的数字，21显然也是可以，因为`dp[1][2]`=1，所以继续添加2之前的数字，921显然不行，所以一共是2种



## 前缀匹配树

**使用数组进行存储**

- 插入：o(n)
- 搜索：o(n)
- 前缀匹配：o(nm),n表示搜索复杂度，m表示匹配复杂度 

**前缀匹配树**

- 插入：o(k)
- 搜索：o(k)
- 前缀匹配：o(k)

> 其中k代表字符串的长度

### 720

208

692