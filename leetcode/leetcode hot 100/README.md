#### 1. 两数之和

暴力枚举：

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int n = nums.length;
        for (int i = 0; i < n - 1; i++){
            for (int j = i + 1; j < n; j++){
                if (nums[i] + nums[j] == target){
                    return new int[]{i, j};
                }
            }
        }
        return new int[0];
    }
}
```

哈希表：

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> hashtable = new HashMap<Integer, Integer>();
        for (int i = 0; i < nums.length; i++){
            // 快速寻找数组中是否存在目标元素
            if (hashtable.containsKey(target - nums[i])){
                return new int[]{hashtable.get(target - nums[i]), i};
            }
            hashtable.put(nums[i], i);
        }
        return new int[0];
    }
}
```

#### 2. 两数相加

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(-1);
        ListNode tail = dummy;
        int carry = 0;
        while (l1 != null || l2 != null){
            // 仅当没有遍历到最高位时才相加。
            int n1 = l1 != null ? l1.val : 0;
            int n2 = l2 != null ? l2.val : 0;
            // 进行相加
            int sum = n1 + n2 + carry;
            tail.next = new ListNode(sum % 10);
            tail = tail.next;
            // 得到下一位的数
            carry = sum / 10;
            // 如果还没遍历到最后一位则继续遍历
            if (l1 != null){
                l1 = l1.next;
            }
            if (l2 != null){
                l2 = l2.next;
            }
        }
        if (carry > 0){
            tail.next = new ListNode(carry);
        }
        return dummy.next;
    }
}
```

#### 3. 无重复字符的最长子串

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        // 哈希集合，记录每个字符是否出现过
        Set<Character> occ = new HashSet<Character>();
        int n = s.length();
        // 右指针，初始值为-1，相当于我们在字符串的左边界的左侧，还没有开始移动
        int rk = -1, ans = 0;
        for (int i = 0; i < n; i++){
            if (i != 0){
                // 左指针向右移动一格，移除一个字符
                occ.remove(s.charAt(i-1));
            }
            while (rk + 1 < n && !occ.contains(s.charAt(rk + 1))){
                // 不断地移动右指针
                occ.add(s.charAt(rk+1));
                ++rk;
            }
            // 第i到rk个字符是一个极长地无重复字符子串
            ans = Math.max(ans, rk - i + 1);
        }
        return ans;
    }
}
```

#### 4. 寻找两个正序数组的中位数

##### 4.1 先将两个数组合并，然后根据奇数和偶数判断返回中位数

1. 特殊情况的判断。即其中一个为空数组；
2. 进行归并排序
   1. 判断是否有数组到头了，如果到头了则全部令为另一个数组；
   2. 进行归并排序，即让小的在前面
3. 对进行排序后的数组取中位数。

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        // num1 和 num2 的长度
        int m = nums1.length, n = nums2.length;
        // 其中一个为空数组的判断
        if (m == 0){
            if (n % 2==0){
                return (nums2[n/2 - 1] + nums2[n/2]) / 2.0;
            }else{
                return nums2[n/2];
            }
        }
        if (n == 0){
            if (m % 2==0){
                return (nums1[m/2 - 1] + nums1[m/2]) / 2.0;
            } else{
                return nums1[m/2];
            }
        }
        // 进行递归排序
        int i=0, j=0, cnt=0;
        int[] ans = new int[m+n];
        while(cnt != (m+n)){
            if (i == m){
                while (j != n){
                    ans[cnt++] = nums2[j++];
                }
                break;
            }
            if (j == n){
                while (i != m){
                    ans[cnt++] = nums1[i++];
                }
                break;
            }

            if (nums1[i] < nums2[j]){
                ans[cnt++] = nums1[i++];
            } else{
                ans[cnt++] = nums2[j++];
            }
        }
        // 取中位数
        if (cnt % 2 == 0){
            return (ans[cnt/2-1] + ans[cnt/2]) / 2.0;
        } else{
            return ans[cnt/2];
        }
    }
}
```

##### 4.2 结合奇偶情况进行判断

同时考虑奇偶两种情况：

用left记录(m+n)/2-1，而用right记录(m+n)/2：

1. 如果m+n为偶数时，则返回left和right的平均值；
2. 如果m+n为奇数时，则返回right；

使用aStart和bStart分别记录num1和num2现在遍历到的元素。

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int m = nums1.length, n = nums2.length;
        int aStart = 0, bStart = 0;
        int len = m + n;
        // 使用left记录len/2-1, 使用right记录len/2
        int left =-1,  right = -1;  
        for (int i = 0; i <= len / 2; i++){
            left = right;
            if ((aStart < m) && ((bStart >= n) || (nums1[aStart] < nums2[bStart]))){
                right = nums1[aStart++];
            } else{
                right = nums2[bStart++];
            }
        }
        // 如果len为偶数，则返回left+right的平均值；
        // 如果len为奇数，则返回right
        if (len % 2 == 0){
            return (left + right) / 2.0;
        } else{
            return right;
        }
    }
}
```

#### 5. 最长回文串

1. 定义数组的含义。`dp[i][j]`表示字符串从i到j的子串（左右闭合）是否为回文串；
2. 找出数组间元素的关系式。`dp[i][j]=(s[i]==s[j])&&(dp[i+1][j-1])`
3. 初始值。
   1. 当子串长度为1时，必为回文串。即`dp[i][i]=true`；
   2. 当子串长度为2时，若两个字符相同则为回文串

