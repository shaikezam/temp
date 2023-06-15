# Supplier Java 8 API
*26-01-2022 - Shai Zambrovski*

------------
## Functional Interfaces
### Before Java 8
As an Object-Oriented Programming language, `Java` has declared almost everthing in the `Object` model (except for some of the primitive data types and primitive methods).

Functions in Java were only a part of a class and to use them we need to use the `Class` or to create an instance of it.
### From Java 8 onwards
Since `Java` 8, `Java` brought us the interface `FunctionalInterface`:

Conceptually, a functional interface has exactly one abstract method, however, they can include any quantity of default and static methods.

Note that instances of functional interfaces can be created with:
- Lambda expressions.
- Method references
- Constructor references.

The purpose of `FunctionalInterface` is to make code more readable, clean, and straightforward.

There are many types of `FunctionalInterface`; `Runnable`, `Callable` and so on.

In this tutorial we will learn about the `Supplier` `FunctionalInterface`.
## Supplier interface
A `FunctionalInterface` that can be used when creating `lambda expressions`, `method references` or `constructor references`, in which accepts no arguments and returns a result.

The `Supplier FunctionalInterface` has only one method named `get() `and it isn't chainable
#### Supplier by method references:
```java
public class Main {

    public static void main(String[] args) {
        Supplier<String> text = Main::getText;

        System.out.println(text.get());
    }

    public static String getText() {
        return "Hello shaikezam.com";
    }

}
```
#### Supplier by lambda expression:
```java
public static void main(String[] args) {
    Supplier<String> text = () -> "Hello shaikezam.com";

    System.out.println(text.get());
}
```
#### Supplier by constructor references:
```java
public class Main {

    public static void main(String[] args) {
        Supplier<TextContainer> text = TextContainer::new;

        System.out.println(text.get());
    }

    public static class TextContainer {

        private String text;

        public TextContainer() { text = "Hello shaikezam.com"; }

        @Override
        public String toString() { return text; }
    }

}
```
For all above examples, the output will be `Hello shaikezam.com`.