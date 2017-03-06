# General Trees

We define a tree T as a set of nodes storing elements such that the nodes have a parent-child relationship that satisfies the following properties:  
* If T is nonempty, it has a special node, called the root of T , that has no parent.
* Each node v of T different from the root has a unique parent node w; every node with parent w is a child of w.

Two nodes that are children of the same parent are __siblings__. A node v is __external__ if v has no children. A node v is __internal__ if it has one or more children. External nodes are also known as __leaves__. A node u is an __ancestor__ of a node v if u = v or u is an ancestor of the parent of v.  

We define a tree ADT using the concept of a __position__ as an abstraction for a node of a tree. An element is stored at each position, and positions satisfy parent-child relationships that define the tree structure. A position object for a tree supports the method:


| Method          | Description                                                                            |
|-----------------|----------------------------------------------------------------------------------------|
| getElement():   | Returns the element stored at this position.                                           |
| root():         | Returns the position of the root of the tree(or null if empty).                        |
| parent(p):      | Returns the position of the parent of position p(or null if p is the root).            |
| children(p):    | Returns an iterable collection containing the children of position p (if any).         |
| numChildren(p): | Returns the number of children of position p.                                          |
| isInternal(p):  | Returns true if position p has at least one child.                                     |
| isExternal(p):  | Returns true if position p does not have any children.                                 |
| size():         | Returns the number of positions (and hence elements) that are contained in the tree.   |
| isEmpty():      | Returns true if the tree does not contain any positions(and thus no elements).         |
| iterator():     | Returns an iterator for all elements in the tree(so that the tree itself is Iterable). |
| position():     | Returns an iterable collection of all positions of the tree.                           |


## Computing Depth and Height
Let p be a position within tree T. The __depth__ of p is the number of ancestors of p, other than p itself. Note that this definition implies that the depth of the root of T is 0. The depth of p can also be recursively defined as follows: 

* If p is the root, then the depth of p is 0.
* Otherwise, the depth of p is one plus the depth of the parent of p.

```JAVA
public int depth(Position<E> p) {
	if (isRoot(p))
		return 0;
	else
		return 1 + depth(parent(p));
}
```

We next define the __height__ of a tree to be equal to the maximum of the depths of its positions (or zero, if the tree is empty). Formally, we define the height of a position p in a tree T as follows:
* If p is a leaf, then the height of p is 0.
* Otherwise, the height of p is one more than the maximum of the heights of p's children.

The overall height of a nonempty tree can be computed by sending the root of the tree as a parameter.
```JAVA
public int height(Position<E> p) {
	int h = 0;
	for (Position<E> c : children(p))
		h = Math.max(h, 1 + height(c));
	return h;
}
```
## Binary Trees









