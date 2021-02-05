# Subsequence

**Subsequence** problems can usually be solved efficiently by DP. When it comes to LeetCode and interview questions, a **subsequence** generally doesn't have to be contiguous from the parent sequence - this is different from **subarray** or **substring**, which are typically a contiguous cut of the parent. Also, the word sequence suggests that there is a specific order that we would need to retain from the parent array structure. Lastly, these problems typically ask us to find the subsequence with maximum length. If the goal is to find all subsequences that follow a particular rule, then backtracking is probably more suited. (Maybe ???) 

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

I am going to make the example 1 above slightly more complicated, to deal with some special cases. I am going to have `text1 = 'abcde'` and `text2 = 'acfea'`. Right off the bat you should see that it doesn't really change the result at all, and my longest common subsequence should remain `'ace'` and therefore the result should be 3.

There isn't any fancy or abstract data structure here - you are just given 2 strings to work with. In this case the subproblem or optimal substructure should just be working with subsets of these strings. The idea is that we want to iterate through the 2 strings with pointers `i` for `text1` and `j` for `text2`. For each `i`, `j` pair, we will get the maximum longest common subsequence between `text1[:i+1]` and `text2[:j+1]`, i.e. substrings of `text1` and `text2` ending at `i`, `j` indices.

Naturally, our memoization container `dp` would be a 2d array storing the result for each `i`, `j` pair. But note that the size of `dp` is not `len(text1) x len(text2)`, because we also added a helper row and a helper column to track the result when the substrings are just empty strings, i.e. the edge cases when `i == 0` or `j == 0`. Therefore, we store the result for `i`, `j` pair at `dp[i+1][j+1]` (effectively, turning the container `dp` into 1-based index instead of 0-based index). For them, the longest common subsequence is obviously also just an empty string with length of 0's.

![LC 1143 Example 1](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_1.jpg)

Now we start iterating. Right off the bat when `i == 0` and `j == 0`, the first character of each string is `'a'`. Therefore we need to add 1 to our previous result. But to which result? Well, the result we want to add to is `dp[0][0]`, because that's the place where we hadn't encountered the common character `'a'`. So `dp[1][1] = dp[0][0] + 1 = 0 + 1 = 1`. 

![LC 1143 Example 2](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_2.jpg)

Moving on, we are at `(0, 1)` working with `'a'` and `'ac'` (trying to store the result at `dp[1][2]`). The ending characters are not common and therefore there is nothing to add here. However, we know from the previous state `dp[1][1]` that we already found a common subsequence of just `'a'` with length 1 - adding one more character in the end doesn't change that. 

But, that's not the only previous result we need to check against. We would also check `dp[0][2]` where we stored the result of working with `''` and `'ac'` which is just length 0. Out of these two results, `1 > 0`, so we take the larger one. Therefore `dp[1][2] = max(dp[1][1], dp[0][2]) = max(1, 0) = 1`. 

![LC 1143 Example 3](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_3.jpg)

Fast forward to `(0, 4)` or `dp[1][5]`, and once again we find a common ending character `'a'` again. However, this doesn't mean that we suddenly have a common subsequence `'aa'` of length 2, because in the `text1` substring there is only 1 `'a'`. So the logic we established earlier still holds - we will add 1 to previous result where we hadn't encountered wither `'a'` - that was `dp[0][4]` and that had a result of length 0. So `dp[1][5] = dp[0][4] + 1 = 0 + 1 = 1`. This effectively means that `dp[1][5]` represents the common subsequence consists of the 2nd `'a'` we encountered in `text2` substring.

![LC 1143 Example 4](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_4.jpg)

Fast forward to `(3, 3)` or `dp[4][4]`. We would have just passed `(3, 2)` where we would have seen a common ending character of `'c'`. Therefore based on the logic `dp[4, 3] = 2`. As for `dp[3][4]`, the result was just 1. 

![LC 1143 Example 5](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_5.jpg)
