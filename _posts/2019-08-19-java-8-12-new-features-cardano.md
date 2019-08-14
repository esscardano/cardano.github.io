---
layout: post
title: Java [8..12] new features
date: 2019-08-14
author: e.caceres
categories: java
description: Java 8,9,10,11 & 12 new features
---

<h2>In Java 8 major changes where introduced to Java. This is a review of the main new features from Java 8 to 12.</h2>
<p>&nbsp;</p>
<table>
<tbody>
<tr style="height: 221.5px;">
<td style="width: 250px; height: 221.5px;" valign="top">
<h2>Java 8</h2>
<ul>
<li><a href="#interfaces">Interaces</a></li>
<li><a href="#functionalprogramming">Functional programming</a></li>
<li><a href="#optional">Optional</a></li>
<li><a href="#dateapi">New Date API</a></li>
<li><a href="#newstring8">New String</a></li>
<li><a href="#tryresources">Try-with-resources</a></li>
<li><a href="#io8">New I/O methods</a></li>
<li><a href="#collections8">New Collections methods&nbsp;</a></li>
</ul>
</td>
<td style="width: 300px; height: 221.5px;" valign="top">
<h2>Java 9</h2>
<ul>
<li><a href="#j91">Collection Factory Methods</a></li>
<li><a href="#j92">Stream API</a></li>
<li><a href="#j93">Private interface methods</a></li>
<li><a href="#j94">HttpClient</a></li>
<li><a href="#j95">Reactive Streams</a></li>
</ul>
</td>
<td style="width: 400px; height: 221.5px;" valign="top">
<h2>Java 10</h2>
<ul>
<li>Local-variable Type Inference // Type inferred by compiler <em>var name = "test";</em></li>
<li>Improved Garbage Collector with G1GC</a></li>
<li>Application Class Data Sharing (for different VMs running the same code or repeated executions)</li>
<li>Improved Container Awareness</li>
</ul>
</td>
<td style="width: 400px; height: 221.5px;" valign="top">
<h2>Java 11</h2>
<ul>
<li><a href="#j111">Scripts execution</a></li>
<li><a href="#j112">Unmodifiable Collections</a></li>
<li><a href="#j113">Annotations in "var"</a></li>
<li><a href="#j114">New methods</a></li>
<li>Opened Java Flight Reccorder and Mission Control for JVM analytics</li>
<li>TSL 1.3 support</li>
<li>Removed:&nbsp;JTA, JAX-WS, JAXB, CORBA, Applets, Java Web Start</li>
</ul>
</td>
<td style="width: 500px; height: 221.5px;" valign="top">
<h2>Java 12</h2>
<ul>
<li><a href="#j121">Strings and compact numbers</a></li>
<li><a href="#j122">Switch expressions</a></li>
<li><a href="#j123">Micro-benchmarking suite with JMH</a></li>
<li><a href="https://blog.idrsolutions.com/2019/03/java-12s-jvm-constants-api-explained-in-5-minutes/">JVM Constants API</a></li>
</ul>
</td>
</tr>
</tbody>
</table>
<p>&nbsp;</p>
<p>&nbsp;</p>
<h2><span style="text-decoration: underline;">Java 8</span></h2>
<h3 id="interfaces">Interfaces</h3>
<p>Prior to java 8, interface in Java can only have abstract methods. Now <strong>default</strong> and <strong>static</strong> methods are allowed in.</p>
<pre><code>
interface MyInterface{
    // Default method do not need to be implemented the implementation classes
    default void newMethod() {
        System.out.println("Newly added default method");
    }

    // Static method in interface is similar to default method except that we
    // cannot override them in the implementation classes.
    static void anotherNewMethod() {
    	System.out.println("Newly added static method");
    }

