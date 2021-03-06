排序算法。快速排序的时间复杂度和空间复杂度。快排的排序算法是一个稳定的算法（腾讯CDG一面）

> | 算法       | 稳定性 | 时间复杂度           | 空间复杂度 | 备注           |
> | -------- | --- | --------------- | ----- | ------------ |
> | 选择排序     | ×   | N2              | 1     |              |
> | 冒泡排序     | √   | N2              | 1     |              |
> | 插入排序     | √   | N ~ N2          | 1     | 时间复杂度和初始顺序有关 |
> | 希尔排序     | ×   | N 的若干倍乘于递增序列的长度 | 1     | 改进版插入排序      |
> | 快速排序     | ×   | NlogN           | logN  |              |
> | 三向切分快速排序 | ×   | N ~ NlogN       | logN  | 适用于有大量重复主键   |
> | 归并排序     | √   | NlogN           | N     |              |
> | 堆排序      | ×   | NlogN           | 1     | 无法利用局部性原理    |

快速排序（深信服一面）

```java
class Solution {
    public int[] sortArray(int[] nums) {
        quickSort(nums, 0, nums.length - 1);
        return nums;
    }

    private void quickSort(int[] nums, int left, int right) {
        if (left < right) {
            int index = partition(nums, left, right);
            quickSort(nums, left, index - 1);
            quickSort(nums, index + 1, right);
        }
    }

    private int partition(int[] nums, int left, int right) {
        int pivot = nums[left];
        while (left < right) {
            while (left < right && nums[right] >= pivot) {
                right --;
            }
            nums[left] = nums[right];
            while (left < right && nums[left] <= pivot) {
                left ++;
            }
            nums[right] = nums[left];
        }
        nums[left] = pivot;
        return left;
    }
}
```

归并排序（腾讯云一面）

```java
class Solution{
    private void merge(int[] nums, int L1, int R1, int L2, int R2){
        int n = R1-L1+R2-L2+2;
        int[] temp = new int[n];
        int i = L1, j = L2, index = 0;
        while (i <= R1 && j <= R2){
            if (nums[i] <= nums[j]){
                temp[index++] = nums[i++];
            } else{
                temp[index++] = nums[j++];
            }
        }
        while (i <= R1){
            temp[index++] = nums[i++];
        }
        while (j <= R2){
            temp[index++] = nums[j++];
        }
        for (int k = 0; k < n; k++){
            nums[L1 + k] = temp[k];
        }
    }
    public void mergeSort(int[] nums, int left, int right){
        if (left < right){
           int mid = (left + right) / 2;
           mergeSort(nums, left, mid);
           mergeSort(nums, mid+1, right);
           merge(nums, left, mid, mid+1, right);
        }
    }
}
```