在进行转移方程的时候，使用两层for循环进行遍历：

1. 第一层for循环遍历选取子字符串的长度；
2. 第二层for循环遍历第二层字符串遍历选取子字符串的下标

```java
class Solution {
    public String longestPalindrome(String s) {
        String result = "";
        int len = s.length();
        // dp[i][j]表示字符串从i到j的子串（左右闭合）是否为回文串；
        boolean dp[][] = new boolean[len][len];

        // 初始化：当子串长度为1时，必为回文串
        for (int i = 0; i < len; i ++){
            dp[i][i] = true;
            result = s.substring(i, i + 1);
        }
        // 初始化：当子串长度为2时，若两个字符相同则为回文串
        for (int i = 0; i < len - 1; i ++){
            if (s.charAt(i) == s.charAt(i + 1)){
                dp[i][i+1] = true;
                result = s.substring(i, i + 2);
            }
        }

        // 找出数组元素的关系式
        for (int l = 3; l <= len; l ++){
            for (int i = 0; i < len - l + 1; i ++){
                // dp[i][j]=(s[i]==s[j])&&(dp[i+1][j-1])
                if ((s.charAt(i) == s.charAt(i + l - 1)) && (dp[i+1][i+l-2])){
                    dp[i][i+l-1] = true;
                    result = s.substring(i, i + l);
                } else{
                    dp[i][i+l-1] = false;
                }
            }
        }
        return result;
    }
}
```

#### 10. 正则表达式匹配

1. 定义数组的含义。`dp[i][j]`表示`s`的前i个字符是否能被`p`的前j个字符匹配
2. 定义数组元素间的关系式
   1. 如果`p.charAt(j)==s.charAt(i)`，则`dp[i][j]==dp[i-1][j-1]`；
   2. 如果`p.charAt(j)=='.'`，则`dp[i][j]=dp[i-1][j-1]`；
   3. 如果`p.charAt(j)=='*'`：
      1. 如果`p.charAt(j-1)!=s.charAt(i)`，则`dp[i][j]=dp[i][j-2]`；
      2. 如果`p.charAt(j-1)==s.charAt(i)`或者`p.charAt(i-1)=='.'`：
         1. `dp[i][j]=dp[i-1][j]`；
         2. 或`dp[i][j]=dp[i][j-1]`；
         3. 或`dp[i][j]=dp[i][j-2]`
3. 初始值。`dp[0][0]=true`

```java
class Solution {
    public boolean isMatch(String s, String p) {
        if (s == null || p == null){
            return false;
        }
        // 定义数组的含义. dp[i][j]表示s的前i个字符是否能被p的前j个字符匹配
        boolean dp[][] = new boolean[s.length()+1][p.length()+1];   
        // 初始化
        dp[0][0] = true;
        for (int i = 0; i < p.length(); i ++){      // 当s的长度为0,并且p仅包含*时可以令匹配
            if (p.charAt(i) == '*' && dp[0][i-1]){
                dp[0][i+1] = true;
            }
        }

        // 定义数组元素间的关系式
        for (int i = 1; i <= s.length(); i++){
            for (int j = 1; j <= p.length(); j++){
                // 当前元素相等,或者当前p的元素为'.'能匹配任意元素
                if(p.charAt(j-1)==s.charAt(i-1) || p.charAt(j-1)=='.'){
                    dp[i][j] = dp[i-1][j-1];
                }
                if (p.charAt(j-1)=='*'){
                    // 如果前一个元素不匹配且不为任意元素,则看其前两个元素以前的p是否能匹配上
                    if (p.charAt(j-2)!=s.charAt(i-1) && p.charAt(j-2) != '.'){
                        dp[i][j] = dp[i][j-2];
                    } else{
                        dp[i][j] = (dp[i-1][j] || dp[i][j-1] || dp[i][j-2]);
                        /*
                        dp[i][j] = dp[i-1][j]   // 多个字符匹配的情况
                        dp[i][j] = dp[i][j-1]   // 单个字符匹配的情况
                        dp[i][j] = dp[i][j-2]   // 没有匹配的情况
                        */
                    }
                } 
            }
        }
        return dp[s.length()][p.length()];
    }
}
```

#### 11. 盛最多水的容器

利用双指针法进行求解，记录最大的面积。

判断`height[left]`和`height[right]`的大小，将较小者向中间靠拢。

```java
class Solution {
    public int maxArea(int[] height) {
        // 定义left 和 right 双指针
        int left = 0, right = height.length - 1;
        int ans = 0;    // 用于存储结果
        while(left <= right) {
            int temp = (right - left) * Math.min(height[left], height[right]);
            if (temp > ans){
                ans = temp;
            }
            // 将双指针指向的值较小的向中间移动
            if (height[left] < height[right]){
                left ++;
            } else{
                right --;
            }
        }
        return ans;
    }
}
```

#### 15. 三数之和

利用双指针法进行求解。

