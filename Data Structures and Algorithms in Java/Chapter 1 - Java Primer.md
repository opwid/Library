```Java
//Code Fragment 1.2

public class Counter {
  private int count; // a simple integer instance variable
  public Counter( ) { } // default constructor (count is 0)
  public Counter(int initial) { count = initial; } // an alternate constructor
  public int getCount( ) { return count; } // an accessor method
  public void increment( ) { count++; } // an update method
  public void increment(int delta) { count += delta; } // an update method
  public void reset( ) { count = 0; } // an update method
}
```
```Java
//Code Fragment 1.3

public class CounterDemo {
  public static void main(String[ ] args) {
  Counter c; // declares a variable; no counter yet constructed
  c = new Counter( ); // constructs a counter; assigns its reference to c
  c.increment( ); // increases its value by one
  c.increment(3); // increases its value by three more
  int temp = c.getCount( ); // will be 4
  c.reset( ); // value becomes 0
  Counter d = new Counter(5); // declares and constructs a counter having value 5
  d.increment( ); // value becomes 6
  Counter e = d; // assigns e to reference the same object as d
  temp = e.getCount( ); // will be 6 (as e and d reference the same counter)
   e.increment(2); // value of e (also known as d) becomes 8

  }
}
```

# Access Control Modifiers
* The __public__ class modifier designates that all classes may access the defined aspect. For example, line 1 of of Code Fragment 1.2 designates `public class Counter {` and therefore all other classes (such as CounterDemo) are allowed to construct new instances of the Counter class, as well as to declare variables and parameters of type Counter. In Java, each public class must be defined in a separate file named classname.java, where "classname" is the name of the class (for example, file Counter.java for the Counter class definition).
* The designation of public access for a particular method of a class allows any other class to make a call to that method. For example, line 5 of Code Fragment 1.2 designates `public int getCount( ) { return count; }` which is why the CounterDemo class may call c.getCount( ).
* If an instance variable is declared as public, dot notation can be used to directly access the variable by code in any other class that possesses a reference to an instance of this class. For example, were the count variable of Counter to be declared as public (which it is not), then the CounterDemo would be allowed to read or modify that variable using a syntax such as c.count.
* The __protected__ modifier specifies that the member can only be accessed within its own package (as with package-private) and, in addition, by a subclass of its class in another package.
* The __private__ modifier specifies that the member can only be accessed in its own class. Neither subclasses nor any other classes have access to such members. For example, we defined the count instance variable of the Counter class to have private access level. We were allowed to read or edit its value from within methods of that class (such as getCount, increment, and reset), but other classes such as CounterDemo cannot directly access that field. Of course, we did provide other public methods to grant outside classes with behaviors that depended on the current count value.
* Finally, we note that if no explicit access control modifier is given, the defined aspect has what is known as __package-private__ access level. This allows other classes in the same package to have access, but not any classes or subclasses from other packages.

![Access Levels](https://github.com/opwid/Library/blob/master/Data%20Structures%20and%20Algorithms%20in%20Java/Images/Access%20Levels.png)  

* The __static__ modifier in Java can be declared for any variable or method of a class. When a variable of a class is declared as static, its value is associated with the class as a whole, rather than with each individual instance of that class. Static variables are used to store "global" information about a class. (For example, a static variable could be used to maintain the total number of instances of that class that have been created.) Static variables exist even if no instance of their class exists. The static variable gets memory only once in class area at the time of class loading. 
* When a method of a class is declared as static, it too is associated with the class itself, and not with a particular instance of the class. That means that the method is not invoked on a particular instance of the class using the traditional dot notation. Instead, it is typically invoked using the name of the class as a qualifier.  

```Java
class Counter {
	
	 int x = 0;
	 static int count = 0;
	
	 Counter(){
		
		this.x = 0;
		count++;
	}
	 Counter(int x){
		
		this.x = x;
		count++;
	}
	 int getX(){
		return x;
	}

}

```
