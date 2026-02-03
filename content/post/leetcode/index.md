+++
title = '【Leetcode】算法刷题日常'
description = ''
date = '2026-02-01T16:12:08+08:00'
draft = true
image = '119571089.jpg'
categories = ['算法', '力扣']
tags = ['leetcode']
+++

---

## Day1

### 无重复字符的最长子串

https://leetcode.cn/problems/longest-substring-without-repeating-characters/description/

![](2026-02-01-17-22-25-PixPin_2026-02-01_17-22-24.jpg)

解法：滑动窗口

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int n = s.length();
        // 因为只有英文字母、数字、符号和空格组成
        // 所以创建一个长度为128（askii码）的数组用来记录字符出现的次数
        int[] cnt = new int[128]; 
        int ans = 0;

        int l = 0;
        char[] charS = s.toCharArray();
        // 固定左l，遍历右r
        for(int r = 0; r < n; r++) {
            char c = charS[r];
            cnt[c]++;
            // r往后遍历，窗口内有字符重复时，左边l收缩窗口
            while(cnt[c] > 1) {
                cnt[charS[l]]--;
                l++;
            }
            // 更新ans为窗口长度
            ans = Math.max(ans, r - l + 1);
        }
        return ans;
    }
}
```

---

### LRU缓存机制

https://leetcode.cn/problems/lru-cache/description/

![](2026-02-01-17-59-00-PixPin_2026-02-01_17-58-59.jpg)

思路：使用带头尾指针的双向链表实现，使用hashMap作为缓存

1. put元素时两种情况
   
   1. put的元素不在链表中，直接链表头插（addToHead），判断容量是否超过上限，如果超过了，就移除(remove)掉最后一个节点
   
   2. put的元素在链表中，从缓存中找到该节点，移动到头部（moveToHead）

2. get获取元素
   
   1. 如果get的元素不在缓存中，返回 -1
   
   2. get的元素在缓存中，返回节点的value，同时更新缓存（moveToHead）

```java
class LRUCache {
    class Node {
        int key;
        int value;
        Node next;
        Node prev;
    }
    private final int capacity; // LRU缓存容量
    private Node head, tail; // 头尾节点
    private final Map<Integer, Node> cache = new HashMap<>(); // Map缓存

    // 初始化
    public LRUCache(int capacity) {
        this.capacity = capacity;
        head = new Node();
        tail = new Node();
        head.next = tail;
        tail.prev = head;
    }

    // put方法
    public void put (int key, int value) {
        Node node = cache.get(key);
        if (node == null) {
            Node newNode = new Node();
            newNode.key = key;
            newNode.value = value;                    
            cache.put(key, newNode);
            addToHead(newNode);
            if (cache.size() > capacity) {
                Node oldNode = popTail();
                cache.remove(oldNode);
            }
        } else {
            node.value = value;
            moveToHead(node);
        }
    }

    // get方法
    public int get(int key) {
        Node node = cache.get(key);
        if (node == null) return -1;

        moveToHead(node);
        return node.value;
    }

    // 辅助方法
    private void moveToHead(Node node) {
        remove(node);
        addToHead(node);
    }
    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    private void addToHead(Node node) {
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
    }
    private Node popTail() {
        Node res = tail.prev;
        remove(res);
        return res;
    }
}
```

---

### 反转链表easy

![](2026-02-01-18-42-43-PixPin_2026-02-01_18-42-41.jpg)

思路：原地反转，使用3个指针

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode cur = head;
        ListNode pre = null;
        while (cur != null) {
            ListNode nxt = cur.next;
            cur.next = pre;
            pre = cur;
            cur = nxt;
        }
        return pre;
    }
}
```

---

### 数组中的第K个最大元素

https://leetcode.cn/problems/kth-largest-element-in-an-array/description/

![](2026-02-01-19-41-56-PixPin_2026-02-01_19-41-54.jpg)

思路：

