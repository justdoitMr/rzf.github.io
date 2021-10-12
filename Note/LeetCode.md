# 算法小抄

# 数据结构

## 数据结构的存储⽅式

数据结构的存储⽅式只有两种：数组（顺序存储）和链表（链式存储）。这句话怎么理解，不是还有散列表、栈、队列、堆、树、图等等各种数据结构吗？

我们分析问题，⼀定要有递归的思想，⾃顶向下，从抽象到具体。你上来就列出这么多，那些都属于「上层建筑」，⽽数组和链表才是「结构基础」。因为那些多样化的数据结构，究其源头，都是在链表或者数组上的特殊操作，API 不同⽽已。

⽐如说「队列」、「栈」这两种数据结构既可以使⽤链表也可以使⽤数组实现。⽤数组实现，就要处理扩容缩容的问题；⽤链表实现，没有这个问题，但需要更多的内存空间存储节点指针。

「图」的两种表⽰⽅法，邻接表就是链表，邻接矩阵就是⼆维数组。邻接矩阵判断连通性迅速，并可以进⾏矩阵运算解决⼀些问题，但是如果图⽐较稀疏的话很耗费空间。邻接表⽐较节省空间，但是很多操作的效率上肯定⽐不过邻接矩阵。

「散列表」就是通过散列函数把键映射到⼀个⼤数组⾥。⽽且对于解决散列冲突的⽅法，拉链法需要链表特性，操作简单，但需要额外的空间存储指针；线性探查法就需要数组特性，以便连续寻址，不需要指针的存储空间，但操作稍微复杂些。

「树」，⽤数组实现就是「堆」，因为「堆」是⼀个完全⼆叉树，⽤数组存储不需要节点指针，操作也⽐较简单；⽤链表实现就是很常⻅的那种「树」，因为不⼀定是完全⼆叉树，所以不适合⽤数组存储。为此，在这种链表「树」结构之上，⼜衍⽣出各种巧妙的设计，⽐如⼆叉搜索树、AVL树、红⿊树、区间树、B 树等等，以应对不同的问题。

综上，数据结构种类很多，甚⾄你也可以发明⾃⼰的数据结构，但是底层存储⽆⾮数组或者链表，⼆者的优缺点如下：

- 数组由于是紧凑连续存储,可以随机访问，通过索引快速找到对应元素，⽽且相对节约存储空间。但正因为连续存储，内存空间必须⼀次性分配够，所以说数组如果要扩容，需要重新分配⼀块更⼤的空间，再把数据全部复制过去，时间复杂度O(N)；⽽且你如果想在数组中间进⾏插⼊和删除，每次必须搬移后⾯的所有数据以保持连续，时间复杂度 O(N)。
- 链表因为元素不连续，⽽是靠指针指向下⼀个元素的位置，所以不存在数组的扩容问题；如果知道某⼀元素的前驱和后驱，操作指针即可删除该元素或者插⼊新元素，时间复杂度 O(1)。但是正因为存储空间不连续，你⽆法根据⼀个索引算出对应元素的地址，所以不能随机访问；⽽且由于每个元素必须存储指向前后元素位置的指针，会消耗相对更多的储存空间。

## 数据结构的基本操作

对于任何数据结构，其基本操作⽆⾮遍历 + 访问，再具体⼀点就是：增删查改。

数据结构种类很多，但它们存在的⽬的都是在不同的应⽤场景，尽可能⾼效地增删查改。话说这不就是数据结构的使命么？

如何遍历 + 访问？我们仍然从最⾼层来看，各种数据结构的遍历 + 访问⽆⾮两种形式：线性的和⾮线性的。

线性就是 for/while 迭代为代表，⾮线性就是递归为代表。再具体⼀步，⽆⾮以下⼏种框架：

**数组遍历框架，典型的线性迭代结构：**

~~~ java
void traverse(int[] arr) {
  for (int i = 0; i < arr.length; i++) {
  	// 迭代访问 arr[i]
  }
}
~~~

**链表遍历框架，兼具迭代和递归结构：**

~~~ java
/* 基本的单链表节点 */
class ListNode {
  int val;
  ListNode next;
}
void traverse(ListNode head) {
  for (ListNode p = head; p != null; p = p.next) {
  	// 迭代访问 p.val
  }
}
void traverse(ListNode head) {
  // 递归访问 head.val
  traverse(head.next)
}
~~~

**⼆叉树遍历框架，典型的⾮线性递归遍历结构：**

~~~ java
/* 基本的⼆叉树节点 */
class TreeNode {
  int val;
  TreeNode left, right;
}
void traverse(TreeNode root) {
  traverse(root.left)
  traverse(root.right)
}
~~~

看⼆叉树的递归遍历⽅式和链表的递归遍历⽅式，相似不？再看看⼆叉树结构和单链表结构，相似不？如果再多⼏条叉，N 叉树你会不会遍历？

