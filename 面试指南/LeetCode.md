[toc]

### 1、[两数之和](https://leetcode-cn.com/problems/two-sum)

------

自己做的

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int[] t = new int[2];
        
        int fn = 0;
        for (int i = 0; i < nums.length; i++) {
            fn = nums[i];
            for (int j = 0; j < nums.length; j++) {
                if(j == i) {
                    continue;
                }
                if(fn+nums[j] == target) {
                    t[0] = i;
                    t[1] = j;
                }
            }
        }
        return t;
    }
}
```

官方答案

```java
// 暴力枚举
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int n = nums.length;
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                if (nums[i] + nums[j] == target) {
                    return new int[]{i, j};
                }
            }
        }
        return new int[0];
    }
}
// 哈希表
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> hashtable = new HashMap<Integer, Integer>();
        for (int i = 0; i < nums.length; ++i) {
            if (hashtable.containsKey(target - nums[i])) {
                return new int[]{hashtable.get(target - nums[i]), i};
            }
            hashtable.put(nums[i], i);
        }
        return new int[0];
    }
}
```

### 2、[两数相加](https://leetcode-cn.com/problems/add-two-numbers)

------

百度参考

```java
public class Example {
    // 两数相加
    public static ListNode addListNode(ListNode l1, ListNode l2) {
        // 生成ListNode链表对象，链表的值为0，没有指向的节点
        ListNode dummyHead = new ListNode(0);
        ListNode p = l1, q = l2, current = dummyHead;

        int value = 0;

        while (p != null || q != null) {
            int pValue = p != null ? p.val : 0;
            int qValue = q != null ? q.val : 0;

            int sum = value + pValue + qValue;

            value = sum / 10;

            current.next = new ListNode(sum % 10);
            current = current.next;

            System.out.println("==" + current);

            if (p != null) {
                p = p.next;
            }
            if (q != null) {
                q = q.next;
            }
        }

        if (value > 0) {
            current.next = new ListNode(value);
        }

        return dummyHead;
    }
}

class ListNode {
    int val;

    ListNode next;

    public ListNode(int val) {
        super();
        this.val = val;
    }

    @Override
    public String toString() {
        return "ListNode [val=" + val + ", next=" + next + "]";
    }
}
```

官方

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode head = null, tail = null;
        int carry = 0;
        while (l1 != null || l2 != null) {
            int n1 = l1 != null ? l1.val : 0;
            int n2 = l2 != null ? l2.val : 0;
            int sum = n1 + n2 + carry;
            if (head == null) {
                head = tail = new ListNode(sum % 10);
            } else {
                tail.next = new ListNode(sum % 10);
                tail = tail.next;
            }
            carry = sum / 10;
            if (l1 != null) {
                l1 = l1.next;
            }
            if (l2 != null) {
                l2 = l2.next;
            }
        }
        if (carry > 0) {
            tail.next = new ListNode(carry);
        }
        return head;
    }
}
```

### 3、[无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters)

------

自己做的

```java
    public static int lengthOfLongestSubstring(String s) {
        if (s.length() == 1) {
            return 1;
        }

        if (s.length() == 0) {
            return 0;
        }
        List<String> list = Arrays.asList(s.split(""));
        List<String> sList = new ArrayList<String>(list);

        Map<String, List<Integer>> map = new HashMap<>();

        for (int i = 0; i < sList.size(); i++) {
            List<String> headList = new ArrayList<String>();
            List<String> tailList = new ArrayList<String>();
            for (int j = i; j >= 0; j--) {
                if (!headList.contains(sList.get(j))) {
                    headList.add(sList.get(j));
                    continue;
                }
                break;
            }

            for (int j = i; j < sList.size(); j++) {
                if (!tailList.contains(sList.get(j))) {
                    tailList.add(sList.get(j));
                    continue;
                }
                break;
            }
            
            if(map.containsKey(sList.get(i))) {
                map.get(sList.get(i)).add(headList.size() > tailList.size() ? headList.size() : tailList.size()); 
            } else {
                List<Integer> tempValueList = new ArrayList<Integer>();
                tempValueList.add(headList.size() > tailList.size() ? headList.size() : tailList.size());
                map.put(sList.get(i), tempValueList);
            }
        }

        
        int maxValue = 0;
        for (Entry<String, List<Integer>> entry : map.entrySet()) {
            List<Integer> maxList = entry.getValue();
            Collections.sort(maxList);
            if(maxList.get(maxList.size()-1) > maxValue) {                
                maxValue = maxList.get(maxList.size()-1);
            }
        }
        
//        System.out.println(map);
//        System.out.println(maxValue);
        
        return maxValue;
    }
```

