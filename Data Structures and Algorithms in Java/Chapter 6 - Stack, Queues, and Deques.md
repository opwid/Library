# Stacks
A stack is a collection of objects that are inserted and removed according to last-in, first-out principle. A user may insert objects into a stack at any time, but may only access or remove the most recently inserted object that remains (at the so-called "top" of the stack).

| Method     | Description                                                                               |
|------------|-------------------------------------------------------------------------------------------|
| push(e):   | Adds element  e to the top of the stack                                                   |
| pop():     | Removes and returns the top element from the stack(or null if the stack is empty).        |
| top():     | Returns the top element of the stack, without removing it(or null if the stack is empty). |
| size():    | Returns the number of elements in the stack.                                              |
| isEmpty(): | Returns a boolean indicating whether the stack is empty.                                  |

##  Simple Array-Based Stack Implementation

As our first implementation of the stack ADT, we store elements in an array, named _data_, with capacity N for some fixed N. We oriented the stack so that the bottom element of the stack is always stored in cell _data[0]_, and the top element of the stack in cell _data[t]_ for index t that is equal to one less than the current size of the stack.  

The array implementation of a stack is simple and efficient. Nevertheless, this implementation has one negative aspect —it relies on a fixed-capacity array, which limits the ultimate size of the stack.  

| Method     | Worst-case Running Time |
|------------|-------------------------|
| push(e):   | O(1)                    |
| pop():     | O(1)                    |
| top():     | O(1)                    |
| size():    | O(1)                    |
| isEmpty(): | O(1)                    |


## Implementing a Stack with a Singly Linked List

Unlike our array-based implementation, the linked-list approach has memory usage that is always proportional to the number of actual elements currently in the stack, and without an arbitrary capacity limit. In designing such an implementation, we need to decide if the top of the stack is at the front or back of the list. There is clearly a best choice here, however, since we can insert and delete elements in constant time only at the front. With the top of the stack stored at the front of the list, all methods execute in constant time.

# Queues

Another fundamental data structure is the queue. It is a close "cousin" of the stack, but a queue is a collection of objects that are inserted and removed according to the first-in, first-out principle. That is, elements can be inserted at any time, but only the element that has been in the queue the longest can be next removed. We usually say that elements enter a queue at the back and are removed from the front.

| Method      | Description                                                                                 |
|-------------|---------------------------------------------------------------------------------------------|
| enqueue(e): | Adds element e to the back of queue.                                                        |
| dequeue():  | Removes and returns the first element from the queue(or null if the queue is empty).        |
| first():    | Returns the first element of the queue,,without removing it(or null if the queue is empty). |
| size():     | Returns the number of elements in the queue.                                                |
| isEmpty():  | Returns a boolean indicating whether the queue is empty.                                    |


## Array-Based Queue Implementation

Let's assume that as elements are inserted into a queue, we store them in an array such that the first element is at index 0, the second element at index 1, and so on. We will replace a dequeued element in the array with a null reference, and maintain an explicit variable f to represent the index of the element that is currently at the front of the queue. Such an algorithm for dequeue would run in O(1) time.  

However, there remains a challenge with the revised approach. With an array of capacity N, we should be able to store up to N elements before reaching any exceptional case. If we repeatedly let the front of the queue drift rightward over time, the back of the queue would reach the end of the underlying array even when there are fewer than N elements currently in the queue. We must decide how to store additional elements in such a configuration. The solution is using array circularly.

## Implementing a Queue with a Singly Linked List
As we did for the stack ADT, we can easily adapt a singly linked list to implement the queue ADT while supporting worst-case O(1)-time for all operations, and without any artificial limit on the capacity. The natural orientation for a queue is to align the front of the queue with the front of the list, and the back of the queue with the tail of the list, because the only update operation that singly linked lists support at the back end is an insertion.











