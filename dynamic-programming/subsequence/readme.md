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

Naturally, our memoization container `dp` would be a 2d array storing the result for each `i`, `j` pair. But note that the size of `dp` is not `len(text1) x len(text2)`, because we also added a helper row and a helper column to track the result when the substrings are just empty strings, i.e. the edge cases when `i == 0` or `j == 0`. Therefore, we store the result for `i`, `j` pair at `dp[i+1][j+1]`. For them, the longest common subsequence is obviously also just an empty string with length of 0.

![LC 1143 Example 1](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_1.jpg)

Now we start iterating. Right off the bat when `i == 0` and `j == 0`, the first character of each string is `'a'`. Therefore we need to add 1 to our previous result. But to which result? Well, the result we want to add to is `dp[0][0]`, because that's the place where we hadn't encountered the common character `'a'`. So `dp[1][1] = dp[0][0] + 1 = 0 + 1 = 0`. Don't worry if you are a little lost here, we will revisit this logic very soon to see why it should be this case.

![LC 1143 Example 2](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_2.jpg)


![LC 1143 Example 3](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/subsequence/assets/1143_example_3.jpg)
