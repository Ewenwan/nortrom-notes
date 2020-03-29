
[toc]

[programming](./prog.md)

# JVM

* java virtual machine notes
    * ***BYTECODE***
    * The bytecodes are executed in an execution engine, which is one part of the virtual machine that can vary in different implementations. On a Java Virtual Machine implemented in software, **the simplest kind of execution engine just interprets the bytecodes one at a time**. Another kind of execution engine, one that is faster but requires more memory, is a **just-in-time compiler**. In this scheme, **the bytecodes of a method are compiled to native machine code the first time the method is invoked**. The native machine code for the method is then cached, so it can be re-used the next time that same method is invoked.
    * ***JAVA INTERPRETER***
    * Sometimes the Java Virtual Machine is called the Java interpreter; however, given the various ways in which bytecodes can be executed, this term can be misleading. While "Java interpreter" is a reasonable name for a Java Virtual Machine that interprets bytecodes, virtual machines also use other techniques (such as just-in-time compiling) to execute bytecodes. Therefore, **although all Java interpreters are Java Virtual Machines, not all Java Virtual Machines are Java interpreters**.
    * ***CLASS LOADER***
    * A Java application can use two types of class loaders: a **"primordial" class loader** and **class loader objects**. The primordial class loader (there is only one of them) is a part of the Java Virtual Machine implementation. For example, if a Java Virtual Machine is implemented as a C program on top of an existing operating system, then the primordial class loader will be part of that C program. **The primordial class loader loads trusted classes, including the classes of the Java API**, usually from the local disk. At run-time, a Java application can install class loader objects that load classes in custom ways, such as by downloading class files across a network. The Java Virtual Machine considers **any class it loads through the primordial class loader to be trusted**, regardless of whether or not the class is part of the Java API. Classes it loads through class loader objects, however, it views with suspicion--by default, it considers them to be untrusted. While the primordial class loader is an intrinsic part of the virtual machine implementation, class loader objects are not. Instead, **class loader objects are written in Java, compiled to class files, loaded into the virtual machine, and instantiated just like any other object**. They are really just another part of the executable code of a running Java application.
    * Because of class loader objects, you donít have to know at compile-time all the classes that may ultimately take part in a running Java application. They enable you to dynamically extend a Java application at run-time.
    * When a loaded class first refers to another class, the virtual machine requests the referenced class from the same class loader that originally loaded the referencing class.
    * Because the Java Virtual Machine takes this approach to loading classes, **classes can by default only see other classes that were loaded by the same class loader**. This is how Javaís architecture enables you to create multiple name-spaces inside a single Java application. A Java application can instantiate multiple class loader objects either from the same class or from multiple classes. It can, therefore, create as many (and as many different kinds of) class loader objects as it needs. Classes loaded by different class loaders are in different name-spaces and cannot gain access to each other unless the application explicitly allows it. When you write a Java application, you can segregate classes loaded from different sources into different name-spaces. In this way, you can use Javaís class loader architecture to control any interaction between code loaded from different sources. You can prevent hostile code from gaining access to and subverting friendly code.
