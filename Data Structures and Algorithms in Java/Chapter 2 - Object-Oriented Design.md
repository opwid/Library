# Inheritance
In object-oriented programming, the mechanism for a modular and hierarchical organization is a technique known as inheritance. This allows a new class to be defined based upon an existing class as the starting point. In object-oriented terminology, the existing class is typically described as the base class, parent class, or superclass, while the newly defined class is known as the subclass or child class. We say that the subclass extends the superclass.  

When inheritance is used, the subclass automatically inherits, as its starting point, all methods from the superclass (other than constructors). The subclass can differentiate itself from its superclass in two ways. It may augment the superclass by adding new fields and new methods. It may also specialize existing behaviors by providing a new implementation that overrides an existing method.  

The subclass inherits all the members and methods that are declared as public or protected. If declared as private it can not be inherited by the subclasses. The private members can be accessed only in its own class. The private members can be accessed through assessor methods as shown in the example below. The subclass cannot inherit a member of the superclass if the subclass declares another member with the same name.  

The constructor in the superclass is responsible for building the object of the superclass and the constructor of the subclass builds the object of subclass. When the subclass constructor is called during object creation, it by default invokes the default constructor of super-class. Hence, in inheritance the objects are constructed top-down. The superclass constructor can be called explicitly using the keyword super, but it should be first statement in a constructor. The keyword super always refers to the superclass immediately above of the calling class in the hierarchy. The use of multiple super keywords to access an ancestor class other than the direct parent is illegal.

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
