# Subsequence

**Subsequence** problems can usually be solved efficiently by DP. On LeetCode, these problems usually have "subsequence" written in the title of the problem. However, we don't really have a problem title in interviews. How do we identify such questions?

When it comes to LeetCode and interview questions, a **subsequence** generally doesn't have to be contiguous from the parent sequence - this is different from **subarray** or **substring**, which are typically contiguous slices of the parents. 

Also, the word **sequence** suggests that there is a specific order that we would need to retain from the parent structure. If the order is not relevant and we are only looking for combinations, then backtracking might be the way to go.

In this type of problems, we only work with vanilla linear data structure arrays or strings, so there isn't any fancy data structure we need to abstract, and naturally the subproblems are simply the same problem for a slice of parent structure. We typically need to iterate through the structure twice to explore all possibilities of starting / ending points of substructures. 

Therefore, the best way to tackle these problems is to first draw a table to visualize the problem - the columns and rows of the table will be each element in the structure we will iterate to, and the table contents will be the specific subproblem we are dealing with at that column-row combination. We will illustrate this method in some examples below.

## Example - 300. Longest Increasing Subsequence

### Problem Description

Given an integer array `nums`, return the length of the longest strictly increasing subsequence.

A subsequence is a sequence that can be derived from an array by deleting some or no elements without changing the order of the remaining elements. For example, `[3,6,2,7]` is a subsequence of the array `[0,3,1,6,2,2,7]`.

**Example 1**:

```
Input: nums = [10,9,2,5,3,7,101,18]
Output: 4
Explanation: The longest increasing subsequence is [2,3,7,101], therefore the length is 4.
```

**Example 2**:

```
Input: nums = [0,1,0,3,2,3]
Output: 4
```

**Example 3**:

```
Input: nums = [7,7,7,7,7,7,7]
Output: 1
```

**Constraints**:

* `1 <= nums.length <= 2500`
* `-104 <= nums[i] <= 104`

### Intuition

Let's work with Example 1 above. In this problem, again we are only given an array to work with, and that makes things simple - the subproblem must be to solve the same problem for a smaller subarray. So for the memoization container `dp`, we will define it so that `dp[i]` stores the length of the longest increasing subsequence (LIS) for the substring that ends at index `i`.

* First of all, for any sequence, the minimum length of the LIS must be 1, because there exists at least a LIS with 1 number.
* Index `0` - we are only working with `[10]`, so the LIS must be itself. Therefore, `dp[0] = 1`. 
* Index `1` - we are working with `[10, 9]`. The LIS ends with number `9` would just be `[9]`, because number `10` is not smaller than number `9`. So `dp[1] = 1`.
* Index `2` - we are working with `[10, 9, 2]`. Still, The LIS ends with number `2` would just be `[2]`, because neither `10` nor `9` is smaller than `2`. So `dp[2] = 1`.
* Index `3` - we are working with `[10, 9, 2, 5]`. Things finally get a bit interesting here. Since `2` is smaller than `5`, we now have a subsequence of `[2, 5]` that's constructed by adding `5` to the end of the LIS that ends with `2`. So `dp[3] = 2`.
* Index `4` - we have `[10, 9, 2, 5, 3]`. Since `2` is smaller than `3`, `dp[4] = 2`.
* Index `5` - we have `[10, 9, 2, 5, 3, 7]`. Since `2` is smaller than `7`, `dp[4] = 2`. `5` is smaller than `7`, so `dp[4] = 3`. Finally, `3` is also smaller than `7`, but note that `dp[4]` is not incremented to `4`. Why so? We know that all of `2`, `5`, and `3` can come right before `7`.
    - If we want `2` to come right before `7`, we just need to get the LIS that ends with `2`, and add `7` to it. So the result is `dp[2] + 1 = 1 + 1 = 2`.
    - If we want `5` to come right before `7`, we just need to get the LIS that ends with `5`, and add `7` to it. So the result is `dp[3] + 1 = 2 + 1 = 3`.
    - If we want `3` to come right before `7`, we just need to get the LIS that ends with `3`, and add `7` to it. So the result is `dp[4] + 1 = 2 + 1 = 3`.
* ...

If we do it in a table, that's like this:

