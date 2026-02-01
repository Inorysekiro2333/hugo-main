+++
title = '【Leetcode】算法刷题日常'
description = ''
date = '2026-02-01T16:12:08+08:00'
draft = true
image = ''
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