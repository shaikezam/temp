# Spring @Qualifier Annotation
*18-01-2022 - Shai Zambrovski*

------------
## Why using @Qualifier?
Almost every `Java` developer knows the `Autowired` keyword (*an annotation in `Spring` that used to automatically wire (inject) dependencies into a bean without explicitly specifying them in the code*).

But what if we have an interface in which two or more beans are implements it?

Lets take the next example:

We have a main class with a Program class that contains an IGenerator instance.

The IGenerator has two implementations `KooGenerator` and `FooGenerator`:
```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        ApplicationContext applicationContext = SpringApplication.run(DemoApplication.class, args);
        Program program = applicationContext.getBean(Program.class);
        System.out.println(program.run());
    }

    @Component
    public class Program {
        @Autowired
        private IGenerator generator;

        public String run() {
            return generator.sayWhoIAm();
        }
    }

    public interface IGenerator {
        String sayWhoIAm();
    }

    @Component("kooGenerator")
    public class KooGenerator implements IGenerator {
        @Override
        public String sayWhoIAm() {
            return "I'm KOO";
        }
    }

    @Component("fooGenerator")
    public class FooGenerator implements IGenerator {
        @Override
        public String sayWhoIAm() {
            return "I'm FOO";
        }
    }
}
```
Once we run the application we will get an exeption: `org.springframework.beans.factory.NoUniqueBeanDefinitionException`, tha's because Spring doesn't know which implementation to take.

Let's take a look of our solutions:
## Using @Qualifier with @Autowired
```java
@Component
public class Program {

  @Autowired
  @Qualifier("kooGenerator")
  private IGenerator generator;

  public String run() {
    return generator.sayWhoIAm();
  }
}
```
By using the name of the implementation in the @Qualifier annotation, we can tell which bean we want to use and to avoid the `NoUniqueBeanDefinitionException`.
## Using @Primary
Spring also allow us to define a primary bean implementation to use, in case of multipe implementation.

Need to remember that `@Qualifier` as bigger priority then `@Primary,` that's means that it's waste to define both of the annotations.

`@Primary `means default implementation, while `@Qualifier` is the specific implementation.
```java
public interface IGenerator {
  String sayWhoIAm();
}

@Component
public class KooGenerator implements IGenerator {
  @Override
  public String sayWhoIAm() {
    return "I'm KOO";
  }
}

@Primary
@Component
public class FooGenerator implements IGenerator {
  @Override
  public String sayWhoIAm() {
    return "I'm FOO";
  }
}
```
## Autowiring by Name
Another way to resolve the bean implementation is via the name of the bean.

This is the default way.

In this case, the `Program` will take the `kooGenerator` implementation as this is the bean name next to the `@Component`.
```java

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        ApplicationContext applicationContext = SpringApplication.run(DemoApplication.class, args);
        Program program = applicationContext.getBean(Program.class);
        System.out.println(program.run());
    }

    @Component
    public class Program {

        @Autowired
        private IGenerator kooGenerator;

        public String run() {
            return kooGenerator.sayWhoIAm();
        }
    }

    public interface IGenerator {
        String sayWhoIAm();
    }

    @Component("kooGenerator")
    public class KooGenerator implements IGenerator {
        @Override
        public String sayWhoIAm() {
            return "I'm KOO";
        }
    }

    @Component("fooGenerator")
    public class FooGenerator implements IGenerator {
        @Override
        public String sayWhoIAm() {
            return "I'm FOO";
        }
    }

}
```