**⼆叉树框架可以扩展为 N 叉树的遍历框架：**

~~~ java
/* 基本的 N 叉树节点 */
class TreeNode {
  int val;
  TreeNode[] children;
}
void traverse(TreeNode root) {
  for (TreeNode child : root.children)
  	traverse(child)
}
~~~

N 叉树的遍历⼜可以扩展为图的遍历，因为图就是好⼏ N 叉棵树的结合体



## 数组

### 数组概念

**基本概念**

- 定义：在**连续的内存空间**中，储存一组**相同类型**的元素
- 数组的访问: 通过索引访问元素。a[0]
- 数组的内存空间是连续的，增删需要移动后面的元素
- 二维数组的内存地址是连续的吗？
  - 二维数组实际是一个线性数组存放着其他数组的首地址

**数组在内存中的存储方式**

![1631793034936](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/195036-259521.png)

需要两点注意的是

- 数组下标都是从0开始的。
- 数组内存空间的地址是连续的

正是因为数组的在内存空间的地址是连续的，所以我们在删除或者增添元素的时候，就难免要移动其他元素的地址。

**操作复杂度分析**

- 访问:o(1)
- 搜索:o(n)
- 插入:o(n)
- 删除:o(n)

> 适合读多写少的情景

### 简单题目

#### 704、二分查找

[二分查找](https://leetcode-cn.com/problems/binary-search/)



![1631793346242](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/195548-464493.png)

**题解**

这是一道基础题目，必须掌握，但是需要注意一下4点：

1. 需要考虑数组为空的情况下，
2. 边界检查：`left <= right`，使用`<=`号，在这里当`left = right`的时候是有意义的。
3. 实际中用`int mid = left + (right-left)/2`; 可以防止`left+right`溢出（超出整数范围）
4. 在这个题目中还有一点就是题目中明确无重复元素，但是我们需要明白的是使用二分查找法最基本的就是保证元素有序。

**下面给出代码：**

~~~ java
class Solution {
    public int search(int[] nums, int target) {
        if(nums.length == 0){
            return -1;
        }
        if(nums == null){
            return -1;
        }
        int left=0;
        int right=nums.length-1;
        int mid=0;
        while(left <= right){
            mid =(left + right)/2;
          //int mid = left + (right-left)/2
            if(nums[mid] == target){
                return mid;
            }else if(nums[mid]< target){
                left =mid+1;
            }else if(nums[mid] > target){
                right =mid-1;
            }
        }
        return -1;
    }
}
~~~

**二分法第二种写法**

如果说定义 target 是在一个在左闭右开的区间里，也就是[left, right) ，那么二分法的边界处理方式则截然不同。

有如下两点：

- `while (left < right)`，这里使用 < ,因为left == right在区间[left, right)是没有意义的
- `if (nums[middle] > target)` right 更新为 middle，因为当前`nums[middle]`不等于`target`，去左区间继续寻找，而寻找区间是左闭右开区间，所以right更新为middle，即：下一个查询区间不会去比较`nums[middle]`

**代码演示**

~~~ java
// 版本二
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size(); // 定义target在左闭右开的区间里，即：[left, right)
        while (left < right) { // 因为left == right的时候，在[left, right)是无效的空间，所以使用 <
            int middle = left + ((right - left) >> 1);
            if (nums[middle] > target) {
                right = middle; // target 在左区间，在[left, middle)中
            } else if (nums[middle] < target) {
                left = middle + 1; // target 在右区间，在[middle + 1, right)中
            } else { // nums[middle] == target
                return middle; // 数组中找到目标值，直接返回下标
            }
        }
        // 未找到目标值
        return -1;
    }
};
~~~

#### 35、搜索插入位置

[搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/)

![1631795197931](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/202638-648086.png)



**题解**

这道题明显是二分查找的变化版本，题目中已说明数组元素有序，并且提示时间复杂度，所以可以想到使用二分法解答。

首先把题目当做二分查找来想，也就是在有序数组中查找某一个元素，查找到就返回元素下表，所以很简单可以写出二分查找算法，如果查不到，那么我们就返回元素待查找元素的插入位置，如果退出循环，说明此事没有查找到元素，而此时`right<left`那么让`right+1`就是元素插入位置，可以思考一下这里是为什么。

另外注意题目描述：

- 排序数组

**代码展示**

~~~ java
class Solution {
    public int searchInsert(int[] nums, int target) {
        // 数组有序，明显使用二分搜索算法
        if(nums == null)
        return -1;
        if(0 == nums.length)
        return -1;
        int left = 0;
        int right=nums.length -1;
        int mid = 0;
        while(left <= right){
            mid=(left+right)/2;
            if(nums[mid] == target){
                return mid;
            }else if(nums[mid] < target){
                left = mid +1;
            }else if(nums[mid] > target){
                right =mid -1;
            }
        }
        // 退出循环，说明没有找到元素
        return right+1;
    }
}
~~~

