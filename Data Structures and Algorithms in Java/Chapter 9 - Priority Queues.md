
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
































