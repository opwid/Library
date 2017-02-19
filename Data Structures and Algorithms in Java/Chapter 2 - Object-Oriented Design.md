# Inheritance
In object-oriented programming, the mechanism for a modular and hierarchical organization is a technique known as __inheritance__. This allows a new class to be defined based upon an existing class as the starting point. In object-oriented terminology, the existing class is typically described as the base class, parent class, or superclass, while the newly defined class is known as the subclass or child class. We say that the subclass extends the superclass.  

In Java, each class can extend exactly one other class. Because of this property, Java is said to allow only __single inheritance__ among classes. We should also note that even if a class definition makes no explicit use of the extends clause, it automatically inherits from a class, java.lang.Object, which serves as the universal superclass in Java.

When inheritance is used, the subclass automatically inherits, as its starting point, __all methods from the superclass__ (other than __constructors__). The subclass can differentiate itself from its superclass in two ways. It may augment the superclass by adding new fields and new methods. It may also specialize existing behaviors by providing a new implementation that __overrides__ an existing method.  

The subclass inherits __all__ the members and methods that are declared as public or protected. If declared as private it can not be inherited by the subclasses. The private members can be accessed only in its own class. The private members can be accessed through assessor methods as shown in the example below. The subclass cannot inherit a member of the superclass if the subclass declares another member with the same name.  

The __constructor__ in the superclass is responsible for building the object of the superclass and the constructor of the subclass builds the object of subclass. When the subclass constructor is called during object creation, it by default invokes the default constructor of superclass. Hence, in inheritance the objects are constructed top-down. The superclass constructor can be called explicitly using the keyword __super__, but it should be first statement in a constructor. The keyword super always refers to the superclass immediately above of the calling class in the hierarchy. The use of multiple super keywords to access an ancestor class other than the direct parent is illegal.

```JAVA
//BClass.java
public class BClass {
	private int privateX = 0;
	public int publicX = 0;
	
	BClass(int x){
		this.privateX = x;
	}
	public int getPrivateX() {
		return privateX;
	}
	public void setPrivateX(int privateX) {
		this.privateX = privateX;
	}
}

//CClass.java
public class CClass extends BClass{

	CClass(int x){
		super(x);
	}	
}

//ClassTest.java
public class ClassTest {

	public static void main(String[] args) {
		CClass var1 = new CClass(1);
		System.out.println(var1.getPrivateX()); // 1
		
		var1.publicX = 15;
		var1.setPrivateX(12);
		
		System.out.println(var1.getPrivateX()); // 12
		System.out.println(var1.publicX); //15
	}
}

```
## Method Overriding
Declaring a method in subclass which is already present in parent class is known as method overriding. The main advantage of method overriding is that the class can give its own specific implementation to a inherited method without even modifying the superclass.  

__Rules of method overriding in Java__

* Argument list: The argument list of overriding method must be same as that of the method in superclass. The data types of the arguments and their sequence should be maintained as it is in the overriding method.
* Access Modifier: The Access Modifier of the overriding method (method of subclass) cannot be more restrictive than the overridden method of superclass. 
* private, static and final methods cannot be overridden as they are local to the class. However static methods can be re-declared in the sub class, in this case the sub-class method would act differently and will have nothing to do with the same static method of parent class.
* If a class is extending an abstract class or implementing an interface then it has to override all the abstract methods unless the class itself is a abstract class.

__Super keyword in Overriding__  

super keyword is used for calling the parent class method/constructor. super.methodname() calling the specified method of base class while super() calls the constructor of base class.

```JAVA
/*
Method overriding and super keyword example
*/

//SuperClass.java
public class SuperClass {
	
	public void print(){
		System.out.println("SuperClass: print()");
	}
}

//SubClass.java
public class SubClass extends SuperClass{

	public void print(){
		System.out.println("SubClass: print()");
		super.print();
	}
}


//OverridingTest.java 
public class OverridingTest {

	public static void main(String[] args) {
		
		SubClass var1 = new SubClass();
		var1.print();
	}
}

```