官方

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        // 哈希集合，记录每个字符是否出现过
        Set<Character> occ = new HashSet<Character>();
        int n = s.length();
        // 右指针，初始值为 -1，相当于我们在字符串的左边界的左侧，还没有开始移动
        int rk = -1, ans = 0;
        for (int i = 0; i < n; ++i) {
            if (i != 0) {
                // 左指针向右移动一格，移除一个字符
                occ.remove(s.charAt(i - 1));
            }
            while (rk + 1 < n && !occ.contains(s.charAt(rk + 1))) {
                // 不断地移动右指针
                occ.add(s.charAt(rk + 1));
                ++rk;
            }
            // 第 i 到 rk 个字符是一个极长的无重复字符子串
            ans = Math.max(ans, rk - i + 1);
        }
        return ans;
    }
}
```

### 4、[寻找两个正序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays)

------

自己做的

```java
public static double findMedianSortedArrays(int[] nums1, int[] nums2) {
    int[] c = new int[nums1.length + nums2.length];
    System.arraycopy(nums1, 0, c, 0, nums1.length);
    System.arraycopy(nums2, 0, c, nums1.length, nums2.length);

    Arrays.sort(c);

    if (c.length % 2 != 0) {
        return c[c.length / 2];
    }
    return (double)(c[c.length / 2] + c[c.length / 2 - 1]) / 2;
}
```

官方

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int length1 = nums1.length, length2 = nums2.length;
        int totalLength = length1 + length2;
        if (totalLength % 2 == 1) {
            int midIndex = totalLength / 2;
            double median = getKthElement(nums1, nums2, midIndex + 1);
            return median;
        } else {
            int midIndex1 = totalLength / 2 - 1, midIndex2 = totalLength / 2;
            double median = (getKthElement(nums1, nums2, midIndex1 + 1) + getKthElement(nums1, nums2, midIndex2 + 1)) / 2.0;
            return median;
        }
    }

    public int getKthElement(int[] nums1, int[] nums2, int k) {
        /* 主要思路：要找到第 k (k>1) 小的元素，那么就取 pivot1 = nums1[k/2-1] 和 pivot2 = nums2[k/2-1] 进行比较
         * 这里的 "/" 表示整除
         * nums1 中小于等于 pivot1 的元素有 nums1[0 .. k/2-2] 共计 k/2-1 个
         * nums2 中小于等于 pivot2 的元素有 nums2[0 .. k/2-2] 共计 k/2-1 个
         * 取 pivot = min(pivot1, pivot2)，两个数组中小于等于 pivot 的元素共计不会超过 (k/2-1) + (k/2-1) <= k-2 个
         * 这样 pivot 本身最大也只能是第 k-1 小的元素
         * 如果 pivot = pivot1，那么 nums1[0 .. k/2-1] 都不可能是第 k 小的元素。把这些元素全部 "删除"，剩下的作为新的 nums1 数组
         * 如果 pivot = pivot2，那么 nums2[0 .. k/2-1] 都不可能是第 k 小的元素。把这些元素全部 "删除"，剩下的作为新的 nums2 数组
         * 由于我们 "删除" 了一些元素（这些元素都比第 k 小的元素要小），因此需要修改 k 的值，减去删除的数的个数
         */

        int length1 = nums1.length, length2 = nums2.length;
        int index1 = 0, index2 = 0;
        int kthElement = 0;

        while (true) {
            // 边界情况
            if (index1 == length1) {
                return nums2[index2 + k - 1];
            }
            if (index2 == length2) {
                return nums1[index1 + k - 1];
            }
            if (k == 1) {
                return Math.min(nums1[index1], nums2[index2]);
            }
            
            // 正常情况
            int half = k / 2;
            int newIndex1 = Math.min(index1 + half, length1) - 1;
            int newIndex2 = Math.min(index2 + half, length2) - 1;
            int pivot1 = nums1[newIndex1], pivot2 = nums2[newIndex2];
            if (pivot1 <= pivot2) {
                k -= (newIndex1 - index1 + 1);
                index1 = newIndex1 + 1;
            } else {
                k -= (newIndex2 - index2 + 1);
                index2 = newIndex2 + 1;
            }
        }
    }
}
```

