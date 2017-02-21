# The List ADT
In this chapter, we explore several abstract data types that represent a linear sequence of elements, but with more general support for adding or removing elements at arbitrary positions. However, designing a single abstraction that is well suited for efficient implementation with either an array or a linked list is challenging, given the very different nature of these two fundamental data structures.  

| Method     	 Description                                                                                                                                                                      	|
|------------	|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| size():    	| Returns the number of elements in the list.                                                                                                                                      	|
| isEmpty(): 	| Returns a boolean indicating whether the list is empty.                                                                                                                          	|
| get(i):    	| Returns the element of the list having index i; an error condition occurs if i is not in range [0, size()-1]                                                                     	|
| set(i,e):  	| Replaces the element at index i with e, and returns the old element that was replaced; an error condition occurs if i is not in range[0, size( )−1].                             	|
| add(i,e):  	| Inserts a new element e into the list so that it has index i, moving all subsequent elements one index later in the list; an error condition occurs if i is not in range[0,size] 	|
| remove(i): 	| Removes and returns the element at index i, moving all subsequent elements one index earlier in the list; an error condition occurs if i is not in range [0,size()-1]            	|



# Array Lists

An obvious choice for implementing the list ADT is to use an array A, where A[i] stores (a reference to) the element with index i.
```JAVA
import java.util.Collection;
import java.util.Iterator;
import java.util.List;
import java.util.ListIterator;

public class ArrayList<E> implements List<E> {

	public final static int CAPACITY=16;
	private E[] data;
	private int size = 0;

	public ArrayList(){
		this(CAPACITY);
	}

	public ArrayList(int CAPACITY){
		data = (E[]) new Object[CAPACITY];
	}

	@Override
	public boolean add(E e) {
		// TODO Auto-generated method stub
		return false;
	}

	@Override
	public void add(int index, E element) throws IndexOutOfBoundsException, IllegalStateException {
		// TODO Auto-generated method stub
		checkIndex(index, size + 1);
		if(size != data.length){
			for (int k = size-1; k >= index; k--) {
				data[k+1] = data[k];
			}
			data[index] = element;
			size++;
		}else{
			throw new IllegalStateException("Array is full");
		}
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
		// TODO Auto-generated method stub
		return false;
	}

	@Override
	public boolean addAll(int index, Collection<? extends E> c) {
		// TODO Auto-generated method stub
		return false;
	}

	@Override
	public void clear() {
		// TODO Auto-generated method stub

	}

	@Override
	public boolean contains(Object o) {
		// TODO Auto-generated method stub
		return false;
	}

	@Override
	public boolean containsAll(Collection<?> c) {
		// TODO Auto-generated method stub
		return false;
	}

	@Override
	public E get(int index) {
		// TODO Auto-generated method stub
		return data[index];
	}

	@Override
	public int indexOf(Object o) {
		// TODO Auto-generated method stub
		return 0;
	}

	@Override
	public boolean isEmpty() {
		// TODO Auto-generated method stub
		return size == 0;
	}

	@Override
	public Iterator<E> iterator() {
		// TODO Auto-generated method stub
		return null;
	}

	@Override
	public int lastIndexOf(Object o) {
		// TODO Auto-generated method stub
		return 0;
	}

	@Override
	public ListIterator<E> listIterator() {
		// TODO Auto-generated method stub
		return null;
	}

	@Override
	public ListIterator<E> listIterator(int index) {
		// TODO Auto-generated method stub
		return null;
	}

	@Override
	public boolean remove(Object o) {
		// TODO Auto-generated method stub
		return false;
	}

	@Override
	public E remove(int index) throws IndexOutOfBoundsException {
		// TODO Auto-generated method stub
		checkIndex(index, size);
		E temp = data[index];
		for (int i = index; i < size; i++) {
			data[i] = data[i+1];
		}
		size--;
		return temp;
	}

	@Override
	public boolean removeAll(Collection<?> c) {
		// TODO Auto-generated method stub
		return false;
	}

	@Override
	public boolean retainAll(Collection<?> c) {
		// TODO Auto-generated method stub
		return false;
	}

	@Override
	public E set(int index, E element) {
		// TODO Auto-generated method stub
		checkIndex(index, size);
		E temp = data[index];
		data[index] = element;
		return temp;
	}

	@Override
	public int size() {
		// TODO Auto-generated method stub
		return size;
	}

	@Override
	public List<E> subList(int fromIndex, int toIndex) {
		// TODO Auto-generated method stub
		return null;
	}

	@Override
	public Object[] toArray() {
		// TODO Auto-generated method stub
		return null;
	}

	@Override
	public <T> T[] toArray(T[] a) {
		// TODO Auto-generated method stub
		return null;
	}
	protected void checkIndex(int i, int n) throws IndexOutOfBoundsException {
		if (i < 0 || i >= n){
			throw new IndexOutOfBoundsException("Illegal index: " + i);
		}
	}
}

```

