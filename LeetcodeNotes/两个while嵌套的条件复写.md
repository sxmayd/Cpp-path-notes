# 两个while嵌套的条件复写

#### [剑指 Offer 21. 调整数组顺序使奇数位于偶数前面](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)

题目描述：输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数位于数组的前半部分，所有偶数位于数组的后半部分。

```C++
class Solution {
public:
    vector<int> exchange(vector<int>& nums) {
        if(nums.size() == 0) return {};

        int pBegin = 0;
        int pEnd = nums.size() - 1;

        while(pBegin < pEnd){
            // 向后移动pBegin，直到其指向偶数
            while(pBegin < pEnd && (nums[pBegin] & 0x1) != 0) pBegin++;

            // 向前移动pEnd，直到其指向奇数
            while(pBegin < pEnd && (nums[pEnd] & 0x1) == 0) pEnd--;

            swap(nums[pBegin], nums[pEnd]);
        }

        return nums;
    }
};
```

`注`

1. 需要注意在第二层的while循环中同样需要 $pBegin < pEnd$​ 这一条件

2. 有趣的是，上述的代码也可以用下面的形式来表达：

```C++
class Solution {
public:
    vector<int> exchange(vector<int>& nums) {
        int left = 0, right = nums.size() - 1;
        while (left < right) {
            if ((nums[left] & 0x1) != 0) {
                left++;
                continue;
            }
            if ((nums[right] & 0x1) != 1) {
                right--;
                continue;
            }
            swap(nums[left], nums[right]);
        }
        return nums;
    }
};
```

3. 本题还可以用快慢指针来做：

   - 定义快慢双指针 fast 和 low ，fast 在前， low 在后 .
   - fast的作用是向前搜索奇数位置，low 的作用是指向下一个奇数应当存放的位置
   - fast 向前移动，当它搜索到奇数时，将它和 nums[low] 交换，此时 low 向前移动一个位置 .
   - 重复上述操作，直到 fast 指向数组末尾 .

   ```C++
   class Solution {
   public:
       vector<int> exchange(vector<int>& nums) {
           int low = 0, fast = 0;
           while (fast < nums.size()) {
               if (nums[fast] & 1) {
                   swap(nums[low], nums[fast]);
                   low ++;
               }
               fast ++;
           }
           return nums;
       }
   };
   ```


   

