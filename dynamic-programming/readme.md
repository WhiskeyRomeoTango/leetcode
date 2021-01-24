# Dynamic Programming

## Definition

Dynamic Programming (**DP** for short) is a programming / computing technique used commonly for optimization problems. It simplifies a problem by 1) **breaking down the problem into simpler sub-problems**, and 2) **storing the solution to each sub-problem** so that future calculations can be avoided when the sub-problem is revisted. The technique in part 2) of storing solutions of sub-problems is called **Memoization**.

## (Bad?) Example of Fibonacci Sequence

This sounds very cool and efficient, but that was it - most people (including me) still don't understand it when they see this definition for the first time. Many tutorials, teachers, or textbooks then use the classic example of fibonacci sequence to illustrate it. That is, a very costly recursive function of calculating the fibonacci number at position n:

```python
def fibonacci(n): # This recursion has a time complexity of O(2^n) because it creates a binary tree, O(1.618^n) to be exact
    if n == 0: return 0
    elif n == 1: return 1
    else: return fibonacci(n-1) + fibonacci(n-2)
```

becomes

```python
def fibonacci(n): # This function has a time complexity of O(n) because it runs only 1 pass
    dp = [0] * (n + 1)
    dp[0], dp[1] = 0, 1
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```

Personally I think it's a lousy way of introducing DP, because it's a very shallow example used for explaning a technique used for many complicated problems Interviewers won't ever ask you to implement a fibonacci function, and the problem they give you will be much more complex than this.

## How to Identify if DP is Suitable for A Problem

Identifying the problems that can be best solved by DP needs to go back to its definiton. For the first part, the problem needs to be broken down into smaller sub-problems. "Sub-problem" screams recursion. But it doesn't mean we should try DP for any recursive approach we take. The key of DP lies in the second part of its definition, **storing solutions of the sub-problems**. What solutions are worth storing?

The solutions that are worth storing are the ones we have to solve **repeatedly**. Consider the problem where we use [backtracking](https://github.com/WhiskeyRomeoTango/leetcode/tree/main/searching/backtracking) to solve Sudoku. A Sudoku game can be broken down into millions of sub-problems of whether we can put a number k at coordiate (i, j). The issue is that the sub-problems do not **overlap** - i.e. once we solve the problem we would never revisit it again, so memoization would add 0 value there. In the fibonacci example though, because we repeatedly solve the same / overlapping sub-problems, it then becomes worthwhile to store these solutions to be reused later, because we know we need it later.

> It's best to use DP to solve problems with common or overlapping sub-problems.

## How to Solve DP Problems

I really like [YouTuber Reducible's video on DP](https://www.youtube.com/watch?v=aPQY__2H3tE&ab_channel=Reducible). I am going to just list the steps he summarized, and then use a problem to illustrate how to apply these steps.

1. Visualize Examples (w/ Directed Acyclic Graph)
2. Find an Appropriate Sub-problem
3. Find Relationships among Sub-problems
4. Generalize the Relationship
5. Implement by Solving Sub-problems in Order

## Example - 62: Unique Paths 

### Problem Description

A robot is located at the top-left corner of a `m x n` grid (marked 'Start' in the diagram below).

The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid (marked 'Finish' in the diagram below).

How many possible unique paths are there?
