# Range Sum

**Range Sum** is a flavor of DP problem that's frequently asked in interviews. The problem asks you to implement a function / algorithm to find the sum of numbers in a box within a matrix. For example, if we are given a matrix below, and a `3 x 3` box within the matrix (confined by top-left coordinate of `(2, 1)` and bottom-right coordinate of `(4, 3)`). Our function should return the sum of these 9 numbers, `2 + 0 + 1 + 1 + 0 + 1 + 0 + 3 + 0 = 8`.

![Range Sum Example 1](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/range-sum/_assets/example_range_1.jpg)

Now the problem itself isn't very spicy. If we brute force it, we will just need to loop `i*j` times and add each number to the sum. Not too bad. 

However, this is a very common problem in applications of computer vision or computer graphics. In those applications, the matrix is an image, where each coordinate `(i, j)` represents a pixel. The pixel itself then holds `(r, g, b)` values that describe the color of the pixel. We typically need to perform range sum repeatedly, often iterating through the entire image. Then, to complete this series of tasks our runtime is `(i * j) ^ 2`. Let's say a autonomous driving car needs to process the image it sees in front of it with a 1080p feed, that's `(1920 * 1080) ^ 2 = 4,299,816,960,000` operations. That sounds very, very bad.

## Prefix Sum

Let's first consider a special case of range sum where our box always starts from the top-left corner of the matrix at coordinate `(0, 0)`, and this type of range sum is called **prefix sum**. It can be generalized as this - given a matrix `matrix`, return a new matrix `matrixSum` where `matrixSum[i][j]` is the sum of numbers within the box in `matrix` confined by `(0, 0)` and `(i, j)`. 

Let's work out an example when we are at `(2, 2)`.

![Prefix Sum Example 1](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/range-sum/_assets/example_1.jpg)

Using DP to solve the problem requires us to find sub-problems. So what are my sub-problems at `matrixSum[2][2]`?

Well, a sub-problem should be a smaller version of the parent problem. In this case, my sub-problems are getting results of `matrixSum[r][c]` where `r < 2` or `c < 2`. Below are some sub-problems that will help us.

* `matrixSum[1][2]` - the yellow-shaded area

![Prefix Sum Example 2](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/range-sum/_assets/example_2.jpg)

* `matrixSum[2][1]` - the blue-shaded area

![Prefix Sum Example 3](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/range-sum/_assets/example_3.jpg)

* `matrixSum[1][1]`, the green-shaded area, which is just the overlapping box of the previous 2 areas.

![Prefix Sum Example 4](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/range-sum/_assets/example_4.jpg)

A simple relationship can be established here. 

* First, add the sum of the first two boxes together, which is `3 + 0 + 1 + 5 + 6 + 3 = 18` and `3 + 0 + 5 + 6 + 1 + 2 = 17`. Together they sum up to `35`. 
* Then, subtract it by the overlapping box, because we added each number `3 + 0 + 5 + 6 = 14` in this overlapping box twice. This gets us to `21`.
* Finally, we just need to add `matrix[2][2] = 0` which is the single number sitting at coordinate `(2, 2)`. Together they will give us the sum of the 9 numbers confined by this box, which is `3 + 0 + 1 + 5 + 6 + 3 + 1 + 2 + 0 = 21`.

The relationship can be generalized as:

```
matrixSum[i][j] = matrixSum[i-1][j] + matrixSum[i][j-1] - matrixSum[i-1][j-1] + matrix[i][j]
```

Therefore, to complete `matrixSum`, we just need to loop through the matrix, and apply this relationship at each coordinate `(i, j)`. But notice that when `i == 0` or `j == 0`, we will get out of the bounds of the matrix. A simple trick is simply turn `matrixSum` to 1-based index instead of 0-based index. So `(i+1, j+1)` in `matrixSum` represents the sum of the box confined by `(i, j)` in `matrix`. The first row and first column of `matrixSum` are just going to be dummy 0's, and it will take care of these edge cases beautifully. Our relationship is also updated slightly to:

```
matrixSum[i+1][j+1] = matrixSum[i][j+1] + matrixSum[i+1][j] - matrixSum[i][j] + matrix[i][j]
```

Our final solution to prefix sum would look like this.

