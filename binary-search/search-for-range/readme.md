# 34. Find First and Last Position of Element in Sorted Array

## Problem Description

Given an array of integers nums sorted in ascending order, find the starting and ending position of a given target value.

If target is not found in the array, return [-1, -1].

Follow up: Could you write an algorithm with O(log n) runtime complexity?

Example 1:

```
Input: nums = [5,7,7,8,8,10], target = 8
Output: [3,4]
```

Example 2:

```
Input: nums = [5,7,7,8,8,10], target = 6
Output: [-1,-1]
```

Example 3:

```
Input: nums = [], target = 0
Output: [-1,-1]
```

Constraints:

* `0 <= nums.length <= 105`
* `-109 <= nums[i] <= 109`
* `nums is a non-decreasing array`
* `-109 <= target <= 109`

## Intuition Leads to Solution

First of all, we can just brute force and loop through the list to find the answer. But that's not the point for this problem. The follow up of O(log n) time complexity already shouted binary search of some sort!

Consider a vanilla binary search algorithm below, which returns the index of the `target` value in a **sorted** list/array `nums`. 

```python
def binarySearch(nums, target):
    
    # Edge cases when the list is empty, or the target is out of the range
    if not nums:
        return None
    if target < nums[0] or target > nums[-1]:
        return None

    l = 0
    r = len(nums) - 1
    while l <= r:
        m = int((l + r) / 2)
        if target == nums[m]:
            return m
        elif target < nums[m]:
            r = m + 1
        else:
            l = m + 1
    
    return -1
```

This is working greatly, but with a catch. Notice that for the above algorithm, if we pass in a list of `nums=[1, 1, 1]` and `target=1`, then the return is `1`. It doesn't return the first occurrence of the target value. That's because at the first step, we already find the target value as the median value, and the function returns it directly.

This is fine if we are purely searching (e.g. to find if target exists in the list). However, in the case of returning an index, we typically want to return the index of the first occurrence. For that purpose, we can do the following.

```python
def firstOccurrence(nums, target):
    l = 0
    r = len(nums)
    while l < r:
        m = int((l + r) / 2)
        if target <= nums[m]:
            r = m
        else:
            l = m + 1
    
    return l if l < len(nums) and nums[l] == target else None
```

This is different from the vanilla binary search we saw earlier in several ways:

1. The starting right pointer is at `len(nums)`, instead of `len(nums) - 1`;
2. The while loop condition is `l < r` instead of `l <= r`;
3. Instead of returning the index when median value is equal to target, when target is LESS THAN or EQUAL TO the median value, we move to left half;
4. When reducing the list to its left half, we move right pointer to `r = m` instead of `r = m - 1`;
5. Edge case handling is moved to the bottom (note that I got rid of the edge case of empty list, ignore that for now).

To understand this, again we consider the case of `nums=[1, 1, 1]` and `target=1`. Since the target value (1) is always LESS THAN or EQUAL TO the median value(1), we will keep reducing the list to its left half (point 3 above). And, since the left half always have the median value included (point 4 above), if target value exists in the list, it's guaranteed that at least one target value will be in the left half. In the end, when left and right pointer converge and we come out of the loop (point 2 above), the item they point to is either the first occurrence of target, or the immediate smaller item if target is not in list. This is typically referred to as the closed-left and open-right approach. It's the custom for most languages like python (`range(0, 3)` gives `[0, 1, 2]` - doesn't give `3`). 

Finally, we just need to check 1) if l and r converged to the dummy index of `len(nums)` (point 1 above, meaning target value is larger than the largest value in the list), and 2) if the value at left pointer is indeed equal to target value (meaning target value is smaller than the smallest value in the list or not in the list at all). All of the edge cases were taken care of. This already solves half of the problem - to find the first position of element in sorted array.

We can then tweak the algorithm just a tiny bit to (help) us find the last position.

```python
def lowerBound(nums, target):
    l = 0
    r = len(nums)
    while l < r:
        m = int((l + r) / 2)
        if target <= nums[m]: # <= !!!
            r = m
        else:
            l = m + 1
    
    return l


def upperBound(nums, target):
    l = 0
    r = len(nums)
    while l < r:
        m = int((l + r) / 2)
        if target < nums[m]: # < !!!
            r = m
        else:
            l = m + 1
    
    return l
```

Note that these two functions are exactly the same, except for the check condition for `upperBound()` is `if target < nums[m]`, while for `lowerBound()` it's `if target <= nums[m]`. 

We can understand it as the opposite of `lowerBound()` - if target is GREATER THAN or EQUAL TO median value, we move to the right half of the list - but because the right half starts at `m + 1`, eventually the target value will be pushed out of the right half. At the same time, since if target is LESS THAN median, we will keep moving to the left half of the remaining list, the value to the immediate left of left pointer is guaranteed to be target value if it exists in the list. In the edge case when target value is at the end of the list, `l` and `r` will converge to the dummy index of `len(nums)`. So the actual lower bound would be `upperBound(nums, target) - 1`.

Finally we can put these two together. We already discussed that `l = lowerBound(nums, target)` and `r = upperBound(nums, target) - 1`. We will just need to check if `l` indeed points to target value. If yes, then return `[l, r]`, otherwise return `[-1, -1]`.

```python
def searchRange(self, nums, target):

    def lowerBound(nums, target):
        l = 0
        r = len(nums)
        while l < r:
            m = int((l + r) / 2)
            if target <= nums[m]: # <= !!!
                r = m
            else:
                l = m + 1

        return l


    def upperBound(nums, target):
        l = 0
        r = len(nums)
        while l < r:
            m = int((l + r) / 2)
            if target < nums[m]: # < !!!
                r = m
            else:
                l = m + 1

        return l

    # Don't forget the edge case when the list is empty!
    if not nums:
        return [-1, -1]
        
    l = lowerBound(nums, target)
    r = upperBound(nums, target) - 1
    if l == len(nums) or nums[l] != target:
        return [-1, -1]
    return [l, r]
```

## Related Problems

[278. First Bad Version](https://leetcode.com/problems/first-bad-version/submissions/)
