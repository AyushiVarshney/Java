# JVM Components
```java
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

```
# Key Features introduced in Java 11
String methods like isBlank, strip(). Allowed var for lamba expression parameters. HttpClientAPI

# Can the "main()"/static method be overridden 
No, static method can not be overridden since they do not belong to instance. Static always belong to class. If we override static methods that become method hiding.

```Java
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
```

# Can the "main" method be declared as "final" in Java
Yes, we can but this is not of any use because static method anyways can not be overridden.

# Explain the concept of object cloning in Java.
Creation of exact copy of object. Class must implement Cloneable interface otherwise CloneNotSupportedException. By default shallow copy of Object is created.

```Java
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
```

# If the "compareTo()" method is implemented in a class, what is the need to implement the "equals" method as well?
You need compareTo() for ordering. You need equals() (and hashCode()) for equality checks, especially with hash-based collections.  You often need both, especially when using sorted collections. equals() method in Object class by deafult checks for reference eqiality rather than contens.

# Why would you use a final class when its methods cannot be overridden?
For security, immutabilty. 

```Java
public final class Utility {
    public static int add(int a, int b) {
        return a + b;
    }
}
```

This Utility class is meant to be used as a helper class, and preventing inheritance ensures that its behavior remains predictable. All methods of a final class are final by default. 

# Can changes be made to a final or immutable class using reflection in Java?
You can use Java Reflection (java.lang.reflect) to modify private fields, even if they are final as well as static final.
This is possible because reflection allows you to override access restrictions.

```Java
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
```
# Can We Modify a Final Class?
We cannot extend a final class, but we can modify its behavior using reflection.
We cannot remove the final modifier from a class, but we can change fields dynamically.

# why reflection was introduced in java
1. Runtime Inspection of Classes and Objects
Example: Listing all methods of a class at runtime:

```Java
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
```

2. Accessing Private Members for Testing, Serialization
3. Dynamic Object Creation (No Hardcoded Class Name) eg: Used in dependency injection (spring), serialization (java)
   
public class DynamicInstance {
    public static void main(String[] args) throws Exception {
        Class<?> clazz = Class.forName("java.lang.String");
        Object obj = clazz.getConstructor().newInstance(); // Create instance dynamically
        System.out.println("Created object: " + obj.getClass().getName());
    }
}

5. ORM (Hibernate, JPA) → Map Java objects to database tables without writing explicit SQL queries.
   
   Reflection in Spring for dependency injection
   
```Java
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
```
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

# What is an Anonymous Object in Java
is created without assigning it to a reference variable. It is usually used when an object is needed only once.
new Car().drive(); 

# What is a covariant return type in method overriding, and does it support overloading as well?
No for Overloaing as overloading is based on method name and parameters not on return type
```Java
public class Parent{
   Parent callMe(){}
   int printMe(){}
}

public class Chind extends Parent{
   Child callMe(){} //here callMe method is being overriden and it can return subtype (covariant return type)
   double printMe(){} //this is not allowed. For overriding return should also be either same or subtype
}
```
# What are the advantages and disadvantages of using log4j for logging in Java?
Highly configurable (provides confg file), multiple appenders, thread-safe. Provides diff log levels.
Can slow your app if not config properly (like file based logging)
example of log4j.xml file
```Java
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Properties>
        <Property name="LOG_PATTERN">%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %logger{36} - %msg%n</Property>
        <Property name="LOG_FILE">logs/app.log</Property>
    </Properties>

    <Appenders>
        <!-- Console Appender (Logs to Console) -->
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="${LOG_PATTERN}" />
        </Console>

        <!-- File Appender (Logs to a File) -->
        <RollingFile name="FileLogger" fileName="${LOG_FILE}"
                     filePattern="logs/app-%d{yyyy-MM-dd}.log.gz">
            <PatternLayout>
                <Pattern>${LOG_PATTERN}</Pattern>
            </PatternLayout>
            <Policies>
                <TimeBasedTriggeringPolicy />
            </Policies>
        </RollingFile>
    </Appenders>

    <Loggers>
        <!-- Set logging levels for specific packages -->
        <Logger name="com.example" level="DEBUG" additivity="false">
            <AppenderRef ref="FileLogger" />
        </Logger>

        <!-- Root Logger (Default Logging Behavior) -->
        <Root level="INFO">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="FileLogger"/>
        </Root>
    </Loggers>
</Configuration>
```
# what is the difference between dependency and dependencymanagement
dependency tag will immedietly loads the files to classpath. And if we add any dependency in dependencymanagement tag then we dont need to define version of those dependencies in child module hence, all modules will use same versions of those dependencies.