| start\end  | 0: `10` | 1: `9` | 2: `2` | 3: `5` | 4: `3` | 5: `7` |
|:----------:|:-----:|:----:|:----:|:----:|:----:|:----:|
|**0: `10`** | **`LIS(0) = [10]`<br/>`dp[0]=1`** | `10 > 9`<br/>`No LIS` | `10 > 2`<br/>`Not LIS` | `10 > 5`<br/>`Not LIS` | `10 > 3`<br/>`Not LIS` | `10 > 7`<br/>`Not LIS` |
|**1: `9`**  |  | **`LIS(1) = [9]`<br/>`dp[1]=1`** | `9 > 2`<br/>`Not LIS` | `9 > 5`<br/>`Not LIS` | `9 > 3`<br/>`Not LIS` | `9 > 7`<br/>`Not LIS` |
|**2: `2`**  |  |  | **`LIS(2) = [2]`<br/>`dp[2]=1`** | **`2 < 5`<br/>`LIS(3)=LIS(2)+[5]`<br/>`dp[3]=1+1=2`** | **`2 < 3`<br/>`LIS(4)=LIS(2)+[3]`<br/>`dp[4]=1+1=2`** | `2 < 7`<br/>`LIS(5)=LIS(2)+[7]`<br/>`dp[5]=1+1=2` |
|**3: `5`**  |  |  |  | `LIS(3) = [5]`<br/>`dp[3]=1` | `5 > 3`<br/>`Not LIS` | **`5 < 7`<br/>`LIS(5)=LIS(3)+[7]`<br/>`dp[5]=1+2=3`** |
|**4: `3`**  |  |  |  |  | `LIS(4) = [3]`<br/>`dp[4]=1` | **`3 < 7`<br/>`LIS(5)=LIS(4)+[7]`<br/>`dp[5]=1+2=3`** |
|**5: `7`**  |  |  |  |  |  | `LIS(5) = [7]`<br/>`dp[5]=1` |

There are still a few more numbers we haven't checked, but at this point it should already be clear what the pattern is:

1. Iterate through the array at each index `i`;
2. Iterate through all the numbers before `nums[i]` at index `j` such that `j < i`;
3. For all `j`'s such that `j < i`, we already know the length of LIS that ends with `nums[j]` - that's `dp[j]`;
4. For all `j`'s such that `nums[j] < nums[i]`, **we can build a new increasing subsequence that ends with `nums[i]`, by adding it to the end of the LIS that ends with `nums[j]`**;
5. `dp[i]` - the length of the LIS that ends with `nums[i]` - would just be the largest of all such `dp[j] + 1`;
6. The final answer is just the maximum of `dp[i]` among all `i`'s.

Hope you are still with me. The logic above may seem complicated, but the implementation should be very straightforward.

### Solution

```python
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        
        dp = [1] * len(nums)
        maxLength = 1
        
        # Base case when the length of the given array is 1, just return 1
        if len(nums) == 1:
            return maxLength
        
        # Iterations
        for i in range(len(nums)):
            for j in range(i):
                if nums[j] < nums[i]:
                    dp[i] = max(dp[i], 1 + dp[j])
            maxLength = max(maxLength, dp[i])
            
        return maxLength
```

## Example - 1143. Longest Common Subsequence

### Problem Description

Given two strings `text1` and `text2`, return the length of their longest common subsequence.

A subsequence of a string is a new string generated from the original string with some characters(can be none) deleted without changing the relative order of the remaining characters. (eg, "ace" is a subsequence of "abcde" while "aec" is not). A common subsequence of two strings is a subsequence that is common to both strings.

If there is no common subsequence, return 0.

**Example 1**:

```
Input: text1 = "abcde", text2 = "ace" 
Output: 3  
Explanation: The longest common subsequence is "ace" and its length is 3.
```

**Example 2**:

```
Input: text1 = "abc", text2 = "abc"
Output: 3
Explanation: The longest common subsequence is "abc" and its length is 3.
```

**Example 3**:

```
Input: text1 = "abc", text2 = "def"
Output: 0
Explanation: There is no such common subsequence, so the result is 0.
```

**Constraints**:

* `1 <= text1.length <= 1000`
* `1 <= text2.length <= 1000`
* The input strings consist of lowercase English characters only.

### Intuition

I am going to make Example 1 above slightly more complicated, to deal with some special cases. I am going to have `text1 = 'abcde'` and `text2 = 'acfea'`. Right off the bat you should see that it doesn't really change the result at all - my longest common subsequence should remain as `'ace'` and therefore the result should be 3.

Again, there isn't any fancy abstract data structure here. However, it's slightly different from the previous example - we are working with 2 different sequences instead of 1. 

But that doesn't really matter much because our subproblem or optimal substructure should still be just slices of these sequences. The idea is similar to the previous problem - we want to iterate through the 2 strings, character by character, with pointers `i` for `text1` and `j` for `text2`. For each `i`, `j` pair, we will try to get the maximum longest common subsequence between `text1[:i+1]` and `text2[:j+1]`, i.e. substrings of `text1` and `text2` ending at `i`, `j` indices. 

> To do this, we will compare the pair of characters we encounter at each pass. If they are the same character, then we get a common subsequence that's 1 longer than the previously longest subsequence we've encountered. If they aren't the same, then we need to carry the length of current longest subsequence over for future use.

