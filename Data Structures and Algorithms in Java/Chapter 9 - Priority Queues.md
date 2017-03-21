
# The Priority Queue Abstract Data Type

In this chapter, we introduce a new abstract data type known as a priority queue. This is a collection of prioritized elements that allows arbitrary element insertion, and allows the removal of the element that has first priority. When an element is added to a priority queue, the user designates its priority by providing an associated key. The element with the minimal key will be the next to be removed from the queue (thus, an element with key 1 will be given priority over an element with key 2).  

We model an element and its priority as a key-value composite known as an entry.  

| Method        | Description                                                                                                                  |
|---------------|------------------------------------------------------------------------------------------------------------------------------|
| insert(k, v): | Creates an entry with key k and value v in the priority queue.                                                               |
| min():        | Returns (but does not remove) a priority queue entry (k,v) having minimal key; returns null if the priority queue is empty.  |
| removeMin():  | Removes and returns an entry (k,v) having minimal key from the priority queue; returns null if the priority queue is empty.  |
| size():       | Returns the number of entries in the priority queue.                                                                         |
| isEmpty():    | Returns a boolean indicating whether the priority queue is empty.                                                            |

The following table shows a series of operations and their effects on an initially empty priority queue.


| Method      | Return Value | Contents                |
|-------------|--------------|-------------------------|
| insert(5,A) |              | { (5,A) }               |
| insert(9,C) |              | { (5,A), (9,C) }        |
| insert(3,B) |              | { (3,B), (5,A), (9,C) } |
| min()       | (3,B)        | { (3,B), (5,A), (9,C) } |
| removeMin() | (3,B)        | { (5,A), (9,C) }        |
| insert(7,D) |              | { (5,A), (7,D), (9,C) } |


***
## The Comparable Interface
Java provides two means for defining comparisons between object types. The first of these is that a class may define what is known as the natural ordering of its instances by formally implementing the java.lang.Comparable interface, which includes a single method, compareTo.The syntax a.compareTo(b) must return an integer i with the following meaning:  
* i < 0 designates that a < b.
* i = 0 designates that a = b.
* i > 0 designates that a > b.

## The Comparator Interface
In some applications, we may want to compare objects according to some notion other than their natural ordering. For example, we might be interested in which of two strings is the shortest, or in defining our own complex rules for judging which of two stocks is more promising. To support generality, Java defines the java.util.Comparator interface. A comparator is an object that is external to the class of the keys it compares. It provides a method with the signature compare(a, b) that returns an integer with similar meaning to the compareTo method described above.
```JAVA
public class StringLengthComparator implements Comparator<String> {
  public int compare(String a, String b) {
    if (a.length( ) < b.length( )) return −1;
    else if (a.length( ) == b.length( )) return 0;
    else return 1;
  }
}
```
***

## Comparing Two List-Based Implementations

| Method    | Unsorted List | Sorted List |
|-----------|---------------|-------------|
| size      | O(1)          | O(1)        |
| isEmpty   | O(1)          | O(1)        |
| insert    | O(1)          | O(n)        |
| min       | O(n)          | O(1)        |
| removeMin | O(n)          | O(1)        |




# Heaps
The two strategies for implementing a priority queue ADT in the previous section demonstrate an interesting trade-off. When using an unsorted list to store entries, we can perform insertions in O(1) time, but finding or removing an element with minimal key requires an O(n)-time loop through the entire collection. In contrast, if using a sorted list, we can trivially find or remove the minimal element in O(1) time, but adding a new element to the queue may require O(n) time to restore the sorted order.  

In this section, we provide a more efficient realization of a priority queue using a data structure called a binary heap. This data structure allows us to perform both insertions and removals in logarithmic time, which is a significant improvement over the list-based implementations discussed in Priority Queues. The fundamental way the heap achieves this improvement is to use the structure of a binary tree to find a compromise between elements being entirely unsorted and perfectly sorted. 

## The Heap Data Structure
A heap (see Figure 9.1) is a binary tree T that stores entries at its positions, and that satisfies two additional properties: a relational property defined in terms of the way keys are stored in T and a structural property defined in terms of the shape of T itself. The relational property is the following: 

* __Heap-Order Property__: In a heap T, for every position p other than the root, the key stored at p is greater than or equal to the key stored at p’s parent.