快速排序的思想，区域划分，将大于、等于、小于基准的数划分到三个数组中，之后在大于和小于的数组中递归划分，等于的直接返回

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        List<Integer> numList = new ArrayList<>();
        for (int num : nums) {
            numList.add(num);
        }
        return quickSelect(numList, k);
    }

    public int quickSelect(List<Integer> nums, int k) {
        // 随机选择基准数
        Random rand = new Random();
        int n = nums.size();
        int randNum = rand.nextInt(n);
        int pivot = nums.get(randNum);
        // 将大于、小于、等于pivot的元素分别划分到big、small、equal中
        List<Integer> big = new ArrayList<>();
        List<Integer> small = new ArrayList<>();
        List<Integer> equal = new ArrayList<>();
        for (int num : nums) {
            if (num > pivot) {
                big.add(num);
            } else if (num < pivot) {
                small.add(num);
            } else {
                equal.add(num);
            }
        }
        // 第k大的元素在big中，递归划分
        if (k <= big.size()) {
            return quickSelect(big, k);
        }
        // 第k大的元素在small中，递归划分
        if (nums.size() - small.size() < k) {
            return quickSelect(small, k - nums.size() + small.size());
        }
        // 第k大的元素在equal中，直接返回pivot
        return pivot;
    }
}
```

解释：

- big + equal = nums.size - small；

- 如果big + equal的元素个数小于 k ，说明第k大的元素不在big和equal中，只能在small中；

- 因为big和equal中的元素都比small中的大，所以在small中需要找**第 k - (big + equeal的数量) 大的元素**

---

### k个一组反转链表hard

https://leetcode.cn/problems/reverse-nodes-in-k-group/description/

![](2026-02-01-20-08-20-PixPin_2026-02-01_20-08-18.jpg)

思路：

1.链表分区为已翻转部分+待翻转部分+未翻转部分

2.每次翻转前，要确定翻转链表的范围，这个必须通过k次循环来确定

3.需记录翻转链表前驱和后继，方便翻转完成后把已翻转部分和未翻转部分连接起来

4.初始需要两个变量pre和end，pre代表待翻转链表的前驱，end代表待翻转链表的末尾

5.经过k次循环，end 到达末尾，记录待翻转链表的后继next = end.next

6.翻转链表，然后将三部分链表连接起来，然后重置pre和end指针，然后进入下一次循环

7.特殊情况，当翻转部分长度不足k时，在定位end完成后，end==null，已经到达末尾，说明题目已完成，直接返回即可

8.时间复杂度为 O(n *K) 最好的情况为 O(n）最差的情况未 O(n²)

9.空间复杂度为O(1）除了几个必须的节点指针外，我们并没有占用其他空间

```java
public ListNode reverseKGroup(ListNode head, int k) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;

    ListNode pre = dummy;
    ListNode end = dummy;

    while (end.next != null) {
        for (int i = 0; i < k && end != null; i++) end = end.next;
        if (end == null) break;
        ListNode start = pre.next;
        ListNode next = end.next;
        end.next = null;
        pre.next = reverse(start);
        start.next = next;
        pre = start;

        end = pre;
    }
    return dummy.next;
}

private ListNode reverse(ListNode head) {
    ListNode pre = null;
    ListNode curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = pre;
        pre = curr;
        curr = next;
    }
    return pre;
}
```

![](2026-02-01-21-06-27-PixPin_2026-02-01_21-06-26.jpg)

![](2026-02-01-21-06-36-PixPin_2026-02-01_21-06-35.jpg)

![](2026-02-01-21-06-57-PixPin_2026-02-01_21-06-56.jpg)

---

## Day2

### 三数之和

https://leetcode.cn/problems/3sum/description/

![](2026-02-02-16-45-15-PixPin_2026-02-02_16-45-14.jpg)

思路：

- 先排好序，最外层循环为 i 固定为左起第一个从后遍历（从小到大），内层循环双指针j、k，j为i后一个开始往后遍历（从小到大），k从最右边往左边遍历（从大到小）

- 外层去重的点：if (i > 0 && nums[i - 1] == nums[i]) continue;

- 两个外层循环的优化点：
  
  - 如果i + (i+1) + (i+2)  > 0 （最小的三个数已经大于0了）直接break掉，不进循环
  
  - 如果i + (n - 1) + (n - 2) < 0 （当前的i和最大的两个数都还小于0，说明当前的i太小） continue直接遍历下一个i

- 内层循环：
  
  - 如果i + j + k > 0了，说明k大了，k--
  
  - 如果i + j + k < 0了，说明j小了，j++
  
  - 等于0的情况下，去重 j、k ： nums[j] == nums[j - 1] ,j++;   nums[k] == nums[k + 1], k--

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        // 转化为两数之和
        List<List<Integer>> ans = new ArrayList<>();
        Arrays.sort(nums);
        int n = nums.length;
        for (int i = 0; i < n - 2; i++) {
            int x = nums[i];
            if (i > 0 && nums[i - 1] == x) continue; // 去重i

            // 2个优化
            if (x + nums[i + 1] + nums[i + 2] > 0) break;
            if (x + nums[n - 1] + nums[n - 2] < 0) continue;

            int j = i + 1;
            int k = n - 1;

            while (j < k) {
                int s = x + nums[j] + nums[k];
                if (s > 0) k--;
                else if (s < 0) j++;
                else {
                    // 去重j k
                    ans.add(List.of(x, nums[j], nums[k]));
                    for (j++;j < k && nums[j] == nums[j - 1]; j++);
                    for (k--;j < k && nums[k] == nums[k + 1]; k--);
                }
            }
        }
        return ans;
    }
}
```