### 5、[二分查找](https://leetcode-cn.com/problems/binary-search/solution/er-fen-cha-zhao-by-leetcode/)

------

自己写

```java
public int search(int[] num, int target) {
    int left = 0;
    int right = num.length - 1;
    int pivot;

    while (left <= right) {
        pivot = left + (right - left) / 2;
        if (num[pivot] == target) {
            return pivot;
        }
        if (target < num[pivot]) {
            right = pivot - 1;
        } else {
            left = pivot + 1;
        }
    }
    return -1;
}
```

官方

```java
class Solution {
  public int search(int[] nums, int target) {
    int pivot, left = 0, right = nums.length - 1;
    while (left <= right) {
      pivot = left + (right - left) / 2;
      if (nums[pivot] == target) return pivot;
      if (target < nums[pivot]) right = pivot - 1;
      else left = pivot + 1;
    }
    return -1;
  }
}
```

### 6、[ 第一个错误的版本](https://leetcode-cn.com/problems/first-bad-version/)

------

官方

```java
public class Solution extends VersionControl {
    public int firstBadVersion(int n) {
        int left = 1, right = n;
        while (left < right) { // 循环直至区间左右端点相同
            int mid = left + (right - left) / 2; // 防止计算时溢出
            if (isBadVersion(mid)) {
                right = mid; // 答案在区间 [left, mid] 中
            } else {
                left = mid + 1; // 答案在区间 [mid+1, right] 中
            }
        }
        // 此时有 left == right，区间缩为一个点，即为答案
        return left;
    }
}
```

### 7、[搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/)

------

自己写

```java
public static int searchInsert(int[] nums, int target) {
    int left = 0; 
    int right = nums.length - 1;
    int pivot = 0;

    while( left < right) {
        pivot = left + (right - left) / 2;
        if(nums[pivot] == target) {
            return pivot;
        }
        if(nums[pivot] > target) {                
            right = pivot - 1;
        } 
        if(nums[pivot] < target) {
            left = pivot + 1;
        }
    }

    for (int i = 0; i < nums.length; i++) {
        if(nums[i] >= target) {
            return i;
        }
        if(i == (nums.length - 1)) {
            return i + 1;
        }
    }

    return 0;
}
```

