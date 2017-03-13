
# Binary Search Trees
In this chapter, we use a search-tree structure to efficiently implement a sorted map. The three most fundamental methods of of a map are:

| Method     | Description                                                                                                                         |
|------------|-------------------------------------------------------------------------------------------------------------------------------------|
| get(k):    | Returns the value v associated with key k, if such an entry exists; otherwise returns null.                                         |
| put(k, v): | Associates value v with key k, replacing and returning any existing value if the map already contains an entry with key equal to k. |
| remove(k): | Removes the entry with key equal to k, if one exists, and returns its value; otherwise returns null.                                |



Binary trees are an excellent data structure for storing entries of a map, assuming we have an order relation defined on the keys. In this chapter, we define a binary search tree as a proper binary tree such that each internal position p stores a key-value pair (k,v) such that:

* Keys stored in the left subtree of p are less than k.
* Keys stored in the right subtree of p are greater than k.


## Searching Within a Binary Search Tree
The most important consequence of the structural property of a binary search tree is its namesake search algorithm. We can attempt to locate a particular key in a binary search tree by viewing it as a decision tree. In this case, the question asked at each internal position p is whether the desired key k is less than, equal to, or greater than the key stored at position p, which we denote as key(p). If the answer is "less than," then the search continues in the left subtree. If the answer is "equal," then the search terminates successfully. If the answer is "greater than," then the search continues in the right subtree. Finally, if we reach a leaf, then the search terminates unsuccessfully.

```JAVA
Algorithm TreeSearch(p, k):
	if p is external then
		return p //unsuccessful search
	else if k == key(p) then
		return p //successful search
	else if k < key(p) then
		return TreeSearch(left(p), k)
	else //we know that k > key(p)
		return TreeSearch(right(p), k)
```

Since we spend O(1) time per position encountered in the search, the overall search runs in O(h) time, where h is the height of the binary search tree T. Admittedly, the height h of T can be as large as the number of entries, n, but we expect that it is usually much smaller. Later in this chapter we will show various strategies to maintain an upper bound of O(log n) on the height of a search tree T.

## Insertion

The map operation put(k, v) begins with a search for an entry with key k. If found, that entry's existing value is reassigned. Otherwise, the new entry can be inserted into the underlying tree by expanding the leaf that was reached at the end of the failed search into an internal node. The binary search-tree property is sustained by that placement (note that it is placed exactly where a search would expect it).

```JAVA
Algorithm TreeInsert(k, v):
	Input: A search key k to be associated with value v
	p = TreeSearch(root( ), k)
	if k == key(p) then
		Change p's value to (v)
	else
		expandExternal(p, (k, v))
```
## Deletion
To delete an entry with key k, we begin by calling TreeSearch(root(), k) to find the position p storing an entry with key equal to k (if any). If the search returns an external node, then there is no entry to remove. Otherwise, we distinguish between two cases (of increasing difficulty):

* Node to be deleted is leaf: Simply remove from the tree. 
* Node to be deleted has only one child: Copy the child to the node and delete the child
*  Node to be deleted has two children: Find inorder successor of the node. Copy contents of the inorder successor to the node and delete the inorder successor. Note that inorder predecessor can also be used.

As with searching and insertion, this algorithm for a deletion involves the traversal of a single path downward from the root, possibly moving an entry between two positions of this path, and removing a node from that path and promoting its child. Therefore, it executes in time O(h) where h is the height of the tree.

## Performance of a Binary Search Tree

Almost all operations have a worst-case running time that depends on h, where h is the height of the current tree. This is because most operations rely on traversing a path from the root of the tree, and the maximum path length within a tree is proportional to the height of the tree. Most notably, our implementations of map operations get, put, and remove, and most of the sorted map operations, each begins with a call to the treeSearch utility.  

A binary search tree T is therefore an efficient implementation of a map with n entries only if its height is small. In the best case, T has height h = ⌈log(n+ 1)⌉− 1, which yields logarithmic-time performance for most of the map operations. In the worst case, however, T has height n, in which case it would look and feel like an ordered list implementation of a map. Such a worst-case configuration arises, for example, if we insert entries with keys in increasing or decreasing order.


