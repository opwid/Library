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