    // Typical abstract method that must be implemented in the implementation classes
    void existingMethod(String str);
}
</code></pre>
<h3 id="functionalprogramming">Functional programming</h3>
<h4>Functional interface (@FunctionalInterface)</h4>
<p>Interface with only one abstract method (default or static methods don't count!!!)</p>
<pre><code>@FunctionalInterface public interface Comparator {
  int compare(T o1, T o2);
  // Methods implemented from Object (e.g. equals)
  boolean equals(Object obj);
}</code></pre>
<h3>Lambda expression</h3>
<p>Implementation of an anonymous class. The type is a functional interface. Do not creates an object (not using new) so it is much cheaper (do not need to gets the memory, static and non-static blocks, executes the constructor and all super constructors). It is an object without identity.</p>
<pre><code>// (params) -&gt; {body}
Comparator c = (String s1, String s2) -&gt; Integer.compare(s1.length(), s2.length));
// Compiler can infer the type
Comparator c = (s1, s2) -&gt; Integer.compare(s1.length(), s2.length));

// using an anonymous class
Runnable r = new Runnable() {
  @Override
  public void run() {
    System.out.println("Hello");
  }
// using a lambda expression
Runnable r2 = () -&gt; System.out.println("Hello");};

//Anonymous Consumer (this  instantiate an implementation of the Consumer interface using an anonymous class) vs Lambda
Consumer&lt;String&gt; printConsumer= new Consumer&lt;String&gt;() {
    public void accept(String name) {
        System.out.println(name);
    }
}; // same than: name -&gt; System.out.println(name)</code></pre>
<h3>Functional classes predefined</h3>
<ul>
<li>Supplier(() -&gt; T)</li>
<li>Consumer(T -&gt; ()), BiConsumer((T , Z) -&gt; ())</li>
<li>Predicate (T -&gt; Boolean), BiPredicate((T , Z) -&gt; Boolean)</li>
<li>Function((T -&gt; Z))</li>
<li>BiFunction((T , Z -&gt; U))</li>
<li>UnaryOperator((T -&gt; T))</li>
<li>BinaryOperator((T , T -&gt; T))</li>
</ul>
<pre><code>
Predicate p1 = s -&gt; s.length() &lt; 20;
Predicate p2 = s -&gt; s.length() &gt; 10;
Predicate p3 = p1.and(p2);

List strings = Arrays.asList("one", "two");
List list = new ArrayList&lt;&gt;();
Consumer print = System.out::println;
Consumer add = list::add;
// Channing consumers!
strings.forEach(print.andThen(add));</code></pre>
<h3>Method reference</h3>
<p>Simplified way to write some labmda expresions</p>
<pre><code>a -&gt; Class.method(a) === Class::method
Comparator c = (i1,i2) -&gt; Integer.compare(i1, i2)
Comparator c = Integer::compare
</code></pre>
<h3>Streams</h3>
<p>A&nbsp;<strong>Stream in Java</strong>&nbsp;can be defined as a sequence of elements from a source that supports aggregate operations on them.&nbsp;</p>
<p>Can be executed only once (the terminate operation closes the stream so only one terminate operation can be executed for one stream). Only declares a pipeline (<strong>very cheap</strong> -until terminate is executed-)</p>
<p>It is a way to process data (in parallel and pipelined). There are 3 kinds of operations: Stream, intermediary and terminate.</p>
<h4>Stream operation</h4>
<p>Creates the stream. A stream is lazy executed. This means it is not executed until the terminate operation does. Then the stream is closed (only can be executed once)</p>
<h4>Intermediary operations</h4>
<p>They are lazy. Only declares a pipeline, do not executes the operation. Returns a stream (Stream&lt;T&gt;).</p>
<ul>
<li>map(Function&lt;T,R&gt;): Applies a function over the Stream. <em>people.map(person -&gt; person.getName())</em></li>
<li>filter(Predicate&lt;T&gt;): Filter elements depending on a Predicate(T -&gt; Boolean). <em>people.filter(person -&gt; person.getName().startsWith("A"))</em></li>
<li>flatMap(Function&lt;T,Stream&lt;R&gt;&gt; flatMapper): Flats object of objects into objects. Unwraps types to apply to Stream operations that do not support them.
<pre class="language-java code-toolbar"><code class=" language-java">Stream<span class="token operator">&lt;</span>String<span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token operator">&gt;</span>	<span class="token operator">-</span><span class="token operator">&gt;</span> flatMap <span class="token operator">-</span><span class="token operator">&gt;</span>	Stream<span class="token operator">&lt;</span>String<span class="token operator">&gt;</span>
Stream<span class="token operator">&lt;</span>Set<span class="token operator">&lt;</span>String<span class="token operator">&gt;&gt;</span>	<span class="token operator">-</span><span class="token operator">&gt;</span> flatMap <span class="token operator">-</span><span class="token operator">&gt;</span>	Stream<span class="token operator">&lt;</span>String<span class="token operator">&gt;</span>
{ {'Jose','John'}, {'Charles'}, {'Mr.T'} } -&gt; flatMap -&gt; {'Jose','John','Charles','Mr.T'}
// List all people of a list of teams (containing a List&lt;String&gt; people)
Function&lt;List&lt;Integer&gt;,Stream&lt;Integer&gt; flatmapper = l -&gt; l.stream();
teams.stream().map(team -&gt; team.getPeople()) // returns Stream&lt;List&lt;String&gt;&gt;
              .flatmap(flatmapper) // returns Stream&lt;String&gt;
              .distinct()
              .collect(Collectors.List());</code></pre>
