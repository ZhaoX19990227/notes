

#### [1. 两数之和](https://leetcode.cn/problems/two-sum/)

给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** *`target`* 的那 **两个** 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

你可以按任意顺序返回答案。

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
    int[] res = new int[2];
    if(nums == null || nums.length == 0){
        return res;
    }
    Map<Integer, Integer> map = new HashMap<>();
    for(int i = 0; i < nums.length; i++){
        //查一下有没有剩下的一个元素与当前元素相加等于target
        int temp = target - nums[i];
        //如果map中有剩下的一个元素
        if(map.containsKey(temp)){
            //如果有那就放入结果集
            res[1] = i;
            res[0] = map.get(temp);
        }
        //如果map中不存在就把当前数作为key，i作为value放入map
        map.put(nums[i], i);
    }
    return res;
    }
}
```

#### [2. 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

给你两个 **非空** 的链表，表	示两个非负的整数。它们每位数字都是按照 **逆序** 的方式存储的，并且每个节点只能存储 **一位** 数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

你可以假设除了数字 0 之外，这两个数都不会以 0 开头。

```java
class Solution {
  public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode head  = new ListNode(0);
    ListNode curr = head;
    int carry = 0;
    int sum;
    while(l1 != null || l2 != null) {
      int x = (l1 == null) ? 0:l1.val;
      int y = (l2 == null) ? 0:l2.val;
      sum = x + y + carry;
      carry = sum / 10;
      curr.next = new ListNode(sum % 10);
      curr = curr.next;
      if(l1 != null) l1 = l1.next;
      if(l2 != null) l2 = l2.next;
    }
    //走到最后carry还是不等于0，就往前新增一个节点
    if(carry != 0) curr.next = new ListNode(carry);
    return head.next;
  }
}
```

#### [3. 无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长子串** 的长度。

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        //定义两个滑动指针 表示滑动窗口的左右边界
        int left = 0;
        int right = 0;
        //默认长度为1
        int maxLength = 1;
        if(s.length() == 0) return 0;
        //set集合用来存放字符
        Set<Character> window = new HashSet<>();
        while(right < s.length()) {
            //取出右边界的字符
            char newRight = s.charAt(right);
            //如果集合中已经存在了，则删除左边界的字符，左边界右移
            while(window.contains(newRight)) {
                window.remove(s.charAt(left));
                left++;
            }
            //如果不存在，则放入滑动窗口中，然后重新计算窗口长度，右边界右移
            window.add(newRight);
            maxLength = Math.max(maxLength,right-left+1);
            right++;
        }
        return maxLength;
    }
}
```

#### [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)

给你一个字符串 `s`，找到 `s` 中最长的回文子串。

```java
class Solution {
    public String longestPalindrome(String s) {
        if (s == null || s.length() == 0) {
            return "";
        }
//         保存起始位置，测试了用数组似乎能比全局变量稍快一点
        int[] range = new int[2];
        char[] str = s.toCharArray();
        for (int i = 0; i < s.length(); i++) {
//             把回文看成中间的部分全是同一字符，左右部分相对称
//             找到下一个与当前字符不同的字符
            i = findLongest(str, i, range);
        }
        return s.substring(range[0], range[1] + 1);
    }
    
    public static int findLongest(char[] str, int low, int[] range) {
//         查找中间部分
        int high = low;
        while (high < str.length - 1 && str[high + 1] == str[low]) {
            high++;
        }
//         定位中间部分的最后一个字符
        int ans = high;
//         从中间向左右扩散
        while (low > 0 && high < str.length - 1 && str[low - 1] == str[high + 1]) {
            low--;
            high++;
        }
//         记录最大长度
        if (high - low > range[1] - range[0]) {
            range[0] = low;
            range[1] = high;
        }
        return ans;
    }
}
```

#### [9. 回文数](https://leetcode.cn/problems/palindrome-number/)

给你一个整数 x ，如果 x 是一个回文整数，返回 true ；否则，返回 false 。

回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

例如，121 是回文，而 123 不是。

