# The List ADT
In this chapter, we explore several abstract data types that represent a linear sequence of elements, but with more general support for adding or removing elements at arbitrary positions. However, designing a single abstraction that is well suited for efficient implementation with either an array or a linked list is challenging, given the very different nature of these two fundamental data structures.  

| Method     	| Description                                                                                                                                                                      	|
|------------	|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| size():    	| Returns the number of elements in the list.                                                                                                                                      	|
| isEmpty(): 	| Returns a boolean indicating whether the list is empty.                                                                                                                          	|
| get(i):    	| Returns the element of the list having index i; an error condition occurs if i is not in range [0, size()-1]                                                                     	|
| set(i,e):  	| Replaces the element at index i with e, and returns the old element that was replaced; an error condition occurs if i is not in range[0, size( )−1].                             	|
| add(i,e):  	| Inserts a new element e into the list so that it has index i, moving all subsequent elements one index later in the list; an error condition occurs if i is not in range[0,size] 	|
| remove(i): 	| Removes and returns the element at index i, moving all subsequent elements one index earlier in the list; an error condition occurs if i is not in range [0,size()-1]            	|
