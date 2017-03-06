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