```java
 class Solution {
    public boolean isPalindrome(int x) {
        if (x == 0) return true;
        if (x < 0 || x % 10 == 0) return false;
// 记录 x 后一半的翻转，如 x = 4334，reversed = 43；然后和x的前半部分进行比较是否相等
      /*
      x = 4334 reversed = 0
      reversed = 4,x = 433
      reversed = 43,x = 43
      
      x = 121 reversed = 0
      reversed = 1, x = 12
      reversed = 12, x = 1
      */
        int reversed = 0;
        while (x > reversed) {
            reversed = reversed * 10 + x % 10;
            x /= 10;
        }
// x有偶数个位数和奇数个位数两种情况
        return x == reversed || x == reversed / 10;
    }
}
```

#### [15. 三数之和](https://leetcode-cn.com/problems/3sum/)

给你一个包含 `n` 个整数的数组 `nums`，判断 `nums` 中是否存在三个元素 *a，b，c ，*使得 *a + b + c =* 0 ？请你找出所有和为 `0` 且不重复的三元组。

**注意：**答案中不可以包含重复的三元组。

```java
class Solution {
  public List<List<Integer>> threeSum(int[] nums) {
    if(nums == null||nums.length < 3) return new ArrayList<>();
    //先排序
    Arrays.sort(nums);
    List<List<Integer>> res = new ArrayList<>();

    for(int i = 0;i < nums.length - 2;i++) {
      if (i > 0 && nums[i] == nums[i - 1]) continue;
      //找到相反数
      int target = -nums[i];
      //定义两个指针，目的是和target之和为0
      int left = i + 1;
      int right = nums.length - 1;
      while(left < right){
        int sum = nums[left] + nums[right];
        if(sum == target) {
          //如果已经找了和target之和为0的两个数，则将nums[i]所在的值和left和right转为链表加入res中
          res.add(Arrays.asList(nums[i],nums[left],nums[right]));
          //去重 如果遇到相同的元素直接越过
          while(left < right && nums[left] == nums[++left]);
          while(left < right && nums[right] == nums[--right]);
        }else if(sum < target) {
          left++;
        }else
          right--;
      }
    }
    return res;
  }
}
```

#### [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/)

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。

```java
class Solution {
    public boolean isValid(String s) {
        Stack<Character> stack = new Stack();
        for(int i = 0; i < s.length(); i++){
            char c = s.charAt(i);
            if('(' == c){
                stack.push(')');
            } else if ('{' == c){
                stack.push('}');
            } else if ('[' == c){
                stack.push(']');
                    //查看此堆栈顶部的对象而不将其从堆栈中移除。
            } else if (stack.isEmpty() || c != stack.peek()){
                return false;
            } else{
                //如果不是所有匹配的括号且栈不为空且不是堆顶的元素，就弹栈
                stack.pop();
            }
        }
        return stack.isEmpty();
    }
}
```

#### [21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。

```java
class Solution {
    public ListNode mergeTwoLists (ListNode list1, ListNode list2) {
        if(list1 == null) return list2;
        if(list2 == null) return list1;

        if(list1.val < list2.val){
            list1.next = mergeTwoLists(list1.next,list2);
            return list1;
        }else{
            list2.next = mergeTwoLists(list1,list2.next);
            return list2;
        }
    }
}
```

#### [25. K 个一组翻转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

给你链表的头节点 `head` ，每 `k` 个节点一组进行翻转，请你返回修改后的链表。

