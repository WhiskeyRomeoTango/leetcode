# Dynamic Programming

## Definition

Dynamic Programming (**DP** for short) is a programming / computing technique used commonly for optimization problems. It simplifies a problem by 1) **breaking down the problem into simpler sub-problems**, and 2) **storing the solution to each sub-problem** so that future calculations can be avoided when the sub-problem is revisted. The technique in part 2) of storing solutions of sub-problems is called **Memoization**.

## (Bad?) Example of Fibonacci Sequence

The definition makes it sound very cool and efficient, but that was it - most people (including me) still don't understand it when they see this definition for the first time. Many tutorials, teachers, or textbooks then use the classic example of fibonacci sequence to illustrate it. A very costly recursive function of calculating the fibonacci number at position n:

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

Personally, I think it's a lousy way of introducing DP. Imagine you are doing fibonacci by hand on paper, you would start writing down `1, 1, 2, 3, 5, 8, ...` until you are at pos `n`. At any position `i`, you always know what are on position `i-1` and `i-2` and you just sum them up to get what's on `i`. The natural way to think about how to implement something by code is to think about how to implement it by hand. By doing that, I am already at the second function. On the other hand, although the first function is very clean and easy to implement, that's not how I would imagine a fibonacci sequence should be implemented.

Therefore, even though it's a great example of showing how impressive DP is, I think it's too shallow as an example to explain a complex technique used for many complex problems. Most of the interview or real-world problems won't be as straightforward as A+B - we need to spice it up a bit to see how DP really shines in some tricky situations.

## How to Identify if DP is Suitable for A Problem

To identify a problem that is best solved by DP, we need to go back to its definition. For the first part, the problem needs to be broken down into smaller sub-problems. In general, we break down a problem into smaller sub-problems in many other algorithms as well, such as backtracking, DFS, BFS, etc. However, this doesn't mean we should try DP for any of those approaches. The key of DP lies in the second part of its definition, **storing solutions of the sub-problems**. What kind of solutions are worth storing?