[152. 乘积最大子数组](https://leetcode-cn.com/problems/maximum-product-subarray/)（阿里云笔试）

```java
class Solution {
    public int maxProduct(int[] nums) {
        // res 记录最终的结果
        // lastMax记录最新的最大值
        // lastMin记录最新的最小值
        // 需要记录lastMin是因为当nums[i]为负数时，与lastMin相乘可能会使得结果最大
        int res = nums[0], lastMax = nums[0], lastMin = nums[0];
        for (int i = 1; i < nums.length; i++){
            // 转移方程：
            // lastMax = max(nums[i]*lastMax, nums[i]*lastMin, nums[i])
            // lastMin = min(nums[i]*lastMax, nums[i]*lastMin, nums[i])
            int a = nums[i];
            int b = nums[i] * lastMax;
            int c = nums[i] * lastMin;
            lastMax = Math.max(a, Math.max(b, c));
            lastMin = Math.min(a, Math.min(b, c));
            res = Math.max(res, lastMax);
        }
        return res;
    }
}
```

[219. 存在重复元素 II](https://leetcode-cn.com/problems/contains-duplicate-ii/)（华为Cloudbu）

```java
class Solution {
    public boolean containsNearbyDuplicate(int[] nums, int k) {
        Set<Integer> occ = new HashSet<Integer>();
        for (int i = 0; i < nums.length; i ++){
            if (occ.contains(nums[i])) return true;
            occ.add(nums[i]);
            if (occ.size() > k) occ.remove(nums[i - k]);
        }
        return false;
    }
}
```

[70. 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)（快手一面）

[121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)（快手一面）

[155. 最小栈](https://leetcode-cn.com/problems/min-stack/)（今日头条一面）

```java
class MinStack{
    Deque<Integer> xStack;
    // 设计一个辅助栈，其记录当前xStack对应的最小值
    // 由于stack栈先进后出的性质
    // 每次xStack入栈时，minStack入栈min(x, minStack.peek())
    // 每次xStack出栈时，minStack只需要出栈栈顶元素
    Deque<Integer> minStack;
    public MinStack(){
        xStack = new LinkedList<>();
        minStack = new LinkedList<>();
        minStack.push(Integer.MAX_VALUE);
    }
    public void push(int x){
        xStack.push(x);
        minStack.push(Math.min(minStack.peek(), x));
    }
    public void pop(){
        xStack.pop();
        minStack.pop();
    }
    public int top(){
        return xStack.peek();
    }
    public int getMin(){
        return minStack.peek();
    }
}
```

[206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)（今日头条一面）

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        // bad case
        if (head == null || head.next == null) return head;
        ListNode last = reverseList(head.next);
        head.next.next = head;
        // 当链表递归反转之后，新的头节点是last，而之前的head变成了最后一个节点，链表的末尾要指向null
        head.next = null;
        return last;
    }
}
```

[146. LRU 缓存机制](https://leetcode-cn.com/problems/lru-cache/)（淘宝一面，深信服一面）

```java
class LRUCache {
    LinkedHashMap<Integer, Integer> map = new LinkedHashMap<>();
    int capacity;

    public LRUCache(int capacity){
        this.capacity = capacity;
    }
    public int get(int key){
        if(!map.containsKey(key)){
            return -1;
        } else{
            makeLast(key);
            return map.get(key);
        }
    }
    public void put(int key, int value){
        map.put(key, value);
        makeLast(key);
        if (map.size() > capacity){
            map.remove(map.keySet().iterator().next());
        }
    }
    public void makeLast(int key){
        int value = map.get(key);
        map.remove(key);
        map.put(key, value);
    }
}
```

输入一个数字，打印出嵌套的正方形。设输入 n；如果 n 是 奇数，则打印 1 ，5 ，9..., n 的嵌套正方形；如果 n 是偶数，则打印出 1 , 5 ,9..., n - 1 的嵌套正方形；最大n 设置为17；例子：如果 输入 n = 9， 则打印出下面的图形（淘宝一面）

```java
class Solution{
    public void printNestedArray(int n){
        if (n % 2 == 0) n--;
        n = n - (n - 1) % 4;
        boolean[][] ans = new boolean[n][n];
        int start = 0;
        int length = n;
        while(length > 0){
            fillArray(ans, start, length);
            start += 2;
            length -= 4;
        }
        for (int i = 0; i < n; i ++){
            for (int j = 0; j < n; j++){
                if (ans[i][j]){
                    System.out.print("+ ");
                } else{
                    System.out.print("  ");
                }
            }
            System.out.println();
        }
    }
    public void fillArray(boolean[][] ans, int start, int length){
        for (int i = start; i < start + length; i++){
            ans[i][start] = true;
            ans[start][i] = true;
            ans[i][start+length-1] = true;
            ans[start+length-1][i] = true;
        }
    }
}
```

一个蛋糕三刀切成四块（腾讯CDG一面）

长度为n的数组，进行逆序排序？（腾讯CDG一面）

```java
class Solution{
    public void reverseArray(int[] nums){
        int right = nums.length - 1;
        int left = 0;
        while(left < right){
            int tmp = nums[right];
            nums[right] = nums[left];
            nums[left] = tmp;
            left ++;
            right --;
        }
    }
}
```

[剑指 Offer 25. 合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)（快手二面）

```java
class Solution{
    public ListNode mergeListNode(ListNode head1, ListNode head2){
        ListNode head = new ListNode(-1);
        ListNode dummy = head;
        while(head1 != null && head2 != null){
            if (head1.val < head2.val){
                dummy.next = new ListNode(head1.val);
                head1 = head1.next;
            } else{
                dummy.next = new ListNode(head2.val);
                head2 = head2.next;
            }
            dummy = dummy.next;
        }
        if (head1 != null){
            dummy.next = head1;
        } else if (head2 != null){
            dummy.next = head2;
        }
        return head.next;
    }
}
```

[剑指 Offer 42. 连续子数组的最大和](https://leetcode-cn.com/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/)（腾讯云二面）

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int res = nums[0];
        for (int i = 1; i < nums.length; i ++){
            nums[i] = Math.max(nums[i], nums[i] + nums[i - 1]);
            res = Math.max(res, nums[i]);
        }
        return res;
    }
}
```

求两个数的最大公约数gcd（腾讯云二面）

```java
class SoluctionTxCloud{
    // 求两个数的最大公约数
    public int gcd(int a, int b){
        if (b == 0) return a;
        return gcd(b, a % b);
    }

    // 求两个数的最大公倍数
    public int lcm(int a, int b){
        return a / gcd(a, b) * b;
    }
}
```

[剑指 Offer 04. 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)（今日头条测开二面）

```java
class Solution{
    public boolean findNumberIn2DArray(int[][] matrix, int target){
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return false;
        }
        int rows = matrix.length;
        int cols = matrix[0].length;
        int row = 0;
        int col = cols - 1;
        while(row < rows && col >= 0){
            if (matrix[row][col] > target){
                col --;
            } else if (matrix[row][col] < target){
                row ++;
            } else{
                return true;
            }
        }
        return false;
    }
}
```

单例模式（腾讯TEG一面）

[704. 二分查找](https://leetcode-cn.com/problems/binary-search/)（腾讯TEG一面）

[20. 有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)（腾讯TEG一面）

最大堆最小堆怎么维护（腾讯TEG一面）

算法题：[逛街](https://www.nowcoder.com/questionTerminal/35fac8d69f314e958a150c141894ef6a)（腾讯CDG事务开发一面）

算法题：[剑指 Offer 42. 连续子数组的最大和](https://leetcode-cn.com/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/)（字节系统研发一面）
