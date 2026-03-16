# ☕ Java Quick Reference

## Primitive Data Types

| Type    | Size   | Range / Notes              |
|---------|--------|----------------------------|
| `byte`  | 8-bit  | -128 to 127                |
| `short` | 16-bit | -32,768 to 32,767          |
| `int`   | 32-bit | ~-2B to 2B                 |
| `long`  | 64-bit | append `L`: `123L`         |
| `float` | 32-bit | append `f`: `3.14f`        |
| `double`| 64-bit | default decimal            |
| `char`  | 16-bit | Unicode character: `'A'`   |
| `boolean`| 1-bit | `true` / `false`           |

## Variables & Arrays

```java
int x = 10;
final double PI = 3.14159;   // constant

int[] arr = {1, 2, 3};
int[] arr = new int[5];
int[][] matrix = new int[3][4];

// Array utilities
Arrays.sort(arr);
Arrays.fill(arr, 0);
int[] copy = Arrays.copyOf(arr, arr.length);
```

## Control Flow

```java
// if / else
if (x > 0) {
    // ...
} else if (x == 0) {
    // ...
} else {
    // ...
}

// switch (Java 14+ switch expression)
String result = switch (day) {
    case "MON", "TUE", "WED" -> "Weekday";
    case "SAT", "SUN"        -> "Weekend";
    default                  -> "Unknown";
};

// for
for (int i = 0; i < 10; i++) { }

// enhanced for
for (String s : list) { }

// while / do-while
while (cond) { }
do { } while (cond);
```

## Classes & OOP

```java
public class Animal {
    private String name;          // encapsulation
    protected int age;

    public Animal(String name, int age) {
        this.name = name;
        this.age  = age;
    }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String speak() { return "..."; }

    @Override
    public String toString() {
        return "Animal{name=" + name + "}";
    }
}

// Inheritance
public class Dog extends Animal {
    public Dog(String name) {
        super(name, 0);
    }

    @Override
    public String speak() { return "Woof!"; }
}

// Interface
public interface Flyable {
    void fly();
    default void land() { System.out.println("Landing"); }
}

// Abstract class
public abstract class Shape {
    abstract double area();
}
```

## Collections Framework

```java
import java.util.*;

// List
List<String> list = new ArrayList<>();
list.add("a"); list.remove("a");
list.get(0);   list.size();
Collections.sort(list);

LinkedList<Integer> ll = new LinkedList<>();
ll.addFirst(1); ll.addLast(2);
ll.poll(); ll.peek();

// Set
Set<String> set = new HashSet<>();
Set<String> sortedSet = new TreeSet<>();

// Map
Map<String, Integer> map = new HashMap<>();
map.put("key", 1);
map.get("key");
map.getOrDefault("key", 0);
map.containsKey("key");
for (Map.Entry<String, Integer> e : map.entrySet()) {
    System.out.println(e.getKey() + "=" + e.getValue());
}

// Queue / Deque
Queue<Integer> q = new LinkedList<>();
q.offer(1); q.poll(); q.peek();

Deque<Integer> dq = new ArrayDeque<>();
dq.offerFirst(1); dq.offerLast(2);

// Priority Queue (min-heap by default)
PriorityQueue<Integer> pq = new PriorityQueue<>();
PriorityQueue<Integer> maxPQ = new PriorityQueue<>(Collections.reverseOrder());
```

## Strings

```java
String s = "Hello World";
s.length();
s.charAt(0);
s.substring(0, 5);
s.indexOf("lo");
s.contains("World");
s.toLowerCase(); s.toUpperCase();
s.trim(); s.strip();
s.replace("Hello", "Hi");
s.split(" ");
String.join(", ", list);
s.toCharArray();
Integer.parseInt("42");
String.valueOf(42);

// StringBuilder (mutable, not thread-safe)
StringBuilder sb = new StringBuilder();
sb.append("Hello").append(" ").append("World");
sb.insert(5, ",");
sb.delete(5, 6);
sb.reverse();
sb.toString();
```

## Exception Handling

```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.err.println(e.getMessage());
} catch (Exception e) {
    e.printStackTrace();
} finally {
    // always runs
}

// Custom exception
public class CustomException extends RuntimeException {
    public CustomException(String msg) { super(msg); }
}

// try-with-resources
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line = br.readLine();
}
```

## Generics

```java
public class Box<T> {
    private T value;
    public Box(T value) { this.value = value; }
    public T getValue()  { return value; }
}

// Bounded type
public <T extends Comparable<T>> T max(T a, T b) {
    return a.compareTo(b) >= 0 ? a : b;
}

// Wildcard
void print(List<?> list) { }
void addNumbers(List<? super Integer> list) { }
```

## Streams (Java 8+)

```java
import java.util.stream.*;

List<Integer> numbers = List.of(1, 2, 3, 4, 5);

// filter + map + collect
List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .collect(Collectors.toList());

// reduce
int sum = numbers.stream().reduce(0, Integer::sum);

// groupingBy
Map<Boolean, List<Integer>> grouped = numbers.stream()
    .collect(Collectors.groupingBy(n -> n % 2 == 0));

// sorted, distinct, limit, skip
numbers.stream().sorted().distinct().limit(3).forEach(System.out::println);

// Optional
Optional<Integer> first = numbers.stream().filter(n -> n > 3).findFirst();
first.ifPresent(System.out::println);
int val = first.orElse(-1);
```

## Functional Interfaces (Java 8+)

```java
// Function<T, R>
Function<String, Integer> len = String::length;

// Predicate<T>
Predicate<Integer> isEven = n -> n % 2 == 0;

// Consumer<T>
Consumer<String> print = System.out::println;

// Supplier<T>
Supplier<List<String>> listFactory = ArrayList::new;

// BiFunction<T, U, R>
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
```

## Concurrency Basics

```java
// Thread
Thread t = new Thread(() -> System.out.println("Running"));
t.start();

// synchronized
synchronized (lock) { ... }
public synchronized void method() { ... }

// Executor Service
ExecutorService executor = Executors.newFixedThreadPool(4);
executor.submit(() -> doWork());
executor.shutdown();

// Callable + Future
Future<Integer> future = executor.submit(() -> computeValue());
Integer result = future.get();  // blocks

// Atomic
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();
```

## Java Memory Model Keywords

| Keyword      | Purpose |
|--------------|---------|
| `volatile`   | Ensures visibility across threads |
| `synchronized` | Mutual exclusion |
| `transient`  | Skip field during serialization |
| `static`     | Class-level (not instance) |
| `final`      | Immutable variable / no override / no subclass |
