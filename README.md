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

# Why would you use a final class when its methods cannot be overridden?
For security, immutabilty. 

public final class Utility {
    public static int add(int a, int b) {
        return a + b;
    }
}
This Utility class is meant to be used as a helper class, and preventing inheritance ensures that its behavior remains predictable. All methods of a final class are final by default. 

# Can changes be made to a final or immutable class using reflection in Java?
You can use Java Reflection (java.lang.reflect) to modify private fields, even if they are final as well as static final.
This is possible because reflection allows you to override access restrictions.

import java.lang.reflect.Field;
import java.lang.reflect.Modifier;

import java.lang.reflect.Field;

class Constants {
    public static final String MESSAGE = "Hello, World!";
}

public class ModifyFinalStatic {
    public static void main(String[] args) throws Exception {
        Field field = Constants.class.getDeclaredField("MESSAGE");
        field.setAccessible(true);

        // Remove the 'final' modifier
        Field modifiersField = Field.class.getDeclaredField("modifiers");
        modifiersField.setAccessible(true);
        modifiersField.setInt(field, field.getModifiers() & ~Modifier.FINAL);

        // Change the constant
        field.set(null, "Hacked!");

        // Verify the change
        System.out.println(Constants.MESSAGE);  // Output: Hacked!
    }
}

# Can We Modify a Final Class?
We cannot extend a final class, but we can modify its behavior using reflection.
We cannot remove the final modifier from a class, but we can change fields dynamically.

# why reflection was introduced in java
1. Runtime Inspection of Classes and Objects
Example: Listing all methods of a class at runtime:

import java.lang.reflect.Method;

public class ReflectionExample {
    public static void main(String[] args) {
        Class<?> clazz = String.class; // Get metadata of String class

        // Print all method names in String class
        for (Method method : clazz.getDeclaredMethods()) {
            System.out.println(method.getName());
        }
    }
}

2. Accessing Private Members for Testing, Serialization
3. Dynamic Object Creation (No Hardcoded Class Name) eg: Used in dependency injection (spring), serialization (java)
public class DynamicInstance {
    public static void main(String[] args) throws Exception {
        Class<?> clazz = Class.forName("java.lang.String");
        Object obj = clazz.getConstructor().newInstance(); // Create instance dynamically
        System.out.println("Created object: " + obj.getClass().getName());
    }
}
4. ORM (Hibernate, JPA) → Map Java objects to database tables without writing explicit SQL queries.
   Reflection in Spring for dependency injection
   public class A{
      public void doSomething(){}
   }

   public class B{
      private A a;
      setA(A a){
         this.a = a;
      }

      public void doWork(){
         a.doSomething();
      }
   }

   public class SpringMain{
      public static void main(String arg[]){
         Class<?> aClazz = Class.forName("A");
         Object aInstance = aClazz.getDeclaredConstructor().newInstance(); //dynamically created A class instance
         Class<?> bClazz = Class.forName("B");
         Object bInstance = bClazz.getDeclaredConstructor().newInstance();

         Method setter = bClazz.getMethod("setA", A.class);
         setter.invoke(bInstance, aInstance);

         Method doWork = bClazz.getMethod("doWork");
         doWork.invoke(bInstance);
      }
   }

# Implement your jackson like serializer and deserializer

# What is the purpose of the "rt.jar" file in Java?
rt.jar (short for "runtime.jar") is a core Java library file that contains all the built-in Java classes required to run Java applications.

# Explain the significance of the Manifest file in Java.
MANIFEST.MF is a special metadata file inside JAR which provides config details about jar's content.
Main-Class ::::::::::::::	Defines the entry point (for running JARs)
Class-Path ::::::::::::::	Specifies dependencies
Implementation-Version ::	Provides version info
Sealed ::::::::::::::::::	Ensures package integrity
Signature :::::::::::::::	Enables secure JARs