---

### 相交链表

https://leetcode.cn/problems/intersection-of-two-linked-lists/description/

![](2026-02-02-17-20-18-PixPin_2026-02-02_17-20-16.jpg)

![](2026-02-02-17-20-30-PixPin_2026-02-02_17-20-28.jpg)

![](2026-02-02-17-20-38-PixPin_2026-02-02_17-20-37.jpg)

思路：独属于理科生的情话：“走过你的来时路，只为与你相遇”

设「第一个公共节点」为 node，「链表headA 」的节点数量为 a，「链表headB」的节点数量为b，「两链表的公共尾部」的节点数量为 c，则有：

- 头节点 headA 到 node 前，共有 a－c 个节点;

- 头节点 headB 到 node 前，共有 b－c 个节点;

考虑构建两个节点指针A，B分别指向两链表头节点headA，headB，做如下操作:

- 指针A 先遍历完链表headA，再开始遍历链表headB，当走到node时，共走步数为:     a+ (b - c)

- 指针B先遍历完链表headB，再开始遍历链表headA，当走到node时，共走步数为:
  b+ (a -c)
  
  如下式所示，此时指针A，B重合，并有两种情况:
  
  a + (b -c) = b+ (a - c)
  
  若两链表有公共尾部（即 c > 0）：指针A，B同时指向「第一个公共节点」node。
  
  若两链表无公共尾部（即 c = 0）：指针A，B同时指向 null。
  
  因此返回A即可。

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if (headA == null || headB == null) {
            return null;
        }
        ListNode pA = new ListNode(-1, headA);
        ListNode pB = new ListNode(-1, headB);
        while(pA != pB) {
            pA = pA != null ? pA.next : headB;
            pB = pB != null ? pB.next : headA;
        }
        return pA;
    }
}
```

---

### 合并区间

https://leetcode.cn/problems/merge-intervals/description/

![](2026-02-02-19-25-32-PixPin_2026-02-02_19-25-28.jpg)

思路：

1. 先按照单个区间的第一个元素排序：`Arrays.sort(intervals, (p, q) -> p[0] - q[0])`

2. 遍历二维数组，对于每个拿到的元素（一维数组 int[ ] p）,如果当前遍历区间的左端点start_i <= 结果集合中最后一个元素的右端点end_i-1，说明是可以合并的。合并之后的右端点取当前遍历区间和结果集合最后一个元素中的大者。
   
   - `if (p[0] <= ans.get(m - 1)[1])`
   
   - `ans.get(m-1)[1] = Math.max(p[1], ans.get(m-1)[1])`

3. 不能合并的单独加入结果集合中

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        Arrays.sort(intervals, (p, q) -> p[0] - q[0]); // 先按照单个区间的第一个元素排序
        List<int[]> ans = new ArrayList<>();
        for (int[] p : intervals) {
            int m = ans.size();

            // 注意边界条件 
            // 如果当前遍历区间左端点 <= 列表中最后一个元素的右端点，说明是可以合并的
            if (m > 0 && p[0] <= ans.get(m - 1)[1]) {
                // 合并之后的右端点取两者之间的大的
                ans.get(m - 1)[1] = Math.max(p[1], ans.get(m - 1)[1]);
            } else {
                ans.add(p);
            }
        }
        return ans.toArray(new int[ans.size()][]);
    }
}
```

---

### 字符串相加

https://leetcode.cn/problems/add-strings/description/

