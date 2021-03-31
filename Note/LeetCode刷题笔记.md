# LeetCode刷题笔记

## 两数之和

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

## 两数相加

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

