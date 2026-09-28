---
title: "Rotate Image"
aliases: []
tags: [leetcode, array, matrix, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 48
leetcode_difficulty: Medium
pattern: "In-place Matrix Transformation"
status: not-started
---

# Rotate Image

## Problem Statement
You are given an `n x n` 2D matrix representing an image. Rotate the image by 90 degrees **clockwise**, **in place** — you are not allowed to allocate another 2D matrix.

Constraints: `n == matrix.length == matrix[i].length`, `1 <= n <= 20`, `-1000 <= matrix[i][j] <= 1000`.

## Examples
```
Input:
[[1,2,3],
 [4,5,6],
 [7,8,9]]
Output:
[[7,4,1],
 [8,5,2],
 [9,6,3]]

Input:
[[5,1,9,11],
 [2,4,8,10],
 [13,3,6,7],
 [15,14,12,16]]
Output:
[[15,13,2,5],
 [14,3,4,1],
 [12,6,8,9],
 [16,7,10,11]]
```

## Intuition & Approach
The naive approach allocates a new `n x n` matrix, and for every cell `(r, c)` in the original, computes its destination `(c, n-1-r)` in the rotated result, then copies the whole thing back. That's O(n^2) time and O(n^2) extra space — it works, but the problem explicitly forbids the extra matrix, forcing an in-place approach.

The trick is to decompose a 90-degree clockwise rotation into two simpler, well-known in-place operations whose composition equals the rotation:

1. **Transpose** the matrix (swap `matrix[i][j]` with `matrix[j][i]` for all `i < j`). This flips the matrix across its main diagonal.
2. **Reverse each row.** Reversing every row after a transpose produces exactly the 90-degree clockwise rotation.

Why does transpose + row-reverse equal clockwise rotation? A clockwise rotation sends `matrix[r][c]` to `result[c][n-1-r]`. A transpose sends `matrix[r][c]` to `temp[c][r]`. Reversing each row of `temp` sends `temp[c][r]` to `result[c][n-1-r]` — matching exactly. Both transpose and row-reversal can be done in-place with O(1) extra space, so their composition is too.

An alternative, equally valid in-place approach is **four-way cell swapping by layer**: for each concentric square "ring" of the matrix, cycle groups of four cells (top → right → bottom → left → top) using one temp variable. This avoids a full transpose pass but is more fiddly to get right; transpose+reverse is easier to reason about and remember.

## Pattern Recognition
> [!tip] Pattern: In-place Matrix Transformation
> Whenever a problem demands you transform a matrix "in place" with no extra matrix, look for a way to decompose the target transformation (rotation, reflection) into a sequence of simpler in-place primitives like transpose and reverse-row/reverse-column.

## Algorithm Steps
1. Transpose the matrix in place: for `i` in `0..n-1`, for `j` in `i+1..n-1`, swap `matrix[i][j]` and `matrix[j][i]`.
2. Reverse every row in place: for each row, swap elements symmetric around the row's center.
3. Return (the matrix is rotated in place).

## Solution (Python)
```python
from typing import List

class Solution:
    def rotate(self, matrix: List[List[int]]) -> None:
        n = len(matrix)

        # Step 1: transpose in place (only upper triangle needs swapping).
        for i in range(n):
            for j in range(i + 1, n):
                matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]

        # Step 2: reverse each row in place.
        for row in matrix:
            row.reverse()
```

## Solution (Java)
```java
class Solution {
    public void rotate(int[][] matrix) {
        int n = matrix.length;

        // Step 1: transpose in place (only upper triangle needs swapping).
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                int temp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = temp;
            }
        }

        // Step 2: reverse each row in place.
        for (int[] row : matrix) {
            int left = 0, right = row.length - 1;
            while (left < right) {
                int temp = row[left];
                row[left] = row[right];
                row[right] = temp;
                left++;
                right--;
            }
        }
    }
}
```

## Complexity Analysis
- **Time:** O(n^2) — transpose visits roughly half the cells, and reversing every row visits all cells once; both are linear in the number of cells.
- **Space:** O(1) — all swaps happen within the given matrix, no auxiliary matrix is allocated.

## Edge Cases
- `n == 1`: a single-cell matrix; transpose and row-reverse are both no-ops, correctly leaving it unchanged.
- `n == 2`: smallest case where the rotation is visually verifiable by hand — good sanity check while debugging.
- Negative numbers or duplicate values in the matrix — the algorithm is value-agnostic, it only moves positions, so this should not matter, but is worth testing.

## Common Mistakes
- Transposing the *entire* matrix including both `(i,j)` and `(j,i)` for every pair, which swaps each pair back to where it started (undoing the transpose) — only iterate `j > i` to avoid double-swapping.
- Reversing columns instead of rows, or transposing across the anti-diagonal instead of the main diagonal, which produces a counter-clockwise rotation or a reflection instead of the required clockwise rotation.

## Related Problems
- [[Minimum-Area-Rectangle]]
- [[Merge-Sorted-Array]]
- [[Construct-Quad-Tree]]

## Follow-up / Variants
A common follow-up: rotate the image 90 degrees **counter-clockwise** in place instead — this is transpose followed by reversing each *column* (or equivalently, reverse rows first, then transpose). Another variant asks for rotation by an arbitrary multiple of 90 degrees, which reduces to `(k % 4)` applications of this same routine.