# In what situations would you use an inner class in Java
In what situations would you use an inner class in Java and when we have tight coupling example Car has Engine so Engine could be inner class for Car.

# What are the access modifiers available in Java?
private: Accessible within class (Not applicate for top level classes as we can only have default/public outer classes)
default: Accessible within package
protected: Accessible within package and subclasses in different package (Not applicate for top level classes)
public: Accessible from everywhere

# what are some common annotation used in Jackson Serialization
@JsonIgnore: That field wont be serialized
@JsonProperty: To change field name
@JsonFormat: to specify date format

# what are some common annotation used in Jackson DeSerialization

# How do you serialize immutable objects.
By defualt no arg constructor is required but we can create custom also if there is no default constructor in class which you are serializing, for immutable objects we need @JsonCreator on the constructor which we have createed and @JsonProperty on parameters passed in contructors.

# What is an Annotation in Java, and how do you create a custom annotation?
Annotations provide metadata that provides extra info about code behaviour.
```Java
@Retention(RetentionPolicy.RUNTIME)  // Available at runtime for reflection
@Target(ElementType.METHOD)          // Can only be applied to methods
@interface LogExecutionTime {
    String value() default "Execution Time Logger";  // Annotation element with default value
}
```
# What is the meaning of the Serial Version UID in Java Serialization?
Allows backward compatibily: If we dont provide seraialId then automatically in id is generated based on class. and if later we do any changes to this class then JVM will generate diff serialId then we will get InvaliCLassException on deserializing.

# what is serialization vs externalization?
Serialization is process of converting objects to bytestream (Done by java) slower due to Reflection overhead.
Externalization give more control what we want to serialize (implment Externalizable interface and override readExternal and writeExternal in which we define what we want to serialize) Example :
```java
public class Employee implements Externalizable {
   int id;
    String name;
    int age;

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeUTF(name);
        out.writeInt(id);
        //not serializing age
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        id = in.readInt();
        name = in.readUTF();
    }
}
```

# Explain the difference between the "path" and "classpath" variables in Java.
path: is used to find java executables like java, javac
classpath: is used for locating .class files and jars

# JVM memory
```Java
├── Method Area (Class Metadata, Static Variables, Constants)
├── Heap Memory (Objects, Instance Variables)
├── Stack Memory (Method Calls, Local Variables, References)
├── PC Register (Current Instruction Address)
└── Native Method Stack (C/C++ Code Execution)
```

# How is Exception Handling implemented in inheritance in Java, and what are the rules associated with it?
Method in subclass should not throw broader checked exception. However, unchecked exceptions can be thrown freely.
```Java
class Parent {
    void show() throws IOException { // Checked Exception (IOException)
        System.out.println("Parent method");
    }
}

class Child extends Parent {
    @Override
    void show() throws Exception { // ❌ Broader Exception (Exception is parent of IOException)
        System.out.println("Child method");
    }

    @Override
    void show() throws java.io.FileNotFoundException { // ✅ Narrower Exception (Subclass of IOException)
        System.out.println("Child method");
    }
}
```
# Define Data Hiding in the context of inheritance in Java.
In inheritance, data hiding means that a subclass does not inherit private fields from its parent class, even though it inherits the methods that operate on those fields.

# What is IS-A and HAS-A (Association, Aggregation and Composition) in Object Oriented Programming.
IS-A : IS-A relationship is created in parent child relationship. Class A extends B
Association (has-a): is created when two objects can exist independenty. example Student and Teacher
```Java
class Student{
   private String name;
   Student(){}
}

class Teacher{
   private String name;
   Teacher(){}
   public void teaches(Student student){} //Teacher teaches Student but both teacher and student can exist independently.
}

Aggregation(has-a) (Dependency Injection): is created when parent contains child, and child can exist separately

class ServiceLayer{
   private RepoLayer repoLayer; //S
   
   @Autowired
   public ServiceLayer(RepoLayer repoLayer){ 
      this.repoLayer = repoLayer;
   }
}

class RepoLayer{
   public RepoLayer(){} // RepoLayer can exist independently
}

Composition(part-of): is created when child can not exist separately. when parent is destroyed, child is also destroyed

class Engine{
   public Engine(){}
}

class Car{
   private Engine engine;
   public Car(){
      engine = new Engine(); // Engine is created inside Car
   }
}
```
# Internal Working of TreeSet
Treemap uses Red black tree which is self balancing binary search tree. The tree performs binary search to find coorect position for element being inserted or deleted.
No null values are allowed in TreeSet. Uses compareTo method for natural Ordreing of elements.

