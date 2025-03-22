---
author: "LongWei"
title: "LeetCodeHot100刷题笔记"
date: "2024-09-18"
tags: ["Hot100"]
categories: ["LeetCode"]
aliases: ["leetcode_hot100"]
ShowToc: true
TocOpen: true
---







## 热题100

### 哈希

##### [1. 两数之和](https://leetcode.cn/problems/two-sum/)

给定： 数组nums 和target

要求：找出两元素之和为target的数组下标



暴力：

```python
for i in range(len(nums)):
    for j in range(i+1, len(nums)):
        if ... 
# O(n^2)
```

题解：

问题分解 =>  给定x 寻找 target-x(这是复杂度的主要部分)

用额外结构记录

```python
hashtable = dict()
for i in range(len(nums)):
    if nums[i] in hashtable:
        return [i, hashtable[nums[i]]]
    else:
        hashtable[nums[i]] = i
# 保存遍历过的元素，参考过去
```



##### [49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/)

```
输入: strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
输出: [["bat"],["nat","tan"],["ate","eat","tea"]]
```

思路：condition => [...]

HashTable,  Key=排序后的Str, Value=List\<String>

```java
HashMap<String, List<String>> hashTable = new HashMap<>();
for(String str : strs){
    char[] charArray = str.toCharArray(); // 将字符串转换为字符数组
    Arrays.sort(charArray); // 对字符数组进行排序
    String orderStr = new String(charArray);
    if (hashTable.containsKey(orderStr)) {
        hashTable.get(orderStr).add(str);
    }else {
        hashTable.put(orderStr, new ArrayList<>());
        hashTable.get(orderStr).add(str);
    }
}

List<List<String>> res = new ArrayList<>();

for (Map.Entry<String, List<String>> entry : hashTable.entrySet()) {
    res.add(entry.getValue());
}
return res;
```





##### [128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)

```java
HashSet<Integer> hashSet = new HashSet<>();
// 去重复，并且查询O(1)
for (int num : nums) {
    hashSet.add(num);
}

List<Integer> start_nums = new ArrayList<>();

// 找所有为起点的数
for (Iterator<Integer> it = hashSet.iterator(); it.hasNext(); ) {
    int num = it.next();
    if (!hashSet.contains(num-1)){
        start_nums.add(num);
    }
}

// 遍历起点x, 遍历 x+1, x+2, ...
int longest_length = 0;
for(int start : start_nums){
    int current_length = 1;
    int start_i = start;
    while (hashSet.contains(start_i+1)) {
        current_length += 1;
        start_i += 1;
    }
    if (current_length > longest_length){
        longest_length = current_length;
    }
}
return longest_length;
}
```



### 双指针

##### [283. 移动零](https://leetcode.cn/problems/move-zeroes/)

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的**相对顺序**(原始数组位置，非不是重新排序)。

**请注意** ，必须在不复制数组的情况下原地对数组进行操作。

```java
if (nums.length == 1){
    return;
}
int i = 0;
int j = 1;
for (;j<nums.length;j++) {
    if (nums[i] == 0 && nums[j] != 0){
        nums[i] = nums[j];
        nums[j] = 0;
    }
    while (i<=j) {
        if (nums[i] != 0) {
            i+=1;
        } else {
            break;
        }
    }
}
for (;i<nums.length;i++) {
    nums[i] = 0;
}
```



##### [11. 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)

给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。

找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

**说明：**你不能倾斜容器。

```java
// 左右双指针，area = h * w, h限制于最矮的边 <= 漏水
int i = 0;
int j = height.length - 1;
int max = 0;
while (i < j) {
    int h = 0;
    if (height[i] < height[j]) {
        h = height[i];
    } else {
        h = height[j];
    }
    int area = h * (j - i);
    if (area > max) {
        max = area;
    }
    if (height[i] < height[j]) {
        i += 1;
    } else {
        j -= 1;
    }
}
return max;
```



##### [15. 三数之和](https://leetcode.cn/problems/3sum/)

给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请你返回所有和为 `0` 且不重复的三元组。



⭐ 分析： 

先排序，递增

 1. 固定a => 遍历b 则 c = - a - b

 2.  ```java
          a
     [-4,-1,-1,0,1,2]
             b     c
     // 若b向右，则c向左，因为 c = - a - b，且nums递增, b增大 => c减小
              
     // e.g.
       a                         a 
     [-3,-1,0,1,2,3,4,5]    => [-3,-1,0,1,2,3,4,5]
          b         c ✔️              b       c||| 
     c不比从最右侧开始     
     ```

	1. 

```java
// 固定a, b => c !!! 由于 nums[b]<=nums[c] 并且nums递增 c不比每次从末尾开始，因为b向右，则c向左。
Arrays.sort(nums);
ArrayList<List<Integer>> res = new ArrayList<>();
for (int a=0; a<nums.length-2; a++) {
    if (a-1>=0 && nums[a-1] == nums[a]) {
        continue;
    }
    if (nums[a] > 0){
        break;
    }
    // 最右侧
    int c = nums.length-1;
    for (int b=a+1; b<nums.length-1; b++) {
        if (b-1>a && nums[b] == nums[b-1]) {
            // b+=1;
            continue;
        }
        int target = -nums[a] - nums[b];

        while (b < c && target <= nums[c]) {
            if (nums[a] + nums[b] + nums[c] == 0) {
                List<Integer> temp = new ArrayList<>();
                temp.add(nums[a]);
                temp.add(nums[b]);
                temp.add(nums[c]);
                res.add(temp);
                break;
            }
            c -= 1;
        }
    }
}
return res;
```



##### [42. 接雨水](https://leetcode.cn/problems/trapping-rain-water/)

局部化 => 每个单位的储水量

- 暴力   |_|  查每个单位的左右最大边，看看这个地方是否有存水的资格

```java
// 暴力 => 分解为局部某小块[单位长度]聚水量
if (height.length < 3)
    return 0;
int ans = 0;
for (int i=1; i<height.length-1; i++) {
    int max_left = 0;
    // 左侧最大值
    for (int left=0; left<i; left++) {
        if (max_left < height[left])
            max_left = height[left];
    }
    if (max_left < height[i]) {
        continue;
    }
    int max_right = 0;
    // 右侧最大值
    for (int right=height.length-1; right>i; right--) {
        if (max_right < height[right])
            max_right = height[right];
    }
    if (max_left>height[i] && max_right>height[i]){
        ans += Math.min(max_left, max_right) - height[i];
    }
}
return ans;
```

预先计算，避免重复。 预先把每个位置的左右最大高计算出来，直接判断是否被两高边夹住 => 能存水

```java
if (height.length < 3)
            return 0;
// 预先存储每个位置的左右最大高，判断该位置是否能蓄水
int[] per_pos_left_max = new int[height.length];
per_pos_left_max[0] = 0;
int[] per_pos_right_max = new int[height.length];
per_pos_right_max[height.length-1] = 0;

for (int i=1; i<height.length; i++) {
    per_pos_left_max[i] = Math.max(per_pos_left_max[i-1], height[i-1]);
}
for (int i=height.length-2; i >= 0; i--) {
    per_pos_right_max[i] = Math.max(per_pos_right_max[i+1], height[i+1]);
}
int ans = 0;
for (int i=0; i<height.length; i++) {
    int min = Math.min(per_pos_left_max[i], per_pos_right_max[i]);
    if (min > height[i])
        ans += min - height[i];
}

return ans;
```



### 滑动窗口

##### [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

滑动窗口 => a[???bca]bcbb => a???b[cab]cbb

左边界直接滑动窗口内第一个重复位置的右侧【左边界右滑一大步】

```java
char[] chars = s.toCharArray();
HashSet<Character> hashSet = new HashSet<>();
int res = 0;
for(int left=0, right=0; right<s.length(); right++) {
    char ch = chars[right];

    while(hashSet.contains(ch)) {
        hashSet.remove(chars[left]);
        left+=1;
    }

    hashSet.add(ch);
    res = Math.max(res, right - left + 1);
}

return res;
```



##### [438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)

描述： 给定s,和目标p，找出在s中所有由p随机组合的字串起点

思路：

1. 【暴力】遍历数组s，每个元素作为起始点，sortStr([start, start+target.length()-1]).equals(sortStr(target)) 
2. 【滑动窗口】 DNA转录？ 1️⃣ 还是遍历整个s数组 2️⃣ 比较方法变为 当前窗口元素出现次数的词频表

```java
List<Integer> ans = new ArrayList<>();

// 意外情况
if (s.length() < p.length()) {
    return ans;
}

// 词频表 index-value == alpha-count
int[] sCount = new int[26];
int[] pCount = new int[26];
for (int i = 0; i < p.length(); i++) {
    sCount[s.charAt(i) - 'a'] += 1;
    pCount[p.charAt(i) - 'a'] += 1;
}
// 首个异构词
if (Arrays.equals(sCount, pCount)) {
    ans.add(0);
}

int left = 0;
int right = p.length()-1;
for(int i=0; i<s.length()-p.length(); i++) {
    // 左侧出界
    sCount[s.charAt(left) - 'a'] -= 1;
    left += 1;
    // 右边界添加新元素
    right += 1;
    sCount[s.charAt(right) - 'a'] += 1;

    if (Arrays.equals(sCount, pCount)) {
        ans.add(left);
    }
}

return ans;
```



### 字串

##### [560. 和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)

给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回 *该数组中和为 `k` 的子数组的个数* 。

子数组是数组中元素的连续非空序列。

思路：

统计个数而非位置 => 前缀和                    

[[???]---?]  利用前缀和-前缀和的差 代表之前的子列和