![](2026-02-02-19-34-41-PixPin_2026-02-02_19-34-40.jpg)

思路：

- 设定 i, j 两个指针分别指向num1和num2的尾部，模拟人工加法

- 计算进位：计算`carry = tmp / 10`，代表当前位相加是否产生进位；

- 添加当前位：计算`tmp = n1 + n2 + carry`，并将当前位`tmp % 10`添加至res头部；

- 当指针 i, j 走过数字首部后，给n1, n2赋值0，相当于给num1, num2中长度较短的数字前面填0，方便后面计算

- 当遍历完num1, num2后跳出循环，并根据carry的值决定是否在头部添加进位1，最终返回res

```java
class Solution {
    public String addStrings(String num1, String num2) {
        StringBuilder res = new StringBuilder("");
        int i = num1.length() - 1, j = num2.length() - 1, carry = 0;
        while (i >= 0 || j >= 0) {
            int n1 = i >= 0 ? num1.charAt(i) - '0' : 0;
            int n2 = j >= 0 ? num2.charAt(j) - '0' : 0;
            int tmp = n1 + n2 + carry;
            carry = tmp / 10;
            res.append(tmp % 10);
            i--;
            j--;
        }
        if (carry == 1) res.append(1);
        return res.reverse().toString();
    }
}
```

---

### 合并k个升序链表hard

https://leetcode.cn/problems/merge-k-sorted-lists/description/

![](2026-02-02-20-40-09-PixPin_2026-02-02_20-40-08.jpg)

思路：分治

- 把lists一分为二，先合并前一半的链表，再合并后一半的链表

- 然后把这两个链表合并成最终的链表

- 如何合并前一半的链表？可以用递归，继续一分为二。如此下去直到只有一个链表，此时无需合并。

```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        return mergeKLists(lists, 0, lists.length);
    }

    // 合并从 lists[i] 到 lists[j-1] 的链表
    private ListNode mergeKLists(ListNode[] lists, int i, int j) {
        int m = j - i;
        if (m == 0) {
            return null; // 注意输入的 lists 可能是空的
        }
        if (m == 1) {
            return lists[i]; // 无需合并，直接返回
        }
        ListNode left = mergeKLists(lists, i, i + m / 2); // 合并左半部分
        ListNode right = mergeKLists(lists, i + m / 2, j); // 合并右半部分
        return mergeTwoLists(left, right); // 最后把左半和右半合并
    }

    // 21. 合并两个有序链表
    private ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode(); // 用哨兵节点简化代码逻辑
        ListNode cur = dummy; // cur 指向新链表的末尾
        while (list1 != null && list2 != null) {
            if (list1.val < list2.val) {
                cur.next = list1; // 把 list1 加到新链表中
                list1 = list1.next;
            } else { // 注：相等的情况加哪个节点都是可以的
                cur.next = list2; // 把 list2 加到新链表中
                list2 = list2.next;
            }
            cur = cur.next;
        }
        cur.next = list1 != null ? list1 : list2; // 拼接剩余链表
        return dummy.next;
    }
}
```

---

## Day3

### 排序数组（手撕快排）

https://leetcode.cn/problems/sort-an-array/description/

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        if (scanner.hasNext()) {
            int n = scanner.nextInt();
            int[] arr = new int[n];
            for (int i = 0; i < n; i++) {
                arr[i] = scanner.nextInt();
            }
            
            quickSort(arr, 0, n - 1);
            
            StringBuilder sb = new StringBuilder();
            for (int num : arr) {
                sb.append(num).append(" ");
            }
            System.out.println(sb.toString().trim()); // 去掉首尾空格
        }
        scanner.close();
    }

    private static void quickSort(int[] arr, int left, int right) {
        if (left < right) {
            int pivotIndex = partition(arr, left, right);
            quickSort(arr, left, pivotIndex - 1);
            quickSort(arr, pivotIndex + 1, right);
        }
    }

    private static int partition(int[] arr, int left, int right) {
        int pivot = arr[left];
        int i = left, j = right;
        while (i < j) {
            while (i < j && arr[j] >= pivot) j--;
            while (i < j && arr[i] <= pivot) i++;
            if (i < j) {
                swap(arr, i, j);
            }
        }
        // 枢纽元素归位
        swap(arr, left, i);
        return i;
    }

    private static void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}