官方

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int n = nums.length;
        int left = 0, right = n - 1, ans = n;
        while (left <= right) {
            int mid = ((right - left) >> 1) + left;
            if (target <= nums[mid]) {
                ans = mid;
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        return ans;
    }
}
```

### 8、[有序数组的平方](https://leetcode-cn.com/problems/squares-of-a-sorted-array/)

自己做

```java
class Solution {
    public int[] sortedSquares(int[] nums) {
        int[] ans = new int[nums.length];
        for (int i = 0; i < nums.length; ++i) {
            ans[i] = nums[i] * nums[i];
        }
        Arrays.sort(ans);
        return ans;
    }
}
```

官方

```java
// 方式一
class Solution {
    public int[] sortedSquares(int[] nums) {
        int n = nums.length;
        int negative = -1;
        for (int i = 0; i < n; ++i) {
            if (nums[i] < 0) {
                negative = i;
            } else {
                break;
            }
        }

        int[] ans = new int[n];
        int index = 0, i = negative, j = negative + 1;
        while (i >= 0 || j < n) {
            if (i < 0) {
                // 全为正数
                ans[index] = nums[j] * nums[j];
                ++j;
            } else if (j == n) {
                // 全为负数
                ans[index] = nums[i] * nums[i];
                --i;
            } else if (nums[i] * nums[i] < nums[j] * nums[j]) {
                ans[index] = nums[i] * nums[i];
                --i;
            } else {
                ans[index] = nums[j] * nums[j];
                ++j;
            }
            ++index;
        }

        return ans;
    }
}
// 方式二
class Solution {
    public int[] sortedSquares(int[] nums) {
        int n = nums.length;
        int[] ans = new int[n];
        for (int i = 0, j = n - 1, pos = n - 1; i <= j;) {
            if (nums[i] * nums[i] > nums[j] * nums[j]) {
                ans[pos] = nums[i] * nums[i];
                ++i;
            } else {
                ans[pos] = nums[j] * nums[j];
                --j;
            }
            --pos;
        }
        return ans;
    }
}
```

### 9、[旋转数组](https://leetcode-cn.com/problems/rotate-array/)

自己写

```java
public static void rotate(int[] nums, int k) {
    int len = nums.length;
    int cycleNum = k % len;

    int[] arrs = new int[len];

    for (int i = 0; i < len; i++) {
        if(((i+cycleNum) / len) == 0 &&  (i+cycleNum) % len < len ) {                
        System.out.println("==");
        arrs[i +cycleNum] = nums[i];
        }
        arrs[(i+cycleNum) % len] = nums[i];
    }

    for (int i = 0; i < arrs.length; i++) {
    	nums[i] = arrs[i];
    }
}
```

官方

```java
class Solution {
    public void rotate(int[] nums, int k) {
        int n = nums.length;
        int[] newArr = new int[n];
        for (int i = 0; i < n; ++i) {
            newArr[(i + k) % n] = nums[i];
        }
        System.arraycopy(newArr, 0, nums, 0, n);
    }
}
```

### 10、[移动零](https://leetcode-cn.com/problems/move-zeroes/)

自己写

```java
public static void moveZeroes(int[] nums) {
    int len = nums.length;

    int count = 0;
    int zeroCount = 0;
    while (count != len) {            
        if(nums[count] == 0) {
            for (int j = count; j < len - 1; j++) {
                nums[j] = nums[j + 1];
            }
            nums[len - 1] = 1;
            count = 0;
            zeroCount++;
            continue;
        }
        count++;
    }

    for (int i = 0; i < zeroCount; i++) {
        nums[len - 1 - i] = 0;
    }
}
```

官方

```java
// 方式一
class Solution {
    public void moveZeroes(int[] nums) {
        int n = nums.length, left = 0, right = 0;
        while (right < n) {
            if (nums[right] != 0) {
                swap(nums, left, right);
                left++;
            }
            right++;
        }
    }

    public void swap(int[] nums, int left, int right) {
        int temp = nums[left];
        nums[left] = nums[right];
        nums[right] = temp;
    }
}
// 方式二（推荐）
class Solution {
	public void moveZeroes(int[] nums) {
		if(nums==null) {
			return;
		}
		//两个指针i和j
		int j = 0;
		for(int i=0;i<nums.length;i++) {
			//当前元素!=0，就把其交换到左边，等于0的交换到右边
			if(nums[i]!=0) {
				int tmp = nums[i];
				nums[i] = nums[j];
				nums[j++] = tmp;
			}
		}
	}
}
```

### 11、[两数之和 II - 输入有序数组](https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/)

自己写

```java
public int[] twoSum(int[] numbers, int target) {
    int[] ansArr = new int[2];

    for (int i = 0; i < numbers.length; i++) {
        int fValue = numbers[i];
        for (int j = i + 1; j < numbers.length; j++) {
            if ((fValue + numbers[j]) == target) {
                ansArr[0] = i + 1;
                ansArr[1] = j + 1;
                return ansArr;
            }
        }

    }
    return ansArr;
}
```

官方

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int low = 0, high = numbers.length - 1;
        while (low < high) {
            int sum = numbers[low] + numbers[high];
            if (sum == target) {
                return new int[]{low + 1, high + 1};
            } else if (sum < target) {
                ++low;
            } else {
                --high;
            }
        }
        return new int[]{-1, -1};
    }
}
```

