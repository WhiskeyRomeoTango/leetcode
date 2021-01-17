# Backtracking

Backtracking by itself is a general algorithm best used for constraint satisfaction problems. The tell-tale signs of a problem best suited for backtracking is **finding all or some solutions that meet certain criteria / constraint**. Specifically, we must be able to test partial solutions against the criteria. Backtracking builds partial solutions **recursively** in a depth-search fashion, and checks them against the criteria. If the partial solution meets the constraint, it expands upon the solution and generates new paths. Otherwise, it **backtracks** to the previously valid path and explore other options. Examples of such problems include:

* Puzzles like Soduku, Crosswords, etc - these usually only have 1 unique solution that needs to be identified.
* Combinations/permutations of certain elements (numbers, characters, ...) that meets certain criteria - these usually ask us for all possible solutions available.

## How Backtracking Works

A backtracking algorithm consists of 3 parts:

* A recursive function that takes in a solution (which could be partial / incomplete)
* A mutation or addition operation performed on the solution
* Validity check for whether the solution is complete and/or valid

Using these 3 parts, we can generalize backtracking with this pseudocode:

```
backtrack(solution):
    if solution is not valid:
        return
    if solution is valid:
        output(solution)
    for p in all permutations(solution):
        backtrack(p)
```

For `output()`, it's usually just pushing the solution to an array, if we are trying to find all the solutions.

## Backtracking and Depth First Search (DFS)

I've seen on LeetCode that some people mix DFS and Backtracking. I do feel that visualize backtracking as a DFS search in tree is very helpful for understanding backtracking. But there is a subtle difference between the two we should understand. In theory, DFS is used only for tree or graph data structures, while backtracking works for all types of data structures. In practice, backtracking creates a search tree via the recursions and permutations, and performs DFS with pruning via the validity and completion checks. 

Consider a simple problem where we need to generate strings using characters `['a', 'b']`. Although there is no explicit tree/graph structure here, we can abstract it:

```
     root
    /    \
   a      b
  / \    / \
 a   b  a   b
     ....
```

For backtracking, there should always be a completion constraint we should meet, otherwise this tree will just be infinitely deep. For example, the length of the string must be shorter than or eqaul to 5 characters. Then, we will essentially depth first search this tree until we are at depth 5, and backtrack. By doing so we effectively pruned out any branch that's over 5 nodes deep. There should also be validity checks to prune the branches further, which may or may not be the same as the completion check. For example, we can have an additional validity check of whether a and b are alternating (i.e. no consecutive a's or b's).

For a vanilla DFS on a tree though, since the data structure is finite, we will eventually reach a null node that represents the end of a branch, that's the only case when we backtrack. Obviously we can also add pruning to it based on certain condition / constraint, and we effectively turned it into backtracking on a tree structure.

Vanilla DFS is best used for checking whether a certain value is in the structure. Backtracking is best used for finding all or some solutions along the searching process based on certain constraints.

## Example - 39: Combination Sum

### Problem Description

Given an array of distinct integers candidates and a target integer target, return a list of all unique combinations of candidates where the chosen numbers sum to target. You may return the combinations in any order.

The same number may be chosen from candidates an unlimited number of times. Two combinations are unique if the frequency of at least one of the chosen numbers is different.

It is guaranteed that the number of unique combinations that sum up to target is less than 150 combinations for the given input.

Example 1:

```
Input: candidates = [2,3,6,7], target = 7
Output: [[2,2,3],[7]]
Explanation:
2 and 3 are candidates, and 2 + 2 + 3 = 7. Note that 2 can be used multiple times.
7 is a candidate, and 7 = 7.
These are the only two combinations.
```

Example 2:

```
Input: candidates = [2,3,5], target = 8
Output: [[2,2,2,2],[2,3,3],[3,5]]
```

Example 3:

```
Input: candidates = [2], target = 1
Output: []
```

Example 4:

```
Input: candidates = [1], target = 1
Output: [[1]]
```

Example 5:

```
Input: candidates = [1], target = 2
Output: [[1,1]]
```

Constraints:

* `1 <= candidates.length <= 30`
* `1 <= candidates[i] <= 200`
* `All elements of candidates are distinct`
* `1 <= target <= 500`

### Solution

Let's use the tree/DFS anology we saw before. Each node in this tree will be a combination of numbers from the list `candidates`, or a possible solution. Each node will also have `len(candidates)` of child nodes, representing the number of additions we can do to get new possible solutions. We will prune out branches when the sum of the combination exceed `target`, while storing the ones that sum up to exactly `target` - that's both our completion and validity check. 

```python
def combinationSum(candidates: [int], target: int) -> [int]:

    results = []

    def backtrack(solution: [int], target: int) -> None:
        # the solution is sorted to avoid duplicates
        solution.sort()
        currSum = sum(solution)
        if currSum > target:
            return
        elif currSum == target:
            # only push when solution is not a duplicate
            if solution not in results:
                results.append(solution)
        else:
            for num in candidates:
                backtrack(solution + [num], target)

    backtrack([], target)
    return results
```

It passed all tests on LeetCode and was accepted, but runtime was 750ms. So I thought we could perhaps optimize it a bit, because it's very vanilla backtracking.

## Example - 37: Suduko Solver

Write a program to solve a Sudoku puzzle by filling the empty cells.

A sudoku solution must satisfy all of the following rules:

* Each of the digits 1-9 must occur exactly once in each row.
* Each of the digits 1-9 must occur exactly once in each column.
* Each of the digits 1-9 must occur exactly once in each of the 9 3x3 sub-boxes of the grid.
* The '.' character indicates empty cells.

Example 1:

```
Input: board = [["5","3",".",".","7",".",".",".","."],["6",".",".","1","9","5",".",".","."],[".","9","8",".",".",".",".","6","."],["8",".",".",".","6",".",".",".","3"],["4",".",".","8",".","3",".",".","1"],["7",".",".",".","2",".",".",".","6"],[".","6",".",".",".",".","2","8","."],[".",".",".","4","1","9",".",".","5"],[".",".",".",".","8",".",".","7","9"]]
Output: [["5","3","4","6","7","8","9","1","2"],["6","7","2","1","9","5","3","4","8"],["1","9","8","3","4","2","5","6","7"],["8","5","9","7","6","1","4","2","3"],["4","2","6","8","5","3","7","9","1"],["7","1","3","9","2","4","8","5","6"],["9","6","1","5","3","7","2","8","4"],["2","8","7","4","1","9","6","3","5"],["3","4","5","2","8","6","1","7","9"]]
Explanation: The input board is shown above and the only valid solution is shown below:
```

Constraints:

* `board.length == 9`
* `board[i].length == 9`
* `board[i][j] is a digit or '.'`
* It is guaranteed that the input board has only one solution.

