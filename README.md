# JVM Components
+-----------------+     +-----------------+               +-----------------+     +-----------------+
|   Classloader   | --> | Bytecode Verifier| -->          |   Memory Area   | --> | Execution Engine |
+-----------------+     +-----------------+               +-----------------+     +-----------------+
 loads .class files     crucial for class loading          |      Heap           |     Responsible for actual execution of code
into JVM memory         linking process. performs          |     Method Area     |       +----------------------------------+ 
                        checkes like compatible datatypes  |      Stack          |       | Interpretor (Executes bytecode   |
                        flow control, operand stack checks,|   PC Registers      |       | instructions one by one) - Slower|
                         potential problems that could     |  Native Method Stack|       +----------------------------------+
                         cause java security               | +-----------------+ |       +----------------------------------+
                                                                                         | Just In Time Compiler (compiles
                                                                                         | frequently executed bytecode to 
                                                                                         | machine code. faster. Ex:
                                                                                         | for(int i=0;i<1000;i++) callme();
                                                                                         | here callMe method is executed freq         
                                                                                         | JVM will identify as hotspot and 
                                                                                         | JIT will convert it to native machine code
                                                                                         | and it will be cached
                                                                                         +-----------------------------------+
                                                                                         +------------------+
                                                                                         | Garbage          |
                                                                                         | COllector        |
                                                                                         +------------------+
# Key Features introduced in Java 11
String methods like isBlank, strip(). Allowed var for lamba expression parameters. HttpClientAPI

# Can the "main()"/static method be overridden 
No, static method can not be overridden since they do not belong to instance. Static always belong to class. If we override static methods that become method hiding.

class Parent {
    public static void myMethod() {
        System.out.println("Parent's static method");
    }
}

class Child extends Parent {
    public static void myMethod() { // This HIDES the Parent's myMethod()
        System.out.println("Child's static method");
    }
}

public class Main {
    public static void main(String[] args) {
        Parent p = new Parent();
        p.myMethod(); // Output: Parent's static method

        Child c = new Child();
        c.myMethod(); // Output: Child's static method

        Parent pc = new Child(); // Reference of Parent type, object of Child type
        pc.myMethod(); // Output: Parent's static method (Hiding, not Overriding)
    }
}

# Can the "main" method be declared as "final" in Java
Yes, we can but this is not of any use because static method anyways can not be overridden.

# Explain the concept of object cloning in Java.
Creation of exact copy of object. Class must implement Cloneable interface otherwise CloneNotSupportedException. By default shallow copy of Object is created.

class MyObject implements Cloneable {
    int x;
    OtherObject obj; // Reference to another object, in shallow copy ojb will share same memory

    public MyObject(int x, OtherObject obj) {
        this.x = x;
        this.obj = obj;
    }

    @Override
    public MyObject clone() throws CloneNotSupportedException {
        return (MyObject) super.clone(); // Shallow copy by default
    }

    //For deep copy
    @Override
    public MyObject clone() throws CloneNotSupportedException {
        MyObject cloned = (MyObject) super.clone(); // Still need the shallow copy

        // Now, create deep copies of the referenced objects:
        cloned.obj = new OtherObject(this.obj.name); // Create a new OtherObject

        return cloned; // Deep copy
    }
}

class OtherObject {
    String name;

    public OtherObject(String name) {
        this.name = name;
    }
}

public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        OtherObject o = new OtherObject("Original");
        MyObject obj1 = new MyObject(10, o);

        MyObject obj2 = obj1.clone(); // Shallow copy

        obj2.x = 20; // Modifies the cloned object's primitive variable
        obj2.obj.name = "Modified"; // Modifies the shared object

        System.out.println(obj1.x); // Output: 10
        System.out.println(obj1.obj.name); // Output: Modified (Shared!)

        System.out.println(obj2.x); // Output: 20
        System.out.println(obj2.obj.name); // Output: Modified (Shared!)
    }
}

# If the "compareTo()" method is implemented in a class, what is the need to implement the "equals" method as well?
You need compareTo() for ordering. You need equals() (and hashCode()) for equality checks, especially with hash-based collections.  You often need both, especially when using sorted collections. equals() method in Object class by deafult checks for reference eqiality rather than contens.

