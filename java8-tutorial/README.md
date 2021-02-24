# Java 8 Study Manual

`Java8 is becoming more and more popular. It provides a lot of new APIs, which greatly improves our daily coding efficiency. This article briefly summarizes some of the new features of java8 and attaches code examples.`


## table of Contents

* [Interface uses default method] (#Interface uses default method)
* [Lambda expression](#lambda expression)
* [Functional interface](#functional interface)
* [Method and Constructor reference](#method-and-constructor-reference)
* [Lambda Range](#lambda-Scope)
  * [Local variables](#local variables)
  * [Access global variables or static variables] (#Access global variables or static variables)
  * [default method](#default method)
* [Method Reference](#Method Reference)
* [Built-in Functional Interfaces](#Built-in-functional-interfaces)
  * [Judge Predicates](#Judge-predicates)
  * [Function Functions](#Function-functions)
  * [Production Suppliers](#产品-suppliers)
  * [Consumers](#consuming-consumers)
  * [Comparators](#Comparators-comparators)
  * [Other function interface](#Other function interface)
* [Optional Optionals](#optional-optionals)
* [流 Streams](#流-streams)
  * [Filter Filter](#滤-filter)
  * [Map Map](#Map-map)
  * [Mapping flatMap](# Mapping-flatMap)
  * [Sort Sorted](#sort-sorted)
  * [Reduce Reduce](# Reduce-reduce)
  * [Count Count](#Count-count)
  * [Match Match](#match-match)
  * [Skip skip](#skip-skip)
  * [Output limit](#Output-limit)
  * [Output collect](#output-collect)
* [Parallel Streams](#parallel-streams)
  * [Serial Sort](#serial-sort)
  * [Parallel Sort](#parallel-sort)
* [Collection Maps](#集-maps)
* [DATE API](#日-api)
  * [Clock](#clock)
  * [Time Zone] (#时间区)
  * [Local time] (#local time)
  * [Local date](#local date)
  * [Local date and time] (#local date and time)
* [Note](#note)


## Interface uses default method

Java 8 allows us to use unstructured methods such as the `default` modifier in interface classes to implement interfaces.
 
This feature is described in detail in [virtual extension methods](http://stackoverflow.com/a/24102730).

Example:

```
interface Formula {
    double calculate(int a);

    default double sqrt(int a) {
        return Math.sqrt(a);
    }
}
```
In addition to the abstract method `calculate`, the `Formula` interface class also defines the default method `sqrt`. The implementing class must implement the abstract method `calculate`. The default modification method `sqrt` can also be used outside the class.


```
Formula formula = new Formula() {
    @Override
    public double calculate(int a) {
        return sqrt(a * 100);
    }
};

formula.calculate(100); // 100.0
formula.sqrt(16); // 4.0
```

The formula uses an anonymous implementation class. The code is very redundant and requires 6 lines of code to implement a simple calculation `sqrt(a * 100)`. In the next section, a method can be implemented more cleverly with Java 8.



## Lambda expression

A simple example, sorting a collection of strings

```
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");

Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
```

The static method `Collections.sort` receives the list collection and a comparator to implement the sorting function. Usually you need to create an anonymous comparator and pass it to the sort method.

In order to replace anonymous class instances, Java 8 introduces a very concise syntax, **lambda expression**:


```
Collections.sort(names, (String a, String b) -> {
    return b.compareTo(a);
});
```

Of course, you can also use shorter and more readable writing, as above.


```
Collections.sort(names, (String a, String b) -> b.compareTo(a));
```

It can be further streamlined, leaving only one line of code, omitting the `{}` and `return` methods, as above.


```
names.sort((a, b) -> b.compareTo(a));
```
The List class now provides the `sort` method. At the same time, the java compiler can automatically recognize the parameter types, so you can ignore them when coding. Next, let us learn more about how lambda expressions are widely used.



## Functional interface

How does lambda expression recognize the system type of Java? Each lambda corresponds to a type specified by the interface. Therefore, every _functional interface_ definition must contain an abstract method declaration. The type of each lambda expression needs to match the abstract method. Since the default method is not abstract, you need to add the default method to the functional interface.

We can define an interface as a lambda expression arbitrarily, and it needs to contain an abstract method inside. To ensure that the interface meets the specification, you need to add the @FunctionalInterface annotation. Once you try to add the second abstract method, the compiler will automatically detect and throw a compilation error.


Example:

```
@FunctionalInterface
interface Converter<F, T> {
    T convert(F from);
}
```

```
Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
Integer converted = converter.convert("123");
System.out.println(converted); // 123
```
Code: com.winterbe.java8.samples.lambda.Lambda2

Remember, if you omit `@FunctionalInterface`, the code is also valid.


## Method and Constructor reference

The code example above can be further simplified by using static method references:

```
Converter<String, Integer> converter = Integer::valueOf;
Integer converted = converter.convert("123");
System.out.println(converted); // 123
```

Java 8 allows you to pass method or constructor references, such as `::`. The above example demonstrates referencing a static method. In addition, we can also use the method of class instance objects.


```
class Something {
    String startsWith(String s) {
        return String.valueOf(s.charAt(0));
    }
}
```

```
Something something = new Something();
Converter<String, String> converter = something::startsWith;
String converted = converter.convert("Java");
System.out.println(converted); // "J"
```

Let us understand how the `::` key word is used in the constructor. First define a class structure as follows:


```
class Person {
    String firstName;
    String lastName;

    Person() {}

    Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```

Next, we define a person factory interface to create new persons:

```
interface PersonFactory<P extends Person> {
    P create(String firstName, String lastName);
}
```

Different from the traditional implementation, we implement it by calling the constructor method:

```
PersonFactory<Person> personFactory = Person::new;
Person person = personFactory.create("Peter", "Parker");
```

We use `Person::new` to trigger the constructor function of Person. The Java compiler can automatically select the appropriate constructor function to match `PersonFactory.create`.

Code: com.winterbe.java8.samples.lambda.Lambda2

## Lambda Scope

Lambda expressions to access external variables are similar to anonymous objects. You can access final modified local local variables.

### Local variables

We can read final modified local variables

```
final int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2); // 3
```
Unlike anonymous objects, the variable `num` need not be forced to be finalized. The following writing is also valid:

```
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2); // 3
```

`num` must be implicitly final when the code is compiled. The following wording compilation will report an error:

```
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);
num = 3;
```

### Access global variables or static variables

Contrary to local variables, we can read or write global variables or static global variables in lambda expressions.

```
Code: com.winterbe.java8.samples.lambda.Lambda4
class Lambda4 {
    static int outerStaticNum;
    int outerNum;
To
// method
    void testScopes() {
        Converter<Integer, String> stringConverter1 = (from) -> {
            outerNum = 23;
            return String.valueOf(from);
        };

        Converter<Integer, String> stringConverter2 = (from) -> {
            outerStaticNum = 72;
            return String.valueOf(from);
        };
    }
}
```

### default method

Remember the example formula in the first section? A default method `sqrt` is defined in the `Formula` interface class, which can be accessed by every formula instance object including anonymous objects. But this does not apply to lambda expressions.

The lambda expression statement cannot directly access the default method. The following wording compiles:

```
Formula formula = (a) -> sqrt(a * 100);
```

## Method reference

Method references and lambda expressions have the same characteristics (for example, they both need a target type and need to be converted into an instance of a functional interface), but we don’t need to provide a method body for the method reference, we can directly pass the method name Use existing methods.

There are three cases of method reference:

* 1) Class + dynamic method
* 2) Class + static method
* 3) Class instance object + dynamic method
* 4) Class instance object + static method (invalid, incorrect writing)

```
    public static void main(String[] args) {

        // 1) Class + dynamic method
        BiConsumer<LinkedHashSet, Object> biConsumer1 = LinkedHashSet::add;
        LinkedHashSet s1 = new LinkedHashSet();
        biConsumer1.accept(s1, "aaa");
        System.out.println(s1);

        // 2) Class + static method
        BiConsumer<String, Long> biConsumer2 = Utils::concatStatic;
        biConsumer2.accept("first_param", 6L);

        // 3) Class instance object + dynamic method
        BiConsumer<String, Long> biConsumer3 = new Utils()::concat;
        biConsumer3.accept("first_param", 7L);

        // 4) Class instance object + static method
        // Error: (35, 48) java: Invalid method reference, static limit method reference
        // BiConsumer<String, Long> biConsumer4 = new Utils()::concatStatic;
        // biConsumer4.accept("first_param", 8L);

    }
```

Array construction method that receives int parameters

```
IntFunction<int[]> arrayMaker = int[]::new;
int[] array = arrayMaker.apply(10) // Create an array int[10]

```

## Built-in Functional Interfaces


JDK 1.8 API provides many built-in functional interfaces. Some of them are from older versions of Java, such as `Comparator` or `Runnable`. Extend those existing interfaces through the `@FunctionalInterface` annotation to achieve lambda support.

At the same time, Java 8 API also provides some new functional interfaces to meet the needs of multiple scenarios. Some new features are from [Google Guava](https://code.google.com/p/guava-libraries/) third party library.

### Judgment Predicates

Predicates internally defines a boolean type judgment method with an input parameter. This interface also contains many default methods to satisfy various complex logical expressions, such as (and, or, not)

```
Code: com.winterbe.java8.samples.lambda.Lambda3

Predicate<String> predicate = (s) -> s.length()> 0;

predicate.test("foo"); // true
predicate.negate().test("foo"); // false

Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```

### Functions

The function receives an input parameter and returns a result. The `default` method is used to link multiple functions together (executed before compose, executed after andThen)

```
Code: com.winterbe.java8.samples.lambda.Lambda3

Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);

backToString.apply("123"); // "123"
```

### Production Suppliers

Suppliers produces results of the specified type. Unlike Functions, Suppliers does not accept any parameters.

```
Code: com.winterbe.java8.samples.lambda.Lambda3

Supplier<Person> personSupplier = Person::new;
personSupplier.get(); // new Person
```

Case: Java 8 introduced an overloaded version of the log method. This version of the log method accepts a Supplier as a parameter. The function signature of this alternative version of the log method is as follows:

```
public void log(Level level, Supplier<String> msgSupplier)
You can call it in the following ways:

logger.log(Level.FINER, () -> "Problem: "+ generateDiagnostic());
If the level of the logger is set appropriately, the log method will execute the Lambda expression passed in as a parameter internally (note: conventional writing, execute all input methods first, get the actual parameters, and then execute the method). Lazy evaluation can effectively avoid some unnecessary performance overhead.

The internal implementation of the Log method introduced here is as follows:
public void log(Level level, Supplier<String> msgSupplier){
    if(logger.isLoggable(level)){
        log(level, msgSupplier.get());
    }
}
```

https://my.oschina.net/bairrfhoinn/blog/142985

### Consumers

Consumers means processing a single input parameter, and provides the andThen'default' method for subsequent processing.

```
Code: com.winterbe.java8.samples.lambda.Lambda3

Consumer<Person> greeter = (p) -> System.out.println("Hello, "+ p.firstName);
greeter.accept(new Person("Luke", "Skywalker"));
```

### Compare Comparators

Comparators existed in older versions of Java. Java 8 adds a variety of `default` methods


```
Code: com.winterbe.java8.samples.lambda.Lambda3

Comparator<Person> comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);

Person p1 = new Person("John", "Doe");
Person p2 = new Person("Alice", "Wonderland");

comparator.compare(p1, p2); //> 0
comparator.reversed().compare(p1, p2); // <0
```

Comparator and reversed are used in combination to support the sorting of multiple fields, the default is from small to large, or from large to small

```
Code: com.winterbe.java8.samples.functional.Comparator1

List<Score> list = new ArrayList<>();
list.add(new Score("xiaohong", 90L, 91L));
list.add(new Score("xiaoming", 85L, 90L));
list.add(new Score("wanggang", 90L, 96L));
list.add(new Score("xiaoma", 85L, 70L));

// First according to the language from small to large, if equal, according to mathematics from small to large
Collections.sort(list, Comparator.comparing(Score::getYuwen).thenComparing(Score::getShuxue));
System.out.println("First according to language from small to large, if equal, according to mathematics from small to large");
list.forEach(System.out::println);

// First according to language, from largest to smallest, if equal, from largest to smallest in mathematics
Comparator c1 = Comparator.comparing(Score::getYuwen).reversed();
Comparator c2 = Comparator.comparing(Score::getShuxue).reversed();
Collections.sort(list, c1.thenComparing(c2));
System.out.println("First according to language from big to small, if equal, according to mathematics from big to small");
list.forEach(System.out::println);
```

### Other function interfaces

Like BinaryOperator, etc.

Code location: com.winterbe.java8.samples.functional

## Optionals

Optionals is not a functional interface, mainly to prevent `NullPointerException`. The next section will focus on the introduction, now let us first understand how Optionals work.

Optional is a simple container for storing null or non-null values. Imagine that a method with a return value sometimes returns null. In contrast, Java 8 returns `Optional` instead of `null`.

```
Code: com.winterbe.java8.samples.stream.Optional1

Optional<String> optional = Optional.of("bam");

optional.isPresent(); // true
optional.get(); // "bam"
optional.orElse("fallback"); // "bam"

optional.ifPresent((s) -> System.out.println(s.charAt(0))); // "b"
```

## Streams

`java.util.Stream` can perform one or more operations on the element list. Stream operations can be intermediate values ​​or final results.
The final operation returns a certain type of result, while the intermediate operation returns the stream itself. So you can chain multiple method calls in one line of code. Streams are created in `java.util.Collection`, such as list or set (map does not support). Stream can be executed sequentially or in parallel.

 Streams is very powerful, so I wrote a separate article introducing [Java 8 Streams Tutorial](http://winterbe.com/posts/2014/07/31/java8-stream-tutorial-examples/). Code Base [Sequency](https://github.com/winterbe/sequency)
 
* Intermediate operations: filter, map, mapToInt, mapToLong, mapToDouble, flatMap, sorted, distinct, limit, skip, of, iterate
* Termination operation: forEach, count, collect, reduce, toArray, anyMatch, allMatch, noneMatch, findAny, findFirst, max, min
* Primitive type specialization stream: IntStream, LongStream, DoubleStream

### Filter

Filter filters all elements through the `predicate` judgment function. This operation is an intermediate operation and needs to be terminated to trigger execution.

```
Code: com.winterbe.java8.samples.stream.Stream_filter
stringCollection
    .stream()
    .filter((s) -> s.startsWith("a"))
    .forEach(System.out::println);

// "aaa2", "aaa1"
```

### Mapping Map

`map` is an intermediate process operation that transforms elements into another form with the help of function expressions. The following example converts each string to uppercase string. But you can also use `map` to convert each object to another type. The final output type depends on the function expression you pass in.

```
stringCollection
    .stream()
    .map(String::toUpperCase)
    .sorted((a, b) -> b.compareTo(a)) //from big to small
    .forEach(System.out::println);

// "DDD2", "DDD1", "CCC", "BBB3", "BBB2", "AAA2", "AAA1"
```
### Mapping flatMap

If it involves one-to-many mapping, you need to put the mapping result into a Stream. The effect of using the flatMap method is that the multiple results after conversion are not mapped to a stream respectively, but are mapped to the content of the stream.

```
Code: com.winterbe.java8.samples.stream.Stream_flatMap

List<String> lists = Arrays.asList("Hello", "World");
lists.stream().flatMap(word-> Stream.of(word.split("")))
        .distinct()
        .forEach(System.out::println);
```

### Sort Sorted

Sorted is an intermediate operation that returns an ordered view of the stream. Unless you pass a custom `Comparator`, the elements are sorted by the default `smallest to large`.

```
Code: com.winterbe.java8.samples.stream.Stream_sorted

//Default sort
stringCollection.stream().sorted().forEach(System.out::println);
System.out.println(stringCollection);
Special note: `sorted` just creates a sorted view of the stream, and does not change the order of the original collection. So the order of `stringCollection` has not changed.
```

```
//Custom order, sort by string, from big to small
stringCollection
        .stream()
        .map(String::toUpperCase)
        .sorted((a, b) -> b.compareTo(a))
        .forEach(System.out::println);
```

### Reduce

Termination operation, through the given function expression to process the two elements before and after the stream, or the intermediate result and the next element. Lambda repeatedly combines each element until the stream is reduced to a value. For example, sum or find the largest element.

```
Code: com.winterbe.java8.samples.stream.Stream_reduce

// Split the stream data list into multiple batches, the sum is initially 0, and each batch is executed (sum, p) -> sum = sum + p.age to get the partial sum of sum. Parallel computing ideas
// Finally pass (sum1, sum2) -> sum1 + sum2 to calculate the final sum
// (sum1, sum2) -> sum1 + sum2, mainly suitable for parallelism, parallelStream(), single thread is invalid.

private static void test3(List<Person> persons) {
    Integer ageSum = persons.parallelStream().reduce(0, (sum, p) -> sum += p.age, (sum1, sum2) -> sum1 + sum2);
    System.out.println(ageSum);
}
```

For more reduce usage, please refer to: https://blog.csdn.net/io_field/article/details/54971679

#### Count Count

Count is a terminal operation that returns the total number of elements in a long list.

```
Code: com.winterbe.java8.samples.stream.Stream_count
long startsWithB =
    stringCollection
        .stream()
        .filter((s) -> s.startsWith("b"))
        .count();
        
System.out.println(startsWithB); // 3
```

### Match

Various matching operations are used to determine whether the stream conditions are met. After all operations are completed, a boolean type result is returned.

```
Code: com.winterbe.java8.samples.stream.Stream_match
List<String> stringCollection = new ArrayList<>();
stringCollection.add("ddd2");
stringCollection.add("aaa2");
stringCollection.add("bbb1");
stringCollection.add("aaa1");
stringCollection.add("bbb3");
stringCollection.add("ccc");
stringCollection.add("bbb2");
stringCollection.add("ddd1");

// Only one condition is required
boolean anyStartsWithA = stringCollection.stream().anyMatch((s) -> s.startsWith("a"));
System.out.println("anyMatch:" + anyStartsWithA); // true

// All conditions must be met
boolean allStartsWithA = stringCollection.stream().allMatch((s) -> s.startsWith("a"));
System.out.println("allMatch:" + allStartsWithA); // false

// All conditions must not be met
boolean noneStartsWithZ = stringCollection.stream().noneMatch((s) -> s.startsWith("z"));
System.out.println("noneMatch:" + noneStartsWithZ); // true

// return any element
Optional<String> anyE = stringCollection.stream().findAny();
System.out.println("findAny:" + anyE.get());

//Return the first element
Optional<String> firstE = stringCollection.stream().findFirst();
System.out.println("findFirst:" + firstE.get());
```

### skip skip

Returns a stream that discards the first n elements

```
Code: com.winterbe.java8.samples.stream.Stream_skip
// Throw away the first three elements
stringCollection
    .stream()
    .skip(3)
    .forEach(System.out::println);

```

### Output limit

Take only the first N results

```
Code: com.winterbe.java8.samples.stream.Stream_limit
// Take the first three elements
stringCollection
    .stream()
    .limit(3)
    .forEach(System.out::println);

```

### Output collect

Accept various practices as parameters, accumulate the elements in the stream into a summary result

Common examples:

* Group a transaction list by currency, and get the sum of all transaction amounts of the currency (return a Map\<Currency, Integer>)
* Divide the transaction list into two groups, expensive and inexpensive (return a Map<Boolean, List\<Transaction>>)
* Create multi-level groupings, such as grouping transactions by city, and then further grouping by expensive or inexpensive

Common methods of Collectors:

* Collectors.toList, get the List list
* Collectors.toSet, get Set collection
* Collectors.joining, join strings through `joiner`
* Collectors.groupingBy(Function<? super T,? extends K>), group by K value, return Map<K, List>
* Collectors.groupingBy(Function<? super T,? extends K>, Collector<? super T,A,D>), two-level grouping, get two-level Map
* Collectors.partitioningBy(Predicate<? super T> predicate), partition is a special case of grouping, and returns a Boolean value, which means that the key of the obtained grouping Map can only be Boolean, so it can be divided into two groups
* Collectors.maxBy, to find the maximum value, you need to pass a custom Comparator
* Collectors.reducing, generalized reduction summary.
* Collectors.toMap, get the Map collection. Note: If the key is repeated, an exception will be thrown, which requires special handling


```
Code: com.winterbe.java8.samples.stream.Stream_collect

// Change the string to uppercase and concatenate it with a comma
List<String> citys = Arrays.asList("USA", "Japan", "France");
String cityS = citys.stream().map(x -> x.toUpperCase()).collect(Collectors.joining(", "));
        
// Group by gender
Map<String, List<Student>> maps = studentList.stream().collect(Collectors.groupingBy(Student::getSex));

// Group by gender first, then group by age
Map<String, Map<String, List<Student>>> maps = studentList.stream()
   .collect(Collectors.groupingBy(Student::getSex,
      Collectors.groupingBy(s -> {
          if (s.getAge() <20) {
              return "low age";
          } else {
              return "High age";
          }
      })));

// Divide into two groups by age 25
Map<Boolean, List<Student>> maps = studentList.stream().collect(Collectors.partitioningBy(s -> {
    if (s.getAge() <25) {
        return true;
    } else {
        return false;
    }
}));

// find the oldest person
Optional<Student> optional1 = studentList.stream().collect(Collectors.maxBy(Comparator.comparing(Student::getAge)));
optional1.ifPresent(System.out::println);

// total age
// The reducing parameter, the first one: the initial value. The second one: conversion function. Third: Cumulative function
int sum = studentList.stream().collect(Collectors.reducing(0, Student::getAge, Integer::sum));

// Convert to a Map<sex,name>, note: if the key is repeated, an exception will be thrown
// If the key is repeated, overwrite the previous one with the value of the latter
private static void test8(List<Student> studentList) {
    Map<String, String> sexNameMap = studentList.stream().collect(Collectors.toMap(p -> {
        return p.getSex();
    }, p2 -> {
        return p2.getName();
    }, (oldValue, newValue) -> newValue));
    System.out.println(sexNameMap);
}
```


## Parallel Streams

As described below, streams can be executed serially or in parallel. The serial execution of the stream is done by a single thread. Parallel stream processing is executed simultaneously on multiple threads.

The following example will demonstrate how to significantly improve performance through parallel stream processing.

First we create a large collection of List elements:

```
Code: com.winterbe.java8.samples.stream.Stream_reduce

int max = 1000000;
List<String> values ​​= new ArrayList<>(max);
for (int i = 0; i <max; i++) {
    UUID uuid = UUID.randomUUID();
    values.add(uuid.toString());
}
```

Now we measure the time it takes to sort the streams of this collection.

### Serial Sort

```java
long t0 = System.nanoTime();

long count = values.stream().sorted().count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1-t0);
System.out.println(String.format("sequential sort took: %d ms", millis));

// sequential sort took: 899 ms
```

Code: com.winterbe.java8.samples.stream.Streams3

### Parallel Sort

```java
long t0 = System.nanoTime();

long count = values.parallelStream().sorted().count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1-t0);
System.out.println(String.format("parallel sort took: %d ms", millis));

// parallel sort took: 472 ms
```

Code: com.winterbe.java8.samples.stream.Streams3

As you can see, the two code snippets are almost the same. But parallel sorting can be about 50% faster. So what you have to do is replace `stream()` with `parallelStream()`.

## Collection Maps

As mentioned earlier, Maps does not directly support streams. The Map interface does not provide `stream()` related methods. However, you can use `map.keySet().stream()`, `map.values().stream()` and `map.entrySet().stream()` to create streams based on keys, values ​​or entries.

In addition, maps also provide some new and useful methods to support routine tasks.

Code: com.winterbe.java8.samples.misc.Maps1

```java
Map<Integer, String> map = new HashMap<>();

for (int i = 0; i <10; i++) {
    map.putIfAbsent(i, "val" + i);
}

map.forEach((id, val) -> System.out.println(val));
```

 If `putIfAbsent` is empty, execute put, otherwise return the value corresponding to the key, which can avoid the redundancy of a null judgment code. `forEach` is operated internally by BiConsumer.

The following example describes how map uses functions to calculate:

```java
map.computeIfPresent(3, (num, val) -> val + num);
map.get(3); // val33

map.computeIfPresent(9, (num, val) -> null);
map.containsKey(9); // false

map.computeIfAbsent(23, num -> "val" + num);
map.containsKey(23); // true

map.computeIfAbsent(3, num -> "bam");
map.get(3); // val33 (executed when missing)
```

Next, we learn how to delete entries by a given key, provided that it currently has a kv mapping:

```java

// The value corresponding to 3 is equal to "val3" before the delete action is performed
map.remove(3, "val3");
map.get(3); // val33

map.remove(3, "val33");
map.get(3); // null
```

Other useful methods:

```java
// Return the value associated with the key, otherwise return the following default value
map.getOrDefault(42, "not found"); // not found
```

Merging entries in the map is also very easy:

```java
map.merge(9, "val9", (value, newValue) -> value.concat(newValue));
map.get(9); // val9

map.merge(9, "concat", (value, newValue) -> value.concat(newValue));
map.get(9); // val9concat
```

If the key-value key-value pair does not exist, it is merged into the map. Otherwise, execute the function function to change its value.


## Date API

Java 8 provides a brand new date and time API, located under the `java.time` package. The new date API is equivalent to the [Joda-Time](http://www.joda.org/joda-time/) library, but it is also [not the same](http://blog.joda.org/2009/ 11/why-jsr-310-isn-joda-time_4941.html). The following example covers the most important parts of this new API.

### Clock

The clock provides access to the current date and time. The clock knows the time zone and can be used instead of `System.currentTimeMillis()` to retrieve the current time (in milliseconds) since Unix EPOCH. A certain moment on the timeline is represented by `Instant`. `Instant` can create legacy `java.util.Date` objects.

Clock provides access to the current date and time. Clocks are aware of a timezone and may be used instead of `System.currentTimeMillis()` to retrieve the current time in milliseconds since Unix EPOCH. Such an instantaneous point on the time-line is also represented by the class `Instant`. Instants can be used to create legacy `java.util.Date` objects.

```java
Clock clock = Clock.systemDefaultZone();
long millis = clock.millis();

Instant instant = clock.instant();
Date legacyDate = Date.from(instant); // legacy java.util.Date
```
Code: com.winterbe.java8.samples.time.LocalTime1

### Time zone

The time zone is represented by `ZoneId`, which provides many static methods. The time zone defines an important offset in the conversion between instantaneous and local date and time.

```java
System.out.println(ZoneId.getAvailableZoneIds());
// prints all available timezone ids

ZoneId zone1 = ZoneId.of("Europe/Berlin");
ZoneId zone2 = ZoneId.of("Brazil/East");
System.out.println(zone1.getRules());
System.out.println(zone2.getRules());

// ZoneRules[currentStandardOffset=+01:00]
// ZoneRules[currentStandardOffset=-03:00]
```

Code: com.winterbe.java8.samples.time.LocalTime1

### local time

LocalTime represents the time without a time zone, such as 10 PM or 17:30:15. The following example creates two local times with time zones. We compare these two times and calculate the difference in hours or minutes between the two.

```java
LocalTime now1 = LocalTime.now(zone1);
LocalTime now2 = LocalTime.now(zone2);

System.out.println(now1.isBefore(now2)); // false

long hoursBetween = ChronoUnit.HOURS.between(now1, now2);
long minutesBetween = ChronoUnit.MINUTES.between(now1, now2);

System.out.println(hoursBetween); // -3
System.out.println(minutesBetween); // -239
```

LocalTime provides many factory methods for creating various new instances, including parsing time strings.

```java
LocalTime late = LocalTime.of(23, 59, 59);
System.out.println(late); // 23:59:59

DateTimeFormatter germanFormatter =
    DateTimeFormatter
        .ofLocalizedTime(FormatStyle.SHORT)
        .withLocale(Locale.GERMAN);

LocalTime leetTime = LocalTime.parse("13:37", germanFormatter);
System.out.println(leetTime); // 13:37
```

Code: com.winterbe.java8.samples.time.LocalTime1

### Local Date

LocalDate represents a different date, such as 2014-03-11. It is immutable and completely similar to LocalTime. The example demonstrates adding or subtracting days, months, and years to calculate new dates. Remember, every operation will return a new instance.


```java
LocalDate today = LocalDate.now();
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
LocalDate yesterday = tomorrow.minusDays(2);

LocalDate independenceDay = LocalDate.of(2014, Month.JULY, 4);
DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
System.out.println(dayOfWeek); // FRIDAY
```

Parsing LocalDate from a string is as simple as parsing LocalTime:

```java
DateTimeFormatter germanFormatter =
    DateTimeFormatter
        .ofLocalizedDate(FormatStyle.MEDIUM)
        .withLocale(Locale.GERMAN);

LocalDate xmas = LocalDate.parse("24.12.2014", germanFormatter);
System.out.println(xmas); // 2014-12-24
```

Code: com.winterbe.java8.samples.time.LocalDate1

### Local date and time

LocalDateTime represents the date and time. It contains the above date and time to form an instance. `LocalDateTime` is immutable, it works like LocalTime and LocalDate. We can use date and time to retrieve certain fields:

Code: com.winterbe.java8.samples.time.LocalDateTime1

```java
LocalDateTime sylvester = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);

DayOfWeek dayOfWeek = sylvester.getDayOfWeek();
System.out.println(dayOfWeek); // WEDNESDAY

Month month = sylvester.getMonth();
System.out.println(month); // DECEMBER

long minuteOfDay = sylvester.getLong(ChronoField.MINUTE_OF_DAY);
System.out.println(minuteOfDay); // 1439
```

It can be converted to time through the additional information of the time zone. Easily convert the instance to an old date of type `java.util.Date`.

```java
Instant instant = sylvester
        .atZone(ZoneId.systemDefault())
        .toInstant();

Date legacyDate = Date.from(instant);
System.out.println(legacyDate); // Wed Dec 31 23:59:59 CET 2014
```
Formatting date-time is just like formatting date or time. Instead of using a predefined format, we can use a custom mode to create a formatter.

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MM dd, yyyy - HH:mm");
        LocalDateTime parsed = LocalDateTime.parse("07 25, 2017 - 14:00", formatter);
        System.out.println(parsed); // 2017-07-25T14:00
```

Unlike `java.text.NumberFormat`, `DateTimeFormatter` is immutable and **thread safe**.

For more syntax details, [Refer here](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html).


## Comment

The annotations in Java 8 are repeatable. Let us illustrate with an example.

First, we define a wrapper annotation, which contains an array of actual annotations:

Code: com.winterbe.java8.samples.misc.Annotations1

```java
@interface Hints {
    Hint[] value();
}

@Repeatable(Hints.class)
@interface Hint {
    String value();
}
```

Java 8 allows us to use multiple annotations of the same type by declaring the annotation @Repeatable.

### Form 1: Use container annotations (old way)

```java
@Hints({@Hint("hint1"), @Hint("hint2")})
class Person {}
```

### Form 2: Use repeatable annotations (new way)

```java
@Hint("hint1")
@Hint("hint2")
class Person {}
```

Form 2, the java compiler implicitly sets the @Hints annotation. This is very important for reading annotation information via reflection.
```java
Hint hint = Person.class.getAnnotation(Hint.class);
System.out.println(hint);                   // null

Hints hints1 = Person.class.getAnnotation(Hints.class);
System.out.println(hints1.value().length);  // 2

Hint[] hints2 = Person.class.getAnnotationsByType(Hint.class);
System.out.println(hints2.length);          // 2
```

Although we never defined `@Hints` annotation on the `Person` class, it can still be read with `getAnnotation(Hints.class)`. However, it is more convenient that `getAnnotationsByType` can access all annotations with `@Hint`.


In addition, Java 8 annotations have been extended with two new targets:


```java
@Target({ElementType.TYPE_PARAMETER, ElementType.TYPE_USE})
@interface MyAnnotation {}
```