1. 首先对数组进行排序；
2. 利用顺序`i`对数组进行遍历，然后`left`定义第`i+1`位置，`right`定义第`len-1`位置。
3. 向中间移动

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        // 数组长度小于3则直接返回
        if (nums == null || nums.length < 3){
            return result;
        }
        Arrays.sort(nums);
        for (int i = 0; i < nums.length-2; i ++){
            if (nums[i] > 0){   // 如果当前数字打羽0，则三数之和一定大于0，所以结束循环
                break;
            }
            // 去重
            if ( i > 0 && nums[i] == nums[i-1]){
                continue;
            }
            // 定义双指针
            int left = i + 1, right = nums.length - 1;
            while (left < right){
                // 和小于0则将left右移
                if (nums[i] + nums[left] + nums[right] < 0){
                    left ++;
                // 和大于0则将right左移
                } else if (nums[i] + nums[left] + nums[right] > 0){
                    right --;
                } else{
                    // 和等于0
                    result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                    // 进行去重
                    while (left < right && nums[left] == nums[left + 1]){
                        left ++;
                    }
                    while (left < right && nums[right] == nums[right - 1]){
                        right --;
                    }
                    left ++;
                    right --;
                }
            }
        }
        return result;
    }
}
```

#### 17. 电话号码的字母组合

首先使用哈希表存储每个数字对应的所有可能的字母，然后进行回溯操作。

回溯过程中维护一个字符串，表示已有的字母排列（如果未遍历完电话号码的所有数字，则已有的字母排列是不完整的）。该字符串初始为空。每次取电话号码的一位数字，从哈希表中获取数字对应的所有可能的字母，并将其中的一个字母插入到已有的字母排列后面，然后继续处理电话号码的后一位数字，直到处理完电话号码中的所有数字，即得到一个完整的字母排列。然后进行回退操作，遍历其余的字母排序。

```java
class Solution {
    public List<String> letterCombinations(String digits) {
        List<String> combinations = new ArrayList<String>();
        if (digits.length() == 0){
            return combinations;
        }
        // 利用哈希表存储每个数字对应的所有可能的字母，然后进行回溯操作
        Map<Character, String> phoneMap = new HashMap<Character, String>() {{
            put('2', "abc");
            put('3', "def");
            put('4', "ghi");
            put('5', "jkl");
            put('6', "mno");
            put('7', "pqrs");
            put('8', "tuv");
            put('9', "wxyz");
        }};
        backtrack(combinations, phoneMap, digits, 0, new StringBuffer());
        return combinations;
    }
    public void backtrack(List<String> combinations, Map<Character, String> phoneMap, String digits, int index, StringBuffer combination){
        if (index == digits.length()){  // 如果处理到电话号码中的所有数字，即得到一个完整的字母排序
            combinations.add(combination.toString());
        } else{
            char digit = digits.charAt(index);
            // 当前数字对应的字符串的长度
            String letters = phoneMap.get(digit);
            int lettersCount = letters.length();
            for (int i = 0; i < lettersCount; i ++){
                combination.append(letters.charAt(i));
                backtrack(combinations, phoneMap, digits, index + 1, combination);  // 进行递归
                combination.deleteCharAt(index);    // 使用完后则进行递归删除
            }
        }
    }
}
```

#### 19. 删除链表的倒数第N个节点

```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        // 令头节点为空以方便处理要删除第一个节点的情况
        ListNode dummy = new ListNode(0, head);
        int length = getLength(head);
        ListNode cur = dummy;
        // 遍历至要删除的节点之前
        for (int i = 1; i < length - n + 1; ++i){
            cur = cur.next;
        }
        // 删除节点
        cur.next = cur.next.next;
        ListNode ans = dummy.next;
        return ans;
    }

    public int getLength(ListNode head){
        // 先遍历一遍得到长度
        int length = 0;
        while(head != null){
            length ++;
            head = head.next;
        }
        return length;
    }
}
```

#### 20. 有效的括号

 利用栈进行判断：

若为`(,{,[`，则将对应的`),},]`添加进栈中；

否则，如果栈为空或者出栈元素不匹配，则说明括号无效。

```java
class Solution {
    public boolean isValid(String s) {
        Stack<Character> stack = new Stack<Character>();
        for (int i = 0; i < s.length(); i ++){
            char cur = s.charAt(i);
            // 若为(, {, [, 则让对应元素入栈
            if (cur == '('){
                stack.push(')');
            } else if (cur == '{'){
                stack.push('}');
            } else if (cur == '['){
                stack.push(']');
            // 否则，栈空或出栈元素不匹配则返回false
            } else if (stack.empty() || cur != stack.pop()){
                return false;
            }
        }
        return stack.empty();
    }
}
```

#### 21. 合并两个有序链表

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        // head 确定链表头
        ListNode head = new ListNode(-1);
        // tail 用于更新
        ListNode tail = head;
        while (l1 != null && l2 != null){
            if (l1.val < l2.val){
                tail.next = new ListNode(l1.val);
                l1 = l1.next;
            } else{
                tail.next = new ListNode(l2.val);
                l2 = l2.next;
            }
            tail = tail.next;
        }
        // 合并尚未完全合并的链表
        tail.next = (l1 == null) ? l2 : l1;
        return head.next;
    }
}
```

