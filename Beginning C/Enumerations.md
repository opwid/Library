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

You can specify your own integer value for any or all of the enumerators explicitly. Although the names you use for enumerators must be unique, there is no requirement for the enumerator values themselves to be unique. Unless you have a specific reason for making some of the values the same, it is usually a good idea to ensure that they are unique. Here's how you could define the Weekday type so that the enumerator values start from 1:

```C
enum Weekday {Monday = 1, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday};
```
Now the enumerators Monday through Sunday will correspond to values 1 through 7. The enumerators that follow an enumerator with an explicit value will be assigned successive integer values. This can cause enumerators to have duplicate values, as in the following example:
```C
enum Weekday {Monday = 5, Tuesday = 4, Wednesday, Thursday = 10, Friday = 3, Saturday, Sunday};
```
Wednesday will be set to Tuesday+1 so it will be 5, the same as Monday. Similarly Saturday and Sunday will be set to 4 and 5, so they also have duplicate values.  

You can create variables of an enumeration type without specifying a tag, so there's no enumeration type name. For example:
```C
enum {red, orange, yellow, green, blue, indigo, violet} shirt_color=blue;
```

















