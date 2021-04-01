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