**小结**

下面给出使用二分法结题的模板。

> ~~~ java
> class Solution {
>     public int searchInsert(int[] nums, int target) {
>         int left = 0, right = nums.length - 1; // 注意
>         while(left <= right) { // 注意
>             int mid = (left + right) / 2; // 注意
>             if(nums[mid] == target) { // 注意
>                 // 相关逻辑
>             } else if(nums[mid] < target) {
>                 left = mid + 1; // 注意
>             } else {
>                 right = mid - 1; // 注意
>             }
>         }
>         // 相关返回值
>         return 0;
>     }
> }
> ~~~
>
> 另外注意题目中的描述，有序数组，无重复，都要想到二分法。

#### [69、 x 的平方根](https://leetcode-cn.com/problems/sqrtx/)

[69、x 的平方根](https://leetcode-cn.com/problems/sqrtx/)

![1631868468352](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1631868468352.png)

**解答**

这一带题目很简单，但是有一点需要注意，就是溢出问题，两个数的乘积已经超过int类型的最大值，所以中间我们要使用long类型。

**代码实现**

~~~ java
class Solution {
    public int mySqrt(int x) {
        if(x == 0 ){
            return 0;
        }
        if(x == 1){
            return 1;
        }
        long nextNum=0;
         for(int i=1;i<=x/2;i++){
            int sqrtNum=i;
            nextNum=(long)(sqrtNum+1)*(sqrtNum+1);
            sqrtNum *= sqrtNum;
            if(sqrtNum <x && nextNum > x){
                return i;
            }else if(sqrtNum == x){
                return i;
            }else 
            {
               continue;
            }
        }
        return -1;
    }
}
~~~



### 中等

#### 34、在排序数组中查找元素的第一个和最后一个位置

[在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

![1631796728054](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/205209-388231.png)

**题解**

根据题目描述，升序排列数组，所以很明显使用二分查找法，但是平常的二分法都是查找一个target，本题目中需要查找两个边界，所以想到两次使用二分查找算法，分别查找到左边界和右边界。

**代码演示**

~~~ java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        // 有序数组，但是可能存在重复元素
       int []res={-1,-1};
        if(nums == null)
            return res;
        if(0 == nums.length)
            return res;
        int left = 0;
        int right=nums.length -1;
        int mid = 0;
        // 首先查找左边界
        while(left <= right){
            mid=(left+right)/2;
            if(nums[mid] <= target){
                if(nums[mid] == target)
                    res[0]=mid;
                left = mid +1;
            }else if(nums[mid] > target){
                right =mid -1;
            }
        }
        left =0;
        right=nums.length-1;
        // 下面开始找右边界
         while(left <= right){
            mid=(left+right)/2;
            if(nums[mid] < target){
                left = mid +1;
            }else if(nums[mid] >= target){
                 if(nums[mid] == target)
                res[1]=mid;
                right =mid -1;
            }
        }
        // 退出循环，说明没有找到元素
        Arrays.sort(res);
        return res;
    }
}
~~~













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

双指针技巧还可以分为两类，一类是「快慢指针」，一类是「左右指针」。前者解决主要解决链表中的问题，比如典型的判定链表中是否包含环；后者主要解决数组（或者字符串）中的问题，比如二分查找。   

- 普通双指针
  - 两个指针向同一个方向移动
- 对撞双指针
  - 两个指针面对面移动
- 快慢双指针
  - 慢指针+快指针

### 快慢指针的常见算法

#### 141

[环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

**题目描述**

![1633999394835](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/12/084318-127316.png)、

用两个指针，一个每次前进两步，一个每次前进一步。如果不含有环，跑得快的那个指针最终会遇到 null，说明链表不含环；如果含有环，快指针最终会超慢指针一圈，和慢指针相遇，说明链表含有环。

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

#### 142

**题目描述**

![1634001226037](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/12/091357-610875.png)

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
    public ListNode detectCycle(ListNode head) {
        if(head == null || head.next == null)
        return null;
       
        ListNode slow,fast;
        slow = fast=head;
        while(fast != null && fast.next != null){
            slow=slow.next;
            fast=fast.next.next;
            if(slow == fast){
                slow = head;
                 while(fast != slow){
                    fast=fast.next;
                    slow=slow.next;
                }
                return slow;
            }
        }
        return null; 
    }
}
~~~

可以看到，当快慢指针相遇时，让其中任一个指针重新指向头节点，然后让它俩以相同速度前进，再次相遇时所在的节点位置就是环开始的位置。这是为什么呢？

第一次相遇时，假设慢指针 slow 走了 k 步，那么快指针 fast 一定走了 2k 步，也就是说比 slow 多走了 k 步（也就是环的长度）

![1634001356771](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/12/091602-243179.png)

