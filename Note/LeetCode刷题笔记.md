# LeetCode刷题笔记

[TOC]

## 1，两数之和

[两数之和](https://leetcode-cn.com/problems/two-sum/)

**方法一**

暴力法

~~~ java
class Solution {
    public int[] twoSum(int[] nums, int target) {

        int []arr=new int[2];

        // 使用暴力法
        // 首先判断数组是否是合法的
        if(nums == null || nums.length == 0){
            return null;
        }

        for(int i=0;i< nums.length;i++){
            for(int j=i+1;j<nums.length;j++){
                if(nums[i]+nums[j] == target){
                    arr[0]=i;
                    arr[1]=j;
                }
            }
        }
        return arr;

    }
}
//时间复杂度是o(n2)
~~~

**方法二**

使用哈希表，键值存储数组元素的值，值存储数组元素的下表，遍历数组，使用target值减去每遍历的一个值，然后判断结果是否在hashmap中，如果在的话返回两个元素的下表即可。

~~~ java
class Solution {
    public int[] twoSum(int[] nums, int target) {

    // N is the size of nums
    // Time Complexity: O(N)
    // Space COmplexity: O(N)
        int[] result = new int[2];
        // hashmap:如果key相同的话，会做覆盖操作
        HashMap<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            map.put(nums[i], i);
        }
        for (int j = 0; j < nums.length; j++) {
            int diff = target - nums[j];
            if (map.containsKey(diff) && map.get(diff) != j) {
                result[0] = j;
                result[1] = map.get(diff);
                return result;
            }
        }
        return result;

    }
}
~~~

## 2，两数相加

[两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

**迭代法**

思路和合并两个链表一样

~~~ java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {

        // 使用迭代法
        int nextInt=0;
        int totle=0;
        // 第一个是头节点
        ListNode res=new ListNode(0);
        ListNode curr=res;
        while(l1 != null && l2 != null){
            totle=l1.val+l2.val+nextInt;
            curr.next=new ListNode(totle % 10);
            nextInt=totle /10;
            l1=l1.next;
            l2=l2.next;
            curr=curr.next;
        }

        while(l1 != null){
            totle = l1.val+nextInt;
            curr.next=new ListNode(totle % 10);
            nextInt=totle/10;
            l1=l1.next;
            curr=curr.next;
        }

        while(l2 != null){
            totle = l2.val+nextInt;
            curr.next=new ListNode(totle % 10);
            nextInt=totle/10;
            l2=l2.next;
            curr=curr.next;
        }
        if(nextInt != 0){
            curr.next=new ListNode(nextInt);
        }

        return res.next;

    }
}
~~~

**使用递归的思路**

~~~ java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {


        // 使用递归方法
        int total=l1.val+l2.val;
        int nextInt=total/10;
        // res是新创建的节点
        ListNode res=new ListNode(total % 10);
        if(l1.next != null || l2.next != null || nextInt != 0){
            if(l1.next != null){
                l1=l1.next;
            }else{
                l1=new ListNode(0);
            }

            if(l2.next != null){
                l2=l2.next;
            }else{
                l2=new ListNode(0);
            }
            l1.val=l1.val+nextInt;
            res.next=addTwoNumbers(l1,l2);
        }
        return res;

    }
}
~~~

## 20，有效的括号

[有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

**解法一**

使用栈数据结构

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
                stack.push(chars[i]);
               // return false;
            }
        }
    }
    return stack.isEmpty()?true:false;

    }
}
~~~

## 21，合并两个有序链表

[合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

**链表递归矿建**

~~~ java
//所有对链表的操作，都可以在递归框架的基础上进行改进
public static void show(ListNode node){
//        出口出来后，返回函数调用位置
        if(node == null){
            return ;
        }

//        顺序打印就在递归前面输出结果
        System.out.println(node.data);
        show(node.next);
//        倒序输出就在递归遍历后面打印结果,也就是逆序打印结果
//        System.out.println(node.data);
    }

//数组的递归遍历框架
public static void printArray(List list,int index){
        
        if(index == list.size()){
            return ;
        }
  //在这里输出表示顺序输出
        System.out.println(list.get(index));
        printArray(list,index+1);
  //在这里打印表示逆序输出
   			System.out.println(list.get(index));
    }
~~~

**使用迭代法**

~~~ java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {

        // 使用迭代法解决
        ListNode node=new ListNode(0);
        ListNode curr=node;
        while(l1!= null && l2!= null){
            if(l1.val > l2.val){
                curr.next=l2;
                l2=l2.next;
                curr=curr.next;
            }else if(l1.val <= l2.val){
                curr.next=l1;
                l1=l1.next;
                curr=curr.next;
            }
        }

        if(l1!=null){
            curr.next=l1;
        }
        if(l2!= null){
            curr.next=l2;
        }
        return node.next;

    }
}
~~~

**使用递归解法**