Well, the solutions that are worth storing are those to the **common** sub-problems we have to solve **repeatedly**. Consider the problem where we use [backtracking](https://github.com/WhiskeyRomeoTango/leetcode/tree/main/searching/backtracking) to solve Sudoku. A Sudoku game can be broken down into millions of sub-problems of putting a number `k` at an empty coordiate `(i, j)`. The issue is that these sub-problems never **overlap** - once we solve the sub-problem we would move on to the next sub-problem and never need to revisit it again. The solutions of preceeding sub-problems doesn't (at least directly) affect the following sub-problems, so memoization adds 0 value there. In the fibonacci example though, since we repeatedly solve the same / overlapping sub-problems, it then becomes worthwhile to store these solutions to be reused later.

> It's best to use DP to solve problems with common or overlapping sub-problems.

## How to Solve DP Problems

I really like [YouTuber Reducible's video on DP](https://www.youtube.com/watch?v=aPQY__2H3tE&ab_channel=Reducible). I am going to just list the steps he summarized (with a bit of my addition), and then use a problem to illustrate how to apply these steps.

0. Determine Whether DP is Useful for the Problem (already discussed above)
1. Identify the Variables and Visualize Examples
2. Find an Appropriate Sub-problem
3. Find Relationships among Sub-problems
4. Generalize the Relationship
5. Implement by Solving Sub-problems in Order
6. (Optional) Optimize Space Complexity of Memoization

## Example - 62. Unique Paths 

### Problem Description

A robot is located at the top-left corner of a `m x n` grid (marked 'Start' in the diagram below).

The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid (marked 'Finish' in the diagram below).

How many possible unique paths are there?

![Diagram](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/_assets/robot_maze.png)

### Step 0: Determine Whether DP is Useful for the Problem

Just for the fun, I implemented a backtracking algorithm to solve this problem, as shown below. Although I tested extensively and confirmed this is working, it exceeded time limit on LeetCode OJ, which indicates that my algorithm is too slow.

```python
def uniquePaths(m: int, n: int) -> int:

    paths = 0

    def backtrack(i, j, paths):
        if i == n-1 and j == m-1:
            paths += 1
            return paths
        else:
            if i < n - 1:
                paths = backtrack(i + 1, j, paths)
            if j < m - 1:
                paths = backtrack(i, j + 1, paths)
        return paths

    return backtrack(0, 0, paths)
```

Why would this take so long? The problem is that we are literally trying every single step to try to generate a path to the finish point, and once we reach it, we would backtrack all the way back to the starting point, and then take the other direction and do it all over again. Similar to the recursive, non-DP fibonacci algorithm, at each function call, we generate a binary tree to either go right or go down. The key is that throughout this process we will revist the same position we've been before, and **run the same calculation repeatedly**. This results in a time complexity of O(2^n) which is very very bad.

Therefore, we know we can use DP for this problem to enhance efficiency. Now we can start actually solving it.

### Step 1: Identify the Variables and Visualize Examples

I'm not going to use the diagram above as an example. Instead we'll just try a simpler one of a 3x2 maze below.

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

Since the dumb robot can only move right or down in the grid, we can draw a simple graph of the paths it may take. It's actually going to be a binary tree, where the left branch is the new location when it moves down, and the right branch when it moves right. From this graph it's actually already clear that in this example there are 3 paths for the robot to get to the finish point. We also see in this graph that the tree is not perfect since we have the cases of the robot couldn't move further right or down when it hits the edge.

```
          (0, 0)
          /    \
     (1, 0)    (0, 1)
     /    \    /
(2, 0)    (1, 1)
     \    /
     (2, 1)
```

### Step 2: Find an Appropriate Sub-problem

A sub-problem has to be a simpler / smaller version of the parent problem. In this example, a sub-problem `paths(i, j)` is basically to find the # of paths for the robot to reach a preceding finish point of `(i, j)` where `i < m-1` or `j < n-1`. For example, to solve for `paths(1, 1)`, we get this following sub-tree from the tree above. And we can see that the robot has 2 paths to reach `(1, 1)`. We can also break down this further, where `paths(1, 0) = 1` and `paths(0, 1) = 1`.

```
          (0, 0)
          /    \
     (1, 0)    (0, 1)
          \    /
          (1, 1)
```

### Step 3: Find Relationships among Sub-problems

If we inspect `paths(1, 1)` a bit closer, we can see that there are 2 paths the robot can get there, from `(1, 0)` or `(0, 1)`. For the latter two, there is only 1 path for each position from `(0, 0)`. So in the end there are only 2 paths to get from `(0, 0)` to `(1, 1)`.

For `paths(2, 0)`, there is only 1 path - that is from `(0, 0)` to `(1, 0)` to `(2, 0)`. No detour here.

Finally `paths(2, 1)` which is also just our ultimate problem. The robot can get there from either `(2, 0)` or `(1, 1)`. We already saw that there are 1 and 2 paths to get to them respectively, and therefore there will be `2 + 1 = 3` paths to get to `(2, 1)`.

### Step 4: Generalize the Relationship

By now it should already be very clear that a simple relationship can be established here - a robot at `(i, j)` would have only come from `(i-1, J)` or `(i, j-1)`, representing the position on the top or on the left of it. There is further restriction that the robot couldn't have come from `(i, j)` if `i < 0` or `j < 0`.

Since the problem is about counting the # of paths, then `paths(i, j) = paths(i-1, j) + paths(i, j-1)`, following the same constraint above as well.

### Step 5: Implement by Solving Sub-problems in Order

Before we solve these subproblems, we need to create a data structure to perform memoization, i.e. storing the solutions. Normally we will just go with the same data structure of the given input. In this example however, we are just given `m` and `n`, so we'll just create a `m x n` 2-d array / list structure to hold the solutions while we iterate through the board. 

Note that since we start at `(0, 0)` and need to satisfy the relationship we established above, we will set `dp[0][0] = 1`. This also guarantees that our algorithm will get the correct solution for the edge case where the robot works in a `1x1` grid, where it's already at the finish point at the start.

```python
dp = [ [0] * m ] * n
dp[0][0] = 1
```

Since the robot can only go right or down, we can just iterate the grid normally (i.e. top-down, left-right). In the loops, we will just skip `(0, 0)` as discussed above since it's a special case. Otherwise, we will add `dp[i-1][j]` to the solution if `i != 0`, and also add `dp[i][j-1]` if `j != 0`. Eventually, when we are out of the loops, we just need to return `dp[-1][-1]` which will be solution for the bottom-right coordinate.

```python
def uniquePaths(m: int, n: int) -> int:
        
    dp = [ [0] * m ] * n
    dp[0][0] = 1

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

    return dp[n-1][m-1]
```

## Example - 1641. Count Sorted Vowel Strings

### Problem Description

Given an integer `n`, return the number of strings of length `n` that consist only of vowels (`a`, `e`, `i`, `o`, `u`) and are **lexicographically sorted**.

A string `s` is **lexicographically sorted** if for all valid `i`, `s[i]` is the same as or comes before `s[i+1]` in the alphabet.

Example 1:

```
Input: n = 1
Output: 5
Explanation: The 5 sorted strings that consist of vowels only are ["a","e","i","o","u"].
```

Example 2:

```
Input: n = 2
Output: 15
Explanation: The 15 sorted strings that consist of vowels only are
["aa","ae","ai","ao","au","ee","ei","eo","eu","ii","io","iu","oo","ou","uu"].
Note that "ea" is not a valid string since 'e' comes after 'a' in the alphabet.
```

Example 3:

```
Input: n = 33
Output: 66045
```

### Step 0: Determine Whether DP is Useful for the Problem

I first encountered this problem when I was LeetCoding the backtracking tag. There I implemented the backtracking algorithm below that solved this problem in over 7,000 ms, which sounded very slow.

```python
def countVowelStrings(self, n: int) -> int:
    
    candidates = ['a', 'e', 'i', 'o', 'u'] # make sure this is sorted
    results = []

    def backtrack(s: str, index: int) -> None:
        if len(s) == n:
            results.append(s)
            return
        else:
            for i in range(index, len(candidates)):
                backtrack(s + candidates[i], i)

    backtrack('', 0)
    return len(results)
```

Imagine if you are asked to do this by hand, this backtracking approach would be exactly what you do. For example, let's set `n=3`. Note that we made sure the list `candidates` was sorted, and we kept track of our position within `candidates` so we generate only lexicographically sorted strings. Therefore, this backtracking algorithm will first generate `'aaa'`, and from there we will backtrack and subsequently generate `'aae'`, `'aai'`, `'aao'`, `'aau'`. After that, it's time to backtrack to the preceeding character, and we start with `e` in the 2nd position, `'aee'`, `'aei'`, ... 

For each position `i`, we calculate the number of characters we can based on what's in position 'i-1'. 

* If `a[i]` is `'a'`, we can put all 5 candidates next;
* If `a[i]` is `'e'`, we can put 4 candidates next, excluding `'a'`;
* If `a[i]` is `'i'`, we can put 3 candidates next, excluding `'a'` and `'e'`;
* If `a[i]` is `'o'`, we can put 2 candidates next, excluding `'a'`, `'e'`, and `'i'`;
* If `a[i]` is `'u'`, we can put only 1 candidate next, which is just `'u'` itself.

We do this by repeatedly looping through the list `candidates` 5, 4, 3, 2, and 1 times at each position, which becomes a time sink when n gets larger. However we can remember this pattern and memonize it, so we don't have to loop through it again.

### Step 1: Identify the Variables and Visualize Examples

The only changing variable in this problem is the length of the string we will genenrate. Let's first visualize it, again using example of `n=3`. Instead of drawing graphs, we will try to use a table here. The table we are using here tracks the number of strings that ends with a certain vowel at each string length `n`. 

| Length | a | e | i | o | u | Total | Note |
| - | - | - | - | - | - | - | - |
| `n = 0` | 0 | 0 | 0 | 0 | 0 | **0** | No string |
| `n = 1` | 1 | 1 | 1 | 1 | 1 | **5** | 1 for each vowel |
| `n = 2` | 1 | 2 | 3 | 4 | 5 | **15** | `a` can only go after itself, so there is only 1 `aa`; `e` can go after `a` and `e`, so there are `ae`, `ee`; `i` can go after `a`, `e`, `i`, so there are `ai`, `ei`, `ii`; ... |
| `n = 3` | 1 | 3 | 6 | 10 | 15 | **35** | `a` can only go after itself, so there is only 1 `aaa`; `e` can go after `a` and `e`, so there are `aae`, `aee`, `eee`; `i` can go after `a`, `e`, `i`, so there are `aai`, `aei`, `aii`, `eei`, `eii`, `iii`; ... |

### Step 2: Find an Appropriate Sub-problem

We've pretty much already finished this in the last step. Very intuitively, the sub-problem is just to find the number of possible strings of length `i` where `i < n`. The solutions for `n = 0, 1, 2, 3` are just respective `0`, `5`, `15`, `35`. 

### Step 3: Find Relationships among Sub-problems

In the example above, what's the relationship between `n = 1` and `n = 2`? Well, at Step 0 we generalized the rules of adding new vowel characters after existing strings. Here, we will just need to expand on it to see how the solution of preceeding sub-problem determines the solution of the following sub-problem. This is going to get a bit repetitive so bear with me.

* We can only put `a` after `a`, and there is only 1 `a` generated from `n = 1`, so there is only 1 possible location for `a` for `n = 2`;
* We can only put `e` after `a` and `e`, and there are 1 `a` and 1 `e` generated from `n = 1`, so there are 1 + 1 = 2 possible locations for `e` for `n = 2`;
* We can only put `i` after `a`, `e`, `i`, and there are 1 `a`, 1 `e`, 1 `i` generated from `n = 1`, so there are 1 + 1 + 1 = 3 possible locations for `i` for `n = 2`;
* We can only put `o` after `a`, `e`, `i`, `o`, and there are 1 `a`, 1 `e`, 1 `i`, 1 `o` generated from `n = 1`, so there are 1 + 1 + 1 + 1 = 4 possible locations for `o` for `n = 2`;
* We can put `u` after all 5 vowels, and there are 1 `a`, 1 `e`, 1 `i`, 1 `o`, 1 `u` generated from `n = 1`, so there are 1 + 1 + 1 + 1 + 1 = 5 possible locations for `u` for `n = 2`.

What about the relationship between `n = 2` and `n = 3`? It's the same logic.

* There is 1 `a` generated from `n = 2`, so there is only 1 possible location for `a`;
* There are 1 `a` and 2 `e` generated from `n = 2`, so there are 1 + 2 = 3 possible locations for `e`;
* There are 1 `a`, 2 `e`, 3 `i` generated from `n = 2`, so there are 1 + 2 + 3 = 6 possible locations for `i`;
* There are 1 `a`, 2 `e`, 3 `i`, 4 `o` generated from `n = 2`, so there are 1 + 2 + 3 + 4 = 10 possible locations for `o`;
* There are 1 `a`, 2 `e`, 3 `i`, 4 `o`, 5 `u` generated from `n = 2`, so there are 1 + 2 + 3 + 4 + 5 = 15 possible locations for `u`.

### Step 4: Generalize the Relationship

The relationship should be very clear to you by now. Imagine the table we created was just a `n x 5` 2d list structure `dp`. We can generalize the relationship as below:

| Length | a | e | i | o | u |
| - | - | - | - | - | - |
| `n - 1` | `dp[n-1][0]` | `dp[n-1][1]` | `dp[n-1][2]` | `dp[n-1][3]` | `dp[n-1][4]` |
| `n` | `dp[n-1][:1]` | `dp[n-1][0][:2]` | `dp[n-1][0][:3]` | `dp[n-1][0][:4]` | `dp[n-1][0][:5]` |

Therefore, `dp[n] = [ dp[n-1][:j] for j in range(1, 6) ]`. The solution we need is the count, so in the end we can just sum up the 5 numbers in `dp[n]` to get the final answer.

### Step 5: Implement by Solving Sub-problems in Order

Note that our relationship doesn't hold between `dp[0]` and `dp[1]`. Therefore, we will hard code those base cases. After that, we would just need to iterate from `2` to `n` and apply this relationship we identified. The time complexity here is `O(5*n)` or just `O(n)`, same for space complexity. 

```python
def countVowelStrings(n: int) -> int:

    dp = [[0, 0, 0, 0, 0], [1, 1, 1, 1, 1]]

    for i in range(2, n+1):
        newDp = [ sum(dp[i-1][:j]) for j in range(1, 6) ]
        dp.append(newDp)

    return sum(dp[-1])
```

### Step 6: Optimize Space Complexity of Memoization

DP is the classic example where we sacrafice space to optimize space complexity. However, we should always try to see if we can optimize the space complexity as well. This may be optional for an interview because of shortage of time. But it's definitely a plus if you can nail it.

Note that for this problem, `dp[n]` is solely dependent on `dp[n-1]`. This is different from the previous robot problem we saw, where `dp[i][j]` depend on `dp[i-1][j]` and `dp[i][j-1]` - this is hard to keep track of when we iterate to a new row.

Therefore, instead of memoizing the entire solution set, we just need to memoize the solution to the previous subproblem. Therefore we achieved constant extra space complexity of `O(1)`!

```python
def countVowelStrings(n: int) -> int:

    dp, newDp = [1, 1, 1, 1, 1], [1, 1, 1, 1, 1]

    for i in range(2, n + 1):
        newDp = [ sum(dp[:j]) for j in range(1, 6) ]
        dp = newDp[:] # need to create a hard copy here, otherwise the two variables point to the same list and mess up our calculations above

    return sum(newDp)
```