设相遇点距环的起点的距离为 m，那么环的起点距头结点 head 的距离为 k - m，也就是说如果从 head 前进 k - m 步就能到达环起点。

巧的是，如果从相遇点继续前进 k - m 步，也恰好到达环起点。

![1634001416588](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/12/091703-3349.png)

所以，只要我们把快慢指针中的任一个重新指向 head，然后两个指针同速前进，k - m 步后就会相遇，相遇之处就是环的起点了。

#### 876

**题目描述**

![1634002258197](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/12/093101-187315.png)

类似上面的思路，我们还可以让快指针一次前进两步，慢指针一次前进一步，当快指针到达链表尽头时，慢指针就处于链表的中间位置。

**代码实现**

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
    public ListNode middleNode(ListNode head) {
        if(head == null){
            return null;
        }
        ListNode slow=head,fast=head;
        while(fast != null && fast.next != null){
            slow=slow.next;
            fast=fast.next.next;
        }
        return slow;

    }
}
~~~

当链表的长度是奇数时，slow 恰巧停在中点位置；如果长度是偶数，slow 最终的位置是中间偏右：

#### 22

**题目描述**

![1634002858196](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/12/094122-947491.png)

**代码实现**

~~~ java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode getKthFromEnd(ListNode head, int k) {
        if(head == null || k<0){
            return null;
        }
        ListNode slow=head,fast=head;
        while(k>0){
            fast=fast.next;
            k--;
        }
        while(fast!= null){
            fast=fast.next;
            slow=slow.next;
        }
        return slow;

    }
}
~~~

我们的思路还是使用快慢指针，让快指针先走 k 步，然后快慢指针开始同速前进。这样当快指针走到链表末尾 null 时，慢指针所在的位置就是倒数第 k 个链表节点（为了简化，假设 k 不会超过链表长度）

### 左右指针的常用算

#### 1

**题目描述**

![1634006702251](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1634006702251.png)

**代码实现**

~~~ java
class Solution {
    public int[] twoSum(int[] nums, int target) {

       int[] res = new int[2];
    if(nums == null || nums.length == 0){
        return res;
    }
    Map<Integer, Integer> map = new HashMap<>();
    for(int i = 0; i < nums.length; i++){
        int temp = target - nums[i];
        if(map.containsKey(temp)){
            res[1] = i;
            res[0] = map.get(temp);
        }
        map.put(nums[i], i);
    }
    return res;
    }
}
~~~

只要数组有序，就应该想到双指针技巧。解法有点类似二分查找，通过调节 left 和 right 可以调整 sum 的大小.

但是这一道题目的特点是数组无序，那么就不能使用简单的左右指针，如果有序，可以使用左右指针。

#### 881