`k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点保持原有顺序。

你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。

```java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        if(head == null||head.next == null) return head;
        //例如 1 2 3 4 5   k = 3  反转后就是 3 2 1 | 4 5   分成两个链表
        //两个指针指向第一个链表的头节点，tail是可以滑动的
        ListNode firstHead = head;
        ListNode firstTail = head;
        //每k个节点反转，那么就是tail要右移k - 1次，直到尾节点的位置 
        //这期间，有可能出现空指针异常，表明没有k-1个，则不反转
        for(int i = 0;i < k - 1;i++) {
            firstTail = firstTail.next;
            if(firstTail == null) return firstHead;
        }
        //第二个链表的头节点就是第一个链表的尾节点的下一个节点
        ListNode secondHead = firstTail.next;
        //然后将两个链表断开
        firstTail.next = null;
        //对第一个链表进行反转
        reverse(firstHead);
        //反转后和第二个反转后的链表相连
        firstHead.next = reverseKGroup(secondHead,k);
        //此时第一个链表的头节点已经成了第一个链表的尾节点，所以返回tail即可
        return firstTail;
    }
    private ListNode reverse(ListNode head){
        if(head == null || head.next == null) return head;
        //获取头节点的下一个节点
        ListNode next = head.next;
        //递归获取后面的节点
        ListNode newHead = reverse(next);
        //将满足要求的节点和节点反转
        next.next = head;
        //此时头节点已经到了最后，设置下一个节点为空
        head.next = null;
        return newHead;
    }
}
```

#### [53. 最大子数组和](https://leetcode-cn.com/problems/maximum-subarray/)

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

```java
class Solution {
    public int maxSubArray(int[] nums) {
//动态规划思想：记录第一个下标对应的值，从第二位开始遍历，每次将所在的值和前面数组最大的值进行比较
        int index = nums[0];
        int max = nums[0];
        for(int i = 1;i < nums.length;i++) {
            index = Math.max(index + nums[i],nums[i]);
            max = Math.max(index,max);
        }
        return max;
    }
}
```

#### [102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)

给你二叉树的根节点 `root` ，返回其节点值的 **层序遍历** 。 （即逐层地，从左到右访问所有节点）。

![image-20220627174216696](/Users/zhaoxiang/Desktop/肥崽LeetCode记录.assets/image-20220627174216696.png)

```java
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
  public List<List<Integer>> levelOrder(TreeNode root) {
      if(root == null) return new ArrayList<>();
    //ArrayList可以动态扩容
      List<List<Integer>> res = new ArrayList<>();
    //因为涉及删除节点，放到list中，所以使用queue，先进先出。同样deque继承了queue，LinkedList实现了Deque。
      Queue<TreeNode> queue = new LinkedList<TreeNode>();

      queue.add(root);
      while(!queue.isEmpty()) {
          int count = queue.size();
          List<Integer> list = new ArrayList<Integer>();
          while(count > 0) {
            //从队列中移除第一个节点，返回被删除的节点
              TreeNode node = queue.poll();
            //加入到集合中，最后加入到res中
              list.add(node.val);
              if(node.left!=null) queue.add(node.left);
              if(node.right!=null) queue.add(node.right);
            //queue不能动态扩容，所以在上面的while要注意count必须>0
              count--;
          }
          res.add(list);
      }
        return res;
   }
}
```

#### [103. 二叉树的锯齿形层序遍历](https://leetcode.cn/problems/binary-tree-zigzag-level-order-traversal/)

给你二叉树的根节点 `root` ，返回其节点值的 **锯齿形层序遍历** 。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）。

```java
// 解法一：
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        ArrayList<List<Integer>> res = new ArrayList<>();
        LinkedList<TreeNode> queue = new LinkedList<>();
        if (root != null) {
            queue.add(root);
        }
        while (!queue.isEmpty()) {
            ArrayList<Integer> temp = new ArrayList<>();
            int size = queue.size();
            while (size > 0) {
                TreeNode node = queue.poll();
                temp.add(node.val);
                if (node.left != null) {
                    queue.add(node.left);
                }
                if (node.right != null) {
                    queue.add(node.right);
                }
                size--;
            }
            if (res.size() % 2 != 0) {
                Collections.reverse(temp);
            }
            res.add(new ArrayList<>(temp));
        }
        return res;
    }
}