### 12、[反转字符串](https://leetcode-cn.com/problems/reverse-string/)

自己写

```java
public void reverseString(char[] s) {
    char temp = ' ';
    for (int i = 0, n = s.length - 1; i < s.length / 2; i++, n--) {
        temp = s[i];
        s[i] = s[n];
        s[n] = temp;
    }
}
```

官方

```java
public void reverseString(char[] s) {
    int n = s.length;
    for (int left = 0, right = n - 1; left < right; ++left, --right) {
        char tmp = s[left];
        s[left] = s[right];
        s[right] = tmp;
    }
}
```

### 13、[反转字符串中的单词 III](https://leetcode-cn.com/problems/reverse-words-in-a-string-iii/)

自己写

```java
class Solution {
    public String reverseWords(String s) {
        String str = "";
        
        String reversal = "";
        for (int i = 0; i < s.length(); i++) {
            if(!String.valueOf(s.charAt(i)).equals(" ")) {
                reversal = reversal + String.valueOf(s.charAt(i));
                if(i == s.length() - 1) {
                    str = str + (str.length() == 0 ? "" : " ") + splice(reversal);
                    return str;
                }
                continue;
            }
            
            str = str + (str.length() == 0 ? "" : " ") + splice(reversal);
            reversal = "";
        }
        
        return str;
    }

    public String splice(String reversal) {
        String temp = "";
        for (int j = reversal.length() - 1; j >= 0; j--) {
            temp = temp + reversal.charAt(j);
        }
        return temp;
    }
}
```

官方

```java
class Solution {
    public String reverseWords(String s) {
        StringBuffer ret = new StringBuffer();
        int length = s.length();
        int i = 0;
        while (i < length) {
            int start = i;
            while (i < length && s.charAt(i) != ' ') {
                i++;
            }
            for (int p = start; p < i; p++) {
                ret.append(s.charAt(start + i - 1 - p));
            }
            while (i < length && s.charAt(i) == ' ') {
                i++;
                ret.append(' ');
            }
        }
        return ret.toString();
    }
}
```

### 14、[子集](https://leetcode-cn.com/problems/subsets/)

官方

```java
class Solution {
    List<Integer> t = new ArrayList<Integer>();
    List<List<Integer>> ans = new ArrayList<List<Integer>>();

    public List<List<Integer>> subsets(int[] nums) {
        dfs(0, nums);
        return ans;
    }

    public void dfs(int cur, int[] nums) {
        if (cur == nums.length) {
            ans.add(new ArrayList<Integer>(t));
            return;
        }
        t.add(nums[cur]);
        dfs(cur + 1, nums);
        t.remove(t.size() - 1);
        dfs(cur + 1, nums);
    }
}
```

### 15、[链表的中间结点](https://leetcode-cn.com/problems/middle-of-the-linked-list/)

自己写

```java
public static ListNode middleNode(ListNode head) {
    int count = 1;
    ListNode l = head;
    while (l.next != null) {
        l = l.next;
        count++;
    }

    ListNode l2 = head;
    for (int i = 0; i < count / 2; i++) {
        l2 = l2.next;
    }
    return l2;
}
```

官方

```java
// 方式一
class Solution {
    public ListNode middleNode(ListNode head) {
        ListNode[] A = new ListNode[100];
        int t = 0;
        while (head != null) {
            A[t++] = head;
            head = head.next;
        }
        return A[t / 2];
    }
}
// 方式二
class Solution {
    public ListNode middleNode(ListNode head) {
        int n = 0;
        ListNode cur = head;
        while (cur != null) {
            ++n;
            cur = cur.next;
        }
        int k = 0;
        cur = head;
        while (k < n / 2) {
            ++k;
            cur = cur.next;
        }
        return cur;
    }
}
// 方式三
class Solution {
    public ListNode middleNode(ListNode head) {
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        return slow;
    }
}
```

### 16、[删除链表的倒数第 N 个结点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