[救生艇](https://leetcode-cn.com/problems/boats-to-save-people/)

**题目描述**

![1634007027410](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/12/105038-356905.png)

**思路**

注意到这一刀题目是求最优解问题，所以我们可以先对数组进行原地排序，然后使用对碰指针解决。

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

### 滑动窗口代码框架

~~~ java

  /* 滑动窗口算法框架 */
    public static void slidingWindow(String s, String t) {
//        定义窗口
        HashMap<Character, Integer> window = new HashMap<>();
//        定义目标字符串
        HashMap<Character, Integer> need = new HashMap<>();
        char[] source = s.toCharArray();
        char[] chars= t.toCharArray();
//        添加字符串到目标map
        for (char c : chars) {
            if(need.containsKey(c)){
                need.put(c,need.getOrDefault(c,0)+1);
            }else {
                need.put(c,1);
            }

        }
//定义左右窗口指针
        int left = 0, right = 0;
//        定义窗口中有效字符变量
        int valid = 0;
        while (right < s.length()) {
            // c 是将移入窗口的字符
            char c = source[right];
            // 右移窗口
            right++;
            // 进行窗口内数据的一系列更新
        ...

            /*** debug 输出的位置 ***/
            System.out.println("left ="+left+"  right ="+right);
            /********************/

            // 判断左侧窗口是否要收缩
            while (window needs shrink) {
                // d 是将移出窗口的字符
                char d = source[left];
                // 左移窗口
                left++;
                // 进行窗口内数据的一系列更新
            ...
            }
        }
    }
~~~

这个算法技巧的时间复杂度是 O(N)，⽐字符串暴⼒算法要⾼效得多。

其中两处...表示的更新窗口数据的地方，到时候你直接往里面填就行了。

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

### 76、Hard

[最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

**题目描述**

![1633915171876](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/091937-717855.png)

#### 思路

就是说要在`S`(source) 中找到包含`T`(target) 中全部字母的一个子串，且这个子串一定是所有可能子串中最短的。其中找到的子串可以不按T串的顺序。

**滑动窗口思路**

1. 我们在字符串`S`中使用双指针中的左右指针技巧，初始化`left = right = 0`，**把索引左闭右开区间[left, right)称为一个「窗口」**。
2. 我们先不断地增加`right`指针扩大窗口`[left, right)`，直到窗口中的字符串符合要求（包含了`T`中的所有字符）。**注意，这是开始收缩左边界的条件**
3. 此时，我们停止增加`right`，转而不断增加`left`指针缩小窗口`[left, right)`，直到窗口中的字符串不再符合要求（不包含`T`中的所有字符了）。同时，每次增加`left`，我们都要更新一轮结果。
4. 重复第 2 和第 3 步，直到`right`到达字符串`S`的尽头。

这个思路其实也不难，**第 2 步相当于在寻找一个「可行解」，然后第 3 步在优化这个「可行解」，最终找到最优解，**也就是最短的覆盖子串。左右指针轮流前进，窗口大小增增减减，窗口不断向右滑动，这就是「滑动窗口」这个名字的来历。

下面画图理解一下，`needs`和`window`相当于计数器，分别记录`T`中字符出现次数和「窗口」中的相应字符的出现次数。

![1633915415873](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/092358-401605.png)

此时window窗口中已经包含了T子串的所有字符，所以开始收缩左边界窗口。

![1633915446901](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/092409-184753.png)

左边界收缩两个字符

![1633915615463](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/092656-358186.png)

![1633915652547](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/092745-373950.png)

之后重复上述过程，先移动`right`，再移动`left`…… 直到`right`指针到达字符串`S`的末端，算法结束。

#### 代码思路

首先，初始化`window`和`need`两个哈希表，记录窗口中的字符和需要凑齐的字符：

```java
//        存放窗口内部的字符
        HashMap<Character, Integer> window = new HashMap<Character, Integer>();
//        存放target字符
        HashMap<Character, Integer> need = new HashMap<Character, Integer>();
```

然后，使用`left`和`right`变量初始化窗口的两端，不要忘了，区间`[left, right)`是左闭右开的，所以初始情况下窗口没有包含任何元素：

```java
int left = 0, right = 0;
int valid = 0; 
while (right < s.size()) {
    // 开始滑动
}
```

**其中valid变量表示窗口中满足need条件的字符个数**，如果`valid`和`need.size`的大小相同，则说明窗口已满足条件，已经完全覆盖了串`T`。

解释一下这里为什么`valid`和`need.size`大小相同的话就表示已经满足条件，因为T中可能含有重复的字符，比如`ABACA`这样的，明显字符A有3个，所以使用HashMap的目的就是为了去重操作。

**现在开始套模板，只需要思考以下四个问题**：

**1、**当移动`right`扩大窗口，即加入字符时，应该更新哪些数据？

**2、**什么条件下，窗口应该暂停扩大，开始移动`left`缩小窗口？

**3、**当移动`left`缩小窗口，即移出字符时，应该更新哪些数据？

**4、**我们要的结果应该在扩大窗口时还是缩小窗口时进行更新？

如果一个字符进入窗口，应该增加`window`计数器；如果一个字符将移出窗口的时候，应该减少`window`计数器；当`valid`满足`need`时应该收缩窗口；应该在收缩窗口的时候更新最终结果。

#### 代码实现

```java
class Solution {
    public String minWindow(String s, String t) {
//        首先把两个字符串转换为字符数组
        char[] s1 = s.toCharArray();
        char[] t1 = t.toCharArray();
//        定义窗口
        HashMap<Character, Integer> window = new HashMap<Character, Integer>();
//        存放target字符
        HashMap<Character, Integer> need = new HashMap<Character, Integer>();
//        其中need中的某一个字符可能不止一个
        for(char c:t1){
            if(need.containsKey(c)){
//                字符串t中肯恩存在重复的字符
                need.put(c,need.getOrDefault(c,0)+1);
            }else {
                need.put(c,1);
            }
        }
        // 记录最小覆盖子串的起始索引及长度
        int start = 0;
        int len = Integer.MAX_VALUE;
//        定义窗口的位置
        int left = 0, right = 0;
//        窗口中有效的字符个数
        int valid = 0;
        while (right <s1.length) {
            // 向右开始滑动窗口
            char c=s1[right];
//            窗口增加一位
            right++;
//            判断t字符串中是否包含字符c
            if (need.containsKey(c)) {
//                把当前字符添加进窗口，然后对应值+1
                window.put(c,window.getOrDefault(c,0)+1);
                if(window.get(c).equals(need.get(c))){
//                    有效字符个数+1
                    valid++;
                }
            }
//            判断左侧的字符是否需要收缩
//            这个是本题的左边界收缩条件，当窗口中有效字符的个数等于need中字符的个数，已经包含重复的字符
            while (valid == need.size()) {
                // 在这里更新最小覆盖子串
                if (right - left < len) {
                    start = left;
                    len = right - left;
                }
                // d 是将移出窗口的字符
                char d = s1[left];
                // 左移窗口
                left++;
                // 进行窗口内数据的一系列更新
                if (need.containsKey(d)) {
                    if (window.get(d).equals(need.get(d)))
                        valid--;
                    window.put(d,window.get(d)-1);
                }
            }
        }
        // 返回最小覆盖子串
        return len == Integer.MAX_VALUE? "" : s.substring(start,start+len);
    }
}
```

需要注意的是，当我们发现某个字符在`window`的数量满足了`need`的需要，就要更新`valid`，表示有一个字符已经满足要求。而且，你能发现，两次对窗口内数据的更新操作是完全对称的。

当`valid == need.size()`时，说明`T`中所有字符已经被覆盖，已经得到一个可行的覆盖子串，现在应该开始收缩窗口了，以便得到「最小覆盖子串」。

移动`left`收缩窗口时，窗口内的字符都是可行解，所以应该在收缩窗口的阶段进行最小覆盖子串的更新，以便从可行解中找到长度最短的最终结果。

### 567、Medium

[字符串排列](https://leetcode-cn.com/problems/permutation-in-string/)

**题目描述**

![1633916523426](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1633916523426.png)

这个题目已经明确说明是找目标串的一个排列，所以与目标串的顺序无关。所以我们找到的子串和Target串的长度应该相同，这就是我们收缩左侧边界的条件。

#### 代码思路

初始状态，让窗口的左右指针全部指向0的位置。

![1633918794447](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/101957-363927.png)

接下来一致增加右侧边界，直到window窗口中包含子串ab为止。

![1633919023927](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/102353-922113.png)

现在已经达到收缩左侧边界的条件，因为window窗口中已经包含了子串ab，也就是有效字符vaild现在为2。但是现在总的子窗口的长度是大于2的，我们要找的是window窗口中vaild=need窗口时候的条件。

现在左侧left边界增加1，判断left处的字符是否包含在need窗口中，如果不包含的话就继续收缩，如果包含，那么就更新vaild有效字符个数和window窗口中的值。

![1633919145173](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/102550-917791.png)

继续收缩左侧边界的值，直到有效字符的个数等于need窗口的大小位置，返回结果。

![1633919303117](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1633919303117.png)



#### 代码实现

**左边界收缩条件**

Target串长度==找到的子串长度的时候。

~~~ java
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        //        首先判断两个串是否是合法的
        if(s1.equals(null) || s2.equals(null)){
            return false;
        }
//        将字符串转换为字符数组
        char[] target = s1.toCharArray();
        char[] source = s2.toCharArray();
        HashMap<Character, Integer> window = new HashMap<>();
        HashMap<Character, Integer> need = new HashMap<>();
        for(char c :target){
            if(need.containsKey(c)){
                need.put(c,need.getOrDefault(c,0)+1);
            }
            else {
                need.put(c,1);
            }
        }
//        定义窗口的左右边界
        int left = 0,right=0;
//        定义窗口中有效字符的个数
        int vaild=0;

        while(right <s2.length()){
//           获取新添加进来的字符
            char newChar=source[right];
//            右侧窗口向右移动一位
            right++;
//            首先判断新的字符是否在need窗口中
            if(need.containsKey(newChar)){
//                把新的字符添加到window窗口
                window.put(newChar,window.getOrDefault(newChar,0)+1);
//                开始判断有效字符
                if(need.get(newChar).equals(window.get(newChar))){
                    vaild++;
                }
            }

//            开始收缩左侧的边界,窗口中有效字符个数等于need窗口的大小
            while(right - left >=s1.length()){
//                返回条件
                if(need.size() == vaild){
                    return true;
                }

//                获取左边界的字符
                char leftChar=source[left];
//                窗口向左边移动
                left++;
//                更新窗口
                if(need.containsKey(leftChar)){
                    if(need.get(leftChar).equals(window.get(leftChar)))
                        vaild--;
                    window.put(leftChar,window.getOrDefault(leftChar,0)-1);
                }
            }
        }
        return false;
    }
}
~~~

对于这道题的解法代码，基本上和最小覆盖子串一模一样，只需要改变两个地方：

**1、**本题移动`left`缩小窗口的时机是窗口大小大于`t.size()`时，因为排列嘛，显然长度应该是一样的。

**2、**当发现`valid == need.size()`时，就说明窗口中就是一个合法的排列，所以立即返回`true`。

至于如何处理窗口的扩大和缩小，和最小覆盖子串完全相同。

### 483

[找到字符串中所有字母异位词](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/)

**题目描述**

![1633919635108](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/103356-724335.png)

#### 思路

什么时候开始收缩左侧的边界？

- 当`right-left >=T.length`的时候开始收缩左侧边界。因为这个时候，才有可能窗口中包含了T串，想象如果`right-left < T.length`的时候，是不是不可能包含T子串。

什么时候找到子串T？

- 当`need.length==vaild`的时候，就找到目标串T了。

#### 代码实现

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        if(s.equals(null) || p.equals(null)){
            return null;
        }

        ArrayList<Integer> list = new ArrayList<>();
        char[] s1 = s.toCharArray();
        char[] t1 = p.toCharArray();
//        存放窗口内部的字符
        HashMap<Character, Integer> window = new HashMap<Character, Integer>();
//        存放target字符
        HashMap<Character, Integer> need = new HashMap<Character, Integer>();

        for(char c:t1){
            if(need.containsKey(c)){
//                字符串t中肯恩存在重复的字符
                need.put(c,need.getOrDefault(c,0)+1);
            }else {
                need.put(c,1);
            }
        }
        // 记录最小覆盖子串的起始索引及长度
        int start = 0, len = Integer.MAX_VALUE;
//        定义窗口的位置
        int left = 0, right = 0;
        int valid = 0;//窗口中有效的字符个数
        while (right <s1.length) {
            // 向右开始滑动窗口
            char c=s1[right];
//            窗口增加一位
            right++;
//            判断t字符串中是否包含字符c
            if (need.containsKey(c)) {
//                把当前字符添加进窗口，然后对应值+1
                window.put(c,window.getOrDefault(c,0)+1);
                if(window.get(c).equals(need.get(c))){
//                    有效字符个数+1
                    valid++;
                }
            }
//            判断左侧的字符是否需要收缩
            while ((right - left)>= p.length()) {

                // 在这里更新最小覆盖子串
                if (valid == need.size()) {
                    list.add(left);
                }
                // d 是将移出窗口的字符
                char d = s1[left];
                // 左移窗口
                left++;
                // 进行窗口内数据的一系列更新
                if (need.containsKey(d)) {
                    if (window.get(d).equals(need.get(d)))
                        valid--;
                   window.put(d,window.get(d)-1);
                }
            }
        }
        // 返回最小覆盖子串
        return list;
    }
}
```





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

### 111

[二叉树的最小深度](https://leetcode-cn.com/problems/minimum-depth-of-binary-tree/)

**BFS**

~~~ java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public int minDepth(TreeNode root) {
        
        // 首先判断根节点是否是空的
        if( root == null){
            return 0;
        }
        // bfs必备工具，队列
        Queue queue=new LinkedList();
        // 开始是树的根节点，所以先将根节点加入队列中,offer()是将元素添加到队列的末尾
        queue.offer(root);
        // 定义树根的高度为1
        int lowHeigh=1;
        // 逐个遍历队列中的节点
        while(!queue.isEmpty()){
            // 返回对垒中元素的个数
            int len=queue.size();
            // 逐个遍历与当前节点相邻的节点
            for(int i=0;i<len;i++){
                // 抛出队列首的元素
                TreeNode node=(TreeNode)queue.poll();
              // 判断当前节点是否是叶子结点
              if(node.left == null && node.right == null){
                  return lowHeigh;
              }
                //  判断当前节点的左边是否有节点
                if(node.left != null){
                    queue.offer(node.left);
                }
                if(node.right != null){
                    queue.offer(node.right);
                }
            }
            lowHeigh ++;
        }
        return lowHeigh;
    }
}
~~~

### 109

[开密码锁](https://leetcode-cn.com/problems/zlDJc7/)

~~~ java
class Solution {
    public int openLock(String[] deadends, String target) {
        // 使用一个哈希表记录不能访问的元素
        Set<String> deads=new HashSet<String>();
        for(String s:deadends){
            deads.add(s);
        }
        // 使用一个访问数组，记录访问过的元素
        Set visited= new HashSet<String>();
        // 辅助队列
        Queue q=new LinkedList<String>();
        // 首先添加初始元素
        q.offer("0000");
        visited.add("0000");
        int step=0;
        while(!q.isEmpty()){
            // 队列的大小代表树中每一层元素的个数
            int len=q.size();
            for(int i=0;i<len;i++){
                // 首先抛出队列首页元素
                String s=(String)q.poll();
                if(deads.contains(s)){
                    continue;
                }
                if(target.equals(s)){
                    return step;
                }
                // 访问当前元素的所有相邻元素
                for(int j=0;j<4;j++){
                    // 向队列中添加元素的时候，首先判断是否访问过这个元素
                    String up=plusOne(s,j);
                    if(!visited.contains(up)){
                        q.offer(up);
                        visited.add(up);
                    }
                
                    String down=subOne(s,j);
                    if(!visited.contains(down)){
                        q.offer(down);
                        visited.add(down);
                    }
                }
            }
            step++;
        }
        return -1;
    }

    public String plusOne(String str,int j){
        char []c=str.toCharArray();
        if(c[j] == '9'){
            c[j]='0';
        }else{
            c[j]+=1;
        }
        return new String(c);
    }

    public String subOne(String str,int j){
        char []c=str.toCharArray();
        if(c[j] == '0'){
            c[j]='9';
        }else{
            c[j]-=1;
        }
        return new String(c);
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



# 刷题笔记

在开始刷算法题目之前，我们先来看看各种类型算法考察的频率：

![1631788406934](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/183328-535505.png)

## 算法复杂度分析

### 什么是时间复杂度

**时间复杂度是一个函数，它定性描述该算法的运行时间**。

在软件开发中，时间复杂度就是用来方便开发者估算出程序运行的答题时间。

那么该如何估计程序运行时间呢，通常会估算算法的操作单元数量来代表程序消耗的时间，这里默认CPU的每个单元运行消耗的时间都是相同的。

假设算法的问题规模为n，那么操作单元数量便用函数f(n)来表示，随着数据规模n的增大，算法执行时间的增长率和f(n)的增长率相同，这称作为算法的渐近时间复杂度，简称时间复杂度，记为 O(f(n))。

### 什么是大O

说到时间复杂度，**大家都知道O(n)，O(n^2)，却说不清什么是大O**。

算法导论给出的解释：**大O用来表示上界的**，当用它作为算法的最坏情况运行时间的**上界**，就是对任意数据输入的运行时间的上界，也可以理解为最坏情况下算法的时间复杂度。

同样算法导论给出了例子：拿插入排序来说，插入排序的时间复杂度我们都说是O(n^2) 。

输入数据的形式对程序运算时间是有很大影响的，在数据本来有序的情况下时间复杂度是O(n)，但如果数据是逆序的话，插入排序的时间复杂度就是O(n^2)，也就对于所有输入情况来说，最坏是O(n^2) 的时间复杂度，所以称插入排序的时间复杂度为O(n^2)。

同样的同理再看一下快速排序，都知道快速排序是O(nlogn)，但是当数据已经有序情况下，快速排序的时间复杂度是O(n^2) 的，**所以严格从大O的定义来讲，快速排序的时间复杂度应该是O(n^2)**。

**但是我们依然说快速排序是O(nlogn)的时间复杂度，这个就是业内的一个默认规定，这里说的O代表的就是一般情况，而不是严格的上界**。如图所示：

![1631789680730](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/185442-259319.png)

> **面试中说道算法的时间复杂度是多少指的都是一般情况**，也可以说是平均时间复杂度。

### 不同数据规模的差异

如下图中可以看出不同算法的时间复杂度在不同数据输入规模下的差异。

![1631789815767](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/185657-517642.png)

在决定使用哪些算法的时候，不是时间复杂越低的越好（因为简化后的时间复杂度忽略了常数项等等），要考虑数据规模，如果数据规模很小甚至可以用O(n^2)的算法比O(n)的更合适（在有常数项的时候）。

就像上图中 O(5n^2) 和 O(100n) 在n为20之前 很明显 O(5n^2)是更优的，所花费的时间也是最少的。

那为什么在计算时间复杂度的时候要忽略常数项系数呢，也就说O(100n) 就是O(n)的时间复杂度，O(5n^2) 就是O(n^2)的时间复杂度，而且要默认O(n) 优于O(n^2) 呢 ？

这里就又涉及到大O的定义，**因为大O就是数据量级突破一个点且数据量级非常大的情况下所表现出的时间复杂度，这个数据量也就是常数项系数已经不起决定性作用的数据量**。

例如上图中20就是那个点，n只要大于20 常数项系数已经不起决定性作用了。

**所以我们说的时间复杂度都是省略常数项系数的，是因为一般情况下都是默认数据规模足够的大，基于这样的事实，给出的算法时间复杂的的一个排行如下所示**：

O(1)常数阶 < O(logn)对数阶 < O(n)线性阶 < O(n^2)平方阶 < O(n^3)(立方阶) < O(2^n) (指数阶)

但是也要注意大常数，如果这个常数非常大，例如10^7 ，10^9 ，那么常数就是不得不考虑的因素了。

### 复杂表达式的化简

有时候我们去计算时间复杂度的时候发现不是一个简单的O(n) 或者O(n^2)， 而是一个复杂的表达式，例如：

~~~ java
O(2*n^2 + 10*n + 1000)
~~~

那这里如何描述这个算法的时间复杂度呢，一种方法就是简化法。

去掉运行时间中的加法常数项 （因为常数项并不会因为n的增大而增加计算机的操作次数）。

~~~java
O(2*n^2 + 10*n)
~~~

去掉常数系数（上文中已经详细讲过为什么可以去掉常数项的原因）。

~~~ java
O(n^2 + n)
~~~

只保留保留最高项，去掉数量级小一级的n （因为n^2 的数据规模远大于n），最终简化为：

~~~ java
O(n^2)
~~~

所以最后我们说：这个算法的算法时间复杂度是O(n^2) 。