</li>
<li>peek(Consumer&lt;T&gt;)<code class=" language-java"></code></li>
</ul>
<h4>Teminate operation</h4>
<p>Executes all the stream pipeline and then closes the Stream. So the Stream can be executed only once.&nbsp; Examples:</p>
<ul>
<li>forEach(Consumer&lt;? super T&gt; action): Executes a function for each element of the collection.</li>
<li>collect:&nbsp; Mutable reduction. Put results into container like list. <em>collect(Collectors.toList())</em> <em>collect(Collectors.groupingBy(Person::Age))</em> // this returns a map where the key are ages and values people of this values.</li>
<li>reduce (Object identity, BinaryOperator&lt;T&gt; accumulator).&nbsp;Identity is the initial value.&nbsp; BinaryOperator extends BiFunction with (T, T) -&gt; T.&nbsp;The reduction of a stream empty argument is the identity. If has only one argument then is this argument. If it is executed in parallel needs a combiner.</li>
<li>Aggregation: min,max,sum, count...</li>
<li>Boolean: allMatch,noneMatch,anyMatch...</li>
<li>Optional: findFirst(), findAny()...</li>
</ul>
<p>Examples</p>
<pre><code>
people.stream().forEach(System.out::println);
Map&lt;Integer,List&gt; result = persons.stream().collect(Collectors.groupingBy(Person::getAge());
Map&lt;Integer,Long&gt; result = persons.stream().collect(Collectors.groupingBy(Person::getAge(),Collectors.counting()); //downstream collector counts the number of people of each age
Map&lt;Integer,List&gt; result = persons.stream().collect(Collectors.groupingBy(Person::getName(),Collectors.toList());

ages.stream().reduce(0,(age1,age2) -&gt; age1+age2); (T,T-&gt;T)
[-10,-5].stream().reduce(0, Integer::max) // max(0,max(-10,-5))) = 0<br>
</code></pre>
<h3 id="optional">Optional</h3>
<p>Store an Object or a void value.</p>
<pre><code>Optional opt = Optional.ofNullable(myNullableString);
List people = ...
Optional minAgeBiggerThan20 = people.stream()
                      .map(person -&gt; person.getAge()) // Stream, intermediary
                      .filter(age -&gt; age &gt; 20) // Stream, intermediary
                      .min(Comparator.naturalOrder()); // Optional, terminal
</code></pre>
<h3 id="dateapi">New Date API</h3>
<ul>
<li>New package: java.time</li>
<li>Precision is nanosecond</li>
<li>Instant.MIN 1 billions years ago / Instant.MAX Dec 31 of the year 1.000.000.000</li>
<li>New types: Instant, Duration, LocalDate, Period, LocalTime, ZoneDateTime</li>
<li>Old Date is interoperable with the new types e.g. <em>Date.from(instant);</em></li>
</ul>
<pre><code>Instant instant = Instant.now() //is the current instant. Is inmutable (small cost)
Duration elapsed = Duration.between(startInstant, endInstant); // amount of time between two instants
long millis = elapsed.toMillis();

// LocalDate: Is just a date with a day presition (not nanosecond presition)
LocalDate now = LocalDate.now();
LocalDate birth = LocalDate.of(1985, Month.APRIL, 23);
Period: Amount of time between to LocalDates;
Period p = birth.until(LocalDate.now(); p.getYears();
long days = birth.until(LocalDate.now(), ChronoUnit.DAYS);

// DateAdjuster: Useful to add or substract amounts of time to Instants or LocalDate
LocalDate nextSunday = LocalDate.now().with(TemporalAdjusters.next(DayOfWeek.SUNDAY));

LocalTime: Is just a time of a day
LocalTime time = LocalTime.of(10, 20); // 10:20

// Zone Time: Implements the time different time zones
String ukTimeZone= ZoneId.getAvailableZoneIds().of("Europe/London");
ZonedDateTime.of(1943, Month.APRIL.getValue(), 23, // year month day
		 10, 0, 0, 0,			   // h / mn / s / nanos
                 ZoneIf.of("Europe/London")); // 1943-04-23T10:00-00:01:15[Europe/London]

ZonedDateTime z = ZonedDateTime.of(LocalDate.of(2014,Month.MARCH,12), Localtime.of(9,30), ZoneId.of("Europe/London"));
z.plus(Period.ofMonth(1));
z.withZoneSameInstant(ZoneId.of("US/Central")); // change time zone

System.out.println(DateTimeFormatter.ISO_DATE_TIME.format(z)); // format date to print it

//Use java.time instead java.util.Date
LocalDate date1 = LocalDate.of(1985,12,5);
date1.datesUntil(LocalDate.now(),Period.ofYears(1)).map(date -&gt; Year.of(date.getYear())).count():
</code></pre>
<h3 id="newstring8">String</h3>
<pre><code>// Stream of chars
IntStream stream = "Hello".chars();
stream.mapToObj(letter -&gt; (char)letter).map(Character::toUpperCase).forEach(System.out::print);

//Concatenation: Use StringJoiner
StringJoiner sj = new StringJoiner(", ");
sj.add("one").add("two").add("three"); // one, two, three
StringJoiner sj = new StringJoiner(", ", "{","}");
sj.add("one").add("two").add("three"); // {one, two, three}
String.join(", ","one","two","three");
</code></pre>
<h3 id="tryresources">Try with resources</h3>
<p>Autoclose the try enclosed object. Implements AutoCloseable interface (Java 7)</p>
<pre><code>Scanner scanner = null;
try {
    scanner = new Scanner(new File("test.txt"));
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
    // Multiple catch exception also added in Java 7
} catch (FileNotFoundException | IOException e) {
    e.printStackTrace();
} finally {
    if (scanner != null) {
        scanner.close();
    }
}
// this is equivalent, and closes automatically the scanner object since it is inside the try-with-resources
try (Scanner scanner = new Scanner(new File("test.txt"))) {
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException fnfe) {
    fnfe.printStackTrace();
}
</code></pre>
<h3 id="io8">Java I/O</h3>
<p>Reading text files (File.lines)</p>
<pre><code>
// try with resources from Java 7 + Files.lines from Java 8
// Stream implements AutoCloseable and closes the file
Path path = Paths.get("d:", "tmp", "debug.log");
try (Stream&lt;String&gt; stream = Files.lines(path)) {
	stream.filter(line -&gt; line.contains("ERROR")).findFirst().ifPresent(System.out::println);
} catch (IOException ioe) {
	// handle exception
}

Reading directory entries (File.list) - Only visitrs first level / Files.walk -&gt; visits the whole subtree Files or the deph level set Files.walk(path,2);
try (Stream&lt;String&gt; stream = Files.list(path)) {
	stream.filter(path -&gt; path.toFile().isDirectory()).forEach(System.out::println);
} catch (IOException ioe) {
	// handle exception
}
</code></pre>
<h3 id="collections8">Collections</h3>
<p>Reading text files (File.lines)</p>
<pre><code>
Collection&lt;String&gt; list = new ArrayList&lt;&gt;(strings);
boolean b = list.removeIf(s -&gt; s.length() &gt; 4); // returns if the list was modified
list.replaceAll(String::toUpperCase); // modifies the list
list.sort(Comparator.naturalOrder()); // order the list using the comparator
list.stream().collect(Collectors.joining(", "));

// Comparator:
Comparator&lt;Person&gt; compareLastName = Comparator.comparing(Person::getLastName).thenComparing(Person::getFirstName);
compareLastName.reversed(); // compares the reverse
Comparator c = Comparator.naturalOrder();
Comparator c = Comparator.nullsFirst(Comparator.naturalOrder()); // or nullsLast

// Utility methods:
long max = Long.max(1L,2L);
BinaryOperator sum = (l1, l2) -&gt; l1 + l2; // = Long::sum;
int hash = new Long.hashCode(3141592653589793238L); // -1985256439

Map&lt;String, Person&gt; map = ...
Person p = map.getOrDefault(key,defaultPerson); // prevents a null
map.putIfAbsent(key,person);
map.replace(key,person);<br><br><br></code></pre>
<h2><span style="text-decoration: underline;">Java 9</span></h2>
<h3 id="j91">Collection Factory Methods</h3>
<p>Initialization methods for collections that creates Inmutable Collection (In Java 11 the name is changed to&nbsp;Unmodifiable Collection because although the collection itself is not modifiable, the elements contained are not inmutable)</p>
<pre><code>
List&lt;Integer&gt; list = List.of(1,2,3)
Set&lt;Integer&gt; set = Set.of(1,2,3)
Map&lt;String,Integer&gt; map = Map.of("Key1",1,"Key2",2) // up to 10 key/value  -&gt; remember iteration order not guaranteed
Map.ofEntries(Map.entry("key","value"), Map.entry("key","value") // no limit

list.add(4) -&gt; UnsupportedOperationException
list.getClass() -&gt; InmutableCollection$ListN

List.of(1).getClass() -&gt; InmutableCollection$List1 (optimized implementation for single/dual collections)<br>
</code></pre>
<h3 id="j92">Stream API</h3>
<p>New methods added in the Stream API.</p>
<pre><code>
// For ordered streams
takeWhile(Predicate &lt;? super T&gt; p) -&gt; list.takeWhile(a -&gt; a &lt; 4)  [1,4,5] -&gt; [1]
dropWhile(Predicate &lt;? super T&gt; p) -&gt; drop.takeWhile(a -&gt; a &lt; 4)  [1,4,5] -&gt; [4,5]

// Print code inside comments -&gt; File.lines -&gt; Stream
Files.lines(Path.get("file.html")).dropWhile(l -&gt; !l.contains("&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;")).skip(1).takeWhile(l -&gt; !l.contains("&gt;&gt;&gt;&gt;&gt;")).forEach(...);

// ofNullable(T t)
Stream.ofNullable(null).count() -&gt; 0 // Stream.ofNullable(getBook()).count() -&gt; 1
Stream.ofNullable(getPossibleNull()).flatmap(b -&gt; b.authors.stream()).forEach(System.out::println);

iterate(T seed, Predicate &lt;? super T&gt; hasNext, UnaryOperator next)

// Stream collectors: collect -&gt; Transform Stream into a Collection
Stream.of(1,2,3).map(n -&gt; n + 1).collect(Collectors.toList()); -&gt; [2,3,4]
Stream.of(1,2,3,4).collect(groupingBy(i -&gt; i % 2, toList())); -&gt; {0=[2], 1=[1,3,3]}

// Optional: Holds single value or null, new methods added
ifPresentOrElse(Consumer action, Runnable emptyAction)
or(Supplier&lt;Optional&lt;T&gt;&gt; supplier)
Stream&lt;T&gt; stream()

Stream&lt;Optional&lt;Integer&gt;&gt; optionals = Stream.of(Optional.of(1), Optional.empty(), Optional.of(2));
Stream ints = optionals.flatMap(Optional::steam);
ints.forEach(System.out::print); -&gt; 1 2

myObject.ifPresentOrElse(System.out::println, () -&gt; System.out.println("Object null"));
</code></pre>
<h3 id="j93">Private interface methods</h3>
<p>New methods added in the Stream API.</p>
<pre><code>
public interface Price {
	double getPrice();

	default int getDefaultPrice1() {
		return 1 + defaultPriceInternal();
	}

	default int getDefaultPrice2() {
		return 2 + defaultPriceInternal();
	}

	private int defaultPriceInternal() { // vision only in the interface
		return 1;
	}
}
</code></pre>
<h3 id="j94">Http Client</h3>
<p>New HTTP API.</p>
<ul>
<li>Supports HTTP2/Websockets</li>
<li>Reactive Streams Integration</li>
<li>1. HttpClient.Builder</li>
<li>2. HttpRequest.Builder</li>
<li>3. HttpResponse.Builder</li>
</ul>
<pre><code>
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder(URI.create("https://...")).GET().build();
//Blocking synchronous
HttpResponse response = client.send(request,HttpResponse.BodyHandler.asString());
// Non-blocking asyncrhonous
CompletableFuture&lt;HttpResponse&gt; response = client.sendAsync(request,HttpResponse.BodyHandler.asString());
response.thenAccept(r-&gt; {System.out.println(r.body())});
response.join(); // waits for the completable future to be completed
</code></pre>
<h3 id="j95">Reactive Streams</h3>
<pre>Flow API - To be usable for other thrid party libraries (Akka Streams, RxJava2, Spring 5)<br><br></pre>
<p>&nbsp;</p>
<h2><span style="text-decoration: underline;">Java 11</span></h2>
<h3 id="j111">Scripting</h3>
<p>Java 11 can be executed as script directly in the OS.</p>
<pre><code>// With the shebang we can execute scripts directly in command line:
./listfiles

#!/usr/bin/java --source 11
import java.nio.file.*;

public class ListFiles {
	public static void main(String[] args) throws Exception {
		Files.walk(Paths.get(args[0])).forEach(System.out::println);
	}
}
</code></pre>
<h3 id="j112">Unmodifiable Collections</h3>
<p>Java 11 changes from inmutable to unmodifiable because although the object itself is inmutable, the objects contained by the unmodifiable collection can be mutable (e.g. UnmodifiableList of Person -mutable-).&nbsp;<a href="https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html#unmodifiable">https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html#unmodifiable</a></p>
<h3 id="j113">Annotations in "var"</h3>
<p>Use of var to allow annotations in lambdas (@Nonnull var a, @Nullable var b) -&gt; a.concat(b)</p>
<h3 id="j114">New methods</h3>
<pre><code>// isBlank, repeat, strip, trim
"".isBlank() -&gt; true
"na".repeat(3) -&gt; "nanana"
"\n\t  text \u2005".strip() -&gt; "text"
"\n\t  text \u2005".trim() -&gt; "text \u2005"
"1\n2\n".lines().forEach(System.println::out) -&gt;
1
2
strings.stream().filter(Predicate.not(String::isBlank))).forEach(...)
Optional.isEmpty() -&gt; true if empty object
</code></pre>
<h2 style="text-decoration: underline;">Java 12</h2>
<h3 id="j121">String and compact numbers</h3>
<pre><code>// New String methods
"hello".indent(2) -&gt; "  hello\n"
"1.aaa\n2.bbb"indent(5).lines().forEach(System.out::println) -&gt;
     1.aaa
     2.bbb

String::transform
StringUtils.words(StringUtils.clean(text));

// Compact numbers
// 1K -&gt; 1000
// 1M -&gt; 1000000

Teeing Collector. Collectors.teeing(c1,c2,combiner)

Stream.of(10,20,30).collect(Collectors.teeing(
	Collectors.summingInt(Integer::valueOf),
	Collectors.counting(),
	(sum, count) -&gt; sum / count)); -&gt; 10+20+30 / 3

Switch expressions

String monthName = switch(monthNumber) {
	case 1 -&gt; monthName = "January";
	case 2,3 -&gt; {String month = "February"; break monnth;}
	default -&gt; monthName = "Unknown";
};
</code></pre>
<h3 id="j122">Switch Expressions</h3>
<pre><code>Switch expressions

String monthName = switch(monthNumber) {
	case 1 -&gt; monthName = "January";
	case 2,3 -&gt; {String month = "February"; break monnth;}
	default -&gt; monthName = "Unknown";
};
</code></pre>
<h3 id="j123">Micro-benchmarking suite with JMH</h3>
<pre><code>@Benchmarkmode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@State(Scope.Benchmark)
@Fork(1) //number of JVM forked
// @Setup @Teardown
public class mybenchmark {
@Benchmark
public void testMethod() {}
}
</pre></code></body></html>