#### 22. 括号生成

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> ans = new ArrayList<String>();
        backtrack(ans, new StringBuffer(), 0, 0, n);
        return ans;
    }
    public void backtrack(List<String> ans, StringBuilder cur, int open, int close, int max){
        /*
        ans: 存储的答案
        cur: 当前字符串
        open: 当前(的数量
        close: 当前)的数量
        max: 最多几个括号
        */

        // 如果满足长度要求了，则添加至ans并返回
        if (cur.length() == 2 * max){
            ans.add(cur.toString());
            return;
        }
        // (的数量小于max则添加
        if (open < max){
            cur.append('(');
            backtrack(ans, cur, open+1, close, max);
            cur.deleteCharAt(cur.length() - 1);
        }
        // )的数量小于(则添加
        if (close < open){
            cur.append(')');
            backtrack(ans, cur, open, close+1, max);
            cur.deleteCharAt(cur.length() - 1);
        }
    }
}
```

#### 23. 合并K个升序链表

```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        // 头尾节点
        ListNode head = new ListNode(-1);
        ListNode tail = head;
        int len = lists.length;
        while (true){
            // 定义一个最小节点和对应的索引
            ListNode min_node = null;
            int min_idx = -1;
            for (int i = 0; i < len; i ++){
                // 循环，找到所有链表最小的头
                if (lists[i] == null){
                    continue;
                }
                if (min_node == null || lists[i].val < min_node.val){
                    min_node = lists[i];
                    min_idx = i;
                }
            }
            if (min_idx != -1){
                tail.next = min_node;
                tail = tail.next;
                lists[min_idx] = lists[min_idx].next;
            } else{ //全为空了则返回
                return head.next;
            }
        }

    }
}
```

#### 31. 下一个排列

字典序算法：

> 1. 从右往左，找出第一个左边小于右边的数，设为list[a]；
> 2. 从右往左，找出第一个大于list[a]的数，设为list[b]；
> 3. 交换list[a]，list[b]；
> 4. 将list[a]后面的数据，从小往大排列

```java
class Solution {
    public void nextPermutation(int[] nums) {
        // 从右往左，找出第一个左边小于右边的数，设为list[a]
        int i = nums.length - 2;
        while(i >= 0 && nums[i] >= nums[i+1]){
            i --;
        }
        if (i >= 0){
            // 从右往左，找出第一个大于list[a]的数，设为list[b]
            int j = nums.length - 1;
            while(j >=0 && nums[j] <= nums[i]){
                j --;
            }
            // 交换list[a], list[b]
            int tmp = nums[i];
            nums[i] = nums[j];
            nums[j] = tmp;
        }
        if (i == -1){ // 如果找不到下一个排列，则进行重排列
            Arrays.sort(nums);
        } else{ // 将list[a]后面的数据，从小往大排列
            Arrays.sort(nums, i+1, nums.length);
        }
    }
}
```

#### 32. 最长有效括号

我们始终保持栈底元素为当前已经遍历过的元素中[最后一个没有被匹配的右括号的下标]，这样的做法主要是考虑了边界条件的处理，栈里其他元素维护左括号的下标：

- 对于每个遇到的(，我们将它的下标放入栈中
- 对于每个遇到的)，我们先弹出栈顶元素表示匹配了当前右括号
  - 如果栈为空，说明当前的右括号为没有被匹配的右括号，我们将其下标放入栈中来更新[最后一个没有被匹配的右括号的下标]
  - 如果栈不为空，当前右括号的下标减去栈顶元素，即为[以该右括号为结尾的最长有效括号的长度]

```java
public class Solution {
    public int longestValidParentheses(String s) {
        int maxans = 0;
        Stack<Integer> stack = new Stack<Integer>();
        stack.push(-1);
        for (int i = 0; i < s.length(); i++) {
            // 对于每个遇到的(，我们将它的下标放入栈中
            if (s.charAt(i) == '(') {
                stack.push(i);
            } else {
                // 对于每个遇到的)，我们先弹出栈顶元素表示匹配了当前右括号
                stack.pop();
                // 如果栈为空，说明当前的右括号为没有被匹配的右括号
                // 我们将其下标放入栈中来更新[最后一个没有被匹配的右括号的下标]
                if (stack.empty()) {
                    stack.push(i);
                } else {
                    // 如果栈不为空，当前右括号的下标减去栈顶元素
                    // 即为[以该右括号为结尾的最长有效括号的长度]
                    maxans = Math.max(maxans, i - stack.peek());
                }
            }
        }
        return maxans;
    }
}
```

#### 39. 组合总和

```java
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> ans = new ArrayList<List<Integer>>();
        backtrack(ans, new ArrayList<Integer>(), 0, candidates, target);
        return ans;
    }

    public void backtrack(List<List<Integer>> ans, List<Integer> cur, int idx, int[] candidates, int target){
        if (idx == candidates.length){
            return;
        }
        if (target == 0){
            ans.add(new ArrayList<Integer>(cur));
            return;
        }
        // 直接跳过
        backtrack(ans, cur, idx+1, candidates, target);
        // 选择当前数
        if (target - candidates[idx] >= 0){
            cur.add(candidates[idx]);
            // 由于可以重复选择某个数，故在此idx不加1 
            backtrack(ans, cur, idx, candidates, target - candidates[idx]);
            cur.remove(cur.size() - 1);
        }
    }
}
```

#### 42. 接雨水

```java
class Solution {
    public int trap(int[] height) {
        int sum = 0;
        int[] max_left = new int[height.length]; // max_left 用于记录每个元素左边的最大值
        int[] max_right = new int[height.length]; // max_right 用户记录每个元素右边的最大值
        // 从左到右动态规划更新max_left
        for (int i = 1; i < height.length; i ++){
            max_left[i] = Math.max(max_left[i-1], height[i-1]);
        }
        // 从右到左动态规划更新max_right
        for (int i = height.length - 2; i >= 0; i --){
            max_right[i] = Math.max(max_right[i+1], height[i+1]);
        }
        for (int i = 1; i < height.length - 1; i ++){
            int min = Math.min(max_left[i], max_right[i]);  // 每个柱子左边和右边最大值的最小值
            if (min > height[i]){
                sum += min - height[i];
            }
        }
        return sum;
    }
}
```

#### 46. 全排列

```java
class Solution {
    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> ans = new ArrayList<List<Integer>>();
        int[] visited = new int[nums.length];   // 用户标记每个元素是否被访问过
        backtrack(ans, new ArrayList<Integer>(), visited, nums);
        return ans;
    }

    public void backtrack(List<List<Integer>> ans, List<Integer> cur, int[] visited, int[] nums){
        if (cur.size() == nums.length){
            ans.add(new ArrayList<Integer>(cur));
            return;
        }
        // 遍历所有元素
        for (int i = 0; i < nums.length; i ++){
            if (visited[i]==1){ // 如果该元素应该被访问过则跳过
                continue;
            }
            // 标记访问
            visited[i] = 1;
            cur.add(nums[i]);
            backtrack(ans, cur, visited, nums);
            // 标记未被访问
            visited[i] = 0;
            cur.remove(cur.size() - 1);
        }
    }
}
```

#### 48. 旋转图像

```java
class Solution {
    public void rotate(int[][] matrix) {
        int n = matrix.length;
        int[][] matrix_new = new int[n][n];
        for (int i = 0; i < n; i++){
            for (int j = 0; j < n; j++){
                // 第i行的第j个位置，会旋转至第n-1-i列的第j个位置
                matrix_new[j][n-1-i] = matrix[i][j];
            }
        }
        for (int i = 0; i < n; i ++){
            for (int j = 0; j < n; j ++){
                matrix[i][j] = matrix_new[i][j];
            }
        }
    }
}
```

#### 49. 字母异位词分组

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<String, List<String>>();
        for (String str:strs){
            char[] array = str.toCharArray();
            Arrays.sort(array);
            String key = new String(array);
            // getOrDefault()方法获取指定key对应的value，如果找不到key，则返回设置的默认值
            List<String> list = map.getOrDefault(key, new ArrayList<String>());
            list.add(str);
            map.put(key, list);
        }
        return new ArrayList<List<String>>(map.values());
    }
}
```

