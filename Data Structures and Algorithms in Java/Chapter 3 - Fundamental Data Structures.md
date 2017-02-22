
# The Insertion-Sort Algorithm
The algorithm proceeds by considering one element at a time, placing the element in the correct order relative to those before it. We start with the first element in the array, which is trivially sorted by itself. When considering the next element in the array, if it is smaller than the first, we swap them. Next we consider the third element in the array, swapping it leftward until it is in its proper order relative to the first two elements. We continue in this manner with the fourth element, the fifth, and so on, until the whole array is sorted.
```JAVA
public class Program {
	public static void main(String[] args) {

		int[] arr = {12,1,2,6,7,23,56,3,546,23,6,45,62};
		
		for (int i = 1; i < arr.length; i++) {

			int curr = arr[i];
			int j = i;
			for (int k = j-1; k >= 0; k--) {

				if(arr[k]>curr){
					arr[k+1] = arr[k];
					j--;
				}else{
					break;
				}
			}
			arr[j] = curr;
		}	
	}
}
/*
1 12 2 6 7 23 56 3 546 23 6 45 62 
1 2 12 6 7 23 56 3 546 23 6 45 62 
1 2 6 12 7 23 56 3 546 23 6 45 62 
1 2 6 7 12 23 56 3 546 23 6 45 62 
1 2 6 7 12 23 56 3 546 23 6 45 62 
1 2 6 7 12 23 56 3 546 23 6 45 62 
1 2 3 6 7 12 23 56 546 23 6 45 62 
1 2 3 6 7 12 23 56 546 23 6 45 62 
1 2 3 6 7 12 23 23 56 546 6 45 62 
1 2 3 6 6 7 12 23 23 56 546 45 62 
1 2 3 6 6 7 12 23 23 45 56 546 62 
1 2 3 6 6 7 12 23 23 45 56 62 546
*/

```

# java.util Methods for Arrays
## fill(A,x)
```JAVA
import java.util.Arrays;
public class Program {
	public static void main(String[] args) {
		
		int[] arr = new int[10];
		Arrays.fill(arr, 12);
		for (int i = 0; i < arr.length; i++) {
			System.out.print(arr[i] + " ");
		}
	}
}

/*
12 12 12 12 12 12 12 12 12 12
*/
```
## toString(A)
```JAVA
import java.util.Arrays;
public class Program {
	public static void main(String[] args) {
		
		int[] arr = new int[10];
		Arrays.fill(arr, 12);
		System.out.println(Arrays.toString(arr));
	}
}

/*
[12, 12, 12, 12, 12, 12, 12, 12, 12, 12]
*/
```
## sort(A)
```
import java.util.Arrays;
public class Program {
	public static void main(String[] args) {

		int[] arr = {12,1,2,6,7,23,56,3,546,23,6,45,62};
		Arrays.sort(arr);
		System.out.println(Arrays.toString(arr));
		
	}
}

/*
[1, 2, 3, 6, 6, 7, 12, 23, 23, 45, 56, 62, 546]
*/
```
# Singly Linked List

Arrays are great for storing things in a certain order, but they have drawbacks. The capacity of the array must be fixed when it is created, and insertions and deletions at interior positions of an array can be time consuming if many elements must be shifted.  

In this section, we introduce a data structure known as a linked list, which provides an alternative to an array-based structure. A linked list, in its simplest form, is a collection of nodes that collectively form a linear sequence. In a singly linked list, each node stores a reference to an object that is an element of the sequence, as well as a reference to the next node of the list.  

The linked list instance must keep a reference to the first node of the list, known as the __head__. The last node of the list is known as the __tail__. We can identify the tail as the node having null as its next reference.  

An important property of a linked list is that it does not have a predetermined fixed size; it uses space proportional to its current number of elements. When using a singly linked list, we can easily insert an element at the head of the list. The main idea is that we create a new node, set its element to the new element, set its next link to refer to the current head, and set the list's head to point to the new node. We can also easily insert an element at the tail of the list, provided we keep a reference to the tail node. In this case, we create a new node, assign its next reference to null, set the next reference of the tail to point to this new node, and then update the tail reference itself to this new node.  

Removing an element from the head of a singly linked list is essentially the reverse operation of inserting a new element at the head. Unfortunately, we cannot easily delete the last node of a singly linked list. Even if we maintain a tail reference directly to the last node of the list, we must be able to access the node before the last node in order to remove the last node. But we cannot reach the node before the tail by following next links from the tail. The only way to access this node is to start from the head of the list and search all the way through the list. But such a sequence of link-hopping operations could take a long time. If we want to support such an operation efficiently, we will need to make our list doubly linked.  

| Method        | Description                                                  |
|---------------|--------------------------------------------------------------|
| size()        | Returns the number of elements in the list.                  |
| isEmpty()     | Returns true if the list is empty, and false otherwise.      |
| first()       | Returns (but does not remove) the first element in the list. |
| last()        | Returns (but does not remove) the last element in the list.  |
| addFirst(e)   | Adds a new element to the front of the list.                 |
| addLast(e)    | Adds a new element to the end of the list.                   |
| removeFirst() | Removes and returns the first element of the list.           |


```JAVA
public class SinglyLinkedList<E> {
	
	private static class Node<E>{
		
		private E element;
		private Node<E> next;
		
		public Node(E e, Node<E> n){
			element = e;
			next = n;
			
		}
		public E getElement(){
			return element;
		}
		
		public Node<E> getNext(){
			return next;
		}
		
		public void setNext(Node<E> n){
			next = n;
		}
	}
	
	private Node<E> head = null;
	private Node<E> tail = null;
	private int size = 0;
	
	public SinglyLinkedList(){
		
	}
	
	public int size(){
		return size;
	}
	
	public boolean isEmpty(){
		return size==0;
	}
	
	public E first(){
		if(size == 0){
			return null;
		}
		return head.getElement();
	}
	
	public E last(){
		if(size == 0){
			return null;
		}
		return tail.getElement();
	}
	
	public void addFirst(E e){
		head = new Node<>(e, head);
		if(size == 0){
			tail = head; //new node becomes tail also
		}
		size++;
	}
	
	public void addLast(E e){
		Node<E> newest = new Node<E>(e,null);
		if(size==0){
			head = newest;
		}else{
			tail.setNext(newest);
		}
		
		tail = newest;
		size++;
		
	}
	
	public E removeFirst(){
		if(size == 0){
			return null;
		}
		E temp = head.getElement();
		head = head.getNext();
		size--;
		if(size == 0){
			tail = null;
		}
		return temp;
	}
	/*
	Singly list normally does not implement removeLast(). Time complexity: O(n)
	*/
	
	public E removeLast(){
		if(size == 0){
			return null;
		}
		E temp = tail.getElement();
		Node<E> tempNode = new Node<E>(null,head);
		for(int i = 0; i<size-1; i++){
			tempNode = tempNode.next;
		}
		tail = tempNode;
		size--;
		
		return temp;		
	}
 
}
```














































