## The AND Operator &&
```C
#include <stdio.h>

void main(){

	printf("%d\n", 0 && 0); // 0
	printf("%d\n",-1 && 5); // 1
	printf("%d\n",-1 && 0); // 0

}
```

## The OR Operator ||
```C
#include <stdio.h>

void main(){

	printf("%d\n", 0 || 0); // 0
	printf("%d\n",-1 || 5); // 1
	printf("%d\n",-1 || 0); // 1

}
```

## The Not Operator !

```C
#include <stdio.h>

void main(){

	printf("%d\n", !0); // 1
	printf("%d\n", !-1); // 0
	printf("%d\n", !5); // 0

}
```

## The Conditional Operator

```C
#include <stdio.h>

void main(){

	int x = 12;
	int y = 19;
	int max = x > y ? x : y;

	printf("%d\n",max); // 19
}
```

## Bitwise Operators
* & Bitwise AND operator
* | Bitwise OR operator
* ^ Bitwise XOR operator
* ~ Bitwise NOT operator, also called 1's complement
* >> Bitwise shift right operator
* << Bitwise shift left operator

The fact that you only get a 1 bit when both of the bits being combined are 1 means that you can use the & operator to select a part of an integer variable or even just a single bit. You first define a value, usually called a mask, that you use to select the bit or bits that you want. It will contain a bit value of 1 for the bit positions you want to keep and a bit value of 0 for the bit positions you want to discard. You can then AND this mask with the value that you want to select from.
```C
#include <stdio.h>

void main(){

	unsigned int personalData;
	int male = 0x1;
	int german = 0x2;
	int french = 0x4;
	int italian = 0x8;
	
	personalData = 0b00001001; // personalData |= french | male;

	if(personalData & male){
		if(personalData & german){
			printf("He is German\n");
		}else if(personalData & (french | italian)){
			printf("He is either French or Italian\n");
		}
	}else if(!(personalData & male)){
		if(personalData & german){
			printf("She is German\n");
		}else if(personalData & (french | italian)){
			printf("She is either French or Italian\n");
		}
	}
	
	//He is either French or Italian

	personalData &= ~male; // Resetting male to female

	if(personalData & male){
		if(personalData & german){
			printf("He is German\n");
		}else if(personalData & (french | italian)){
			printf("He is either French or Italian\n");
		}
	}else if(!(personalData & male)){
		if(personalData & german){
			printf("She is German\n");
		}else if(personalData & (french | italian)){
			printf("She is either French or Italian\n");
		}
	}
	
	//She is either French or Italian
}
```

