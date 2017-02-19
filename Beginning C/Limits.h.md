
| Type      	| Lower Limit  	| Upper Limit 	|
|-----------	|--------------	|-------------	|
| char      	| CHAR_MIN     	| CHAR_MAX    	|
| short     	| SHRT_MIN     	| SHRT_MAX    	|
| int       	| INT_MIN      	| INT_MAX     	|
| long      	| LONG_MIN     	| LONG_MAX    	|
| long long 	| LLONG_MIN    	| LLONG_MAX   	|

The lower limits for the unsigned integer types are all 0, so there are no symbols for these. The symbols corresponding to the upper limits for the unsigned integer types are UCHAR_MAX, USHRT_MAX, UINT_MAX, ULONG_MAX, and ULLONG_MAX.

```C
#include <stdio.h>
#include <limits.h>

void main(){

	printf("CHAR_MIN: %d, CHAR_MAX: %d\n", CHAR_MIN, CHAR_MAX);
	printf("SHORT_MIN:%hd , SHORT_MAX: %hd\n", SHRT_MIN, SHRT_MAX);
	printf("INT_MIN: %d, INT_MAX: %d\n", INT_MIN, INT_MAX);
	printf("LONG_MIN: %ld, LONG_MAX: %ld\n", LONG_MIN, LONG_MAX);
	printf("LLONG_MIN: %lld, LLONG_MAX: %lld\n", LLONG_MIN, LLONG_MAX);

	printf("UCHAR_MIN: 0, UCHAR_MAX: %u\n", UCHAR_MAX);
	printf("USHORT_MIN: 0, USHORT_MAX: %u\n", USHRT_MAX);
	printf("USHORT_MIN: 0, UINT_MAX: %u\n", UINT_MAX);
	printf("ULONG_MIN: 0, ULONG_MAX: %lu\n", ULONG_MAX);
	printf("ULLONG_MIN: 0, ULLONG_MAX: %llu\n", ULLONG_MAX);

}

CHAR_MIN: -128, CHAR_MAX: 127
SHORT_MIN:-32768 , SHORT_MAX: 32767
INT_MIN: -2147483648, INT_MAX: 2147483647
LONG_MIN: -9223372036854775808, LONG_MAX: 9223372036854775807
LLONG_MIN: -9223372036854775808, LLONG_MAX: 9223372036854775807
UCHAR_MIN: 0, UCHAR_MAX: 255
USHORT_MIN: 0, USHORT_MAX: 65535
USHORT_MIN: 0, UINT_MAX: 4294967295
ULONG_MIN: 0, ULONG_MAX: 18446744073709551615
ULLONG_MIN: 0, ULLONG_MAX: 18446744073709551615
```