例如： nums=[[1,2,3,4,5,6],-2,2,6], k=6

​       preSum=[0,1,3,6,10,15,21,[19,21],<em style="color:red;">27</em>] , 利用HashMap存储子列前缀和和其次数，如次数大于0，说明子列1与子列2前缀和相等=>它们之间的和为0.

​     当preSum=27时，次数num=6, 本身[6]为子列，并且[-2,2,6]也是 

```java
public static int subarraySum(int[] nums, int k) {
    int count = 0;
    // preSum -> count
    HashMap<Integer, Integer> preSum = new HashMap<>();
    preSum.put(0, 1);
    int sum = 0;
    for (int i = 0; i < nums.length; i++) {
        sum += nums[i];
        if (preSum.containsKey(sum - k)) {
            count += preSum.get(sum - k);
        }
        preSum.put(sum, preSum.getOrDefault(sum, 0) + 1);
    }

    return count;
}
```



##### [239.滑动窗口中的最大值](https://leetcode.cn/problems/sliding-window-maximum/)

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

返回 *滑动窗口中的最大值* 。

思路 =>

1.  ?[???]?? 维护一个最大堆，每次删除&添加，记录结果
2.  for(数组)=>{for(窗口大小)=>{挑选出最大的元素}}   -- 暴力遍历 
3.  维护一个单调队列: [max...min] 每次滑动窗口
   1.  循环(将右侧新元素与队列min比较，大了，则删除Last_Min)，简单插入排序=> 因为只要窗口内的最大值，那么小的值没有必要维护，删了还减轻比较开销。
   2. 将最左侧的元素出队

```java
int n = nums.length;
int ans_idx = 0;
int[] ans = new int[n-k+1];

Deque<Integer> deque = new LinkedList<>(); // 存储下标就行
// 第一个窗口
for (int i = 0; i < k; i++) {
    while (!deque.isEmpty() && nums[i] > nums[deque.peekLast()]) {
        deque.removeLast();
    }
    deque.offerLast(i);
}

ans[ans_idx++] = nums[deque.peekFirst()];
// 开始移动
for (int i = k; i < n; i++) {
    while (!deque.isEmpty() && nums[i] > nums[deque.peekLast()]) {
        deque.removeLast();
    }
    deque.offerLast(i);

    while (deque.peekFirst() < i-k+1) {
        deque.removeFirst();
    }
    ans[ans_idx++] = nums[deque.peekFirst()];
}
return ans;
```





##### \[X待写X][76. 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)

给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""` 。  \# 唯一

**示例 1：**

```
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
解释：最小覆盖子串 "BANC" 包含来自字符串 t 的 'A'、'B' 和 'C'。
```

扩张[右边界]与收缩[左边界]

1️⃣利用HashMap进行判断是否重合

```java
public boolean isCovered(Map<Character, Integer> targetMap, Map<Character, Integer> curMap) {
    Set<Character> keySet = targetMap.keySet();
    for (Character key : keySet) {
        Integer tgt = targetMap.getOrDefault(key, 0);
        Integer crt = curMap.getOrDefault(key, 0);
        if (tgt > crt)
            return false;
    }
    return true;
}

public String minWindow(String s, String t) {
    Map<Character, Integer> targetMap = new HashMap<>();
    Map<Character, Integer> curMap = new HashMap<>();

    for (int i = 0; i < t.length(); i++) {
        char c = t.charAt(i);
        targetMap.put(c, targetMap.getOrDefault(c, 0)+1);
    }
    int minLen = Integer.MAX_VALUE; int startLoc = 0;
    int left = 0, right = 0;
    for (right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        curMap.put(c, curMap.getOrDefault(c, 0)+1);
        while (isCovered(targetMap, curMap)) {
            if (minLen > (right - left)){
                minLen = right - left;
                startLoc = left;
            }
            char ch = s.charAt(left);
            curMap.put(ch, curMap.get(ch)-1);
            left += 1;
        }
    }

    if (minLen < Integer.MAX_VALUE) return s.substring(startLoc, startLoc+minLen+1);
    else return "";
}
```





### 普通数组

#### [53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/)

线段树做法

```java
// 例 [-2,4,5,-1][-1,2,3,-4]
// l:{
//	iSum=6,lSum=7,rSum=8,mSum=9
// }
// r:{
//	iSum=0,lSum=4,rSum=1,mSum=5
// }
public class Status {
    public int iSum, lSum, rSum, mSum;

    public Status(int iSum, int lSum, int rSum, int mSum) {
        this.iSum = iSum;  // 区间总和
        this.lSum = lSum;  // 区间从左边界开始最大和
        this.rSum = rSum;  // 区间从右边界开始最大和
        this.mSum = mSum;  // 区间最大和
    }
}

public Status getMaxSum(int[] nums, int start, int end) {
    if (start == end) {
        return new Status(nums[start], nums[start], nums[start], nums[start]);
    }
    int mid = (start + end) >> 1;
    Status l = getMaxSum(nums, start, mid);
    Status r = getMaxSum(nums, mid+1, end);
    return popPush(l, r);
}

// 
public Status pushUp(Status l, Status r) {
    int iSum = l.iSum + r.iSum;  // 区间总和
    // 1. [x?][??] or [xx][x?]
    int lSum = Math.max(l.lSum, l.iSum+r.lSum);  
    // 2. [??][?x] or [?x][xx] 
    int rSum = Math.max(r.rSum, l.rSum+r.iSum);  
    // 1. [?xx?][???] 2. [???][?xx?] 3. [?xx][xx?]
    int mSum = Math.max(Math.max(l.mSum, r.mSum), l.rSum+r.lSum); 
    return new Status(iSum,lSum,rSum,mSum);
}

public int maxSubArray(int[] nums) {
    return getMaxSum(nums, 0, nums.length-1).mSum;
}
```

2️⃣ 每次记录符合个数，避免 重复的对两字串进行判断复合关系

```java
public String minWindow(String s, String t) {
    Map<Character, Integer> targetMap = new HashMap<>();
    Map<Character, Integer> curMap = new HashMap<>();

    for (int i = 0; i < t.length(); i++) {
        char c = t.charAt(i);
        targetMap.put(c, targetMap.getOrDefault(c, 0) + 1);
    }
    int minLen = Integer.MAX_VALUE; int startLoc = 0;
    int curLen = 0; int tgtLen = targetMap.keySet().size();
    int left = 0, right;

    for (right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (targetMap.containsKey(c)) {
            curMap.put(c, curMap.getOrDefault(c, 0)+1);
            if (targetMap.get(c).equals(curMap.get(c))) {
                curLen += 1;
            }
        }
        while (curLen == tgtLen) {
            if (minLen > (right-left)){
                minLen = right-left;
                startLoc = left;
            }
            char ch = s.charAt(left);
            if (curMap.containsKey(ch)) {
                curMap.put(ch, curMap.get(ch)-1);
                if (curMap.get(ch) < targetMap.get(ch))
                    curLen -=1;
            }
            left += 1;
        }
    }

    if (minLen < Integer.MAX_VALUE) return s.substring(startLoc, startLoc+minLen+1);
    else return "";
}
```



#### [41. 缺失的第一个正数](https://leetcode.cn/problems/first-missing-positive/)

```
示例 1：
输入：nums = [1,2,0]
输出：3
解释：范围 [1,2] 中的数字都在数组中。

示例 2：
输入：nums = [3,4,-1,1]
输出：2
解释：1 在数组中，但 2 没有。

示例 3：
输入：nums = [7,8,9,11,12]
输出：1
解释：最小的正数 1 没有出现。

### 缺失的最小的正整数，给定数组nums, 那么目标一定在[1, N+1]中。
例子 
1. [x?????]? => 1
2. [???x??]? =>[-1<x, ...]
3. [??????]x =>N+1  # 给定的数组刚好占满[1-N],那么最小的正整数为N+1
```

暴力： 

将数组放入HashSet(查询时间O(1)=>空间换时间)中，然后检查[1-N]是否在HashSet中，不在返回i，全在返回N+1；



思路2✔️. 

构建hashMap，映射条件为 i=>i+1, 给定下标i，则元素值为i+1; 

```
例：
  nums = [3,4,-1,1]
=>     i: 0,1, 2,3
=>nums = [-1,4,3,1]  i  0, 1,2,3
=>nums = [-1,1,3,4] => [1,-1,3,4]  => i=1处异常 => ans=i+1=2
```



JAVA语法：

```java
// 传递给方法的是引用的拷贝，而不是引用本身
public void rotate(int[] nums, int k) {
	int n = nums.length;
    int[] ans = new int[n];
    // ...
    nums = ans; //❌❌❌ nums的引用是原数组引用的拷贝
}

例：  a -> [???] 
    rotate(a) => rotate(nums) {
    // nums->[???]<-a
}
// nums != a
```





### 矩阵

#### [73. 矩阵置零](https://leetcode.cn/problems/set-matrix-zeroes/)

```java
使用额外标记数组 col,row 标识某列\行是否存在0，刷新为0的col或row
		   col: 0,1,0   row    判断 mark col[0][j]==0 || row[i][0]==0
输入：matrix = [[1,1,1],  0
               [1,0,1],  1
               [1,1,1]]  0
额外空间=> O(M+N)
输出：[[1,0,1],
      [0,0,0],
      [1,0,1]]
    
利用第一行和第一列充当标记数组，并且使用两个变量标识第一列行是否刷为0 空间复杂度O(1)
                1 1 1
 输入：matrix =  1[0,1]
                1[1,1]
