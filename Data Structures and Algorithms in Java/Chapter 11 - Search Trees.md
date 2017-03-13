
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

We can nevertheless take comfort that, on average, a binary search tree with n keys generated from a random series of insertions and removals of keys has expected height O(log n).  

# Balanced Search Trees

We will explore four search-tree algorithms that provide stronger performance guarantees than binary search tree in the worst-case. Three of the four data structures (AVL trees, splay trees, and red-black trees) are based on augmenting a standard binary search tree with occasional operations to reshape the tree and reduce its height.  

The primary operation to rebalance a binary search tree is known as a rotation. During a rotation, we “rotate” a child to be above its parent.  

![11.8](https://github.com/opwid/Library/blob/master/Data%20Structures%20and%20Algorithms%20in%20Java/Images/11.8.png)  

To maintain the binary search-tree property through a rotation, we note that if position x was a left child of position y prior to a rotation (and therefore the key of x is less than the key of y), then y becomes the right child of x after the rotation, and vice versa. Furthermore, we must relink the subtree of entries with keys that lie between the keys of the two positions that are being rotated. For example, in Figure 11.8 the subtree labeled T2 represents entries with keys that are known to be greater than that of position x and less than that of position y. In the first configuration of that figure, T2 is the right subtree of position x; in the second configuration, it is the left subtree of position y. If used wisely, this operation can be performed to avoid highly unbalanced tree configurations.  

One or more rotations can be combined to provide broader rebalancing within a tree. One such compound operation we consider is a __trinode restructuring__. For this manipulation, we consider a position x, its parent y, and its grandparent z. The goal is to restructure the subtree rooted at z in order to reduce the overall path length to x and its subtrees.  

```
restructure(x):
Input: A position x of a binary search tree T that has both a parent y and a grandparent z
Output: Tree T after a trinode restructuring (which corresponds to a single or double rotation) involving positions x, y, and z
1- Let (a, b, c) be a left-to-right (inorder) listing of the positions x, y, and z, and let (T1, T2, T3, T4) be a left-to-right (inorder) listing of the four subtrees of x, y, and z not rooted at x, y, or z.
2- Replace the subtree rooted at z with a new subtree rooted at b.
3- Let a be the left child of b and let T 1 and T 2 be the left and right subtrees of a, respectively.
4- Let c be the right child of b and let T 3 and T 4 be the left and right subtrees of c, respectively.
```

In practice, the modification of a tree T caused by a trinode restructuring operation can be implemented through case analysis either as a single rotation or as a double rotation. In any of the cases, the trinode restructuring is completed with O(1) running time.

# AVL Trees
In this section, we describe a simple balancing strategy that guarantees worst-case logarithmic running time for all the fundamental map operations.  

__Height-Balance Property__: For every internal position p of T, the heights of the children of p differ by at most 1.  

Any binary search tree T that satisfies the height-balance property is said to be an __AVL tree__.  


![11.10](https://github.com/opwid/Library/blob/master/Data%20Structures%20and%20Algorithms%20in%20Java/Images/11.10.png)  

An immediate consequence of the height-balance property is that a subtree of an AVL tree is itself an AVL tree. The height-balance property also has the important consequence of keeping the height small, as shown in the following proposition.  
__Proposition 11.1__: The height of an AVL tree storing n entries is O(log n).  

By Proposition 11.1 and the analysis of binary search trees given in Section 11.1, the operation get, in a map implemented with an AVL tree, runs in time O(log n), where n is the number of entries in the map. Of course, we still have to show how to maintain the height-balance property after an insertion or deletion.

## Insertion

Given a binary search tree T, we say that a position is balanced if the absolute value of the difference between the heights of its children is at most 1, and we say that it is unbalanced otherwise.

An insertion of a new entry in a binary search tree results in a leaf position p being expanded to become internal, with two new external children. This action may violate the height-balance property (see, for example, Figure 11.11a), yet the only positions that may become unbalanced are ancestors of p, because those are the only positions whose subtrees have changed. Therefore, let us describe how to restructure T to fix any unbalance that may have occurred.  

![11.11](https://github.com/opwid/Library/blob/master/Data%20Structures%20and%20Algorithms%20in%20Java/Images/11.11.png) 

In particular, let z be the first position we encounter in going up from p toward the root of T such that z is unbalanced. Also, let y denote the child of z with greater height (and note that y must be an ancestor of p). Finally, let x be the child of y with greater height (there cannot be a tie and position x must also be an ancestor of p, possibly p itself ). We rebalance the subtree rooted at z by calling the trinode restructuring method, restructure(x).

## Deletion

Deletion may violate the height-balance property in an AVL tree. In particular, if position p represents a (possibly external) child of the removed node in tree T, there may be an unbalanced node on the path from p to the root of T.  

![11.13](https://github.com/opwid/Library/blob/master/Data%20Structures%20and%20Algorithms%20in%20Java/Images/11.13.png)  

As with insertion, we use trinode restructuring to restore balance in the tree T. In particular, let z be the first unbalanced position encountered going up from p toward the root of T, and let y be that child of z with greater height (y will not be an ancestor of p). Furthermore, let x be the child of y defined as follows: if one of the children of y is taller than the other, let x be the taller child of y; else (both children of y have the same height), let x be the child of y on the same side as y (that is, if y is the left child of z, let x be the left child of y, else let x be the right child of y). We then perform a restructure(x) operation.  

The restructured subtree is rooted at the middle position denoted as b in the description of the trinode restructuring operation. The height-balance property is guaranteed to be locally restored within the subtree of b. Unfortunately, this trinode restructuring may reduce the height of the subtree rooted at b by 1, which may cause an ancestor of b to become unbalanced. So, after rebalancing z, we continue walking up T looking for unbalanced positions. If we find another, we perform a restructure operation to restore its balance, and continue marching up T looking for more, all the way to the root.

## Performance of AVL Trees

By Proposition 11.1, the height of an AVL tree with n entries is guaranteed to be O(log n). Because the standard binary search-tree operation had running times bounded by the height, and because the additional work in maintaining balance factors and restructuring an AVL tree can be bounded by the length of a path in the tree, the traditional map operations run in worst-case logarithmic time with an AVL tree.

# (2,4) Trees

It is a particular example of a more general structure known as a multiway search tree, in which internal nodes may have more than two children. Linked data structure representation can be used for a multiway search tree. When using a general tree to implement a multiway search tree, we must store at each node one or more key-value pairs associated with that node. That is, we need to store with w a reference to some collection that stores the entries for w.  

During a search for key k in a multiway search tree, the primary operation needed when navigating a node is finding the smallest key at that node that is greater than or equal to k. For this reason, it is natural to model the information at a node itself as a sorted map.  

The primary efficiency goal for a multiway search tree is to keep the height as small as possible.  

One form of a multiway search tree that keeps the tree balanced while using small secondary data structures at each node is the (2, 4) tree, also known as a 2-4 tree or 2-3-4 tree. This data structure achieves these goals by maintaining two simple properties:
* Size Property: Every internal node has at most four children.
* Depth Property: All the external nodes have the same depth.

## (2,4)-Tree Operations






















