The **difference** from the previous problem is that this is a true 2-dimensional probblem, whereas in the previous one, we visualized the 1-dimensional problem in a 2-dimensional way. As a result, our memoization container `dp` would now be a 2d array storing the result for each `i`, `j` pair. But note that the size of `dp` is not `len(text1) x len(text2)`, because we also added a helper row and a helper column to track the result when the substrings are just empty strings, i.e. the edge cases when `i == 0` or `j == 0`. Therefore, we store the result for `i`, `j` pair at `dp[i+1][j+1]` (effectively, turning the container `dp` into 1-based index instead of 0-based index). For them, the longest common subsequence is obviously just an empty string with length of 0.

![LC 1143 Example 1](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_1.jpg)

Now we start iterating. Right off the bat, the first character of each string is `'a'` when `i == 0` and `j == 0`. Therefore we need to add 1 to our previous result. But to which result? Well, the result we want to add to is `dp[0][0]`, because that's **the last place where we hadn't encountered the common character `'a'`**. So `dp[1][1] = dp[0][0] + 1 = 0 + 1 = 1`. 

![LC 1143 Example 2](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_2.jpg)

Moving on, we are at `(0, 1)` working with `'a'` and `'ac'` (trying to store the result at `dp[1][2]`). The ending characters are not common and therefore there is nothing to add here. However, we know from the previous state `dp[1][1]` that we already found a common subsequence of just `'a'` with length 1 - adding one more character in the end doesn't change that. 

But, that's not the only previous result we need to check against. We would also check `dp[0][2]` where we stored the result of working with `''` and `'ac'` which is just length 0. Out of these two results, `1 > 0`, so we take the larger one. Therefore `dp[1][2] = max(dp[1][1], dp[0][2]) = max(1, 0) = 1`. 

![LC 1143 Example 3](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_3.jpg)

Fast forward to `(0, 4)` or `dp[1][5]`, and we actually find a common ending character `'a'`, again. However, this doesn't mean that we suddenly have a common subsequence `'aa'` of length 2, because in the `text1` substring there is only 1 `'a'`. So the logic we established earlier still holds - we will add 1 to previous result where we hadn't encountered with `'a'` - it was `dp[0][4]` with a result of length 0. So `dp[1][5] = dp[0][4] + 1 = 0 + 1 = 1`. 

Effectively, `dp[1][5]` creates an alternative branch where the common subsequence consists of the 2nd `'a'` we encountered in `text2` substring. Since both branches have the same length 1, they will coexist in `dp` for the time being, until one branch gets taken out later when we find a longer subsequence in the other branch. 

![LC 1143 Example 4](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_4.jpg)

Fast forward to `(2, 2)` or `dp[3][3]`. We would have just passed `(2, 1)` where we would have seen a common ending character of `'c'`. So `dp[3, 2] = 2`. As for `dp[2][3]`, the result was just 1. Hence, `dp[3][3] = max(dp[2][3], dp[3][2]) = max(1, 2) = 2`.

![LC 1143 Example 5](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_5.jpg)

Fast forward to `(2, 4)` or `dp[3][5]`. **This is a very important step in fully understanding the logic**. We have 2 branches colliding here. On the left, we have `dp[2][5]` with the common subsequence of length 1 consisting of the 2nd `'a'` we encountered in `text2`. On the top, we have `dp[3][4]` with the common subsequence of length 2 consisting of the 1st `'a'` we encountered in `text2`, and a `'c'`. Since the problem wants us to find the longest subsequence, we will need to go with the larger of the two - `dp[3][4]`. Therefore, `dp[3][5] = max(dp[2][5], dp[3][4]) = max(1, 2) = 2`. 

From now on, we would no longer consider the alternative branch where we took the 2nd `'a'` in `text2`, because it's already shorter than the longest subsequence we got from the other branch, and we are already at the end of `text2` so it won't get any longer.

![LC 1143 Example 6](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_6.jpg)

The rest of the journey will be rather simple. We'd encounter another common character `'e'` at `(4, 4)` and our longest length is now 3. The result to the final problem is just `dp[5][5]` where we checked `text1` and `text2` themselves, and therefore the final result is just 3.

![LC 1143 Example 7](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_7.jpg)

I hope this lengthy walkthrough of the example is helpful for explaining the logic, because at least for me it wasn't clear at first. Now, we can summarize this logic and generalize the relationships among the subproblems.

```python
if text1[i] == text2[j]:
    dp[i+1][j+1] = dp[i][j] + 1
else:
    dp[i+1][j+1] = max(dp[i][j+1], dp[i+1][j]) 
```

### Solution

```python
class Solution:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        
        # Initialize the dp container as 2d array in 1-based index
        dp = [ [0] * (len(text2) + 1) for _ in range(len(text1) + 1) ]
        
        # Iterate through each character of the two strings and solve subproblems
        for i in range(len(text1)):
            for j in range(len(text2)):
                if text1[i] == text2[j]:
                    dp[i+1][j+1] = dp[i][j] + 1
                else:
                    dp[i+1][j+1] = max(dp[i][j+1], dp[i+1][j])
                    
        return dp[-1][-1]
```
