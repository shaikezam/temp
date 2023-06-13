# Handling Properties
*22-12-2019 - Shai Zambrovski*

------------

Must of Java application need to use with key/value pairs for persist configuration metadata, user information or any data that related to a program or application.
So we need to have am intuitive way to manage key/value mechanism to:

- set or get values.
- Set default value for non-exist pairs.

for this, Java provides us the `java.util.Properties` class that manage the key/value pair as a String.

The `Properties` class provides us basically two methods:
- `setPropertySet` value (create or modify) to a specific key.
- `getPropertyGet` the value of a specific key; `null` is returned if key doesn't exists, unless we provide default value.

So how we can persist those properties?

Using streams we can basically persist two types of properties files format; `.properties` and `.xml` files.
## `.properties` format
- Can be handled with `outputStream` & `inputStream` (if we work with bytes) or `reader` & `writer` (if we work with characters).
- Key/value separated with `:` or `=.`
- Comments starts with `#` or `!`.

### Write to `.properties` file:
```java
Properties props = new Properties();
props.setProperty("courseName", "Java Fundamentals");
props.setProperty("courseLocation", "Auditorium");
try (Writer writer = Files.newBufferedWriter(Paths.get("courses.properties"))) {
   props.store(writer, "2019 courses"); // In case we don't want any comments, need to pass a null in the 2nd argument
}
```
`courses.properties` file:

    #2019 courses
    #Wed Dec 25 14:03:57 IST 2019
    courseLocation=Auditorium
    courseName=Java Fundamentals

### Read from `.properties` file:
```java
Properties props = new Properties();
try (Reader reader = Files.newBufferedReader(Paths.get("courses.properties"))) {
    props.load(reader);
} catch(...) {}
String name = props.getProperty("courseName"); //Java Fundamentals
String location = props.getProperty("courseLocation"); //Auditorium
```
## `.xml` format
- Can be handled only with `outputStream` & `inputStream`.
- Key will persisted as a key attribute and value will persisted as an element value.
- Comments will persisted as an element.

### Write to `.xml` file:
```java
Properties props = new Properties();
props.setProperty("courseName", "Java Fundamentals");
props.setProperty("courseLocation", "Auditorium");
try (OutputStream os = Files.newOutputStream(Paths.get("courses.xml"))) {
    props.storeToXML(os, "2019 courses"); // In case we don't want any comments, need to pass a null in the 2nd argument
}
```
`courses.xml` file:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <comment>2019 courses</comment>
    <entry key="courseLocation">Auditorium</entry>
    <entry key="courseName">Java Fundamentals</entry>
</properties>
```
### Read from `.xml` file:
```java
Properties props = new Properties();
try (InputStream is = Files.newInputStream(Paths.get("courses.xml"))) {
   props.loadFromXML(is);
} catch (...) {}
String name = props.getProperty("courseName"); //Java Fundamentals
String location = props.getProperty("courseLocation"); //Auditorium
```
## Default Properties mechanism
In some cases and scenarios we want to have a default values to provide an abstraction for some configurations.

To achive such feature, we can pass to the `Properties` constructor a `Properties` instance that will supply a default values for a keys that doesn't exists in the current `Properties` instance.