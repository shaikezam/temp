# Serialized And Deserialized Objects
*26-12-2019 - Shai Zambrovski*

------------
Java have a build-in mechanism that can migrate objects to stream of bytes and store them in a file and vice versa.

Those abilities called `Serialization` and `Derialization`.

`Serialization` and `Derialization` are platfrom independent, that means that we can `serialize` in one platfrom and `deserialize` in a different platform.

In order to use this mechanism:
- The desired object must implements the `java.io.Serializable` interface.
- All the fields in the desired object must implements the `Serializable` interface.
- If there is field who doesn't implements the `Serializable` interface he must marked as `transient`.

Please notice, `Serializable` interface is a `marker interface`:

*An empty interface (no field or methods) that provides run-time type information about objects.
We will demostrate this mechanism using simple Course class:*
We will demostrate this mechanism using simple `Course` class:
```java
public class Course implements Serializable {
    private String name;
    private String location;
    private transient int maxStudentsCapacity;

    public Course(String name, String location, int studentsCapacity) {
        this.name = name;
        this.location = location;
        this.maxStudentsCapacity  = studentsCapacity;
    }
}
```
## Serializing
For serializing an object we need to use the `java.io.ObjectOutputStream` class :
```java
Course javaCourse = new Course("Java Fundamentals", "Auditorium", 50);
try(ObjectOutputStream os = new ObjectOutputStream(Files.newOutputStream(Paths.get("course.dat")))) {
    os.writeObject(javaCourse);
} catch (IOException e) {...}
```
## Deserializing
For deserializing an object we need to use the `java.io.ObjectInputStream` class:
```java
try(ObjectInputStream is = new ObjectInputStream(Files.newInputStream(Paths.get("course.dat")))) {
    Course javaCourse = (Course) is.readObject();
} catch (IOException e) {...}
  catch (ClassNotFoundException e) {...}
```
If we print the `javaCourse` we see that the `maxStudentsCapacity` property is 0 altough we set it to 50.

That because this field marked as `transient` and it didn't write to the `course.dat` file, and 0 is the default int value.
## Serializing Version Compatibility
If we take a look on a very reasonable scenario in-which we want to add to our `Course` class a `courseYear` property that will holds the year of the course:
```java
public class Course implements Serializable {
    private String name;
    private String location;
    private transient int maxStudentsCapacity;
    private String courseYear;

    public Course(String name, String location, int maxStudentsCapacity, String courseYear) {
        this.name = name;
        this.location = location;
        this.maxStudentsCapacity = maxStudentsCapacity;
        this.courseYear = courseYear;
    }
}
```
And now, lets try to `Deserializing` again the `course.dat` file again, we will get:
```java
java.io.InvalidClassException: com.company.Course; local class incompatible: stream classdesc serialVersionUID = -3353359511573316605, local class serialVersionUID = 498427908211648568

```
We get an execption because the `Course` class definition we trying to `Deserializing` is very different from the newley class definition.

One class has the serial version UID `-3353359511573316605`, while the other class has the serial version UID `498427908211648568`, and that why we get this execption.

The serial version UID it's a private, static final, long class property that indicates version compatibility (all version must have the same value), and it can be calculated through two ways:
- **Run Time** - can be calculated by the properties name, interfaces, members and types - will cause to incompatible versions.
- **Design Time** - can be calculated by the IDE or the serialver tool.

To overcome the InvalidClassException issue there are two ways:
- **IDE solution** - manually add in the class `private static final long serialVersionUID` and let the IDE generate value.
- **Serialver solution** - nevigate to the root folder of `Course.class` folder and open a cmd and type `serialver -show` and a new window will open:

![](https://shaikezam.com/style/serialver.png)

we need to pass the full class name: `com.company.Course` and click show:

![](https://shaikezam.com/style/serialverwithvalue.png)

And now, if we `Deserializing` again the `course.dat` file we will get:

`Course{name='Java Fundamentals', location='Auditorium', maxStudentsCapacity=0, courseYear='null'}`
Please notice that `courseYear` value is `'null',` because this is the default value of `String`, let's understand how we can override it.
## Customizing Serializing
We can Customizing `Serializing` and `Deserializing` using `writeObject` and `readObject` methods in which called through reflection.

Can be very useful we we want to change the format of the `Serializing` or to support for older versions of serialized classes.
```java
public class Course implements Serializable {
    private String name;
    private String location;
    private transient int maxStudentsCapacity;
    private String courseYear;

    public Course(String name, String location, int maxStudentsCapacity, String courseYear) {
        this.name = name;
        this.location = location;
        this.maxStudentsCapacity = maxStudentsCapacity;
        this.courseYear = courseYear;
    }


    private void writeObject(ObjectOutputStream os) throws IOException, ClassNotFoundException {
        os.defaultWriteObject();
    }

    private void readObject(ObjectInputStream is) throws IOException, ClassNotFoundException {
        ObjectInputStream.GetField fields = is.readFields();
        this.name = (String) fields.get("name", "defaultName");
        this.location = (String) fields.get("location", "defaultlocation");
        this.courseYear = (String) fields.get("courseYear", "defaultCourseYear");
    }
}
```
### readObject
To override the values in which returns from the `Deserializing` we need to get the fields and pass as the 2nd argument the default values.
### writeObject
Very useful for custom `Serializing` if we want to override some property before we `Serializing`.