#### 53. 最大子序列和

1. 定义数组的含义。`dp[i]`表示0~i时数组分片的最大子序列之和；
2. 找出数组元素间的关系式。`dp[i]=max(dp[i-1]+nums[i], nums[i])`；
3. 初始值。`dp[0]=nums[0]`

```java
class Solution {
   public int maxSubArray(int[] nums) {
        // 1. 定义数组的函数。dp[i]表示0~i时数组分片的最大子序列之和
        int[] dp = new int[nums.length];
        // 3. 初始值
        dp[0] = nums[0];
        int max = dp[0];
        for (int i = 1; i < nums.length; i ++){
            // 2. 数组元素间的关系式
            dp[i] = Math.max(dp[i-1] + nums[i], nums[i]);
            // 实时维护一个max
            if (dp[i] > max){
                max = dp[i];
            }
        }
        return max;
    }
}
```

对内存进行优化：

```java
class Solution {
   public int maxSubArray(int[] nums) {
        int sum = nums[0];
        int max = nums[0];
        for (int i = 1; i < nums.length; i ++){
            sum = Math.max(sum + nums[i], nums[i]);
            if (sum > max){
                max = sum;
            }
        }
        return max;
    }
}
```

#### 55. 跳跃游戏

```java
class Solution {
    public boolean canJump(int[] nums) {
        // 实时维护最远可以到达位置
        int rightMost = 0;
        for (int i = 0; i < nums.length; i ++){
            // 对于当前遍历到的位置x，如果它在最远可以到达的位置范围内
            if (rightMost >= i){
                // 更新最远可以到达的位置
                if (nums[i] + i > rightMost){
                    rightMost = nums[i] + i;
                }
                // 如果最远可以到达的位置大于等于数组中的最后一个位置
                // 那么说明最后一个位置可以到达，直接返回答案true
                if (rightMost >= nums.length - 1){
                    return true;
                }
            }
            
        }
        return false;
    }
}
```