![9.1](https://github.com/opwid/Library/blob/master/Data%20Structures%20and%20Algorithms%20in%20Java/Images/9.1.png) 

As a consequence of the heap-order property, the keys encountered on a path from the root to a leaf of T are in nondecreasing order. Also, a minimal key is always stored at the root of T. This makes it easy to locate such an entry when min or removeMin is called, as it is informally said to be "at the top of the heap".  

For the sake of efficiency, as will become clear later, we want the heap T to have as small a height as possible. We enforce this requirement by insisting that the heap T satisfy an additional structural property; it must be what we term complete:

* __Complete Binary Tree Propert__: A heap T with height h is a complete binary tree if levels 0, 1, 2, ..., h-1 of T have the maximal number of nodes possible (namely, level i has 2<sup>i</sup> nodes, for 0 ≤ i ≤ h − 1) and the remaining nodes at level h reside in the leftmost possible positions at that level.  


__The Height of a Heap__: A heap T storing n entries has height h = ⌊log n⌋.  

## Implementing a Priority Queue with a Heap

The Height of a Heap has an important consequence, for it implies that if we can perform update operations on a heap in time proportional to its height, then those operations will run in logarithmic time. Let us therefore turn to the problem of how to efficiently perform various priority queue methods using a heap.  

### Adding an Entry to the Heap 

Let us consider how to perform insert(k, v) on a priority queue implemented with a heap T. We store the pair (k, v) as an entry at a new node of the tree. To maintain the __complete binary tree property__, that new node should be placed at a position p just beyond the rightmost node at the bottom level of the tree, or as the leftmost position of a new level, if the bottom level is already full (or if the heap is empty).  

#### Up-Heap Bubbling After an Insertion  

After this action, the tree T is complete, but it may violate the __heap-order property__. Hence, unless position p is the root of T (that is, the priority queue was empty before the insertion), we compare the key at position p to that of p's parent, which we denote as q. If key k<sub>p</sub> ≥ k<sub>q</sub>, the heap-order property is satisfied and the algorithm terminates. If instead k<sub>p</sub> < k<sub>q</sub>, then we need to restore the heap-order property, which can be locally achieved by swapping the entries stored at positions p and q. This swap causes the new entry to move up one level. Again, the heap-order property may be violated, so we repeat the process, going up in T until no violation of the heap-order property occurs.  

The upward movement of the newly inserted entry by means of swaps is conventionally called __up-heap bubbling__. A swap either resolves the violation of the heap-order property or propagates it one level up in the heap. In the worst case, up-heap bubbling causes the new entry to move all the way up to the root of heap T. Thus, in the worst case, the number of swaps performed in the execution of method insert is equal to the height of T.  

### Removing the Entry with Minimal Key

Let us now turn to method removeMin of the priority queue ADT. We know that an entry with the smallest key is stored at the root r of T (even if there is more than one entry with smallest key). However, in general we cannot simply delete node r, because this would leave two disconnected subtrees.  

Instead, we ensure that the shape of the heap respects the complete binary tree property by deleting the leaf at the last position p of T, defined as the rightmost position at the bottommost level of the tree. To preserve the entry from the last position p, we copy it to the root r (in place of the entry with minimal key that is being removed by the operation).

#### Down-Heap Bubbling After a Removal
We are not yet done, however, for even though T is now complete, it likely violates the __heap-order property__. If T has only one node (the root), then the heap-order property is trivially satisfied and the algorithm terminates. Otherwise, we distinguish two cases, where p initially denotes the root of T:

* If p has no right child, let c be the left child of p.
* Otherwise (p has both children), let c be a child of p with minimal key.

If key k<sub>p</sub> ≤ k<sub>c</sub>, the heap-order property is satisfied and the algorithm terminates. If instead k<sub>p</sub> > k<sub>c</sub>, then we need to restore the heap-order property. This can be locally achieved by swapping the entries stored at p and c. It is worth noting that when p has two children, we intentionally consider the smaller key of the two children.  

Having restored the heap-order property for node p relative to its children, there may be a violation of this property at c; hence, we may have to continue swapping down T until no violation of the heap-order property occurs. This downward swapping process is called __down-heap bubbling__. A swap either resolves the violation of the heap-order property or propagates it one level down in the heap. In the worst case, an entry moves all the way down to the bottom level. Thus, the number of swaps performed in the execution of method removeMin is, in the worst case, equal to the height of heap T, that is, it is ⌊log n⌋.

## Array-Based Implementation of a Complete Binary Tree

The array-based representation of a binary tree (see Chapter 8) is especially suitable for a complete binary tree. We recall that in this implementation, the elements of the tree are stored in an array-based list A such that the element at position p is stored in A with index equal to the level number f(p) of p, defined as follows:

* If p is the root, then f(p) = 0.
* If p is the left child of position q, then f(p) = 2f(q) + 1.
* If p is the right child of position q, then f(p) = 2f(q) + 2.

For a tree with of size n, the elements have contiguous indices in the range [0, n-1] and the last position of is always at index n-1.  

![9.4](https://github.com/opwid/Library/blob/master/Data%20Structures%20and%20Algorithms%20in%20Java/Images/9.4.png) 

The array-based heap representation avoids some complexities of a linked tree structure. Specifically, methods insert and removeMin depend on locating the last position of a heap. With the array-based representation of a heap of size n, the last position is simply at index n-1. Locating the last position in a heap implemented with a linked tree structure requires more effort.























