# Singleton in Java
*18-01-2022 - Shai Zambrovski*

------------
## What is it Singleton design pattern?
Singleton is a `creational design pattern` in which restricts the creation of an object to only one instance.

In most implementations, a hidden constructor - declared `private` - ensures that the class can never be instantiated from outside the class.

We will use the singleton design pattern  when it’s not necessary to have many instances of the same object, spending memory to do something that a single instance can do.

This tutorial is not related to the `Spring` bean deafult scope - singleton (in which it's singleton per application context).
## Eager Singleton classs initialization
The simple singleton java implementation.

The object's instance is created while the class is loading, which leads to some drawbacks:
- If we will not use the Object, it still will be created even if the application will not use it which can leads to waste of resources.
- There isn't really error handling or any exceptions handling in this approach.

```java
public class FileSystemEagerSingleton {
    private static final FileSystemEagerSingleton instance = new FileSystemEagerSingleton();

    //private  - ensures that the class can never be instantiated from outside the class.
    private FileSystemEagerSingleton() {
    }

    public static FileSystemEagerSingleton getInstance() {
        return instance;
    }
}
```
## Singleton classs initialization through static block
Similar to the above implementation except that it can have an exception handling (be aware that it also created even if the application will not use it, i.e. it will create in the class loading).

```java
public class FileSystemStaticSingleton {
    private static FileSystemStaticSingleton instance;

    //private  - ensures that the class can never be instantiated from outside the class.
    private FileSystemStaticSingleton() {
    }

    static {
        try {
            instance = new FileSystemStaticSingleton();
        } catch (Exception ex) {
            // error handling
        }
    }

    public static FileSystemStaticSingleton getInstance() {
        return instance;
    }
}
```
## Lazy Singleton classs initialization
In this approce, we now can handle the exceptions handling and the Object's instance will be create when we will call in the 1st time we call to the `getInstance() `method.

This will work great in a `single-thread` environment, when we will run in `multi-thread` environment, we will have some issue that the Object's instance can be created twice (or more) if multiple threads will be inside the `if` statement.

That's means that the singleton approach will be broken.
```java
public class FileSystemLazySingleton {
    private static FileSystemLazySingleton instance;

    //private  - ensures that the class can never be instantiated from outside the class.
    private FileSystemLazySingleton() {
    }

    public static FileSystemLazySingleton getInstance() {
        if (instance == null) {
            instance = new FileSystemLazySingleton();
        }
        return instance;
    }
}
```
## Thread safe Singleton classs initialization
We can wrap the `getInstance()` method with the `synchronized` keyword, to enable it to be thread safe (only 1 thread can execute this method).

The implementation will work great but with one disadvantage:

Because only 1 thread can access the `getInstance()` method we have created a performance issue in here, in case a lot of threads want to get the Object's instance, they will have to wait for no reasone as the Object already has been created already.
```java
public class FileSystemThreadSafeSingleton {
    private static FileSystemThreadSafeSingleton instance;

    //private  - ensures that the class can never be instantiated from outside the class.
    private FileSystemThreadSafeSingleton() {
    }

    public static synchronized FileSystemThreadSafeSingleton getInstance() {
        if (instance == null) {
            instance = new FileSystemThreadSafeSingleton();
        }
        return instance;
    }
}
```
## Double Checked Thread safe Singleton classs initialization
The solution for this will be Double Check Singleton.

We will set the thread safe mechanism only in the 2nd if statement, so anyone who need the Object's instance (and it already been created) do not need to wait.

```java
public class FileSystemDoubleCheckedSingleton {
    private static FileSystemDoubleCheckedSingleton instance;

    //private  - ensures that the class can never be instantiated from outside the class.
    private FileSystemDoubleCheckedSingleton() {
    }

    public static FileSystemDoubleCheckedSingleton getInstance() {
        if (instance == null) {
            synchronized (FileSystemDoubleCheckedSingleton.class) {
                if (instance == null) {
                    instance = new FileSystemDoubleCheckedSingleton();
                }
            }
        }
        return instance;
    }
}
```
## The Reflection problem
Well, it does not really a problem, but all above solutions are not relevant in case we create another instance via the reflection's `getDeclaredConstructors` API and set it's accessible to `true`.

In order to solve it, we can create singleton using `Enum`.
```java
public enum FileSystemEnumSingleton {

    INSTANCE;

    public static void writeToFile(final String fileName, final String data) throws IOException {
        try (Writer writer = Files.newBufferedWriter(Paths.get(fileName))) {
            writer.write(data);
        }
    }
}
```