自己写

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode[] arrListNode = new ListNode[30];

    int i = 0;
    while (head != null) {
        arrListNode[i++] = head;
        head = head.next;
    }

    if (i == 1) {
        return new ListNode().next;
    }

    if(i == n) {
        return arrListNode[1];
    }

    ListNode l = arrListNode[i - n - 1];
    l.next = arrListNode[i - n].next;

    return arrListNode[0];
}
```

官方

```java
// 根据栈
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0, head);
        Deque<ListNode> stack = new LinkedList<ListNode>();
        ListNode cur = dummy;
        while (cur != null) {
            stack.push(cur);
            cur = cur.next;
        }
        for (int i = 0; i < n; ++i) {
            stack.pop();
        }
        ListNode prev = stack.peek();
        prev.next = prev.next.next;
        ListNode ans = dummy.next;
        return ans;
    }
}
// 双指针
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0, head);
        ListNode first = head;
        ListNode second = dummy;
        for (int i = 0; i < n; ++i) {
            first = first.next;
        }
        while (first != null) {
            first = first.next;
            second = second.next;
        }
        second.next = second.next.next;
        ListNode ans = dummy.next;
        return ans;
    }
}
```

### 17、[整数反转](https://leetcode-cn.com/problems/reverse-integer)

自己写

```java
public static int reverse(int x) {
    long value = x;
    boolean symbol = false;
    if (x < 0) {
        value = -(long)x;
        symbol = true;
    }

    char arr[] = String.valueOf(value).toCharArray();

    String number = "";
    for (int i = arr.length - 1; i >= 0; i--) {
        number = number + arr[i];
    }

    if (symbol) {
        if(-Long.valueOf(number) < Integer.MIN_VALUE) {
            return 0;
        }
        return -Integer.valueOf(number);
    } else {
        if(Long.valueOf(number) > Integer.MAX_VALUE) {
            return 0;
        }
        return Integer.valueOf(number);
    }
}
```

官方

```java
class Solution {
    public int reverse(int x) {
        int rev = 0;
        while (x != 0) {
            if (rev < Integer.MIN_VALUE / 10 || rev > Integer.MAX_VALUE / 10) {
                return 0;
            }
            int digit = x % 10;
            x /= 10;
            rev = rev * 10 + digit;
        }
        return rev;
    }
}
```

### 18、[删除有序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array)

自己写

```java
public static int removeDuplicates(int[] nums) {
    List<Integer> list = new ArrayList<Integer>();

    for (int i = 0; i < nums.length; i++) {
        if(!list.contains(nums[i])) {
            list.add(nums[i]);
        }
    }

    for (int i = 0; i < list.size(); i++) {
        nums[i] = list.get(i);
    }

    return list.size();
}
```

官方

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int n = nums.length;
        if (n == 0) {
            return 0;
        }
        int fast = 1, slow = 1;
        while (fast < n) {
            if (nums[fast] != nums[fast - 1]) {
                nums[slow] = nums[fast];
                ++slow;
            }
            ++fast;
        }
        return slow;
    }
}
```

### 19、[字符串的排列](https://leetcode-cn.com/problems/permutation-in-string/)

自己写

```java
public boolean checkInclusion(String s1, String s2) {
    List<String> list = new ArrayList<>();

    for (int i = 0; i < s2.length() - s1.length() + 1; i++) {
        list.add(s2.substring(i, i + s1.length()));
    }

    for (int i = 0; i < list.size(); i++) {
        char[] arrS2 = list.get(i).toCharArray();
        Arrays.sort(arrS2);

        char[] arrS1 = s1.toCharArray();
        Arrays.sort(arrS1);

        if(new String(arrS2).equals(new String(arrS1))) {
            return true;
        }
    }

    return false;
}
```

官方

```java
public boolean checkInclusion(String s1, String s2) {
    int n = s1.length(), m = s2.length();
    if (n > m) {
        return false;
    }
    int[] cnt = new int[26];
    for (int i = 0; i < n; ++i) {
        --cnt[s1.charAt(i) - 'a'];
    }
    int left = 0;
    for (int right = 0; right < m; ++right) {
        int x = s2.charAt(right) - 'a';
        ++cnt[x];
        while (cnt[x] > 0) {
            --cnt[s2.charAt(left) - 'a'];
            ++left;
        }
        if (right - left + 1 == n) {
            return true;
        }
    }
    return false;
}
```