#### 56. 合并区间

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        if (intervals.length == 0){
            return new int[0][2];
        }
        // 将列表中的区间按照左端点升序排序
        Arrays.sort(intervals, new Comparator<int[]>(){
            public int compare(int[] interval1, int[] interval2){
                return interval1[0] - interval2[0];
            }
        });
        // 用数组merge存储最终答案
        List<int[]> merged = new ArrayList<int[]>();
        for(int i = 0; i < intervals.length; i++){
            int L = intervals[i][0], R = intervals[i][1];
            // 如果当前区间的左端点在数组merged中最后一个区间的右端点之后，那么它们不会重合；
            // 直接将这个区间加入数组merged末尾
            if (merged.size() == 0 || merged.get(merged.size() - 1)[1] < L){
                merged.add(new int[]{L, R});
            } else{
                // 用当前区间的右端点更新数组merged中最后一个区间的右端点
                // 将其置为两者较大值
                merged.get(merged.size() - 1)[1] = Math.max(merged.get(merged.size() - 1)[1], R);
            }
        }
        return merged.toArray(new int[merged.size()][]);
    }
}
```

#### 62. 不同路径

1. 定义数组的含义。`dp[i][j]`表示走到i行j列时的可能性；
2. 找出数组元素间的关系式。`dp[i][j]=dp[i-1][j]+dp[i][j-1]`；
3. 初始值。`dp[i][0]=1`和`dp[0][j]=1`

```java
class Solution {
    public int uniquePaths(int m, int n) {
        // 定义数组的含义
        int dp[] = new int[n];
        // 初始值
        for (int j = 0; j < n; j ++){
            dp[j] = 1;
        }
        // 数组元素间的关系式
        for (int i = 1; i < m; i ++){
            for (int j = 1; j < n; j ++){
                dp[j] = dp[j-1] + dp[j];
            }
        }
        return dp[n-1];
    }
}
```

#### 64. 最小路径和

1、 定义数组的含义。`dp[i][j]`代表到位置i,j时路径的最小值；

2、 找出数组元素之间的关系式。`dp[i][j]=min(dp[i-1][j], dp[i][j-1])+gird[i][j]`;

3、初始值。`dp[0][j]=gird[0][0]+gird[0][1]+...+gird[0][j]`并且`dp[i][0]=gird[0][0]+gird[1][0]+...+gird[i][0]`。

```java
class Solution {
    public int minPathSum(int[][] grid) {
        int numRow = grid.length;
        int numCol = grid[0].length;
        // 用dp[i][j]代表到位置i,j时路径的最小值
        int dp[][] = new int[numRow][numCol];
        // 初始化的值
        int tmp = 0;
        for (int i = 0; i < numRow; i ++){
            tmp = tmp + grid[i][0];
            dp[i][0] = tmp;
        }
        tmp = 0;
        for (int i = 0; i < numCol; i ++){
            tmp = tmp + grid[0][i];
            dp[0][i] = tmp;
        }

        // 定义数组元素之间的关系式:dp[i][j]=min(dp[i-1][j], dp[i][j-1])+gird[i][j]
        for (int i = 1; i < numRow; i ++){
            for (int j = 1; j < numCol; j++){
                dp[i][j] = Math.min(dp[i-1][j], dp[i][j-1]) + grid[i][j];
            }
        }

        return dp[numRow-1][numCol-1];
    }
}
```

#### 70. 爬楼梯

```java
class Solution {
    public int climbStairs(int n) {
        // 定义数组的含义
        int[] dp = new int[n+1];
        // 初始化
        dp[0] = 1;
        dp[1] = 1;
        for (int i = 2; i < n + 1; i ++){
            // 定义数组间的关系
            dp[i] = dp[i-1] + dp[i-2];
        }
        return dp[n];
    }
}
```

进行内存优化：

```java
class Solution {
    public int climbStairs(int n) {
        // 初始化
        int first = 1;
        int second = 1;
        int tmp;
        for (int i = 2; i < n + 1; i ++){
            tmp = first + second;
            first = second;
            second = tmp;
        }
        return second;
    }
}
```

#### 72. 编辑距离

1. 定义数组的含义。`dp[i][j]`表示当word1到i位置转换成work2到j位置需要最少步数；
2. 数组元素之间的关系式。
   1. 当`word[i-1]==word[j-1]`时，`dp[i][j]=dp[i-1][j-1]`；
   2. 否则，`dp[i][j]=min(dp[i-1][j-1],dp[i][j-1],dp[i-1][j])+1`
3. 初始化。`dp[0][j]=Σj, dp[i][0]=Σi`。

```java
class Solution {
    public int minDistance(String word1, String word2) {
        // 1. 定义数组的含义
        int len1 = word1.length();
        int len2 = word2.length();
        int dp[][] = new int[len1+1][len2+1];
        // 2. 对数组进行初始化
        for (int i = 1; i <= len1; i++){
            dp[i][0] = dp[i-1][0] + 1;
        }
        for (int j = 1; j <= len2; j++){
            dp[0][j] = dp[0][j-1] + 1;
        }
        // 3. 定义数组之间的关系式
        for (int i = 1; i <= len1; i++){
            for (int j = 1; j <= len2; j++){
                if (word1.charAt(i-1) == word2.charAt(j-1)){
                    dp[i][j] = dp[i-1][j-1];
                } else{
                    dp[i][j] = Math.min(Math.min(dp[i-1][j-1], dp[i-1][j]), dp[i][j-1]) + 1;
                }
            }
        }
        return dp[len1][len2];
    }
}
```

进行优化

```java
class Solution {
    public int minDistance(String word1, String word2) {
        /*
            由于动态规划的计算无需用到二维矩阵，只需要用到临近的一些元素
            故仅开一个一维矩阵并开一些临时变量保存中间结果
        */
        // 1. 定义数组的含义
        int len1 = word1.length();
        int len2 = word2.length();
        int dp[] = new int[len2+1];
        // 2. 对数组进行初始化
        for (int j = 1; j <= len2; j++){
            dp[j] = dp[j-1] + 1;
        }
        // 3. 定义数组之间的关系式
        for (int i = 1; i <= len1; i++){
            int temp = dp[0];
            // 相当于对之前的二维矩阵进行初始化
            dp[0] = i;
            for (int j = 1; j <= len2; j++){
                int pre = temp; // pre相当于dp[i-1][j-1]
                temp = dp[j]; // temp相当于dp[i-1][j]
                if (word1.charAt(i-1) == word2.charAt(j-1)){
                    dp[j] = pre;
                } else{
                    dp[j] = Math.min(Math.min(pre, temp), dp[j-1]) + 1;
                }
            }
        }
        return dp[len2];
    }
}
```

#### 75. 颜色分类

```java
class Solution {
    public void sortColors(int[] nums) {
        int left = 0, right = nums.length - 1;
        for (int i = 0; i < nums.length; i ++){
            while(i <= right && nums[i] == 2){
                int tmp = nums[right];
                nums[right] = nums[i];
                nums[i] = tmp;
                right --;
            }
            if (nums[i] == 0){
                int tmp = nums[left];
                nums[left] = nums[i];
                nums[i] = tmp;
                left ++;
            }
        }
    }
}
```

#### 78. 子集

```java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> ans = new ArrayList<List<Integer>>();
        backtrack(ans, new ArrayList<Integer>(), nums, 0);
        return ans;
    }

    public void backtrack(List<List<Integer>> ans, List<Integer> cur, int[] nums, int idx){
        ans.add(new ArrayList<Integer>(cur));

        for (int i = idx; i < nums.length; i++){
            cur.add(nums[i]);
            backtrack(ans, cur, nums, i+1);
            cur.remove(cur.size()-1);
        }
    }
}
```

#### 79. 单词搜索

```java
class Solution {
    private boolean[][] visited;

