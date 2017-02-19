stdlib.h defines success by zero or EXIT_SUCCESS and failure by 1 or EXIT_FAILURE. However, a true statement is one that evaluates to a nonzero number. A false statement evaluates to zero. When you perform comparison with the relational operators, the operator will return 1 if the comparison is true, or 0 if the comparison is false.

```C
#include <stdio.h>

void main(){

	printf("%d\n", 1 == 1); // 1
	printf("%d\n", 1 == 2); // 0
	
}
```