// 解法二
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        Queue<TreeNode> queue = new ArrayDeque<>();
        List<List<Integer>> res = new ArrayList<>();
        if(root == null) 	return res;
        int level = 0;    //奇数层从左往右，偶数层从右往左
        queue.add(root);
        while(!queue.isEmpty()){
            int count = queue.size();  //记录当前层的节点数
            level++;
            List<Integer> list = new ArrayList<>();
            List<Integer> temp = new ArrayList<>();
            for(int i = 0;i < count;i++){
                TreeNode cur = queue.poll();
                temp.add(cur.val);
                if(cur.left != null)queue.add(cur.left);
                if(cur.right != null)queue.add(cur.right);
            }
            if(level % 2 == 0){
                for(int i = temp.size() - 1;i >= 0;i--){
                    list.add(temp.get(i));
                }
            }else{
                list = temp;
            }
            res.add(list);
        }
        return res;
    }
}
```



#### [121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)

给定一个数组 prices ，它的第 i 个元素 prices[i] 表示一支给定股票第 i 天的价格。

你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 0 。

```java
class Solution {
    public int maxProfit(int[] prices) {
        //DP动态规划 前i天的最大收益 = max{前i-1天的最大收益，第i天的价格-前i-1天中的最小价格}

        //1.记录【今天之前买入的最小值】
        //2.计算【今天之前最小值买入，今天卖出的获利】，也即【今天卖出的最大获利】
        //3.比较【每天的最大获利】，取最大值即可
       if(prices.length <= 1) return 0;
       int max = 0;
       int min = prices[0];
       for(int i = 1;i < prices.length;i++) {
           max = Math.max(prices[i] - min,max);
           min = Math.min(min,prices[i]);
       }
       return max;
    }
}
```

#### [141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/)

给你一个链表的头节点 head ，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。注意：pos 不作为参数进行传递 。仅仅是为了标识链表的实际情况。

如果链表中存在环 ，则返回 true 。 否则，返回 false。

可以使用快慢指针法， 分别定义 fast 和 slow指针，从头结点出发，fast指针每次移动两个节点，slow指针每次移动一个节点，如果 fast 和 slow指针在途中相遇 ，说明这个链表有环。

为什么fast 走两个节点，slow走一个节点，有环的话，一定会在环内相遇呢，而不是永远的错开呢？

首先第一点： fast指针一定先进入环中，如果fast 指针和slow指针相遇的话，一定是在环中相遇。

因为fast是走两步，slow是走一步，其实相对于slow来说，fast是一个节点一个节点的靠近slow的，所以fast一定可以和slow重合。

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
      ListNode fast = head;
			ListNode slow = head;
		// 空链表、单节点链表一定不会有环
		while (fast != null && fast.next != null) {
			fast = fast.next.next; // 快指针，一次移动两步
			slow = slow.next;      // 慢指针，一次移动一步
			if (fast == slow) {   // 快慢指针相遇，表明有环
				return true;
			}
		}
		return false; // 正常走到链表末尾，表明没有环
    }
}
```

#### [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)

给定一个链表的头节点  head ，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。

如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。如果 pos 是 -1，则在该链表中没有环。注意：pos 不作为参数进行传递，仅仅是为了标识链表的实际情况。不允许修改 链表。

```java
- 如果存在环，那么快慢指针会在环内相遇，但是相遇的点，不一定是环的起点
- 将快指针放回head，快慢指针以相同的步进移动，最终，会在环的入口相遇
  public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode fast = head;
        ListNode slow = head;
        while (fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;
            if (fast == slow) {
                fast = head;
                while (fast != slow) {
                    fast = fast.next;
                    slow = slow.next;
                }
                return slow;
            }
        }
        return null;
    }
}
```



####  [146. LRU 缓存 ](https://leetcode-cn.com/problems/lru-cache/) 使用HashMap

