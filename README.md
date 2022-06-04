# Generics Boundaries

## Learning Goals

-
- Learning Goal 2

Sometimes when you create a method that accepts generics, you may still want to
restrict the types of objects that can be passed into your method because your
logic only makes sense for specific types of objects.

For example, you may have a generic method that needs to operate on numbers, but
can operate on numbers of any type. Consider a method that adds all the numbers
in a list. Using unbounded generics, you might define this method as follows:

```java
    private static <E> Double sumList(List<E> numberList) {
        double sum = 0.0;
        for (E element: numberList) {
            Number number = (Number)element;
            sum += number.doubleValue();
        }

        return sum;
    }
```

Note that in order to get the numerical value from our `numberList`, we need to
convert each element of the list of `Number`. This works fine if call the method
with a list of objects of a type compatible with `Number`:

```java
    public static void main(String[] args) {
        List<Number> numbers = new ArrayList<Number>();
        numbers.add(10.0);
        numbers.add(20);
        numbers.add(30);

        Number sum = sumList(stringNumbers);
        System.out.println("sum = " + sum);
    }
```

But remember that one of the goals of using generics was to make our code more
flexible, while still remaining type safe, meaning that we want the Java
compiler to be able to give us an error when we're not using the right types.

In this case, I can modify the code above to actually pass a `List` of `String`
objects instead of `Number` objects and the code will continue to compile fine:

```java
    public static void main(String[] args) {
        List<String> stringNumbers = new ArrayList<String>();
        stringNumbers.add("0");
        stringNumbers.add("10");
        stringNumbers.add("20");

        // this code compiles
        Number sum = sumList(stringNumbers);
        System.out.println("sum = " + sum);
    }
```

Running this code, however, gives us a `ClassCastException` because we cannot
cast a `String` into a `Number`:

```java
Exception in thread "main" java.lang.ClassCastException: class java.lang.String cannot be cast to class java.lang.Number (java.lang.String and java.lang.Number are in module java.base of loader 'bootstrap')
	at com.flatiron.generics.GenericsRunner.sumList(GenericsRunner.java:38)
	at com.flatiron.generics.GenericsRunner.main(GenericsRunner.java:20)
```

The problem is with this line of code in the `sumList()` method:

```java
            Number number = (Number)element;
```

Here is how we can specify an upper bound wildcard for the generic type in our
`sumList()` method:

```java
    private static Double sumList(List<? extends Number> numberList) {
        double sum = 0.0;
        for (Number number: numberList) {
            sum += number.doubleValue();
        }

        return sum;
    }
```

The syntax for specifying an upper bound wildcard is as follows:

- In the generic parameter type definition, use the `? extends BaseType`
  notation where `?` is the wildcard and `BaseType` is the class that should be
  the upper bound of your parameter type
- Note that in this case `extends` applies to class extensions as well as
  interface implementations

In this case, all objects that are passed into this method must be a subclass of
the `Number` class.

With this code, we now get a compile-time error when we try to pass the wrong
type into our method:

![Wrong Generic Type](https://curriculum-content.s3.amazonaws.com/java-mod-2/type-casting/wrong-generic-type.png)

With the upper bound wildcard above, we are restricting the unknown type of the
`sumList()` method to classes that are subclasses of `Number`.

We can also specify a lower bound, which will restrict the unknown type of a
method to classes that are super classes of the class we specify.

Let's rewrite our `sumList()` method to only accept lists of `Integer` objects:

```java
    private static Integer sumIntegerList(List<Integer> numberList) {
        Integer sum = Integer.valueOf(0);
        for (Integer number: numberList) {
            sum += number;
        }

        return sum;
    }
```

This would work to add a list of `Integer` objects:

```java
    public static void main(String[] args) {
        List<Integer> integers = new ArrayList<Integer>();
        integers.add(10);
        integers.add(20);
        integers.add(30);

        Number sum = sumLIntegerist(integers);
        System.out.println("sum = " + sum);
    }
```

But what if you wanted your `sumIntegerList()` method to be able to add integers
that are held in any class that can hold integer values, which is any class that
is a superclass of the `Integer` class?

Let's examine the `Integer` class hierarchy:

- `Integer extends Number` and
- `Number extends Object`, which yields the following hierarchy:
- `Object` -> `Number` -> `Integer`

Trying to call this `sumIntegerList()` method with a list of `Number` objects
like this:

```java
    public static void main(String[] args) {
        List<Number> numbers = new ArrayList<Number>();
        numbers.add(0);
        numbers.add(10);
        numbers.add(20);

        Number sum = sumIntegerList(numbers);
        System.out.println("sum = " + sum);
    }
```

Gives us the following error:

![Integer List vs Number List](https://curriculum-content.s3.amazonaws.com/java-mod-2/type-casting/integer-list-vs-number-list.png)

This is because the type `List<Integer>` only matches the `Integer` class
specifically. For more flexibility, we can specify a lower bound for our generic
type:

```java
    private static Integer sumIntegerList(List<? super Integer> numberList) {
        Integer sum = Integer.valueOf(0);
        for (Object number: numberList) {
            sum += (Integer)number;
        }

        return sum;
    }
```

The syntax for specifying a lower bound wildcard is as follows:

- In the generic parameter type definition, use the `? super Type` notation
  where `?` is the wildcard and `Type` is the class that should be the lower
  bound of your parameter type

In this case, all objects that are passed into this method must be a `Integer`
or one of the classes it inherits from (directly, i.e. `Number` or indirectly,
which in this case means `Object`).

Now the following code compiles and runs:

```java
    public static void main(String[] args) {
        List<Number> numbers = new ArrayList<Number>();
        numbers.add(0);
        numbers.add(10);
        numbers.add(20);

        Number sum = sumIntegerList(numbers);
        System.out.println("sum = " + sum);
    }
```
