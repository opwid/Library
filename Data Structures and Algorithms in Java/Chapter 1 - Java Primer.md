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
* When a method of a class is declared as static, it too is associated with the class itself, and not with a particular instance of the class. That means that the method is not invoked on a particular instance of the class using the traditional dot notation. Instead, it is typically invoked using the name of the class as a qualifier. Static methods can be useful for providing utility behaviors related to a class that need not rely on the state of any particular instance of that class.  

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
```Java
public class CounterTest {

	public static void main(String[] args) {
		Counter test1 = new Counter(12);
		System.out.println(Counter.count); // 1
		System.out.println(test1.getX()); // 12
		
		Counter test2 = new Counter(24);
		System.out.println(Counter.count); // 2
		System.out.println(test2.getX()); // 24

	}

}
```

* A method of a class may be declared as __abstract__, in which case its signature is provided but without an implementation of the method body. Abstract methods are an advanced feature of object-oriented programming to be combined with inheritance. In short, any subclass of a class with abstract methods is expected to provide a concrete implementation for each abstract method.
* A class with one or more abstract methods must also be formally declared as abstract, because it is essentially incomplete. (It is also permissible to declare a class as abstract even if it does not contain any abstract methods.) As a result, Java will not allow any instances of an abstract class to be constructed, although reference variables may be declared with an abstract type. [What is an abstract class, and when should it be used?](http://www.javacoffeebreak.com/faq/faq0084.html)
* A variable that is declared with the __final__ modifier can be initialized as part of that declaration, but can never again be assigned a new value. If it is a base type, then it is a constant. If a reference variable is final, then it will always refer to the same object (even if that object changes its internal state). If a member variable of a class is declared as final, it will typically be declared as static as well, because it would be unnecessarily wasteful to have every instance store the identical value when that value can be shared by the entire class. 
* Designating a method or an entire class as final has a completely different consequence, only relevant in the context of inheritance. A final method cannot be overridden by a subclass, and a final class cannot even be subclassed.
* All parameters in Java are __passed by value__, that is, any time we pass a parameter to a method, a copy of that parameter is made for use within the method body. So if we pass an int variable to a method, then that variable's integer value is copied. The method can change the copy but not the original. If we pass an object reference as a parameter to a method, then the reference is copied as well. Remember that we can have many different variables that all refer to the same object. Reassigning the internal reference variable inside a method will not change the reference that was passed in.
* Constructors cannot be static, abstract, or final, so the only modifiers that are allowed are those that affect visibility (i.e., public, protected, private, or the default package-level visibility).
* The name of the constructor must be identical to the name of the class it constructs. For example, when defining the Counter class, a constructor must be named Counter as well.
* We don't specify a return type for a constructor (not even void). Nor does the body of a constructor explicitly return anything. When a user of a class creates an instance using a syntax such as `Counter d = new Counter(5);`the new operator is responsible for returning a reference to the new instance to the caller; the responsibility of the constructor method is only to initialize the state of the new instance.
* Java programs can be called from the command line using the java command (in a Windows, Linux, or Unix shell), followed by the name of the Java class whose main method we want to run, plus any optional arguments.
```Bash
user@linux~/.../publictest/bin$: java -cp(or -classpath) . publicpackage.HelloWorld
Hello World!
```
* An important trait of Java's String class is that its instances are immutable; once an instance is created and initialized, the value of that instance cannot be changed. This is an intentional design, as it allows for great efficiencies and optimizations within the Java Virtual Machine.

