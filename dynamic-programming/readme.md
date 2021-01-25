# Dynamic Programming

## Definition

Dynamic Programming (**DP** for short) is a programming / computing technique used commonly for optimization problems. It simplifies a problem by 1) **breaking down the problem into simpler sub-problems**, and 2) **storing the solution to each sub-problem** so that future calculations can be avoided when the sub-problem is revisted. The technique in part 2) of storing solutions of sub-problems is called **Memoization**.

## Example of Fibonacci Sequence

The definition makes it soundd very cool and efficient, but that was it - most people (including me) still don't understand it when they see this definition for the first time. Many tutorials, teachers, or textbooks then use the classic example of fibonacci sequence to illustrate it. That is, a very costly recursive function of calculating the fibonacci number at position n:

```python
def fibonacci(n): # This has a time complexity of O(2^n) as it creates a binary tree at each call, O(1.618^n) to be exact
    if n == 0: return 0
    elif n == 1: return 1
    else: return fibonacci(n-1) + fibonacci(n-2)
```

becomes

```python
def fibonacci(n): # This has a time complexity of O(n) because it runs only 1 pass
    dp = [0] * (n + 1)
    dp[0], dp[1] = 0, 1
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```

Personally I think it's a great example of showing how impressive not the best way to introduce DP, because it's a very shallow example used for explaning a technique used for many complicated problems. 

## How to Identify if DP is Suitable for A Problem

Identifying the problems that can be best solved by DP needs to go back to its definiton. For the first part, the problem needs to be broken down into smaller sub-problems. "Sub-problem" screams recursion. But it doesn't mean we should try DP for any recursive approach we take. The key of DP lies in the second part of its definition, **storing solutions of the sub-problems**. What solutions are worth storing?

The solutions that are worth storing are the ones we have to solve **repeatedly**. Consider the problem where we use [backtracking](https://github.com/WhiskeyRomeoTango/leetcode/tree/main/searching/backtracking) to solve Sudoku. A Sudoku game can be broken down into millions of sub-problems of whether we can put a number k at coordiate (i, j). The issue is that the sub-problems do not **overlap** - i.e. once we solve the problem we would never revisit it again, so memoization would add 0 value there. In the fibonacci example though, because we repeatedly solve the same / overlapping sub-problems, it then becomes worthwhile to store these solutions to be reused later, because we know we need it later.

> It's best to use DP to solve problems with common or overlapping sub-problems.

## How to Solve DP Problems

I really like [YouTuber Reducible's video on DP](https://www.youtube.com/watch?v=aPQY__2H3tE&ab_channel=Reducible). I am going to just list the steps he summarized (with a bit of my addition), and then use a problem to illustrate how to apply these steps.

1. Identify the Variables and Visualize Examples (w/ Directed Acyclic Graph)
2. Find an Appropriate Sub-problem
3. Find Relationships among Sub-problems
4. Generalize the Relationship
5. Implement by Solving Sub-problems in Order
6. (Optional) Optimize Space Complexity of Memoization

## Example - 62: Unique Paths 

### Problem Description

A robot is located at the top-left corner of a `m x n` grid (marked 'Start' in the diagram below).

The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid (marked 'Finish' in the diagram below).

How many possible unique paths are there?

![Diagram](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/_assets/robot_maze.png)

### Step 1: Identify the Variables and Visualize Examples:

I'm not going to use the diagram above as example. Instead we'll just try a simpler one of a 3x3 maze below.

```
---------
| R |   |
---------
|   |   |
---------
|   | F |
---------
```

The only changing variable is the **location** of the robot, and we can use a `(i, j)` coordinate the represent that, where `i` is the row index and `j` is the column index. The start location is `(0, 0)`, and the finish location is `(2, 1)`. 

Since the dumb robot can only move right or down in the grid, we can draw a simple DAG of the paths it may take. It's actually just going to be a binary tree, where the left branch is the new location when it moves down, and the right branch when it moves right. From this graph it's actually already clear that in this example there are 3 paths for the robot to get to the finish point. We also see in this graph that the tree is not perfect since we have the cases of the robot couldn't move further right or down when it hits the edge.

```
       (0, 0)
       /    \
   (1, 0)  (0, 1)
   /    \  /
(2, 0) (1, 1)
   \    /
   (2, 1)
```

### Step 2: Find an Appropriate Sub-problem

A sub-problem has to be simpler version of the parent problem. In this example, a sub-problem `paths(i, j)` is basically to find the # of paths for the robot to reach a preceding finish point of `(i, j)` where `i < m-1` or `j < n-1`. For example, to solve for `paths(1, 1)`, we get this following sub-tree from the tree above. And we can see that the robot has 2 paths to reach `(1, 1)`. We can also break down this further, where `paths(1, 0) = 1` and `paths(0, 1) = 1`.

```
       (0, 0)
       /    \
   (1, 0)  (0, 1)
       \    /
       (1, 1)
```

### Step 3: Find Relationships among Sub-problems

If we inspect `paths(1, 1)` a bit closer, we can see that there are 2 paths the robot can get there, from `(1, 0)` or `(0, 1)`. For the latter two, there is only 1 path for each position from `(0, 0)`. So in the end there are only 2 paths to get from `(0, 0)` to `(1, 1)`.

For `paths(2, 0)`, there is only 1 path - that is from `(0, 0)` to `(1, 0)` to `(2, 0)`. No detour here.

Finally `paths(2, 1)` which is also just our ultimate problem. The robot can get there from either `(2, 0)` or `(1, 1)`. We already saw that there are 1 and 2 paths to get to them respectively, and therefore there will be `2 + 1 = 3` paths to get to `(0, 0)`.

### Step 4: Generalize the Relationship

By now it should already be very clear that a simple relationship can be established here - a robot at `(i, j)` would have only come from `(i-1, J)` or `(i, j-1)`, representing the position on the top or on the left of it. There is further restriction that the robot couldn't have come from `(i, j)` when `i < 0` or `j < 0`.

Also, sicne the problem is about counting the # of paths, then `paths(i, j) = paths(i-1, j) + paths(i, j-1)`, following the same constraint above as well.

### Step 5: Implement by Solving Sub-problems in Order

Before we solve these subproblems, we need to create a data structure to perform memoization, i.e. storing the solutions. Normally we will just use the same data structure of the given problem. In this example, we'll just create `m x n` 2-d array / list structure to hold the solutions while we iterate through the board. 

Note that since we start at `(0, 0)` and we have to be able to have a path to come from it, we will set `dp[0][0] = 1`. So for an edge case where the robot works in a `1x1` grid that it's already at the finish point at the start, it has only 1 path.

```python
dp = [ [1 for col in range(m)] for row in range(n) ]
```

Since the robot can only go right or down, we can just iterate the grid normally (i.e. top-down, left-right). In the loops, we will completely skip `(0, 0)` as discussed above since it's a special case. Otherwise, we will add `dp[i-1][j]` to the solution if `i != 0`, and also add `dp[i][j-1]` if `j != 0`. Eventually, when we are out of the loops, we just need to return `dp[-1][-1]` which will be solution for the bottom-right coordinate.

```python
def uniquePaths(self, m: int, n: int) -> int:
        
    dp = [ [1 for col in range(m)] for row in range(n) ]

    for i in range(n):
        for j in range(m):
            paths = 0
            if i == 0 and j == 0:
                continue
            if i != 0:
                paths += dp[i-1][j]
            if j != 0:
                paths += dp[i][j-1]
            dp[i][j] = paths

    return dp[-1][-1]
```