~~~ java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {


        // 使用递归法
        if(l1 == null || l2 == null){
            return l1 == null? l2:l1;
        }
        if(l1.val <= l2.val){
            l1.next= mergeTwoLists(l1.next,l2);
            return l1;
        }else{
            l2.next=mergeTwoLists(l1,l2.next);
            return l2;
        }


    }
}
~~~

## 22，括号生成

[括号生成](https://leetcode-cn.com/problems/generate-parentheses/)

**回溯法**

~~~ java
class Solution {
    public List<String> generateParenthesis(int n) {

        List <String>res=new ArrayList();
        backtracking(n,res,0,0,"");
        return res;

    }
    private void backtracking(int n ,List <String>res,int left,int right,String str){
        if(right>left){
            return;
        }
        if(left == n && right == n){
            res.add(str);
            return;
        }
        if(left < n){
            backtracking(n,res,left+1,right,str+"(");
        }
        if(right < left){
            backtracking(n,res,left,right+1,str+')');
        }
    }
}
~~~

## 24，两两交换链表中的节点

[两两交换链表中的节点](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)

**迭代法**

~~~ java
class Solution {
    public ListNode swapPairs(ListNode head) {

        // 使用迭代法
        if(head == null || head.next == null){
            return head;
        }
        ListNode res=new ListNode(0);
        res.next=head;
        ListNode cur=res;
        while(cur.next != null && cur.next.next != null){
            ListNode next=head.next;
            ListNode temp=head.next.next;
            cur.next=next;
            next.next=head;
            head.next=temp;
            cur=head;
            head=head.next;
        }
        return res.next;
        
    }
}
~~~

**递归法**

~~~ java
class Solution {
    public ListNode swapPairs(ListNode head) {

       
        // 使用迭代法
        if(head == null || head.next==null){
            return head;
        }
        ListNode next=head.next;
        head.next=swapPairs(head.next.next);
        next.next=head;
        return next;
        
    }
}
~~~

## 27，移除元素

[移除元素](https://leetcode-cn.com/problems/remove-element/)

使用双指针法

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
    // 判断left或者right当前指的值是否等于val
    if(nums[left] == val)
    return left;
    else
    return left+1;

    }
}
~~~

## 35，搜索插入位置

[搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/)

使用二分插入法

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

## 49，字母异位词分组

[字母异位词分组](https://leetcode-cn.com/problems/group-anagrams/)

**使用排序的方法**

~~~ java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {

        // 使用排序的方法
        Map<String,ArrayList> map=new HashMap();
    //遍历字符串数组 
        for(String s:strs){
            String temp=s;
            // 先把字符串转换为字符数组
            char []arr=s.toCharArray();
            // 然后对字符数组进行排序操作
            Arrays.sort(arr);
            String key=new String(arr);
            // 判断map中是否有这个字符串
            if(!map.containsKey(key)){
                map.put(key,new ArrayList());
            }
            map.get(key).add(s);
        }
        return new ArrayList(map.values());

    }
}
//时间复杂度：o(nklogk)
//n表示对字符串数组进行遍历的复杂度，klogk表示排序的复杂度
~~~

## 46，全排列