```python
def prefixSum(matrix):
    matrixSum = [ [0 for j in range(len(matrix[0]) + 1) ] for i in range(len(matrix[0]) + 1) ]
    for i in range(len(matrix)):
        for j in range(len(matrix[i])):
            matrixSum[i+1][j+1] = matrixSum[i][j+1] + matrixSum[i+1][j] - matrixSum[i][j] + matrix[i][j]
    return matrixSum
```
```
> matrix = [[3, 0, 1, 4, 2], [5, 6, 3, 2, 1], [1, 2, 0, 1, 5], [4, 1, 0, 1, 7], [1, 0, 3, 0, 5]]
> prefixSum(matrix)
[[0, 0, 0, 0, 0, 0], [0, 3, 3, 4, 8, 10], [0, 8, 14, 18, 24, 27], [0, 9, 17, 21, 28, 36], [0, 13, 22, 26, 34, 49], [0, 14, 23, 30, 38, 58]]
```

Now, with the prefix sum algorithm in hand, we can try some spicier stuff.

## Example - 304. Range Sum Query 2D - Immutable

### Problem Description

Given a 2D matrix matrix, find the sum of the elements inside the rectangle defined by its upper left corner (row1, col1) and lower right corner (row2, col2).

![Range Sum Example 1](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/range-sum/_assets/example_range_1.jpg)

The above rectangle (with the red border) is defined by (row1, col1) = (2, 1) and (row2, col2) = (4, 3), which contains sum = 8.

**Example:**

```
Given matrix = [
  [3, 0, 1, 4, 2],
  [5, 6, 3, 2, 1],
  [1, 2, 0, 1, 5],
  [4, 1, 0, 1, 7],
  [1, 0, 3, 0, 5]
]

sumRegion(2, 1, 4, 3) -> 8
sumRegion(1, 1, 2, 2) -> 11
sumRegion(1, 2, 2, 4) -> 12
```

Note:
* You may assume that the matrix does not change.
* There are many calls to sumRegion function.
* You may assume that row1 ≤ row2 and col1 ≤ col2.

### Intuition

Note that we are using the same matrix as before. Previously in prefix sum, we anchored our top-left coordinate at `(0, 0)` for the box. In this problem, however, we need to deal with flexible top-left coordinate `(row1, col1)`. Can we use the memoized solutions from prefix sum to solve this generic range sum problem?

Here is the idea. To get the sum in the box with red border, we just need to get the sum from the box with purple border (noticed that it's a sub-problem from the prefix sum?), and subtract it by the sum of the gray-shaded area.

![Range Sum Example 2](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/range-sum/_assets/example_range_2.jpg)

Let's say we already calculated the prefix sum matrix `prefixSum` for the given matrix. The purple box is very straightforward - `prefixSum[5][4] = 38` (remember that we left an empty row on top and an empty column on left, so `prefixSum` is actually 1-index based). What about the gray-shaded area? Well, it turns out we can use the same logic from prefix sum to get that area.  We can break this example down by: 

* `prefixSum[2][4] = 24` - the yellow-shaded box

![Range Sum Example 3](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/range-sum/_assets/example_range_3.jpg)

* `prefixSum[5][1] = 14` - the yellow-shaded box

![Range Sum Example 4](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/range-sum/_assets/example_range_4.jpg)

* `prefixSum[2][1] = 8` - the green-shaded box, which is just the overlapping box of the previous 2 areas.

![Range Sum Example 5](https://github.com/WhiskeyRomeoTango/leetcode/blob/main/dynamic-programming/range-sum/_assets/example_range_5.jpg)

Then, the sum of the red box is just `prefixSum[5][4] - prefixSum[2][4] - prefixSum[5][1] + prefixSum[2][1] = 8`.

To generalize the relationship:

```
rangeSum(row1, col1, row2, col2) = prefixSum[row2+1][col2+1] - prefixSum[row1][col2+1] - prefixSum[row2+1][col1] + prefixSum[row1][col1]
```

### Solution

```python
class NumMatrix:

    def __init__(self, matrix: List[List[int]]):
        self.matrix = matrix
        if not self.matrix:
            self.dp = [[0]]
            return
        self.dp = [ [0 for j in range(len(matrix[0]) + 1)] for i in range(len(matrix) + 1) ] 
        for i in range(len(matrix)):
            for j in range(len(matrix[i])):
                self.dp[i+1][j+1] = self.dp[i][j+1] + self.dp[i+1][j] - self.dp[i][j] + self.matrix[i][j]
        

    def sumRegion(self, row1: int, col1: int, row2: int, col2: int) -> int:
        return self.dp[row2+1][col2+1] - self.dp[row1][col2+1] - self.dp[row2+1][col1] + self.dp[row1][col1]
```
