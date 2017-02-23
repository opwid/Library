
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
	Singly linked list normally does not implement removeLast(). Time complexity: O(n)
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
# Circularly Linked List

In this section, we design a structure known as a circularly linked list, which is essentially a singularly linked list in which the next reference of the tail node is set to refer back to the head of the list (rather than null).  

CircularlyLinkedList class supports all of the public behaviors of our SinglyLinkedList class and one additional update method:

| Method        | Description                                                  |
|---------------|--------------------------------------------------------------|
| rotate()     | Moves the first element to the end of the list.              |

We no longer explicitly maintain the head reference. So long as we maintain a reference to the tail, we can locate the head as tail.getNext(). Implementing the new rotate method is quite trivial. We do not move any nodes or elements, we simply advance the tail reference to point to the node that follows it (the implicit head of the list).  

We can add a new element at the front of the list by creating a new node and linking it just after the tail of the list. To implement the addLast method, we can rely on the use of a call to addFirst and then immediately advance the tail reference so that the newest node becomes the last. Removing the first node from a circularly linked list can be accomplished by simply updating the next field of the tail node to bypass the implicit head.
```JAVA

public class CircularlyLinkedList<E> {

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
	
	private Node<E> tail = null;
	private int size = 0;
	public CircularlyLinkedList(){
		
	}
	public int size(){
		return size;
	}
	public boolean isEmpty(){
		return size == 0;
	}
	public E first(){
		if(isEmpty()){
			return null;
		}
		
		return tail.getNext().getElement();
	}
	public E last(){
		if(isEmpty()){
			return null;
		}
		
		return tail.getElement();
	}
	
	public void rotate(){
		if(tail != null){
			tail = tail.getNext();
		}
	}
	public void addFirst(E e){
		if(size == 0){
			tail = new Node<>(e,null);
			tail.setNext(tail);
		}else{
			Node<E> newest = new Node<E>(e,tail.getNext());
			tail.setNext(newest);
		}
		size++;
	}
	public void addLast(E e){
		addFirst(e);
		tail = tail.getNext();
	}
	public E removeFirst(){
		if(isEmpty()){
			return null;
		}
		Node<E> head = new Node<E>(null,tail.getNext());
		if(head == tail){
			tail = null;
		}else{
			tail.setNext(head.getNext());	
		}
		size--;
		return head.getElement();
		
	}
}
```

# Doubly Linked List 
To provide greater symmetry, we define a linked list in which each node keeps an explicit reference to the node before it and a reference to the node after it. Such a structure is known as a doubly linked list. These lists allow a greater variety of O(1)-time update operations, including insertions and deletions at arbitrary positions within the list. We continue to use the term "next" for the reference to the node that follows another, and we introduce the term "prev" for the reference to the node that precedes it.  

| Method        | Description                                                  |
|---------------|--------------------------------------------------------------|
| size()        | Returns the number of elements in the list.                  |
| isEmpty()     | Returns true if the list is empty, and false otherwise.      |
| first()       | Returns (but does not remove) the first element in the list. |
| last()        | Returns (but does not remove) the last element in the list.  |
| addFirst(e)   | Adds a new element to the front of the list.                 |
| addLast(e)    | Adds a new element to the end of the list.                   |
| removeFirst() | Removes and returns the first element of the list.           |
| removeLast()  | Removes and returns the last element of the list.            |

```JAVA

public class DoublyLinkedList<E> {

	private static class Node<E>{
		private E element;
		private Node<E> prev;
		private Node<E> next;

		public Node(E e, Node<E> p, Node<E> n){
			element = e;
			prev = p;
			next = n;
		}
		public E getElement(){
			return element;	
		}
		public Node<E> getPrev(){
			return prev;
		}
		public Node<E> getNext(){
			return next;
		}
		public void setPrev(Node<E> p){
			prev = p;
		}
		public void setNext(Node<E> n){
			next = n;
		}
	}
	private Node<E> header;
	private Node<E> trailer;
	private int size = 0;

	public DoublyLinkedList(){
		header = new Node<>(null, null, null);
		trailer = new Node<>(null, header, null);
		header.setNext(trailer);

	}
	public int size(){
		return size;
	}
	public boolean isEmpty(){
		return size == 0;
	}
	public E first(){
		if(isEmpty()){
			return null;
		}
		return header.getNext().getElement();
	}
	public E last(){
		if(isEmpty()){
			return null;
		}
		return trailer.getPrev().getElement();
	}
	public void addFirst(E e){
		addBetween(e, header, header.getNext());
	}
	public void addLast(E e){
		addBetween(e, trailer.getPrev(), trailer);
	}
	public E removeFirst( ) {
		if (isEmpty( )) return null;
		return remove(header.getNext( ));
	}
	public E removeLast( ) {
		if (isEmpty( )) return null;
		return remove(trailer.getPrev( ));
	}
	private void addBetween(E e, Node<E> predecessor, Node<E> successor) {
		Node<E> newest = new Node<>(e, predecessor, successor);
		predecessor.setNext(newest);
		successor.setPrev(newest);
		size++;
	}
	private E remove(Node<E> node) {
		Node<E> predecessor = node.getPrev( );
		Node<E> successor = node.getNext( );
		predecessor.setNext(successor);
		successor.setPrev(predecessor);
		size--;
		return node.getElement( );
	}

}

```














