Insertion O(logn)
Deletion O(logn)
Search O(logn)

TreeSet<Integer> set = new TreeSet<>(); // 1 2 3 4
TreeSet<Integer> set = new TreeSet<>(Comparator.reverseOrder());//4 3 2 1

# Ways to call Functional Interface abstract method
```Java
interface MyInterface{
 public void print();
}
1. Using Concrete class and providing implementation (Normal)
2. Using Anonymous Class
3. Using Lambda
4. Using Method Referencing

class Main{
   public static void main(String arg[]){
      MyInterface my = new MyInterface({
         @Override
         public void print(){
            System.out.println("hello");
         }
      });
      my.print(); //anonymous class

      MyInterface my = () -> System.out.println("hello");
      my.print(); //using lambda *Note : Lambda can only be used if Interface is having only one abstract method.

      public static void print(){ //for static methods
         System.out.println("hello");
      }

      MyInterface my = Main::print; //using method referencing. **Note: we have assigned a static method to functional Interface which is allowed in method referencing.

      my.print();

      //If print method implementation is in another class then we can method referencing with instance vriable
      //example print method is in Test Class and object name is test
      MyInterface my = test::print;
      my.print();
   }
}
```
# Streams groupingBy and partitioningBy
partitioningBy is a special case of grouping by which splits the collection into two list false and true based on condition and returns Map<Boolean, List<T>>
```Java
Feature      	           groupingBy()	                              partitioningBy()
Key Type	                Any K (String, Integer, Enum, etc.)	       Always Boolean
Output Map Type	         Map<K, List<T>>	                           Map<Boolean, List<T>>
Number of Groups	        Multiple	                                  Exactly Two
Use Case	                Categorizing data into many groups	        Splitting into true/false groups

class Employee {
   String name;
   int salary;
   int experience;
}
//Usecase about partitioningBy: Find two lists of EMployees having salary larger than 25000 and other list having smaller salaries

class PartitioningBy {
   Map<Boolean, List<Employee>> map = list.stream().collect(Collectors.partitioningBy(l->l.salary() > 25000));
   
   //Result in map : Key     Value
                   //true    Employee List having salary greater than 25000
                   //false   Employee list having salary less than equal to 25000
   
   //Suppose if we want only Employee names instaed of Employee
   
   Map<Boolean, List<String>> map = list.stream().collect(Collectors.partitioningBy(l -> l.salary() > 25000, 
      Collectors.mapping(l->l.name(), Collectors.toList())));
   
   //Suppose we want count of Employees
   
   Map<Boolean, Long> map = list.stream().collect(Collectors.partitioningBy( l -> l.salary() > 25000, Collectors.counting())); 
}

//Use Case : if we want to categorize in more than two sets - Find Emplyoyess having 0 to 3 years of exp, 4 to 6 years of exp and above t years

class GroupingBy {
   Map<String, List<Employee>> map = list.stream().collect(Collectors.groupingBy(l -> {
      if(l.experience <= 3)
         return "Entry Level";
      else if(&& l.experience <= 6)
         return "Mid Level";
      else
         return "Senior Level";
   }));

   //Result in map : Key              Value
                   //Entry Level      Employee List having experience less than equal to 3 years
                   //Mid Level        Employee list having salary less than equal to 6 years
                   //Senior Level     Employee list having salary greater than 6 years
   
}
```


## Multithreading
# What is volatile keyword
Used to ensure visibility of variable when updated accross multiple threads. Thread cache the variables for performance which can lead to stale data issue. voltile makes sure that variable value is always read by main memory

```java
class SharedResource {
    static boolean flag = false;

    public static void main(String[] args) {
        Thread reader = new Thread(() -> {
            while (!flag) {  // May read stale value
                // Looping infinitely due to caching
            }
            System.out.println("Flag changed!");
        });

        Thread writer = new Thread(() -> {
            try { Thread.sleep(1000); } catch (InterruptedException e) {}
            flag = true;  // Other thread may not see this update
            System.out.println("Flag updated!");
        });

        reader.start();
        writer.start();
    }
}
```



