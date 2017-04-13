
# Merge Sort
## Divide-and-Conquer

The first two algorithms we describe in this chapter, merge-sort and quick-sort, use recursion in an algorithmic design pattern called divide-and-conquer. The divide-and-conquer pattern consists of the following three steps:
* Divide: If the input size is smaller than a certain threshold (say, one or two elements), solve the problem directly using a straightforward method and return the solution so obtained. Otherwise, divide the input data into two or more disjoint subsets.
* Conquer: Recursively solve the subproblems associated with the subsets.
* Combine: Take the solutions to the subproblems and merge them into a solution to the original problem.

To sort a sequence S with n elements using the three divide-and-conquer steps, the merge-sort algorithm proceeds as follows:
* Divide: If S has zero or one element, return S immediately; it is already sorted. Otherwise (S has at least two elements), remove all the elements from S and put them into two sequences, S<sub>1</sub> and S<sub>2</sub>, each containing about half of the elements of S; that is, S<sub>1</sub> contains the first ⌊n/2⌋ elements of S, and S<sub>2</sub> contains the remaining ⌈n/2⌉ elements.
* Conquer: Recursively sort sequences S<sub>1</sub> and S<sub>2</sub>.
* Combine: Put the elements back into S by merging the sorted sequences S<sub>1</sub> and S<sub>2</sub> into a sorted sequence.

The merge-sort tree associated with an execution of merge-sort on a sequence of size n has height ⌈log n⌉.

![Merge-sort-example-300px](https://upload.wikimedia.org/wikipedia/commons/c/cc/Merge-sort-example-300px.gif) 

https://upload.wikimedia.org/wikipedia/commons/c/cc/Merge-sort-example-300px.gif