[全排列](https://leetcode-cn.com/problems/permutations/)

回溯法

~~~ java
public  static List<List<Integer>> permute(int[] nums) {

        // 使用回溯法
        Map<Integer,Boolean> visit=new HashMap();
        for(int i=0;i<nums.length;i++){
            visit.put(nums[i],false);
        }
        List<List<Integer>> res=new ArrayList();
        ArrayList<Integer> temp=new ArrayList();
        backTracking(nums,res,visit,temp);

        return res;
    }
    private  static void backTracking(int []nums,List<List<Integer>>res,Map visit,List temp){
        // 首先判断temp数组的长度是否为nums数组的长度
        if(temp.size() == nums.length){
            // 添加temp列表到最终结果中
            res.add(new ArrayList(temp));
            return;
        }

        for(int i=0;i<nums.length;i++){
            if((Boolean) visit.get(nums[i]) == false)
            {
                temp.add(nums[i]);
                visit.put(nums[i],true);
                backTracking(nums,res,visit,temp);
                temp.remove(temp.size()-1);
                visit.put(nums[i],false);
            }
        }

    }
~~~

## 53，最大子序列和

[最大子序列和](https://leetcode-cn.com/problems/maximum-subarray/)

**暴力法**

~~~ java
class Solution {
    public int maxSubArray(int[] nums) {

        // 使用暴力法解决
        if(nums.length == 1 && nums[0] < 0){
            return nums[0];
        }

        int res=Integer.MIN_VALUE;
        for(int i=0;i< nums.length;i++){
            int sum=0;
            for(int j=i;j<nums.length;j++){
                sum+=nums[j];
                res=(res>sum?res:sum);
            }
        }
        return res;
    }
}
~~~

**动态规划**

~~~ java
class Solution {
    public int maxSubArray(int[] nums) {

        // 使用动态规划
        int dp[]=new int[nums.length];
        for(int j=0;j<dp.length;j++){
            dp[j]=nums[0];
        }
        int res=nums[0];
        for(int i=1;i<nums.length;i++){
            dp[i]=(Math.max(dp[i-1]+nums[i],nums[i]));
            res=Math.max(res,dp[i]);
        }
        return res;
    }
}
~~~

## 56，合并区间

[合并区间](https://leetcode-cn.com/problems/merge-intervals/)

使用排序方法

~~~ java
class Solution {
    public int[][] merge(int[][] intervals) {

        ArrayList<int []>res=new ArrayList();

        if(intervals.length == 0 || intervals == null){
            return intervals;
        }
        // 对二维数组进行排序,从小到大排序
        Arrays.sort(intervals,(a,b)->a[0]-b[0]);

        for(int [] temp:intervals){
            if(res.size()== 0|| temp[0]> res.get(res.size()-1)[1] ){
                // 添加数组到集合中
                res.add(temp);
            }else{
                // 否则就进行区间合并
                res.get(res.size()-1)[1]=Math.max(temp[1],res.get(res.size()-1)[1]);
            }
        }
        // 把集合转化为数组
        // toArray方法返回集合res中的所有元素
        return res.toArray(new int[res.size()][2]);

    }
}
~~~

## 57，插入区间

[插入区间](https://leetcode-cn.com/problems/insert-interval/)

和合并区间思路一致

```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {

        ArrayList<int[]>res=new ArrayList();
        if(newInterval.length == 0 || newInterval == null){
            return intervals;
        }

// 题目中给出的区间已经有序，所以不需要进行排序操作
        int [][]arr=new int[intervals.length+1][2];
        for(int i=0;i<intervals.length;i++){
            for(int j=0;j<2;j++){
                arr[i][j]=intervals[i][j];
            }
        }
        arr[arr.length-1][0]=newInterval[0];
        arr[arr.length-1][1]=newInterval[1];

        Arrays.sort(arr,(a,b)->a[0]-b[0]);

        for(int []temp:arr){
            if(res.size() == 0 || temp[0] > res.get(res.size()-1)[1]){
                // 直接添加区间到集合中
                res.add(temp);
            }else{
                // 进行区间合并操作
                 res.get(res.size()-1)[1]=Math.max(temp[1],res.get(res.size()-1)[1]);
            }

        }
          // 把集合转化为数组
        // toArray方法返回集合res中的所有元素
        return res.toArray(new int[res.size()][2]);

    }
}
```

## 77 ，组合

[组合](https://leetcode-cn.com/problems/combinations/)

~~~ java
class Solution {
    public List<List<Integer>> combine(int n, int k) {

        // 回溯法

        List<List<Integer>>res=new ArrayList();
        ArrayList<Integer>temp=new ArrayList();

         backTracking(res,1,n, k,temp);
        
        return res;

    }

    private void backTracking(List<List<Integer>> res,int index,int n,int k,ArrayList temp){

        // 首先找到递归出口条件
        if(temp.size() == k){
            // 把temp集合添加到结果集中，然后返回
            res.add(new ArrayList(temp));
            return;
        }
        // 递归操作
        for(int i=index;i<=n;i++){
            temp.add(i);
          backTracking(res,i+1,n,k,temp);
        //   回溯操作
            temp.remove(temp.size()-1);
        }

    }
}
~~~

## 78，子集

[子集](https://leetcode-cn.com/problems/subsets/submissions/)

**回溯法**

~~~ java
public class Test78 {

    public static void main(String[] args) {
        int nums[] = {1,2,3};
        // 使用回溯法解决
        List<List<Integer>> res=new ArrayList();
        // 首先添加一个空的集合到上面的集合中，空集也算是一个子集
        res.add(new ArrayList());
        // 循环遍历每一个元素，进行回溯,
        //i代表子集中元素的个数，因为空集就是0个，所以从i=1开始遍历，表示子集中有一个元素
        for(int i=1;i<=nums.length;i++){
//            这里使用for循环是为了控制子集的长度
            backTracking(nums,res,i,0,new ArrayList());
        }

        for (List<Integer> list : res) {
            Iterator<Integer> iterator = list.iterator();
            while (iterator.hasNext()) {
                System.out.print(iterator.next() + " ");
            }
            System.out.println();
        }
    }

    /**
     * 使用回溯法
     * @param nums 数组
     * @param list 结果集列表
     * @param len 代表子集的长度
     * @param index 每一次开始回溯的位置
     * @param subList 临时子集列表
     */
    private static void backTracking(int []nums,List<List<Integer>> list,int len,int index,List<Integer> subList){

        // 递归结束条件,如果当前子集的长度等于len，那么就添加到list集合中，并且返回
        if(subList.size() == len){
            list.add(new ArrayList<>(subList));
            return ;
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

