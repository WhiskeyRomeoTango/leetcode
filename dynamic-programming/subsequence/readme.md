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
