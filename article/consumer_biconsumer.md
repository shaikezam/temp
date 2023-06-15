# Consumer & BiConsumer Java 8 API
*21-01-2022 - Shai Zambrovski*

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

In this tutorial we will learn about the `Consumer` & `BiConsumer` `FunctionalInterface`.
## Consumer interface
A `FunctionalInterface` that can be used when creating `lambda expressions` or `method references` (Constructor references irrelevant as Consumer returns nothing) in which accepts an argument and returns nothing.

The `Consumer FunctionalInterface` has only one method named `accept()`, also we can chain multiple Consumers with the `andThen()` method.
#### Consumer by method references:

```java
public class Main {

    public static void main(String[] args) {
        Consumer<String> toUpperCaseConsumer = Program::print;
        toUpperCaseConsumer.accept("Hello shaikezam.com");
    }

    public static class Program {

        public static void print(String text) {
            System.out.println(text.toUpperCase());
        }
    }

}
```
#### Consumer by lambda expressions:
```java
public class Main {

    public static void main(String[] args) {
        Consumer<String> toUpperCaseConsumer = (text) -> System.out.print(text.toUpperCase());
        toUpperCaseConsumer.accept("Hello shaikezam.com");
    }

}
```
#### Chain Consumers with andThen() method:
```java
public class Main {

    public static void main(String[] args) {
        Consumer<TextContainer> toUpperCaseConsumer = textContainer -> {
            String text = textContainer.getText();
            textContainer.setText(text.toUpperCase());
        };

        Consumer<TextContainer> printConsumer = textContainer -> System.out.println(textContainer.getText());

        toUpperCaseConsumer
                .andThen(printConsumer)
                .accept(new TextContainer("Hello shaikezam.com"));
    }

    public static class TextContainer {

        private String text;

        public TextContainer(String text) { this.text = text; }

        public String getText() { return text; }

        public void setText(String text) { this.text = text; }
    }

}
```
For all above examples, the output will be `HELLO SHAIKEZAM.COM`.
## BiConsumer interface
The `BiConsumer` interface is similar to a `Consumer` interface and accepts two input parameters (`T` the 1st parameter and `U` the 2nd paramater) and does not return any result.
```java
public class Main {

    public static void main(String[] args) {
        BiConsumer<TextContainer, String> toUpperCaseBiConsumer = (textContainer, text) -> {
            textContainer.setText(text.toUpperCase());
        };

        TextContainer textContainer = new TextContainer();

        toUpperCaseBiConsumer.accept(textContainer, "Hello shaikezam.com");

        System.out.println(textContainer.getText());
    }

    public static class TextContainer {

        private String text;

        public String getText() { return text; }

        public void setText(String text) { this.text = text; }
    }

}
```
And once again, the output will be `HELLO SHAIKEZAM.COM`.