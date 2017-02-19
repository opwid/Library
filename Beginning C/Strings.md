
You use an array of type char to hold strings. This is the simplest form of string variable. You can declare an array variable like this:
```C
#include <stdio.h>

void main(){
	char string1[6] = "Hello\0";
	char string2[] = "World";
	
	printf("%s %s\n",string1,string2 ); //Hello World
}
```
