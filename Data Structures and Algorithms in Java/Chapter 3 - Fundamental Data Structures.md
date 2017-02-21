
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
