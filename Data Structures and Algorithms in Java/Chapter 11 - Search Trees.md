
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
public Value BinarySearh(Node n, Key k){
		if(n.key == k){
			
			return n.value;
		}else if(n.key.compareTo(k)>0){
			BinarySearh(n.left, k);
			
		}else if(n.key.compareTo(k)<0){
			BinarySearh(n.right, k);
		}
		return null;
	}
```