```



---

### 最大子数组和

https://leetcode.cn/problems/maximum-subarray/description/

![](2026-02-03-19-56-12-PixPin_2026-02-03_19-56-10.jpg)

思路：

- 贪心的思想：

- 对于每一个位置，决定是 **“将当前的数加到之前的子数组中”** 还是 **“从当前的数重新开始一个新的子数组”**

- 如果以 `x` 前一个元素结尾的子数组和 `pre` 是正数（或者非负），那么加上 `x` 后，子数组变长了，和也变大了（或者没有变小），这对后续寻找最大和是有利的。**所以，我们应该“继承”之前的成果**

- 如果 `pre` 是负数，说明之前的子数组是一个“累赘”。加上它只会让总和变小。既然是负数，不如直接扔掉，从 `x` 自己开始算，**这就是“贪心”的体现——只要正收益，不要负累赘。**

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int pre = 0, maxAns = nums[0];
        for (int x : nums) {
            pre = Math.max(pre + x, x);
            maxAns = Math.max(maxAns, pre);
        }
        return maxAns;
    }
}

```



---

### 最长回文子串

思路：动态规划

- 1.状态定义：dp数组

- `dp[l][r]` 的含义是：字符串 `s` 从下标 `l` 到下标 `r` 的子串（即 `s[l...r]`）是否为回文串。

- 2.状态转移方程：`if (s.charAt(l) == s.charAt(r) && (r - l <= 2 || dp[l + 1][r - 1]))`

![](2026-02-03-20-24-07-PixPin_2026-02-03_20-23-53.jpg)

- 一旦确认 `s[l...r]` 是回文（`dp[l][r] = true`），就检查它是否比当前记录的最长回文还长。如果是，就更新起止点和长度。

```java
class Solution {
    public String longestPalindrome(String s) {
         // 用一个 boolean dp[l][r] 标识字符串从i到j这段是否为回文。
         // 如果dp[l][r] = true，要判断dp[l-1][r+1]是否为回文只需要判断l-1  r+1两个位置是否为相同字符
        if (s == null || s.length() < 2) {
            return s;
        }
        int strLen = s.length();
        int maxStart = 0; //最长回文的起点
        int maxEnd = 0; // 最长回文的终点
        int maxLen = 1; // 最长回文的长度

        boolean[][] dp = new boolean[strLen][strLen];

        for (int r = 1; r < strLen; r++) {
            for (int l = 0; l < r; l++) {
                if (s.charAt(l) == s.charAt(r) && (r - l <= 2 || dp[l + 1][r - 1])) { // r - l <= 2是为了处理长度为2和3的短回文情况
                    dp[l][r] = true;
                    if (r - l + 1 > maxLen) {
                        maxLen = r - l + 1;
                        maxStart = l;
                        maxEnd = r;
                    }
                }
            }
        }
        return s.substring(maxStart, maxEnd + 1);
    }
}
```



---

### 合并两个有序链表

https://leetcode.cn/problems/merge-two-sorted-lists/

```java
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode();
        ListNode cur = dummy;
        while (list1 != null && list2 != null) {
            if (list1.val < list2.val) {
                cur.next = list1;
                list1 = list1.next;
            } else {
                cur.next = list2;
                list2 = list2.next;
            }
            cur = cur.next;
        }
        cur.next = list1 != null ? list1 : list2;
        return dummy.next;
    }
}
```



---

### 二叉树的层序遍历

https://leetcode.cn/problems/binary-tree-level-order-traversal/description/

![](2026-02-03-20-17-25-PixPin_2026-02-03_20-17-22.jpg)

思路：模板题

- 使用队列

- 遍历该节点时，判断该节点有无左右孩子，有的话加进队列中

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        // 结果集合[[],[]]
        List<List<Integer>> res = new ArrayList<>();
        // 使用队列维护
        Queue<TreeNode> queue = new ArrayDeque<>();
        if (root != null) {
            queue.add(root);
        }
        while (!queue.isEmpty()) {
            int n = queue.size();
            List<Integer> level = new ArrayList<>();
            for (int i = 0; i < n; i++) {
                TreeNode node = queue.poll();
                level.add(node.val);
                if (node.left != null) {
                    queue.add(node.left);
                }
                if (node.right != null) {
                    queue.add(node.right);
                }
            }
            res.add(level);
        }
        return res;
    }
}
```