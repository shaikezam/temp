# Function & BiFunction Java 8 API
*30-01-2022 - Shai Zambrovski*

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

In this tutorial we will learn about the `Function` & `BiFunction` `FunctionalInterface`.
## Function interface
A `FunctionalInterface` that can be used when creating `lambda expressions` or `method references`.

The `Function FunctionalInterface` has several methods:
#### functional method: `apply()` - Applies this function to the given argument.
```java
public class Main {

    public static void main(String[] args) {
        Function<Double, Double> doubleAndMultipleByTwo = (number) -> Math.pow(number, 2) * 2;

        System.out.println(doubleAndMultipleByTwo.apply(3.0)); //Will print 18.0 to the console.
    }
    
}
```
#### `compose()` - Returns a composed function that first applies the function to its input, and then applies this function to the result.
```java
public static void main(String[] args) {
    Function<Double, Double> doubleFunction = (number) -> Math.pow(number, 2);
    Function<Double, Double> multipleByTwoFunction = (number) -> number * 2;
    Function<Double, Double> doubleAndMultipleByTwo = doubleFunction.compose(multipleByTwoFunction);
  
    System.out.println(doubleAndMultipleByTwo.apply(3.0)); //Will print 36.0 to the console.
}
```
#### `andThen()` - Returns a composed function that first applies this function to its input, and then applies the function to the result.
```java
public static void main(String[] args) {
    Function<Double, Double> doubleFunction = (number) -> Math.pow(number, 2);
    Function<Double, Double> multipleByTwoFunction = (number) -> number * 2;
    Function<Double, Double> doubleAndMultipleByTwo = doubleFunction.andThen(multipleByTwoFunction);

    System.out.println(doubleAndMultipleByTwo.apply(3.0)); //Will print 18.0 to the console.
}
```
### `static identity()` - Returns a function that always returns its input argument.
```java
public static void main(String[] args) {
    List<User> users = List.of(new User("shaikezam"), new User("koo"));
    Map<UUID, User> mapUserToUUID = users
        .stream()
        .collect(Collectors.toMap(User::getId, Function.identity()));

    System.out.println(mapUserToUUID);
	// {e514294e-3ed5-4173-b6f5-b2aca1f55400={id=e514294e-3ed5-4173-b6f5-b2aca1f55400, name='shaikezam'}, 76769999-c2a2-4ca0-9675-d23109cc4e5a={id=76769999-c2a2-4ca0-9675-d23109cc4e5a, name='koo'}}
}

public static class User {
    private UUID id = UUID.randomUUID();
    private String name;

    public User(String name) { this.name = name; }

    public UUID getId() { return id; }

    @Override
    public String toString() {
        return "{" +
            "id=" + id +
            ", name='" + name + '\'' +
            '}';
    }
}
```
In case we want to create a function that get, or return, or even both `primitive` types, we can use a pre defined function such as:
- `IntFunction` - input is `int` argument and the output type is parameterized.
- `ToIntFunction` - input is parameterized argument and the output type is an int.
- `DoubleToIntFunction` - both input & output are primitive types.

There are functions for almost all types.
## BiFunction interface
Similar to `Function`, a `FunctionalInterface` that's represents a function that accepts two arguments and produces a result. This is the two-arity specialization of Function.

The BiFunction interface expose two function:
#### functional method: `apply()` - Applies this function to the given arguments.
```java
public class Main {

    public static void main(String[] args) {
        BiFunction<Integer, Integer, Double> multipleNumbersAndPowerByTwoBiFunction = (num1, num2) -> {
            int multipleResult = num1 * num2;
            return Math.pow(multipleResult, 2);
        };

        System.out.println(multipleNumbersAndPowerByTwoBiFunction.apply(2, 3)); //Will print 36.0 to the console.
    }

}
```
#### `andThen()` - Returns a composed function that first applies this function to its input, and then applies the after function to the result.
```java
public class Main {

    public static void main(String[] args) {
        BiFunction<Integer, Integer, Double> multipleNumbersAndPowerByTwoBiFunction = (num1, num2) -> {
            int multipleResult = num1 * num2;
            return Math.pow(multipleResult, 2);
        };

        Function<Double, String> formatResultBiFunction = (result) -> "The result is " + result;

        BiFunction<Integer, Integer, String> formattedResult = multipleNumbersAndPowerByTwoBiFunction.andThen(formatResultBiFunction);

        System.out.println(formattedResult.apply(2, 3)); //Will print The result is 36.0 to the console.
    }

}
```