```

#### [54. 螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/)

```java
思路： 定义四个边界 top,bottm,left,right
while() {
    1.for ➡️ 并且更新边界 + 判断是否退出
    2.for ⬇️
    3.for ⬅️
    4.for ⬆️
}
```

#### [48. 旋转图像](https://leetcode.cn/problems/rotate-image/)

```java
1 2 3 4     
2 y z 5
3 x k 6
5 6 7 8    
    
分步 先旋转外围 再逐层往内进行旋转
x ➡️ x    
⬆️  ⬇️
x ⬅️ x    
```

```java
int len = matrix.length;
int count = len - 2 > 0?len - 2:1;
for (int i = 0; i < count; i++) {
    int left=i, top=i, right=len-1-i, bottom=len-1-i;
    for (int j = i; j < len-i-1; j++) {
        int temp1, temp2;
        // 1. matrix[top][left]
        temp1 = matrix[top][len-1-i];
        matrix[top][len-1-i] = matrix[i][left];

        // 2. matrix[top][right]
        temp2 = matrix[len-1-i][right];
        matrix[len-1-i][right] = temp1;

        // 3. matrix[bottom][right]
        temp1 = matrix[bottom][i];
        matrix[bottom][i] = temp2;

        // 4. matrix[bottom][left]
        matrix[i][left] = temp1;

        // move
        left += 1;
        top += 1;
        right -= 1;
        bottom -= 1;
    }
}
```

or 

```
123      147        741
456 =.T> 258 =filp> 852 
789      369        963
```



#### [240. 搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/)

编写一个高效的算法来搜索 `*m* x *n*` 矩阵 `matrix` 中的一个目标值 `target` 。该矩阵具有以下特性：

- 每行的元素从左到右升序排列。
- 每列的元素从上到下升序排列。

**有序=>二分查找O(logn)**

```
for int[]row in nums:
	binarySearch(row)
```

**Z字查询**  -- 二分查找树

```java
// small<-medium->large
[1,4,7,11,{15⭐}]                 small<- medium
[2,5,8,12,19]                               ⬇
[3,6,9,16,22]                              large
```





### 链表

⭐多指针⭐

#### [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)

![image-20241127231431488](http://sthda9dn6.hd-bkt.clouddn.com/FuXdTJ06VOfgokxTEMjy9iYRhsSE)

找环的入口 => 

![image-20241127231535959](http://sthda9dn6.hd-bkt.clouddn.com/FozNZ7M_zfZTUTh3cRyRdHbIZtAk)

```python
head => 环入口       记为x
第一次相遇点 => 环入口 记为z
环入口 => 第一次相遇点 记为y

快慢指针 
slow(单步)： x+y
fast(双步)： x+y + n(y+z)  // 多绕了n圈

通过步数建立等式

2(x+y) = x+y + n(y+z)
x+y = n(y+z)
x = n(y+z) - y
x = (n-1)(y+z) + z  # 当只绕一圈相遇时 n=1
x = z

第一次相遇点到入口的距离 == 起点到入口的距离 ⭐⭐⭐
```

```java
public ListNode detectCycle(ListNode head) {
    ListNode fast = head;
    ListNode slow = head;
    while (fast != null && fast.next != null) {
        fast = fast.next.next;
        slow = slow.next;

        if (fast == slow) {
            ListNode index1 = head;
            ListNode index2 = fast;
            while (index1 != index2) {
                index1 = index1.next;
                index2 = index2.next;
            }

            return index1;
        }
    }
    return null;
}
```



#### [160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/)

任务：找到第一个相交点

![image-20240922195913092](http://sthda9dn6.hd-bkt.clouddn.com/FuOt9Xa-A-yghnu9986qECvHxhdM)

```java
// 暴力
// for(nodeA: A)
//   for(nodeB: B)  => nodeA?nodeB
// 利用空间替换时间 => 查找时间利用hashSet O(1)
// All nodeA => hashSet
// for(nodeB:B) nodeB?hashSet
```

填充长度，=> 相等长度

```java
X-?-?-√-√-√
?-?-?-√-√-√
同时遍历
or
从尾部开始？ 知道第一个不相等的Node？   
```

交叉遍历

```java
-lenA-
       -lenC-
-lenB-
// A+C+B => 循环后 可视为 B+A+C
// B+C+A                A+B+C
```



#### [234. 回文链表](https://leetcode.cn/problems/palindrome-linked-list/)

判断对称

![image-20240922201339289](http://sthda9dn6.hd-bkt.clouddn.com/FsQh04lB2Mg2GWh-2m0q62hqYypr)

暴力：

读出来=> array => left,right=> 进行判断

⭐ 快慢指针

```java
// 快慢指针：快一次走一步；慢一次走两步
head: 1-2-3-[3-2-1]-null 偶数
          p      q   
head: 1-2-3-4-[3-2-1]-null  奇数
            p          q

head    - [1-2-3]    
newHead - [1-2-3] // 头插，逆序
判断完
头插回去，进行复原
```



#### 翻转链表

1. 尾插法：顺序构建链表
2. 头插法：逆序构建链表

可以构建空的头结点 => 无需判断第一个节点还是后续节点



#### [138. 随机链表的复制](https://leetcode.cn/problems/copy-list-with-random-pointer/)

给你一个长度为 `n` 的链表，每个节点包含一个额外增加的随机指针 `random` ，该指针可以指向链表中的任何节点或空节点。

构造这个链表的 **[深拷贝](https://baike.baidu.com/item/深拷贝/22785317?fr=aladdin)**。 深拷贝应该正好由 `n` 个 **全新** 节点组成，其中每个新节点的值都设为其对应的原节点的值。新节点的 `next` 指针和 `random` 指针也都应指向复制链表中的新节点，并使原链表和复制链表中的这些指针能够表示相同的链表状态。**复制链表中的指针都不应指向原链表中的节点** 。

⭐利用hash，空间换时间⭐

```
思路：
1. 顺序构建深拷贝链表，利用hashMap (oldNode) -> (new Node)
2. 填充random point 
```



#### [146. LRU 缓存](https://leetcode.cn/problems/lru-cache/)

请你设计并实现一个满足 [LRU (最近最少使用) 缓存](https://baike.baidu.com/item/LRU) 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity` ，则应该 **逐出** 最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

\## 分析:

- 函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行 => 直接定位 要么数组要么hash
- 数量超过 `capacity` ，则应该 **逐出** 最久未使用的关键字

\## 做法

- hashMap \<Key => ObejectAddress>  O(1)定位能力
- 自定义双向链表 head(used)<=>tail(not use)

```java
class LRUCache {
    class BiLinkedList {
        int key,val;
        BiLinkedList prev,next;

        public BiLinkedList() {
        }

        public BiLinkedList(int key, int val) {
            this.key = key;
            this.val = val;
        }
    }

    private Map<Integer, BiLinkedList> map =  new HashMap<>();
    private int capacity; // total
    private int size; // current
    private BiLinkedList head, tail;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.size = 0;
        head = new BiLinkedList();
        tail = new BiLinkedList();
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        // 访问 => 放表头
        if (map.containsKey(key)) {
            BiLinkedList node = map.get(key);
            move2Head(node);
            return node.val;
        } else {
            return -1;
        }
    }

    public void put(int key, int value) {
        // 修改 => 放表头
        if (map.containsKey(key)) {
            map.get(key).val = value;
            move2Head(map.get(key));
        }else {
            // 新增
            if (size >= capacity) {
                BiLinkedList node = deleteTail();// map 也需要删除啊
                map.remove(node.key);
                size -= 1;
            }
            // 插入到表头
            BiLinkedList node = insert2Head(key, value);
            map.put(key, node);
            size += 1;
        }
    }

    public void move2Head(BiLinkedList node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;

        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }

    public BiLinkedList deleteTail() {
        BiLinkedList node = tail.prev;
        tail.prev.prev.next = tail;
        tail.prev = tail.prev.prev;
        return node;
    }

    public BiLinkedList insert2Head(int key, int value) {
        BiLinkedList node = new BiLinkedList(key, value);
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
        return node;
    }
}
```

### 二叉树

#### 翻转二叉树

思路：

1. 递归遍历  - 自上而下
2. 每次遍历某个节点后，将其左右儿子指针调换



#### 对称二叉树

判断二叉树中心轴对称

递归遍历  - 自上而下

双指针： 

1. 左左 | 右右
2. 左右 | 右左

```java
TreeNode q=p=root;

# func start
eq1 = q.val==p.val
eq2 = check(q.left, p.right);
eq3 = check(q.right, p.left);

eq1&&eq2&&eq3
# func end    
```



#### 二叉树直径

任意两结点的边数

思路：

考虑每个结点为枢纽，左侧距离+右侧距离+本身

利用深度

```java

int deep(TreeNode node, int deep):
	if(node == null):
		return 0;
	int leftDeep = deep(node.left);
	int rightDeep = deep(node.right);
	if(...) // 保存最大深度 leftDeep+rightDeep+1
        ...
    return max(leftDeep, rightDeep) + 1;
```



#### 二叉树层序遍历

利用队列

```java
queue.add(root)
while(!queue.isEmpty) {
    node = queue.poll();
    ...
    
}
```



#### 验证二叉搜索树

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FtTjZ9S7R5uo-MFqFDISn0iCa6d-" alt="image-20240927182031344" style="zoom:50%;" />

node.left.val < node.val < node.right.val

思路：

// 递归 => 当前结点 + 左子树 + 右子树

​	该结点大于左子树最大

​	            小于右子树最小

```java
public boolean isValidBST(TreeNode node, long leftMax, long rightLower) {
        if (node == null)
            return true;
        if (node.val <= leftMax || node.val >= rightLower)
            return false;

        return isValidBST(node.left, leftMax, node.val) && isValidBST(node.right, node.val, rightLower);
    }
```



