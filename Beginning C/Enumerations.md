With an enumeration, you define a new integer type where variables of the type have a fixed range of possible values that you specify.
```C
enum Weekday {Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday};
```
This statement defines a type __not__ a variable. Each enumerator is identified by the unique name you assign, and the compiler will assign a value of type int to each name. An enumeration is an integer type, and the enumerators that you specify will correspond to integer values. By default the enumerators will start from zero, with each successive enumerator having a value of one more than the previous one. Thus, in this example, the values Monday through Sunday will have values 0 through 6.  

You could declare a variable of type Weekday and initialize it like this:
```C
enum Weekday today = Wednesday;
```

It is also possible to declare variables of the enumeration type when you define the type.
```C
enum Weekday {Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday} today, tomorrow;
```
Naturally you could also initialize the variable in the same statement, so you could write this:
```C
enum Weekday {Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday} today = Monday, tomorrow = Tuesday;
```
Because variables of an enumeration type are of an integer type, they can be used in arithmetic expressions. You could write the previous statement like this:
```C
enum Weekday {Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday} today = Monday, tomorrow = today + 1;
```
__Note__: Although you specify a fixed set of values for an enumeration type, there is no checking mechanism to ensure that only these values are used in your program. It is up to you to ensure that you use only valid values for a given enumeration type. You can do this by only using the names of enumeration constants to assign values to variables.