请你设计并实现一个满足 [LRU (最近最少使用) 缓存](https://baike.baidu.com/item/LRU) 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity`，则应该 **逐出** 最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

![image-20220627174107181](/Users/zhaoxiang/Desktop/java系统笔记/肥崽LeetCode记录.assets/image-20220627174107181.png)



```java
package leetcode.editor.cn;

import java.util.HashMap;

//leetcode submit region begin(Prohibit modification and deletion)
class LRUCache {
    class Node {
        Node prev;
        Node next;
        int key;
        int value;

        Node(int k, int v) {
            prev = this;
            next = this;
            this.key = k;
            this.value = v;
        }
    }
    HashMap<Integer, Node> map = new HashMap<>();
    int capacity;
    Node head;
    public LRUCache(int capacity) {
        head = new Node(-1, -1);
        this.capacity = capacity;
    }
    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        //如果存在，就更新位置到对头，返回值
        Node node = map.get(key);
        deleteAndAddFirst(node);
        return node.value;
    }
    public void put(int key, int value) {
        //如果存在，则根据key获取到对应的节点，修改他的value并删除该节点移动到头节点
        if (map.containsKey(key)) {
            Node node = map.get(key);
            node.value = value;
            deleteAndAddFirst(node);
        } else {
            //如果不存在 则放入 删除并放到头节点 判断长度 如果大于最大容量 则删除最后一个元素
            Node newNode = new Node(key, value);
            map.put(key, newNode);
            deleteAndAddFirst(newNode);
            if (map.size() > capacity) {
                deleteLast();
            }
        }
    }
    //移除双向链表中的节点并放到头节点
    private void deleteAndAddFirst(Node node) {
        node.next.prev = node.prev;
        node.prev.next = node.next;

        node.next = head.next;
        head.next.prev = node;
        head.next = node;
        node.prev = head;

    }
    //删除最后一个元素
    private void deleteLast() {
        //首先获取最后一个元素
        Node lastNode = head.prev;
        //从双向链表中删除最后一个元素
        lastNode.next.prev = lastNode.prev;
        lastNode.prev.next = lastNode.next;
        //从hashmap中移出key
        map.remove(lastNode.key);
    }
}
```

#### [146. LRU 缓存](https://leetcode-cn.com/problems/lru-cache/) 使用LinkedHashMap

请你设计并实现一个满足 [LRU (最近最少使用) 缓存](https://baike.baidu.com/item/LRU) 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity`，则应该 **逐出** 最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。 

```java
class LRUCache {
    LinkedHashMap<Integer,Integer> map;
    int capacity;
    public LRUCache(int capacity) {
        map = new LinkedHashMap<>(capacity + 1,0.75f,true);
        this.capacity = capacity;
    }

    public int get(int key) {
        if(!map.containsKey(key)) return -1;
        return map.get(key);
    }

    public void put(int key, int value) {
        map.put(key,value);
        if(map.size() > capacity) {
            Iterator it = map.keySet().iterator();
            map.remove(it.next());
        }
    }
}
```

#### [206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)    迭代法

给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        //定义一个当前指针
        ListNode curr = head;
        //定义一个当前指针的前指针
        ListNode prev = null;
        if(head == null||head.next == head) return head;
        while(curr != null){
            //保存当前节点的下一个节点
            ListNode next = curr.next;
            //反转
            curr.next = prev;
            //迭代
            prev = curr;
            curr = next;
        }
        return prev;
    }
}

```

#### [206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)   递归法

给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表 

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        //1. 递归头  终止递归条件
       if(head == null || head.next == null) return head;
        //2. 递归体  自顶向下深入
       ListNode tail = reverseList(head.next);
        //3. 回溯    自底向上跳出
       head.next.next = head;
       head.next = null;
       return tail;
    }
}
```

#### [215. 数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

给定整数数组 `nums` 和整数 `k`，请返回数组中第 `**k**` 个最大的元素。