* questions of JVM
    * [Is there one JVM per Java application?](https://stackoverflow.com/questions/5947207/is-there-one-jvm-per-java-application)
        * Yes, generally speaking, each application will get its own JVM instance and its own OS-level process and each JVM instance is independent of each other.
        * There are some implementation details such as Class Data Sharing, where multiple JVM instances might share some data/memory but those have no user-visible effect to the applications
* 参考
    * [深入探讨 Java 类加载器](https://www.ibm.com/developerworks/cn/java/j-lo-classloader/index.html)

# JAVA semantics

* loading a class
    * `Class.forName("SomeClass");`
        * Returns the Class object associated with the class or interface with the given string name.
        * use the class loader that loaded the class which calls this code
    	* initialize the class (that is, all static initializers will be run)
    * 参考
        * [How to load a jar file at runtime](https://stackoverflow.com/questions/194698/how-to-load-a-jar-file-at-runtime)
        * [Class.forName() vs ClassLoader.loadClass() - which to use for dynamic loading?](https://stackoverflow.com/questions/8100376/class-forname-vs-classloader-loadclass-which-to-use-for-dynamic-loading)
    * method and field
        * `getName()`         返回类的名字
        * `getPackage()`      返回类所在的包
        * `getFields()`       返回所有的public数据成员
        * `getMethods()`      返回所有的public方法
        * `newInstance()`     调用默认的不含参数的构建方法。
* 内部类
    * 内部类又称为嵌套类，可以把内部类理解为外部类的一个普通成员。
    * 内部类访问外部类：内部类可以自由访问外部类成员
    * 外部类访问内部类：因为将内部类理解为外部类的一个普通成员，所以外面的访问里面的需先new一个内部类对象，再去获取成员
* static
    * static method
        * [when to use](https://stackoverflow.com/questions/2671496/java-when-to-use-static-methods): One rule-of-thumb: ask yourself "does it make sense to call this method, even if no Obj has been constructed yet?" If so, it should definitely be static.
    * [static class](https://stackoverflow.com/questions/7486012/static-classes-in-java)
        * Java has static nested classes, but has **no way of making a top-level class static**, simulate a static class like this:
        * Declare your class `final` - Prevents extension of the class since extending a static class makes no sense
        * Make the constructor `private` - Prevents instantiation by client code as it makes no sense to instantiate a static class
        * Make all the members and functions of the class `static` - Since the class cannot be instantiated no instance methods can be called or instance fields accessed
* pass by value
    * [Java is always pass-by-value](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value). Unfortunately, they decided to call the location of an object a "reference". When we pass the value of an object, we are passing the reference to it. This is confusing to beginners. (**for object its something like pass by pointer**)
* final
    * final定义的变量可以看做一个常量，不能被改变；
    * final定义的方法不能被覆盖；(同时也是inline)
    * final定义的类不能被继承。
* reflection
    * 概念
        * 反射机制赋予了java更多的动态语言特性
        * JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。
        * Java反射机制主要提供了以下功能： 在运行时判断任意一个对象所属的类；在运行时构造任意一个类的对象；在运行时判断任意一个类所具有的成员变量和方法；在运行时调用任意一个对象的方法；生成动态代理。
    * 应用
        * [use reflection to create instance](https://docs.oracle.com/javase/tutorial/reflect/member/ctorInstance.html), There are four reflective methods for creating instances of classes
            * `java.lang.reflect.Constructor.newInstance()`
            * `Class.newInstance()`. 
                * **The former is preferred**:
                * Class.newInstance() can **only invoke the zero-argument constructor**, while Constructor.newInstance() may invoke any constructor, regardless of the number of parameters.
                * Class.newInstance() throws any exception thrown by the constructor, regardless of whether it is checked or unchecked. Constructor.newInstance() always wraps the thrown exception with an InvocationTargetException.
                * Class.newInstance() requires that the constructor be visible; Constructor.newInstance() may invoke private constructors under certain circumstances.
            * `Class.getConstructor()` then `Constructor.newInstance()`
                * [Java: newInstance of class that has no default constructor](https://stackoverflow.com/questions/3671649/java-newinstance-of-class-that-has-no-default-constructor)
            * `getDeclaredConstructors()`
                * `myObject.getClass().getDeclaredConstructors(types list).newInstance(args list);` see [Can I use Class.newInstance() with constructor arguments?](https://stackoverflow.com/questions/234600/can-i-use-class-newinstance-with-constructor-arguments)
        * tell types
            * how to tell a class is an Array type?
                * Android Class API `isArray()`
            * how to tell a class is an Primitive type?
                * Android Class API `isPrimitive()`
            * how to tell an object is an Wrapper Object?
                * compare the type with each wrapper type(Integer.Class & Boolean.Class....), `isWrapperType(String.class)`
                *  [Determining if an Object is of primitive type](https://stackoverflow.com/questions/709961/determining-if-an-object-is-of-primitive-type)
            * how determine a class implements an interface?
                * `YourInterface.class.isAssignableFrom(clazz)`
                * [Determine if a Class implements a interface in Java](https://stackoverflow.com/questions/12145185/determine-if-a-class-implements-a-interface-in-java)
            * how determine primitive or object of an array? like int[] Object[]
                * [Is an array a primitive type or an object](https://stackoverflow.com/questions/12806739/is-an-array-a-primitive-type-or-an-object-or-something-else-entirely)
                * `int[].class == yourarraycls`
        * get & convert types
            * how can I convert a Field to its corresponding Object?
                * use `Class#getDeclaredFields()` to get all declared fields of the class. use `Field#get()` to get the value.
                * [How to get the fields in an Object via reflection](https://stackoverflow.com/questions/2989560/how-to-get-the-fields-in-an-object-via-reflection)
            * what about inherited field?
                * [Access to private inherited fields via reflection in Java](https://stackoverflow.com/questions/3567372/access-to-private-inherited-fields-via-reflection-in-java)
                * `Field[] fs = b.getClass().getSuperclass().getDeclaredFields();`
                * `getSuperClass() != (java.lang.Object & null)`
            * what about static Field?
                * [Retrieve only static fields declared in Java class](https://stackoverflow.com/questions/3422390/retrieve-only-static-fields-declared-in-java-class)
                * `java.lang.reflect.Modifier.isStatic(field.getModifiers())`
            * how can I get Type from an object?
                * `Type t = obj.getClass().getGenericSuperclass();`
            * how can I convert a primitive object to its wrapper object?
                * `clazz = ClassUtils.primitiveToWrapper(clazz);`
            * how to get the component type of an array?
                * `array.getClass().getComponentType()`
            * how can I get the elements of an array?
                * [Iterating over arrays by reflection](https://stackoverflow.com/questions/2200399/iterating-over-arrays-by-reflection)
* reference
    * Weak references: a reference that isn't strong enough to force an object to remain in memory, will be **discarded at the next garbage collection cycle**.
    * Soft references: like a weak reference, but less eager to throw away the object to which it refers. The GC will always discard weak references when it can and **retain Soft References when it can**.
    * 参考
        * [Is there a practical use for weak references?](https://stackoverflow.com/questions/8790511/is-there-a-practical-use-for-weak-references)
        * [What's the difference between SoftReference and WeakReference in Java?](https://stackoverflow.com/questions/299659/whats-the-difference-between-softreference-and-weakreference-in-java)

# application

* file operation
    * FileOutputStream & FileInputStream and ObjectOutputStream & ObjectInputStream in Java
    * [How do I serialize an object and save it to a file in Android?](https://stackoverflow.com/questions/4118751/how-do-i-serialize-an-object-and-save-it-to-a-file-in-android)
* serialization
    * Parcel class in Android
    * `writeTypedArray` & `createTypedArray` for read & writing arrays
    * Parcel is faster than other Serializable class
    * [Parcelable vs Serializable](http://www.developerphil.com/parcelable-vs-serializable/)
    * [Read & writing arrays of Parcelable objects](https://stackoverflow.com/questions/10071502/read-writing-arrays-of-parcelable-objects)
    