### 20、[ 图像渲染](https://leetcode-cn.com/problems/flood-fill/)

自己写

```java
class Solution {
    public int[][] floodFill(int[][] image, int sr, int sc, int newColor) {
        int num = image[sr][sc];
        image[sr][sc] = newColor;
        newColor(image, sr, sc, newColor, num);
        return image;
    }

    public void newColor(int[][] image, int sr, int sc, int newColor, int num) {
        int x1 = sr - 1;
        int x2 = sr + 1;
        int y1 = sc - 1;
        int y2 = sc + 1;

        if (y1 >= 0 && y1 < image[sr].length && image[sr][y1] == num && image[sr][y1] != newColor) {
            image[sr][y1] = newColor;
            newColor(image, sr, y1, newColor, num);
        }
        if (y2 >= 0 && y2 < image[sr].length && image[sr][y2] == num && image[sr][y2] != newColor) {
            image[sr][y2] = newColor;
            newColor(image, sr, y2, newColor, num);
        }
        if (x1 >= 0 && x1 < image.length && image[x1][sc] == num && image[x1][sc] != newColor) {
            image[x1][sc] = newColor;
            newColor(image, x1, sc, newColor, num);
        }
        if (x2 >= 0 && x2 < image.length && image[x2][sc] == num && image[x2][sc] != newColor) {
            image[x2][sc] = newColor;
            newColor(image, x2, sc, newColor, num);
        }
    }
}
```

优化

```java
class Solution {
    public int[][] floodFill(int[][] image, int sr, int sc, int newColor) {
        helper(image, sr, sc, newColor, image[sr][sc]);
        return image;
    }

    void helper(int[][] image, int sr, int sc, int newColor, int oldColor) {
        
        if (sr < 0 || sc < 0 || sr >= image.length || sc >= image[0].length 
            || image[sr][sc] != oldColor || newColor == oldColor){
            return;
        }
            
        image[sr][sc] = newColor;

        helper(image, sr - 1, sc, newColor, oldColor);
        helper(image, sr + 1, sc, newColor, oldColor);
        helper(image, sr, sc - 1, newColor, oldColor);
        helper(image, sr, sc + 1, newColor, oldColor); 

    }
}
```

### 21、[岛屿的最大面积](https://leetcode-cn.com/problems/max-area-of-island/)

自己写

```java
class Solution {

    public int maxAreaOfIsland(int[][] grid) {
        int maxValue = 0;
        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[i].length; j++) {
                if (grid[i][j] != 0) {
                    int value = point(grid, i, j);
                    if(value > maxValue) {                        
                        maxValue = value;
                    }
                }
            }
        }
        return maxValue;
    }

    public static int point(int grid[][], int x, int y) {
        grid[x][y] = 0;

        int count = 1;
        int x1 = x - 1;
        int x2 = x + 1;
        int y1 = y - 1;
        int y2 = y + 1;

        if (y1 >= 0 && y1 < grid[x].length && grid[x][y1] == 1 ) {
            count = count + point(grid, x, y1);
        }
        if (y2 >= 0 && y2 < grid[x].length && grid[x][y2] == 1 ) {
            count = count + point(grid, x, y2);
        }
        if (x1 >= 0 && x1 < grid.length && grid[x1][y] == 1 ) {
            count = count + point(grid, x1, y);
        }
        if (x2 >= 0 && x2 < grid.length && grid[x2][y] == 1 ) {
            count = count + point(grid, x2, y);
        }

        return count;
    }
}
```

优化

```java
class Solution {

    public int maxAreaOfIsland(int[][] grid) {
        int maxValue = 0;
        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[i].length; j++) {
                if (grid[i][j] != 0) {
                    int value = point(grid, i, j);
                    if(value > maxValue) {                        
                        maxValue = value;
                    }
                }
            }
        }
        return maxValue;
    }

    public static int point(int grid[][], int x, int y) {
        if(x < 0 || x >= grid.length || y < 0 || y >= grid[0].length || grid[x][y] == 0){
            return 0;
        }
        grid[x][y] = 0;

        int count = 1;

        count = count + point(grid, x+1, y);
        count = count + point(grid, x-1, y);
        count = count + point(grid, x, y+1);
        count = count + point(grid, x, y-1);

        return count;
    }
}
```