请注意，你需要找的是数组排序后的第 `k` 个最大的元素，而不是第 `k` 个不同的元素。小顶堆

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        //排序后的第k个最大元素  小顶堆
        PriorityQueue<Integer> queue = new PriorityQueue<>();
        for(int num : nums) {
            queue.add(num);
            if(queue.size() > k) {
                //如果大于k，则删除最小的元素
                queue.poll();
            }
        }
        return queue.poll();
    }
}
```

#### [215. 数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/) 快速排序

给定整数数组 `nums` 和整数 `k`，请返回数组中第 `k` 个最大的元素。

请注意，你需要找的是数组排序后的第 `k` 个最大的元素，而不是第 `k` 个不同的元素。

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        // 利用快速选择的思想，将数组分成两部分，一部分比第k大，一部分比第k小
        // 这里的k就是第k大的元素（正序）
        k = nums.length - k;
        int low = 0;
        int high = nums.length - 1;
        while (low < high) {
            // 快速选择
            int i = partition(nums, low, high);
// 如果i等于k，说明找到了第k大的元素
            if (i == k) break;
// 第K大的元素在下标为j的右边，继续分区在 arr[i+1, len-1] 区间查找：partition(arr, i+1, len-1)
            else if (i < k) low = i + 1;
// 第K大的元素在下标为j的左边，继续分区在 arr[0, i-1] 区间查找：partition(arr, 0, i-1)
            else high = i - 1;
        }
        return nums[k];
    }
    /**
 * 分区函数，将 arr[high] 作为 pivot 分区点
 * i、j 两个指针，i 作为标记“已处理区间”和“未处理区间”的分界点，也即 i 左边的（low~i-1）都是“已处理区”。
 * j 指针遍历数组，当 arr[j] 小于 pivot 时，就把 arr[j] 放到“已处理区间”的尾部，也即是 arr[i] 所在位置
 * 因此 swap(arr, i, j) 然后 i 指针后移，i++
 * 直到 j 遍历到数组末尾 arr[high]，将 arr[i] 和 arr[high]（pivot点） 进行交换，返回下标 i，就是分区点的下标。
 */
    private int partition(int[] nums, int low, int high) {
        int i = low;
        int pivot = nums[high];
        for (int j = low; j < high; j++) {
            if (nums[j] < pivot) {
                swap(nums, i, j);
                i++;
            }
        }
        swap(nums, i, high);
        return i;
    }


    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

> ##### 对partition方法进行优化

```java
private int partition(int[] arr, int low, int high) {
    if (high > low) {
        //在下标 low 和 high 之间随机选择，然后和下标 high 元素进行交换
        int random = low + new Random().nextInt(high - low);
        swap(arr, high, random);
    }
    int i = low;
    int pivot = arr[high];
    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            swap(arr, i, j);
            i++;
        }
    }
    swap(arr, i, high);
    return i;
}
```

#### [912. 排序数组](https://leetcode-cn.com/problems/sort-an-array/)

给你一个整数数组 `nums`，请你将该数组升序排列。

```java
class Solution {
    public int[] sortArray(int[] nums) {
        partition(nums,0,nums.length - 1);
        return nums;
    }
    public static void partition(int[] nums,int left,int right) {
        if(left >= right) return;
        //将最左边的元素作为分界点
        int index = left;
        //将最后的元素作为基准值
        int pivot = nums[right];
        //遍历
        for(int i = left;i <= right - 1;i++) {
//如果当前值比基准值还小，就将当前值所在的位置和最左的分界点所在的位置进行交换，然后分界点右移，保证分界点左边的元素都是比基准值小的
            if(nums[i] < pivot) {
                swap(nums,i,index);
                index++;
            }
        }
        //最后，将分界点和基准值所在的位置进行交换 因为此时分界点也已经到了最后
        swap(nums,index,right);
        //左边递归
        partition(nums,left,index - 1);
        //右边递归
        partition(nums,index + 1,right);
    }
    public static void swap(int[] nums,int i,int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```







```java
public class test1 {
   /*
   线程1输出 a 5次，线程2输出 b 5次，线程3输出 c 5次    要求输出
    abcabcabcabcabcabc
    */
   public static void main(String[] args) {
       waitNotify wn = new waitNotify(1,5);
        new Thread(()->{
           wn.print("a",1,2);
        }).start();
       new Thread(()->{
           wn.print("b",2,3);
       }).start();
       new Thread(()->{
           wn.print("c",3,1);
       }).start();
   }
}

    class waitNotify {
        private int flag;
        private int loopNUmber;

        waitNotify(int flag, int loopNUmber) {
            this.flag = flag;
            this.loopNUmber = loopNUmber;
        }

        public void print(String str, int waitFlag, int nextFlag) {
            for (int i = 0; i < loopNUmber; i++) {
                synchronized (this) {
                    while(flag!=waitFlag){
                        try {
                            this.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    System.out.print(str);
                    flag = nextFlag;
                    this.notifyAll();
                }
            }
        }
    }
```