    private int[][] direction = {{-1, 0}, {0, -1}, {0, 1}, {1, 0}};
    private int m;
    private int n;
    private String word;
    private char[][] board;
    public boolean exist(char[][] board, String word) {
        m = board.length;
        if (m == 0){
            return false;
        }
        n = board[0].length;
        visited = new boolean[m][n];
        this.word = word;
        this.board = board;
        // 对所有位置进行遍历
        for (int i = 0; i < m; i++){
            for (int j = 0; j < n; j++){
                if (backtrack(i, j, 0)){
                    return true;
                }
            }
        }
        return false;
    }

    private boolean backtrack(int i, int j, int start){
        // i, j表示当前遍历到的board位置
        // start表示当前遍历到的word中的位置

        // 如果当前位置已经是最后一个位置
        if (start == word.length() - 1){
            return board[i][j] == word.charAt(start);
        }
        // 如果当前位置匹配
        if (board[i][j] == word.charAt(start)){
            visited[i][j] = true;
            // 遍历4个方向的位置
            for (int k = 0; k < 4; k++){
                int newX = i + direction[k][0];
                int newY = j + direction[k][1];
                // 如果newX和newY在board内并且没有被访问过
                if (inArea(newX, newY) && !visited[newX][newY]){
                    if (dfs(newX, newY, start+1)){
                        return true;
                    }
                }
            }
            visited[i][j] = false;
        }
        return false;
    }

    // 判断当前位置是否在board内
    private boolean inArea(int x, int y){
        return x >= 0 && x <m && y >=0 && y < n;
    }
}
```

#### 94. 二叉树的遍历

```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> ans = new ArrayList<Integer>();
        inorder(root, ans);
        return ans;
    }
    public void inorder(TreeNode root, List<Integer> ans){
        if (root == null){
            return;
        }
        inorder(root.left, ans);
        ans.add(root.val);
        inorder(root.right, ans);
    }
}
```

#### 96. 不同的二叉搜索树

1. 定义数组的含义。
   1. $$G(i)$$为$n$个节点存在二叉搜索树的个数
   2. $f(i)$为以$i$为根的二叉搜索树的个数
2. 数组元素之间的关系。
   1. $G(n)=f(1)+f(2)+f(3)+f(4)+...+f(n)$
   2. $f(i) = G(i-1)*G(n-i)$
   3. $G(n)=G(0)*G(n-1)+G(1)*G(n-2)+...+G(n-1)G(0)$
3. 初始化。
   1. $G(0)=1$
   2. $G(1)=1$

```java
class Solution {
    public int numTrees(int n) {
        int[] dp = new int[n+1];
        dp[0] = 1;
        dp[1] = 1;
        for (int i = 2; i < n + 1; i ++){
            for (int j = 1; j < i + 1; j ++){
                dp[i] += dp[j-1]*dp[i-j];
            }
        }
        return dp[n];
    }
}
```

#### 101. 对称二叉树

```java
class Solution {
    private boolean ans = true;
    public boolean isSymmetric(TreeNode root) {
        return traverse(root, root);
    }
    public boolean traverse(TreeNode left, TreeNode right){
        if (left == null && right == null){
            return true;
        }
        if (left == null || right == null){
            return false;
        }
        return left.val == right.val && traverse(left.left, right.right) && traverse(left.right, right.left);
    }
}
```



#### 102. 二叉树的层次遍历

1. 首先根元素入队；
2. 当队列不为空的时候
   1. 求当前队列中的长度si;
   2. 依次从队列中取si个元素进行扩展，然后进行下一次迭代

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> ans = new ArrayList<>();
        Queue<TreeNode> q = new LinkedList<>();
        // 根节点元素入队
        q.offer(root);
        if (root == null){
            return ans;
        }
        while(!q.isEmpty()){
            // 求当前队列中的长度
            int curLevelSize = q.size();
            List<Integer> cur = new ArrayList<>();
            // 一次从队列中取si个元素进行扩展，然后进行下一次迭代
            for (int i = 0; i < curLevelSize; i++){
                TreeNode node = q.poll();
                cur.add(node.val);
                if (node.left != null) q.offer(node.left);
                if (node.right != null) q.offer(node.right);
            }
            ans.add(cur);        
        }
        return ans;
    }
}
```

#### 105. 从前序与中序遍历序列构造二叉树

