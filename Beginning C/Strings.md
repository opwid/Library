
You use an array of type char to hold strings. This is the simplest form of string variable. You can declare an array variable like this:
```C
#include <stdio.h>

void main(){
	char string1[6] = "Hello\0";
	char string2[] = "World";
	
	printf("%s %s\n",string1,string2 ); //Hello World
}
```
You can initialize just part of an array of elements of type char with a string. For example:
```C
char str[40] = "To be";
```
The compiler will initialize the first five elements, str[0] to str[4], with the characters of the string constant, and str[5] will contain the null character, '\0'. Of course, space is allocated for all 40 elements of the array, and they're all available to use in any way you want.

## Finding the Lenght of a String
The standard function to obtain the length of a string is strlen(). It requires just the address of the string as the argument. This function will overrun the end of the string when no \0 is present.
```C
#include <stdio.h>
#include <string.h>

void main(){
	char string1[6] = "Hello\0";
	char string2[] = "World";
	
	printf("%zu\n",strlen(string1)); // 5
	printf("%zu\n",strlen(string2)); // 5
	
}
```