#### [230. 二叉搜索树中第 K 小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/)

思路：二叉搜索树

二叉搜索树中序遍历是升序结果

中序遍历



#### [199. 二叉树的右视图](https://leetcode.cn/problems/binary-tree-right-side-view/)

先右子树=>再访问根结点 =>左子树

=> 使用辅助数组标记每层的输出情况



#### [114. 二叉树展开为链表](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/)

每次把左子树插在右子树上，进行递归

![image-20240927182649806](http://sthda9dn6.hd-bkt.clouddn.com/FkzQ3mBG8fCq_JY-mHuc37DEDQyS)



#### [105. 从前序与中序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

思路：前序第一个是Root

​           按照root把中序分成左子树结点和右子树结点



#### [437. 路径总和 III](https://leetcode.cn/problems/path-sum-iii/)

给定一个二叉树的根节点 `root` ，和一个整数 `targetSum` ，求该二叉树里节点值之和等于 `targetSum` 的 **路径** 的数目。

**路径** 不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。



遍历Tree，并且有target和node.val，存储一下preSum，往下找target-preSum结点

```java
public int dfs(TreeNode node, long preSum, int targetSum) {
    int ret = 0;
    if (node != null) {
        int val = node.val;
        if (val+preSum == targetSum) {
            ret++;
        }
        ret += dfs(node.left, val+preSum,targetSum);
        ret += dfs(node.right, val+preSum,targetSum);
        return ret;
    }
    return ret;
}

public int pathSum(TreeNode root, int targetSum) {
    int ans = 0;
    if (root == null){
        return 0;
    }
    int val = root.val;
    ans += dfs(root, 0, targetSum);
    ans += pathSum(root.left, targetSum);
    ans += pathSum(root.right, targetSum);
    return ans;
}
```





#### [236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)

三种情况(局部化)：

1. pq分布在左右子树
2. root有个是q||p, 左or右子树存在另一个

```java
// 三种
boolean leftok = find(node.left)
boolean rightok = find(node.right)
if (leftok&&rightok)||(node.val==p||node.val==q)&&(leftok||rightok)
```

```java
private TreeNode ans = null;

// 如何控制最近组先 => DFS, 优先从底部开始搜索
// 当最近组先被发现后，次一级组先由于(pq均在一侧)，故不满足if判断。
public boolean dfs(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null)
        return false;
    boolean l = dfs(root.left, p, q);
    boolean r = dfs(root.right, p, q);
    if ((l&&r) || ((root == p||root==q)&&(l||r)))
        ans = root;
    return l || r || (root==q || root==p);
}

// 暴力 => 查询某个节点是否包括俩目标节点 ❌ 最近！！！
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    dfs(root, p, q);
    return ans;
}
```



#### [124. 二叉树中的最大路径和](https://leetcode.cn/problems/binary-tree-maximum-path-sum/)

二叉树中的 **路径** 被定义为一条节点序列，序列中每对相邻节点之间都存在一条边。同一个节点在一条路径序列中 **至多出现一次** 。该路径 **至少包含一个** 节点，且不一定经过根节点。

**路径和** 是路径中各节点值的总和。

给你一个二叉树的根节点 `root` ，返回其 **最大路径和** 。



1. 考虑枢纽结点为中心，两侧子树的正向贡献

```java
int dfs(TreeNode node) {
    if (node == null)
        return 0; 
	int lv = max(dfs(node.left), 0);
    int rv = max(dfs(node.right), 0);
    if(lv+r+rv > ans)
        // save
    return max(lv+r, rv+r);
}
```



### 图论

#### [200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/)

给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围。



考点

=> 邻接矩阵的连通分量(有几个不相连的子图)



BFS

```java
// BFS 判断连通性 => answer == 连通子图个数
public void colorMap(char[][] grid, int i, int j) {
    int h = grid.length;
    int w = grid[0].length;
    if (i<0 || j<0 || i>=h || j>=w || grid[i][j]==0)
        return;

    grid[i][j] = '0';
    colorMap(grid, i-1, j);
    colorMap(grid, i+1, j);
    colorMap(grid, i, j-1);
    colorMap(grid, i, j+1);
}

public int numIslands(char[][] grid) {
    int ans = 0;
    int row = grid.length;
    int col = grid[0].length;
    for (int i = 0; i < row; i++) {
        for (int j = 0; j < col; j++) {
            if (grid[i][j] == 1){
                ans++;
                colorMap(grid, i, j);
            }
        }
    }
    return ans;
}
```



#### [994. 腐烂的橘子](https://leetcode.cn/problems/rotting-oranges/)

在给定的 `m x n` 网格 `grid` 中，每个单元格可以有以下三个值之一：

- 值 `0` 代表空单元格；
- 值 `1` 代表新鲜橘子；
- 值 `2` 代表腐烂的橘子。

每分钟，腐烂的橘子 **周围 4 个方向上相邻** 的新鲜橘子都会腐烂。

返回 *直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 `-1`* 。



⭐ 注意每个烂橘子同一时刻均开始

=> queue : [111], => [2222] => [3333]

BFS遍历



#### [207. 课程表](https://leetcode.cn/problems/course-schedule/)

你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1` 。

在选修某些课程之前需要一些先修课程。 先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]` ，表示如果要学习课程 `ai` 则 **必须** 先学习课程 `bi` 。

- 例如，先修课程对 `[0, 1]` 表示：想要学习课程 `0` ，你需要先完成课程 `1` 。

请你判断是否可能完成所有课程的学习？如果可以，返回 `true` ；否则，返回 `false` 。



方法一：

- 检查拓扑序列是否存在 => 入度表 + map\<edge, List\<neighborEdge>>



方法二：

- DFS，状态0：未访问过，1：遍历到未收录(用来判断是否重复遍历到)，2：遍历到且收入囊中；



#### [208. 实现 Trie (前缀树)](https://leetcode.cn/problems/implement-trie-prefix-tree/)

字典树

```java
class TreeNode {
	boolean isEnd;
    TreeNode[] children;
}

children => [abcd..xyz]
               => [abcd..xyz]
```



### 回溯

组合问题 -- 使用树型结构分析所有可能情况



#### [46. 全排列](https://leetcode.cn/problems/permutations/)

```
输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

<img src="http://sthda9dn6.hd-bkt.clouddn.com/Fovg4-r41PreH64RrAkLr7SWlGr2" alt="image-20241003170308450" style="zoom: 67%;" />

```java
public void dfs(int[] nums, int depth, Deque<Integer> path, Set<Integer> used, List<List<Integer>> ans) {
    if (depth == nums.length) {
        ans.add(new ArrayList<>(path));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used.contains(nums[i])) {
            continue;
        }
        path.addLast(nums[i]);
        used.add(nums[i]);
        dfs(nums, depth+1, path, used, ans);
        // 回溯
        path.removeLast();
        used.remove(nums[i]);
    }
}
```



#### [78. 子集](https://leetcode.cn/problems/subsets/)

```
输入：nums = [1,2,3]
输出：[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]
```

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FsOj2YHVApDomp34RzhK29B9gya6" alt="image-20241003171606558" style="zoom: 50%;" />

从{}开始，遍历所有元素，分加入或者不加入

```java
public void recur(int i, int[] nums, Deque<Integer> path) {
    if (i == nums.length){
        ans.add(new ArrayList<>(path));
        return;
    }
    path.addLast(nums[i]);
    recur(i+1, nums, path);
    path.removeLast();
    recur(i+1, nums, path);
}
```



#### [17. 电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)

![image-20241003171820684](http://sthda9dn6.hd-bkt.clouddn.com/FkHkEuV7AkJoQJAlUzHTHGFAw9N7)

```
输入：digits = "23"
输出：["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

考点：多个集合的随机组合(每个集合使用一次)

```java
// 思路

{a  => {e  => {?  => {#  => 
b  	    f 	   ?	  # 
c}    	g} 	   ?}	  #}

// 递归出口 num.len == path.len
```



```java
public void recur(int i, String str, String path) {
    if (i == str.length()-1) {
        for (char c : num2Letter.get(str.charAt(i))) {
            ans.add(path.concat(""+c));
        }
        return;
    }
    // 遍历集合
    for (char c : num2Letter.get(str.charAt(i))) {
        path = path.concat(""+c);   // 追加下一个位置的集合
        recur(i+1, str, path);
        path = path.substring(0, path.length()-1); // 撤销追加的元素
    }
}
```



#### [39. 组合总和](https://leetcode.cn/problems/combination-sum/)

给你一个 **无重复元素** 的整数数组 `candidates` 和一个目标整数 `target` ，找出 `candidates` 中可以使数字和为目标数 `target` 的 所有 **不同组合** ，并以列表形式返回。你可以按 **任意顺序** 返回这些组合。

`candidates` 中的 **同一个** 数字可以 **无限制重复被选取** 。如果至少一个数字的被选数量不同，则两种组合是不同的。 

对于给定的输入，保证和为 `target` 的不同组合数少于 `150` 个。

```
输入：candidates = [2,3,6,7], target = 7
输出：[[2,2,3],[7]]
```

<img src="http://sthda9dn6.hd-bkt.clouddn.com/Fp7v4_n9O8moVkKhVS73GqSwERVZ" alt="image-20241003173238544" style="zoom:67%;" />

要去重复

```java
public void help(int[] candidates, int start, int target, List<Integer> path) {
    if (target < 0)
        return;
    else if (target == 0) {
        ans.add(new ArrayList<>(path));
        return;
    }
    // 利用i控制范围，只看自己及右侧
    for (int i = start; i < candidates.length; i++) {
        int num = candidates[i];
        path.add(num);
        help(candidates, i, target - num, path);
        path.remove(path.size()-1); 
    }
}
```



#### [22. 括号生成](https://leetcode.cn/problems/generate-parentheses/)

数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。

```
输入：n = 3
输出：["((()))","(()())","(())()","()(())","()()()"]
```

1代表(  0代表)

构建选择二叉树

{} => {1} => {11} => {111}

​                             =>{110} => {1101}

​                                            => {1100} => {11001}...

再补充右括号

```java
public void help(int num_1, int num_0, int n, String str) {
    if (num_1 > n || num_1 < num_0)
        return;
    if (num_1 == n) {
        ans.add(str);
        return;
    }
    help(num_1+1, num_0, n, str+"(");
    help(num_1, num_0+1, n, str+")");
}
```

#### [131. 分割回文串](https://leetcode.cn/problems/palindrome-partitioning/)

```java
注意： result.add(new ArrayList<>(path));  // 重新克隆一份，避免浅拷贝的对象地址

path.addLast path.removeLast           path.push         path.pop
    a 0            a 0                    b 0               a 0
    a 1			   a 1                    a 1               a 1
    b 2                                   a 2
```



```java
private Deque<String> path = new ArrayDeque<>();
    private List<List<String>> result = new ArrayList<>();

    public List<List<String>> partition(String s) {
        backtrack(s, 0);
        return result;
    }

    public void backtrack(String s, int startIdx) {
        if (startIdx >= s.length()) {     
            result.add(new ArrayList<>(path));   // 收集叶子节点的结果
        }
        for (int i = startIdx; i < s.length(); i++) {
            if (!isValid(s.substring(startIdx, i+1)))
                continue;
            path.addLast(s.substring(startIdx, i+1));

            backtrack(s, i+1);

            path.removeLast();
        }

    }

    private boolean isValid(String substring) {
        int len = substring.length();
        for (int i = 0; i < len/2; i++) {
            if (substring.charAt(i) != substring.charAt(len-i-1))
                return false;
        }
        return true;
    }
```





#### [79. 单词搜索](https://leetcode.cn/problems/word-search/)

给定一个 `m x n` 二维字符网格 `board` 和一个字符串单词 `word` 。如果 `word` 存在于网格中，返回 `true` ；否则，返回 `false` 。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FijSDwV2-lxsa82DOeBSmacT0pcd" alt="image-20241003174040179" style="zoom: 67%;" />

```java
public boolean exist(char[][] board, String word) {
    int row = board.length;
    int col = board[0].length;
    for (int i = 0; i < row; i++) {
        for (int j = 0; j < col; j++) {
            if (ans==false&&board[i][j] == word.charAt(0)) {
                board[i][j] = '#';
                help(board, i, j, word.substring(1));
                board[i][j] = word.charAt(0); // 恢复
            }
        }
    }
    return ans;
}

// 深度搜索 + 回溯
private void help(char[][] board, int i, int j, String word) {
    if (word.equals("")||word.length()==0){
        ans = true;
        return;
    }
    int row = board.length;
    int col = board[0].length;
    char target = word.charAt(0);
    int[][] offsets = {{1,0},{-1,0},{0,1},{0,-1}};
    for (int[] offset : offsets) {
        int r = i + offset[0];
        int c = j + offset[1];
        if (r<0||c<0||r>=row||c>=col)
            continue;
        if (board[r][c] == target&&board[r][c]!='#'){
            board[r][c] = '#';
            help(board, r, c, word.substring(1));
            board[r][c] = target; // 恢复
        }
    }
}
```



#### [51. N 皇后](https://leetcode.cn/problems/n-queens/)

按照国际象棋的规则，皇后可以攻击与之处在同一行或同一列或同一斜线上的棋子。

**n 皇后问题** 研究的是如何将 `n` 个皇后放置在 `n×n` 的棋盘上，并且使皇后彼此之间不能相互攻击。

给你一个整数 `n` ，返回所有不同的 **n 皇后问题** 的解决方案。

每一种解法包含一个不同的 **n 皇后问题** 的棋子放置方案，该方案中 `'Q'` 和 `'.'` 分别代表了皇后和空位。



<img src="http://sthda9dn6.hd-bkt.clouddn.com/Fqa3uAsWP1n72GA12Wqkom0ns0Vx" alt="image-20241003174230414" style="zoom:67%;" />

```java
{[????]}
  ⬇️ check
{[????]}
...
```



```java
public static void backtracking(char[][] chessboard, int n, int row) {
    if (row == n){
        ans.add(convert(chessboard));
        return;
    }
    for (int i = 0; i < n; i++) {
        if (isValid(chessboard, row, i, n)) {
            chessboard[row][i] = 'Q';
            backtracking(chessboard, n, row+1);
            chessboard[row][i] = '.';
            // break;
        }//else
    }
}

private boolean isValid(char[][] chessboard, int r, int c, int n) {
    // check column
    for (int i = 0; i < r; i++) {
        if (chessboard[i][c] == 'Q') {
            return false;
        }
    }
    // check 45 degree
    for (int i=r-1,j=c-1; i>=0&&j>=0; i--,j--) {
        if (chessboard[i][j] == 'Q') {
            return false;
        }
    }
    // check 135 degree
    for (int i=r-1,j=c+1; i>=0&&j<n; i--,j++) {
        if (chessboard[i][j] == 'Q') {
            return false;
        }
    }
    return true;
}

public List<List<String>> solveNQueens(int n) {
    ans = new ArrayList<>();
    char[][] chessboard = new char[n][n];
    for (char[] chars : chessboard) {
        Arrays.fill(chars, '.');
    }
    backtracking(chessboard, n, 0);

    return ans;
}
```



### 二分查找

#### [153. 寻找旋转排序数组中的最小值](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/)

```
输入：nums = [3,4,5,1,2]
输出：1
```

```markdown
 l    r
[345 12]  // 大    小 // 且每段均递增
   m
1. m落在左段 => 往右找
2. m落在右端 => 往左字段找
3. 终止条件 => left-mid-right 有序
```

```java
public int findMin(int[] nums) {
    int left = 0, right = nums.length-1;
    while (left<=right) {
        int mid = (left+right)/2;
        if (nums[left]<=nums[mid] && nums[mid]<=nums[right]) {
            return nums[left];
        }else if(nums[left]<=nums[mid]) {
            left = mid+1;
        }else {
            right = mid;
        }
    }
    return 0;
}
```



#### [33. 搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/)

```markdown
 l    r
[345 12]  // 大    小 // 且每段均递增
   m

1. num[m] == target 终止条件
2. m落在左段上
   => if(target < num[m])  => [left, m-1]
      else                 => [m+1,right]
3. m落在右段上
   => if(target < num[m])  => [left, m-1]
   	  else                 => [m+1,right]
```

```java
public int search(int[] nums, int target) {
    int left = 0, right = nums.length-1;
    while (left <= right) {
        int mid = (left + right) / 2;
        if (nums[mid] == target) {
            return mid;
        }

        if (nums[mid] >= nums[left]) {
            if (nums[left]<=target && nums[mid]>target) {
                right = mid-1;
            }else {
                left = mid+1;
            }
        }else {
            if (nums[right]>=target && target>nums[mid]) {
                left = mid+1;
            }else {
                right = mid-1;
            }
        }
    }
    return -1;
}
```



#### [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

第一次查左边界 => 第二次查右边界

递增序列

```markdown
输入：nums = [5,7,7,8,8,10], target = 8
输出：[3,4]

注意边界包括情况
```



```java
public int[] searchRange(int[] nums, int target) {
    if (nums.length == 0)
        return new int[]{-1, -1};
    int[] ans = new int[2];
    Arrays.fill(ans, -1);
    int a = searchFirstLoc(nums, target);
    if (a != -1) {
        ans[0] = a;
        int b = searchSecondLoc(nums, target);
        ans[1] = b;
    }
    return ans;
}

// 求右边界
private int searchSecondLoc(int[] nums, int target) {
    int left = 0;
    int right = nums.length-1;
    while (left<right) {
        int mid = (left+right+1) / 2;
        if (nums[mid] == target) {
            // [mid, right]
            left = mid;
        }
        else if (nums[mid] > target) {
            // [left, mid-1]
            right = mid-1;
        }else {
            // [mid+1, right]
            left = mid+1;
        }
    }
    return left;
}

// 计算左边界
private int searchFirstLoc(int[] nums, int target) {
    int left = 0;
    int right = nums.length-1;
    while (left<right) {
        int mid = (left+right) / 2;
        if (nums[mid] == target) {
            // [left, mid]
            right = mid;
        }
        else if (nums[mid] > target) {
            // [left, mid-1]
            right = mid-1;
        }else {
            // [mid+1, right]
            left = mid+1;
        }
    }
    if (nums[left]!=target)
        return -1;
    return left;
}
```



#### [74. 搜索二维矩阵](https://leetcode.cn/problems/search-a-2d-matrix/)

给你一个满足下述两条属性的 `m x n` 整数矩阵：

- 每行中的整数从左到右按非严格递增顺序排列。
- 每行的第一个整数大于前一行的最后一个整数。

给你一个整数 `target` ，如果 `target` 在矩阵中，返回 `true` ；否则，返回 `false` 。



<img src="http://sthda9dn6.hd-bkt.clouddn.com/Fg-E0yML6gnID_NIv4G-v1ujTd2z" alt="image-20241003211758160" style="zoom:67%;" />

坐标i,j => 转换为一维左边 i*col+j

```java
private int coordinate2Index(int i, int j, int col){
    return i*col+j;
}
public boolean searchMatrix(int[][] matrix, int target) {
    int row = matrix.length;
    int col = matrix[0].length;
    int left_i = 0, left_j = 0;
    int right_i = row-1, right_j = col-1;
    while (coordinate2Index(left_i, left_j, col)<=coordinate2Index(right_i,right_j,col)){
        int mid_i = (coordinate2Index(left_i, left_j, col)+coordinate2Index(right_i,right_j,col))/col;
        int mid_j = (coordinate2Index(left_i, left_j, col)+coordinate2Index(right_i,right_j,col))%col;
        if (matrix[mid_i][mid_j] == target)
            return true;
        else if (matrix[mid_i][mid_j] > target) {
            if (mid_j > 0) {
                right_i = mid_i;
                right_j = mid_j - 1;
            }else {
                right_i = mid_i-1;
                right_j = col-1;
            }
        }else {
            if (mid_j < col-1) {
                left_i = mid_i;
                left_j = mid_j+1;
            }else {
                left_i = mid_i+1;
                left_j = 0;
            }
        }
    }
    return false;
}
```

---

#### 寻找两个正序数组的中位数

两个**有序**数组，寻找中位数  => 第K小的元素



删删删，每次删掉一部分[K/2个] => 找第K-K/2小的元素。

⭐分双数与偶数情况

```java
public double findMedianSortedArrays(int[] nums1, int[] nums2) {
    int n = nums1.length;
    int m = nums2.length;

    if ((n+m) % 2 == 0){   // 偶数 => (left + right)/2 
        int a = getKth(nums1, 0, n-1, nums2, 0, m-1, (n+m)/2);   
        int b = getKth(nums1, 0, n-1, nums2, 0, m-1, (n+m)/2+1);
        return (a+b)/2.0;
    }else
        return getKth(nums1, 0, n-1, nums2, 0, m-1, (n+m+1)/2);

}

private int getKth(int[] nums1, int start1, int end1, int[] nums2, int start2, int end2, int k) {
    int len1 = end1 - start1 + 1;
    int len2 = end2 - start2 + 1;

    if (len1 > len2)
        return getKth(nums2, start2, end2, nums1, start1, end1, k);

    if (len1 == 0)
        return nums2[start2 + k - 1];

    if (k == 1)
        return Math.min(nums1[start1], nums2[start2]);
    // 忽略之前删除的  // 防止越界     
    int i = start1 + Math.min(len1, k / 2) - 1;  // 下标从0开始
    int j = start2 + Math.min(len2, k / 2) - 1;  // 下标从0开始

    if (nums1[i] > nums2[j]) {  // 那么nums2中 包含j及其之前的元素，都不可能是第k小的元素，直接逻辑删除
        return getKth(nums1, start1, end1, nums2, j + 1, end2, k - (j - start2 + 1));
    }
    else {
        return getKth(nums1, i + 1, end1, nums2, start2, end2, k - (i - start1 + 1));
    }
}
```

<img src="http://sthda9dn6.hd-bkt.clouddn.com/Fqiwp7YIZNR9tals3QewyhcdJqP5" alt="image-20241201141540488" style="zoom:50%;" />

逻辑：

1. 分偶数与单数， 偶数的话，分别求 第K个和K+1个元素； 单数就求第K个元素就好
2. 让nums1是元素数量小的那个
3. 如果nums1为空，返回nums2中的第k小的元素
4. 如果k等于1(递归入口)，返回两个数组中，头部最小的元素。



---

### 栈

#### 155 最小栈 

思路： 1. 普通栈 2. 记录最小值的栈(元素复用)

Stack: 2431

minStack: 2221 （单调栈）



#### 394 字符串解码

```
输入：s = "3[a]2[bc]"
输出："aaabcbc"
```

NumStack: 存储次数

StrStack： 存储字符

分"[" 和 "]"时机



#### [739. 每日温度](https://leetcode.cn/problems/daily-temperatures/)

下一个更高温度出现在几天后。如果气温在这之后都不会升高，请在该位置用 `0` 来代替

```
输入: temperatures = [73,74,75,71,69,72,76,73]
输出: [1,1,4,2,1,1,0,0]
```

 （单调栈）

index:    [  0,   1,   2,   3,   4,   5,    6,  7]

temp   = [73, 74, 75, 71, 69, 72,76,73]

element:[74,  75, 76, 72,72, 76,-1,-1] 从前往后 （一起找老爸）

target     :[  1,  2,   6,  5,  5,  6,  -1,  -1]



output  : [  1,  1,   4,   2,   1,  1,  0,  0]

过程：

preElstack: [73]   73<74   => [74] 74<75 =>  [75,71,69]  71,69<72=> [75,72] 75,72<76 => [76] ...

res:[74(1),75(2), 76(6), 72(5),72(5), 76(6), x, x]



---

[84. 柱状图中最大的矩形](https://leetcode.cn/problems/largest-rectangle-in-histogram/)

![image-20241031142355410](http://sthda9dn6.hd-bkt.clouddn.com/FksEznZYvvRvLC3SxHCG9uI3Bfgp)

从该点出发 => 向左找左边界 >=该点

从该点出发 => 向右找右边界 >=该点



S=wxh

---

### 堆

#### [215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)

给定整数数组 `nums` 和整数 `k`，请返回数组中第 `**k**` 个最大的元素。

请注意，你需要找的是数组排序后的第 `k` 个最大的元素，而不是第 `k` 个不同的元素。

你必须设计并实现时间复杂度为 `O(n)` 的算法解决此问题。



```python
# 思路：
维护一个最小堆
然后每次比较，如比(之前最大的K个元素的)最小的大，则Poll堆顶，add该元素
```



#### [347. 前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/)

**示例 1:**

```
输入: nums = [1,1,1,2,2,3], k = 2
输出: [1,2]
```

```python
# 思路：
1. 先统计词频
2. 词频排序 <= 只维护需要的k个最值元素即可， 最小堆<=
```



#### [295. 数据流的中位数](https://leetcode.cn/problems/find-median-from-data-stream/)

**中位数**是有序整数列表中的中间值。如果列表的大小是偶数，则没有中间值，中位数是两个中间值的平均值。

- 例如 `arr = [2,3,4]` 的中位数是 `3` 。
- 例如 `arr = [2,3]` 的中位数是 `(2 + 3) / 2 = 2.5` 。



```python
# 思路：
O(1)时间计算中位数，=> 固定其位置，O(1)查询 => 堆顶

左堆(最大堆) ： 右堆(最小堆)

# 插入情况
1. 偶数时， 保持 中位数 => 左堆顶
    # 判断插入位置， 插左边：左堆.add； 插右边: 右最小值=>左堆，左堆.add;
2. 奇数时(左堆比右堆多一个元素)， 保持 中位数 => (左堆顶 + 右堆顶) /2
    # 判断插入位置， 插左边：左最大值=>右堆，左堆.add； 插右边: 右堆.add;
```







---

### 贪心

#### [55. 跳跃游戏](https://leetcode.cn/problems/jump-game/)

给你一个非负整数数组 `nums` ，你最初位于数组的 **第一个下标** 。数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个下标，如果可以，返回 `true` ；否则，返回 `false` 。



**示例 1：**

```
输入：nums = [2,3,1,1,4]
输出：true
解释：可以先跳 1 步，从下标 0 到达下标 1, 然后再从下标 1 跳 3 步到达最后一个下标。
```



*思路：*

```python
nums = [2,3,1,1,4]
    1.  | . .| 
    2.    | . . .|   
    
# 查看从当前位置，是否能够突出重围(覆盖范围)  max(cover, i+nums[i])  <= 尽可能的往远跳
```

25/3/18 第二次思考



#### [45. 跳跃游戏 II](https://leetcode.cn/problems/jump-game-ii/)

跳跃游戏问题的最小步数

**示例 1:**

```
输入: nums = [2,3,1,1,4]
输出: 2
解释: 跳到最后一个位置的最小跳跃数是 2。
     从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。
```

```python
# 思路
nums = [2,3,1,1,4]
        | . .|
          | . . .| √ <= 下一轮往这跳                # 每轮尽可能跳的远
            | .| ×
    
curRoundDist = 0;
nextRoundDist = 0;
count = 0;
for(int i=0; i<nums.length; i++) {
    nextRoundDist = max(nextRoundDist, i+nums[i])
    if(i == curRoundDist){ # 此轮结束
        curRoundDist = nextRoundDist
        count += 1;
        if(nextRoundDist >= nums.length-1)
        	break;
    }
}
```



#### [763. 划分字母区间](https://leetcode.cn/problems/partition-labels/)

**示例 1：**

```
输入：s = "[ababcbaca][defegde][hijhklij]"
输出：[9,7,8]
解释：
划分结果为 "ababcbaca"、"defegde"、"hijhklij" 。
###每个字母最多出现在一个片段中。###
像 "ababcbacadefegde", "hijhklij" 这样的划分是错误的，因为划分的片段数较少。
```



```python
# 思路, 最简单情况，左端点 => 最远右端点
#       其中有元素进行扩展右边界 => 更新最远右端点  <= ###

int[] farthestIdx = new int[26]
for(int i=0; i<s.length; i++) :
    char c = s.charAt(i);
    farthestIdx[c - 'a'] = i
    
left=0; right=0;    
for(int i=0; i<s.length; i++) :
    char c = s.charAt(i);
    right = max(right, farthestIdx[c-'a'])  # 贪心处 
    if(right == i){
        # save ans
        left = right+1;
    }
```



---

### 动态规划

#### 爬楼梯

假设你正在爬楼梯。需要 `n` 阶你才能到达楼顶。

每次你可以爬 `1` 或 `2` 个台阶。你有多少种不同的方法可以爬到楼顶呢？

```python 
# 思路

考虑当前处于第i处台阶，那么如何才能到达现在的i号台阶呢？
依赖关系如下：
1. 从i-1阶 走一步
2. 从i-2阶 走两步

=> dp[i]表示走到i号台阶的所有可能方法，i表示台阶

递推关系：
dp[i] = dp[i-1] + dp[i-2]
```



#### [118. 杨辉三角](https://leetcode.cn/problems/pascals-triangle/)

![image-20241117154256102](http://sthda9dn6.hd-bkt.clouddn.com/FvE6njpkfqfJ5ZGeKRNxc7bLctR_)



```python
# 思路

preRow = [1, 2, 1]
curRow = [1]
for(i of preRow)
	curRow.add(preRow[i] + preRow[i+1])
curRow.add(1)
```



#### [198. 打家劫舍](https://leetcode.cn/problems/house-robber/)

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警**。

给定一个代表每个房屋存放金额的非负整数数组，计算你 **不触动警报装置的情况下** ，一夜之内能够偷窃到的最高金额。



\# 不能相邻的选择

```python
# 思路
dp[i]表示包含第i号房间所能偷取的最大金额, i表示房间号

# 考虑当前偷到第i号房间
# 选择可能方案
1. 偷 => money[i] + dp[i-2]
2. 不偷 => dp[i-1]

dp[i] = max(money[i] + dp[i-2], dp[i-1])
```



#### [322. 零钱兑换]

给你一个整数数组 `coins` ，表示不同面额的硬币；以及一个整数 `amount` ，表示总金额。

计算并返回可以凑成总金额所需的 **最少的硬币个数** 。如果没有任何一种硬币组合能组成总金额，返回 `-1` 。

你可以认为每种硬币的数量是无限的。



\# 最小组合个数装满

```python
# dp[i]表示装满金额i需要的金币数量，i表示金额

# 考虑当前 要装满的金额为i
# 那么当前能够选择的方案如下：
1. dp[i-CoinA] + 1  # 跳跃
2. dp[i-CoinB] + 1
3. dp[i-CoinC] + 1

dp[i] = min(1., 2., 3.)
```



#### [279. 完全平方数](https://leetcode.cn/problems/perfect-squares/)

给你一个整数 `n` ，返回 *和为 `n` 的完全平方数的最少数量* 。

**完全平方数** 是一个整数，其值等于另一个整数的平方；换句话说，其值等于一个整数自乘的积。例如，`1`、`4`、`9` 和 `16` 都是完全平方数，而 `3` 和 `11` 不是。



元素： [1, 4, 9, 16 ...]

\# 题目考点如上

```python
# dp[i]表示的是给与的整数i所需最少数量的完全平方数，i表示该整数。

# 考虑当前的整数为i
# 那么当前能够凑到该整数的方案如下：
1. dp[i-e1*e1]+1
1. dp[i-e2*e2]+1  # 最少步数跳跃，o.O
1. dp[i-e2*e2]+1

dp[i] = min(1. 2. 3.)
```



#### [139. 单词拆分](https://leetcode.cn/problems/word-break/)

给你一个字符串 `s` 和一个字符串列表 `wordDict` 作为字典。如果可以利用字典中出现的一个或多个单词拼接出 `s` 则返回 `true`。



![image-20241117155502397](http://sthda9dn6.hd-bkt.clouddn.com/FnVQo6XwDgecqWsYzZTCkheeKEc-)

```python
# 思路

# dp[i]表示长度为i的字串是否能够被凑出来，i表示该字串

# 考虑当前的字串的能够选择凑出来的可能方案如下：
1. dp[i-len(word1)]==True && subString(i-len(word1),i) in wordDict
...

```



⭐⭐⭐ 总体思路：概率当前位置的所有可能方案，建立线性的依赖关系。



---

#### [300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)

给你一个整数数组 `nums` ，找到其中最长严格递增子序列的长度。

**子序列** 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，`[3,6,2,7]` 是数组 `[0,3,1,6,2,2,7]` 的子序列。

![image-20241118151828079](http://sthda9dn6.hd-bkt.clouddn.com/Fj5EbplWH-n3fozPHBRzsiw71jLr)

```python
# 思路

# 考虑当前元素 与先前 最长严格递增子序列 们 的关系。 

# dp[i] 表示，第0-i个元素的最长严格递增子序列长度，  i表示该元素。 相对于以i结尾的最长严格递增子序列长度

# 初始化全为1，只包含自身。
if[nums[i] > nums[j]] # 递增
dp[i] = max(dp[i], dp[j]+1)


for(int i=1; i<nums.length; i++)
	for(int j=0; j<i; j++)
    	if(nums[i] > nums[j])
        	dp[i] = max(dp[i], dp[j]+1)  # 动态选择，跳跃
```



#### [152. 乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray/)

给你一个整数数组 `nums` ，请你找出数组中乘积最大的非空连续 子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

\# 连续。元素相邻 ✔️!= 元素值相邻❌

**示例 1:**

```
输入: nums = [2,3,-2,4]
输出: 6
解释: 子数组 [2,3] 有最大乘积 6。
```



```python
# 由于有正有负
# 当前值若为负，与先前最大负数相乘 => -- => +

# Min_dp[i] 表示 i号元素及其之前 最小的乘积和
# Max_dp[i] 表示 i号元素及其之前 最大的乘积和

Min_dp[i] = Min(nums[i], Min_dp[i-1]*nums[i], Max_dp[i-1]*nums[i])  # M??_dp[i-1]*nums[i] 保证连续
Max_dp[i] = Max(nums[i], Min_dp[i-1]*nums[i], Max_dp[i-1]*nums[i])  # 统一正负

result = Max(Max_dp[0,...,n])
```





---

#### 0-1背包问题 (2维)

有n件物品和一个最多能背重量为w 的背包。第i件物品的重量是weight[i]，得到的价值是value[i] 。**每件物品只能用一次**，求解将哪些物品装入背包里物品价值总和最大。

![image-20241118152821399](http://sthda9dn6.hd-bkt.clouddn.com/FkaMTah1gqgk50TrO_qMrJJKVjN4)



dp\[i][j] 表示， 任取{0,i}物品，放至容量j的背包中的 总价值。



dp状态转移方程： 

1. 不放物品i： dp\[i][j] = dp\[i-1][j]
2. 放物品i    :    dp\[i][j] = dp\[i-1][j-weight[i]] + value\[i]  # 从任意放置{0,...,i-1}的状态中进行迁移，要求其背包容量足够装下物品i



```java
for (int i = 1; i < n; i++) {  // 物品 
    for (int j = 0; j <= bagweight; j++) {  // 背包
        if (j < weight[i]) {  // 装不下 => 不装
            dp[i][j] = dp[i - 1][j];
        } else {
            dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);  // 装或者不装中价值最大
        }
    }
}
```



遍历顺序=> dp\[i][j] 依赖于 上一行 dp\[i-1][0,...,j]



---

#### 0-1背包问题 (1维)

![image-20241118155137608](http://sthda9dn6.hd-bkt.clouddn.com/Fl9b5IAbYdqcFTPX1NTI0U8gEB8d)

```python
   j    0, 1, 2, 3, 4  # 背包容量
dp[] = [0,15,15,15,15]  # 当前价值, 初始化放置物品1
       [0,15,15,|20,35]   # 放置物品2   [✔️✔️]  max(不放,放)
       [0,15,15,20,|35]   # 放置物品3   [❌] 

dp[j] = max(dp[j], dp[j-weight[i]]+value[i])  # 1. 不放置 2.放置



# 遍历
for(int i = 0; i < weight.size(); i++) { // 遍历物品(关键是重量)，只使用一次
    for(int j = bagWeight; j >= weight[i]; j--) { // 遍历背包容量  从大到小  
        dp[j] = max(dp[j], dp[j - weight[i]] + value[i]); 
        
# 当背包容量 j 小于当前物品的重量 weight[i] 时，根本不可能放入该物品，因此不需要继续更新 dp[j]。我们应该跳过这些容量小于 weight[i] 的情况
```





---

#### 分割等和子集

给你一个 **只包含正整数** 的 **非空** 数组 `nums` 。请你判断是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。



**示例 1：**

```
输入：nums = [1,5,11,5]
输出：true
解释：数组可以分割成 [1, 5, 5] 和 [11] 。
```



0-1背包问题： 和相等 => 堆积sum的一半作为目标



```python
# 1. 求和 sum => target == sum//2

# dp[j] 表示j容量的背包的最大价值
int[] dp = new int[target+1];
# 背包容量==0-target , 目标装满target => dp[target] == target
for(int i=0; i<nums.length; i++)
	for(int j=target; j>=nums[i]; j--) # 背包容量小于当前物品，后续容量的背包没有必要再装了
       dp[j] = max(dp[j], dp[j-nums[i]]+nums[i])  # 不装，装
        
if(dp[target] == target)
	return true
```



#### 32 最长有效括号

```java
// 栈的例子
-1 0 1 2 3 4 5
   ( ( ) ( ) )
   x |.|         Step: 1
   x |.....| 	 Step: 2
 x |.........|   Step: 3
     
-1 0 1 2 3 4 5
   ( ) ( ( ) )
 x |.|
       x |.|
 x |.........|  
```

```java
public int longestValidParentheses(String s) {
    Stack<Integer> stack = new Stack<>();
    int maxLen = 0;
    stack.push(-1);

    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (c == '(') {
            stack.push(i);
        }else {
            stack.pop();  // 这个可能是 相匹配的 "(" or 基准边界

            if (!stack.isEmpty()) { // 相匹配的 "("
                maxLen = Math.max(maxLen, i - stack.peek());
            }else {
                stack.push(i);  // 更新基准边界
            }
        }
    }
    return maxLen;
}
```

**动态规划做法**

dp[i]  子串长度为i的 字符串     的 最长有效括号

<img src="http://sthda9dn6.hd-bkt.clouddn.com/Ft2JzSWTCd8Cz17U9QW517Q7iKYX" alt="image-20241128194352774" style="zoom:50%;" />



```java
# 条件转移方程：
   1. if s[i]  == ")" 
         if s[i-1] == "("                      dp[i-2]+2
			dp[i] = dp[i-2] + 2                  [xxx]()
         else == ")"                                  dp[i-dp[i-1]-2]+dp[i-1]+2
            dp[i] = dp[i-dp[i-1]-2]+dp[i-1]+2              [xxx]     ( [xxx] )
                                                  idx            i-dp[i-1]-1
             
情况1：   [xxx]()       dp[i]=2+dp[i-2]
情况2：   [xxx]([xxx])  dp[i]=2+dp[i-1] + dp[i-2-dp[i-1]]    
              (的idx为i-dp[i-1]-1
```

```java
public int longestValidParentheses(String s) {
    int[] dp = new int[s.length()+1];
    Arrays.fill(dp, 0);
    int maxLen = 0;
    for (int i = 1; i < s.length(); i++) {
        char c = s.charAt(i);
        if (c == ')') {
            if (s.charAt(i-1) == '(' && i-2 >= 0)
                dp[i] = dp[i-2] + 2;
            else if (i-dp[i-1]-1 >= 0 && s.charAt(i-dp[i-1]-1) == '('){
                dp[i] = dp[i-1] + 2 + (i-2-dp[i-1]>=0 ? dp[i-2-dp[i-1]]:0);

            }
            if (dp[i] > maxLen)
                maxLen = dp[i];
        }
    }
    return maxLen;
}
```





---

### 技巧题

#### 只出现一次的数字

*异或*

```python
binary:   8 4 2 1
eg. 4 2 2 1 1
=> 0100 0010 0010 0001 0001
java => ^ (XOR )

0100
0010 => 0110

0110
0010 => 0100

0100
0001 => 0101

0101
0001 => 0100

# 思想：对偶同归于尽
```



---

#### 多数元素

枚举当前可能的候选人 （摩尔投票）

```python
nums = [2,2,1,1,1,2,2]

count =  [1,2,1,0,1,0,1]  # 拥举票数
candidate=[2,2,2,2,1,1,2]  # 当前最有可能的候选元素
```



---

#### 颜色分类 

荷兰国旗问题

[1,2,0,1,0,2] => [0,0,1,1,2,2]  元素分组

双指针法：

[0,...,0]P0 

P0[1,...,1]P2

P2[2,...,2] 

判断i当前的元素视角。进行交换



---

#### 下一个排序

***数字组合递增***

\# 1 树状结构⭐⭐⭐❗❗❗

```python
# 1,2,3
	   		  []
		1      2       3
	   2 3    1  3    1  2
      3   2  3    1  2     1
```

找[23541]下一个排序 [24135]

1 定位 1号元素 [从后往前遍历，非递增的第一个元素] => 3

​    [2]3[541]

2  [541]中找比3大的最小元素  => 4 && 交换 

​    [2]4[531]    => 逆序

3 => [2]4[135]

 ```python
              .2]
        .3         4]
      1 4 .5      1]
    [...]   .4   3]  
             .1 5]
 ```



---

#### [287. 寻找重复数](https://leetcode.cn/problems/find-the-duplicate-number/)

给定一个包含 `n + 1` 个整数的数组 `nums` ，其数字都在 `[1, n]` 范围内（包括 `1` 和 `n`），可知至少存在一个重复的整数。

假设 `nums` 只有 **一个重复的整数** ，返回 **这个重复的数** 。

你设计的解决方案必须 **不修改** 数组 `nums` 且只用常量级 `O(1)` 的额外空间。



```python
idx:   [0,1,2,3,4]
input: [1,3,4,2,2]   // 顺序表，数组作为链表
```



![image-20241127235324909](http://sthda9dn6.hd-bkt.clouddn.com/Ftu4CotzKz2NyoyYeU0nisidJ50O)

等价于找环的入口。

```java
public int findDuplicate(int[] nums) {
    int fast = 0; int slow = 0;
    while(true) {
        fast = nums[nums[fast]];
        slow = nums[slow];
        if(fast == slow) {
            int idx1 = 0;
            int idx2 = fast;
            while(idx1 != idx2) {
                idx1 = nums[idx1];
                idx2 = nums[idx2];
            }
            return idx1;
        }
    }
}
```



原理：由于存在的重复的数字 *target*，因此 ***target* 这个位置一定有起码两条指向它的边**，因此整张图一定存在环，且我们要找到的 *target* 就是这个环的入口  ❗❗❗*仅有一个重复。 重复位置 入度为2 => 环的入口位置*



---

### 多维动态规划

#### [62. 不同路径](https://leetcode.cn/problems/unique-paths/)

一个机器人位于一个 `m x n` 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？



dp\[i][j]表示到该位置的可能的路径数，仅可以往右和下移动(这个限制，创建了转移方程的条件)

dp\[i][j] = dp\[i-1][j] + dp\[i][j-1]   \\\ 仅从上面和左边到该位置



初始化 

```java
  1 1 1 
1
1
1      
```



---

#### [64. 最小路径和](https://leetcode.cn/problems/minimum-path-sum/)

给定一个包含非负整数的 `*m* x *n*` 网格 `grid` ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。

**说明：**每次只能向下或者向右移动一步。



dp\[i][j]: 到该位置最小的数字和(最小的代价)

=Min(dp\[i-1][j], dp\[i][j-1]) + current_grid_value



初始化

```java
  s0 s1 s2   // 累加
t0
t1
t2    
```



---

#### [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)

给你一个字符串 `s`，找到 `s` 中最长的 回文 子串。



dp\[i][j]: 字串i-j是否为回文字串。  

转移条件{

​	如果字串为 ?? 或者 ?x? 且首尾字符相等 => True,

​	如果字串为?xxx? 且首尾字符相等 且 dp\[i+1][j-1]=True => True

}

初始化

```java
s = "caddab"
    
  0 1 2 3 4 5
0 T
1   T
2     T
3       T
4         T
5           T
    
T ca cad cadd cadda caddab
   T  ad  add a[dd]a addab  <=
       T  [dd]  dda   ddab  <=
            T    da    dab
                  T     ab
                         T
       ^
      /   这就是为什么为[i+1][j-1]的原因，利用之前的状态
```



---

#### 最长重复子数组

```
输入：nums1 = [1,2,3,2,1], nums2 = [3,2,1,4,7]
输出：3
解释：长度最长的公共子数组是 [3,2,1] 。
```

子数组为连续的部分



dp\[i][j] 含义， (0 - i-1)和(0 - j-1)的数组的最长重复子数组

转移方程：

​	if nums1[i-1] == nums2[j-1]:

​	  dp\[i][j] = dp\[i-1][j-1] + 1 // 在之前的基础上最长子数组+1

   else:

​      0  // 因为已经不连续了



```java
       ""  "1" "12" "123" "1232" "12321"
    ""  0   0    0     0      0       0
   "3"  0   0    0     1      0       0
  "32"  0   0    1     0     [2]      0
 "321"  0   1    0     0      0      [3]
 "3214" 0   0    0     0      0       0
"32147" 0   0    0     0      0       0     
```

dp\[i][j] = dp\[i-1][j-1] + 1 // 在之前的基础上最长子数组+1⭐⭐⭐



---

#### [1143. 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/)



```
输入：text1 = "abcde", text2 = "ace" 
输出：3  
解释：最长公共子序列是 "ace" ，它的长度为 3 。
```

子序列，相对位置保持不变就行，不要求连续



dp\[i][j] 含义， (0 - i-1)和(0 - j-1)的数组的最长公共子序列

转移方程：

​	if nums1[i-1] == nums2[j-1]:

​	  dp\[i][j] = dp\[i-1][j-1] + 1 // 在之前的基础上最长子数组+1

   else:

​       dp\[i][j] = Max(dp\[i-1][j], dp\[i][j-1]) // 不要求连续



初始化：

```java
     ""  "a" "ab" "abc" "abcd" "abcde"
   "" 0   0    0     0      0       0
  "a" 0   1    1     1      1       1
 "ac" 0   1    1     2      2       2
"ace" 0   1    1     2      2       3
```







---

#### [72. 编辑距离](https://leetcode.cn/problems/edit-distance/)

只能执行 新增，删除，替换三个操作。

```
输入：word1 = "horse", word2 = "ros"
输出：3
解释：
horse -> rorse (将 'h' 替换为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')
```



dp\[i][j] 含义， 将(0 - i-1) 变为 (0 - j-1)的最小操作。



状态转移：

​	if word1[i-1] == word2[j-1]:

​		dp\[i][j] = dp\[i-1][j-1] // 什么都不需要做，忽略掉这个单词

​	else:

​        dp\[i][j] = Min(Min( dp\[i-1][j], dp\[i][j-1] ), dp\[i-1][j-1])

 从上or右，新增一个元素，or左上角修改一个元素



递推方向：

```java
i-1,j-1     i-1,j             
        ↘️     ⬇️
i,j-1  ➡️     i,j        
    
⬇️➡️： 新增一个元素 以匹配
↘️： 修改一个元素，以匹配   
```



初始化：

```java
     ""  "h" "ho" "hor" "hors" "horse"
   "" 0   1   2    3     4      5
  "r" 1
 "ro" 2
"ros" 3      
```





---