### 22、[合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees/)

官方

```java
class Solution {
    public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
        if (t1 == null) {
            return t2;
        }
        if (t2 == null) {
            return t1;
        }
        TreeNode merged = new TreeNode(t1.val + t2.val);
        merged.left = mergeTrees(t1.left, t2.left);
        merged.right = mergeTrees(t1.right, t2.right);
        return merged;
    }
}
```

### 23、[填充每个节点的下一个右侧节点指针](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/)

自己写

```java
class Solution {
    public Node connect(Node root) {
        if(root == null) {
            return new Node().next;
        }
        if(root.left == null) {
            return new Node();
        }
        ct(root);  
        return root;
    }

    public void ct(Node root) {
        if(root.left == null) {
            return;
        }
        root.left.next = root.right;
        if(root.left.right != null) {
            ct1(root.left.right, root.right.left);
        }
        ct(root.left);
        ct(root.right);
    }

    public void ct1(Node root1, Node root2) {
        root1.next = root2;
        if(root1.right != null) {
            ct1(root1.right, root2.left);
        }
    }
}
```

官方

```java
class Solution {
    public Node connect(Node root) {
        if (root == null) {
            return root;
        }
        // 从根节点开始
        Node leftmost = root;
        while (leftmost.left != null) {
            // 遍历这一层节点组织成的链表，为下一层的节点更新 next 指针
            Node head = leftmost;
            while (head != null) {    
                // CONNECTION 1
                head.left.next = head.right;
                // CONNECTION 2
                if (head.next != null) {
                    head.right.next = head.next.left;
                }
                // 指针向后移动
                head = head.next;
            }
            // 去下一层的最左的节点
            leftmost = leftmost.left;
        }
        return root;
    }
}
```

### 23、[合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

官方

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        } else if (l2 == null) {
            return l1;
        } else if (l1.val < l2.val) {
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        } else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }

    }
}
```

### 24、[反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

自己写

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if(head == null) {
            return new ListNode().next;
        }
        Stack<ListNode> stack = new Stack<>();

        while (head != null) {
            stack.add(head);
            head = head.next;
        }

        int len = stack.size();
        ListNode arrListNode[] = new ListNode[len];
        for (int i = 0; i < len; i++) {
            arrListNode[i] = new ListNode(stack.pop().val);
        }
        
        for (int i = 0; i < arrListNode.length - 1; i++) {
            arrListNode[i].next = arrListNode[i+1];
        }

        return arrListNode[0];
    }
}
```

官方

```java
// 迭代
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode curr = head;
        while (curr != null) {
            ListNode next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    }
}
// 递归
class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode newHead = reverseList(head.next);
        head.next.next = head;
        head.next = null;
        return newHead;
    }
}
```

### 25、[组合](https://leetcode-cn.com/problems/combinations/)

精选

```java
class Solution {
    public List<List<Integer>> combine(int n, int k) {
        List<List<Integer>> res = new ArrayList<>();
        if (k <= 0 || n < k) {
            return res;
        }
        // 从 1 开始是题目的设定
        Deque<Integer> path = new ArrayDeque<>();
        dfs(n, k, 1, path, res);
        return res;
    }

    private void dfs(int n, int k, int begin, Deque<Integer> path, List<List<Integer>> res) {
        // 递归终止条件是：path 的长度等于 k
        if (path.size() == k) {
            res.add(new ArrayList<>(path));
            return;
        }

        // 遍历可能的搜索起点
        for (int i = begin; i <= n; i++) {
            // 向路径变量里添加一个数
            path.addLast(i);
            // 下一轮搜索，设置的搜索起点要加 1，因为组合数理不允许出现重复的元素
            dfs(n, k, i + 1, path, res);
            // 重点理解这里：深度优先遍历有回头的过程，因此递归之前做了什么，递归之后需要做相同操作的逆向操作
            path.removeLast();
        }
    }
}
```