```java
class Solution {
    /* 主函数 */
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        return build(preorder, inorder, 0, preorder.length-1, 0, inorder.length-1);
    }
    /* 根据preorder和inorder构造树*/
    public TreeNode build(int[] preorder, int[] inorder, int preLeft, int preRight, int inLeft, int inRight){
        // bad case
        if (preLeft > preRight || inLeft > inRight){
            return null;
        }

        // 前序遍历的第一个元素为当前root
        int preVal = preorder[preLeft];
        int inIdx = -1;
        // 找到其在中序遍历中对应的位置
        for (int i = inLeft; i <= inRight; i ++){
            if (inorder[i] == preVal){
                inIdx = i;
            }
        }
        TreeNode root = new TreeNode(preVal);
        // 递归调用左右子树
        root.left = build(preorder, inorder, preLeft+1, preLeft+inIdx-inLeft, inLeft, inIdx-1);
        root.right = build(preorder, inorder, preLeft+inIdx-inLeft+1, preRight, inIdx+1, inRight);
        return root;
    }
}
```

从中序和后序遍历序列构造二叉树

```java
class Solution {
    /* 主函数 */
    public TreeNode buildTree(int[] inorder, int[] postorder) {
        return build(inorder, postorder, 0, inorder.length-1, 0, postorder.length-1);
    }
    /* 根据preorder和inorder构造树*/
    public TreeNode build(int[] inorder, int[] postorder, int inLeft, int inRight, int postLeft, int postRight){
        // bad case
        if (inLeft > inRight || postLeft > postRight){
            return null;
        }
        // 后序遍历最后一个元素为当前root
        int postVal = postorder[postRight];
        int inIdx = -1;
        // 找到中序遍历中对应的位置
        for (int i = inLeft; i <= inRight; i++){
            if(inorder[i] == postVal){
                inIdx = i;
                break;
            }
        }
        TreeNode root = new TreeNode(postVal);
        // 递归调用左右子树
        root.left = build(inorder, postorder, inLeft, inIdx-1, postLeft, postLeft+inIdx-inLeft-1);
        root.right = build(inorder, postorder, inIdx+1, inRight, postLeft+inIdx-inLeft, postRight-1);
        return root;
    }
}
```

#### 114. 二叉树展开为链表

```java
class Solution {
    public void flatten(TreeNode root) {
        if (root == null){
            return;
        }
        flatten(root.left);
        flatten(root.right);

        /**** 后续遍历位置 ****/
        // 1. 左右子树已经被拉平成一条链表
        TreeNode left = root.left;
        TreeNode right = root.right;

        // 2. 将左子树作为右子树
        root.left = null;
        root.right = left;

        // 3. 将原先的右子树接到当前右子树的末端
        TreeNode p = root;
        while(p.right != null){
            p = p.right;
        }
        p.right = right;
    }
}
```

#### 226. 翻转二叉树

```java
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if (root == null){
            return null;
        }
        TreeNode temp = root.left;
        root.left = root.right;
        root.right = temp;

        invertTree(root.left);
        invertTree(root.right);

        return root;
    }
}
```

#### 297. 二叉树的序列化和反序列化

```java
public class Codec {
    private String SEP = ",";
    private String NULL = "#";
    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        // bad case
        if (root == null) return "";
        // 初始化队列
        Queue<TreeNode> q = new LinkedList<>();
        // 将root加入队列
        q.offer(root);
        StringBuilder sb = new StringBuilder();

        while (!q.isEmpty()){
            TreeNode cur = q.poll();
            /* 层级遍历位置 */
            if (cur == null){
                sb.append(NULL).append(SEP);
                continue;
            }
            sb.append(cur.val).append(SEP);
            /***************/
            q.offer(cur.left);
            q.offer(cur.right);
        } 
        // 转换为string
        return sb.toString();
    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        if (data.isEmpty()) return null;
        String[] nodes = data.split(SEP);
        // 第一个元素就是root 的值
        TreeNode root = new TreeNode(Integer.parseInt(nodes[0]));

        // 队列q 记录父节点，将root加入队列
        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);
        for (int i = 1; i < nodes.length;){
            // 队列中存的都是父节点
            TreeNode parent = q.poll();
            // 父节点对应的左侧子节点的值
            String left = nodes[i++];
            if (!left.equals(NULL)){
                parent.left = new TreeNode(Integer.parseInt(left));
                q.offer(parent.left);
            } else{
                parent.left = null;
            }
            // 父节点对应的右侧右子节点的值
            String right = nodes[i++];
            if (!right.equals(NULL)){
                parent.right = new TreeNode(Integer.parseInt(right));
                q.offer(parent.right);
            }
        }
        return root;
    }
}
```

#### 543. 二叉树的直径

```java
class Solution {
    private int ans = 1;
    public int diameterOfBinaryTree(TreeNode root) {
        traverse(root);
        // 直径等于路径经过节点数的最大值减1
        return ans - 1;
    }
    public int traverse(TreeNode root){
        if (root == null){
            return 0;
        }
        int L = traverse(root.left);    // 左儿子为根的子树的深度
        int R = traverse(root.right);   // 右儿子为根的子树的深度
        ans = Math.max(ans, L+R+1);     // 计算d_node即L+R+1并更新ans
        return Math.max(L, R) + 1;      // 返回该节点为根的子树的深度
    }
}
```

#### 617. 合并二叉树

```java
class Solution {
    public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
        if (root1 == null) return root2;
        if (root2 == null) return root1;
        TreeNode merged = new TreeNode(root1.val + root2.val);
        merged.left = mergeTrees(root1.left, root2.left);
        merged.right = mergeTrees(root1.right, root2.right);
        return merged;
    }
}
```