## Hibernate
# JPA vs Hibernate
JPA is abstraction interface while Hibernate is implementation of JPA. Eg save method is abstract in JPA and hibernate will have actual code of this method

# How to create Primary Key in JPA
using @Id and @GenerateValue() if we want auto increamenting otherwise just use @Id

```java
   if @GenerateValue(strategy = GenerationType.AUTO) then strategy will be decides based on database. eg for SQL auto increment and sequence generator for Oracle
      @GenerateValue(strategy = GenerationType.IDENTITY) for mysql autoincrement
      @GenerateValue(strategy = GenerationType.SEQUENCE) Hibernate automatically creates a sequence named hibernate_sequence, The default allocationSize = 50, meaning 
                                                         Hibernate will fetch 50 IDs at a time from the sequence. There could be gaps in IDs but performance optimized
      @GenerateValue(strategy = GenerationType.SEQUENCE) if used with sequencegenerator then employee_sequence will be auto created if spring.jpa.hibernate.ddl-auto=update

      @Entity
      @SequenceGenerator(name = "emp_seq", sequenceName = "employee_sequence", allocationSize = 1)
      public class Employee {
         @Id
         @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "emp_seq")
         private Long id;
      }

//we can also create using UUID for distributed system but it has 32 long key and slow
```java
   @Entity
   public class Employee {
      @Id
      @GeneratedValue(generator = "UUID")
      @GenericGenerator(name = "UUID", strategy = "org.hibernate.id.UUIDGenerator")
      private String id;
   }
```


# How to make composite primary key
Using @EmbeddedId

```java
@Embeddable
public class EmployeePK implements Serializable {
    private Long empId;
    private String department;

    // Override equals() and hashCode()
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        EmployeePK that = (EmployeePK) obj;
        return Objects.equals(empId, that.empId) && Objects.equals(department, that.department);
    }

    @Override
    public int hashCode() {
        return Objects.hash(empId, department);
    }
}
then use @EmbeddedId in Entity
@Entity
public class Employee {
    @EmbeddedId
    private EmployeePK id;

    private String name;
}
```

# Inheritence in ORM
```java
   Strategy                  Pros                	                  Cons	                                          Best Use Case
   Single Table              Fastest, simple	                       Wastes space(NULL columns)	                    Most cases unless too many unused columns
   Table per Class	          No NULLs	                              Duplicates common fields, harder queries	      Rarely used, only if all child entities are very different
   Joined Table	             Best normalization, avoids NULLs	      Requires JOINs (slower queries)	               Large apps with complex relationships
🚀 Recommendation:
Use Single Table unless you need a strict normalized structure.
Use Joined Table for large projects where normalization is important.

@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "EMP_TYPE", discriminatorType = DiscriminatorType.STRING)
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}

@Entity
@DiscriminatorValue("MANAGER")
public class Manager extends Employee {
    private int teamSize;
}

@Entity
@DiscriminatorValue("ENGINEER")
public class Engineer extends Employee {
    private String techStack;
}
```

# @JoinColumn

@JoinColumn is used to define the foreign key column in an entity.
✔ Helps in renaming the column and setting constraints (nullable, unique, etc.).
✔ Without @JoinColumn, JPA assigns a default name like {entity_name}_id.
✔ Used in One-to-One and Many-to-One relationships.

# find employee having salary equal to employee salary mark
WITH mark_salary AS (SELECT salary FROM Employee WHERE name = 'Mark' LIMIT 1)
SELECT name FROM Employee e INNER JOIN mark_salary m ON e.salary = m.salary;

select name from employee where salary = (select salary from employee where name = 'mark' limit 1);

# What changes you have to when migrating from mysql to oracle
1. Change mysql connector in pom to ojdbc driver
2. change application.properties config (url, username, password, dialect and driver)
3. check schema if tables has autoincrement. auto increment need to change to sequence and trigger(if not using hibernate) in oracle
4. change limit to pagination

# Connection pooling vs Thread pooling
```java
// In connection pooling new connection is created everytime to execute query. SPringboot uses default HikariCP to manage connection pool
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=admin
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.hikari.maximum-pool-size=10  # Max 10 connections in the pool (if its not defined default value 10 will be used)

//In thread pooling, a fixed number of threads are created on startup instead of creating new thread everytime
ExcetorService service = Excetors.newFixedThreadPool(10);
```
