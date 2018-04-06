# StreamPractices
Practices on java Streams
Classes to support functional-style operations on streams of elements, such as map-reduce transformations on collections. For example: 
     int sum = widgets.stream()
                      .filter(b -> b.getColor() == RED)
                      .mapToInt(b -> b.getWeight())
                      .sum();
 

Here we use widgets, a Collection<Widget>, as a source for a stream, and then perform a filter-map-reduce on the stream to obtain the sum of the weights of the red widgets. (Summation is an example of a reduction operation.) 

The key abstraction introduced in this package is stream. The classes Stream, IntStream, LongStream, and DoubleStream are streams over objects and the primitive int, long and double types. Streams differ from collections in several ways: 
•No storage. A stream is not a data structure that stores elements; instead, it conveys elements from a source such as a data structure, an array, a generator function, or an I/O channel, through a pipeline of computational operations.
•Functional in nature. An operation on a stream produces a result, but does not modify its source. For example, filtering a Stream obtained from a collection produces a new Stream without the filtered elements, rather than removing elements from the source collection.
•Laziness-seeking. Many stream operations, such as filtering, mapping, or duplicate removal, can be implemented lazily, exposing opportunities for optimization. For example, "find the first String with three consecutive vowels" need not examine all the input strings. Stream operations are divided into intermediate (Stream-producing) operations and terminal (value- or side-effect-producing) operations. Intermediate operations are always lazy.
•Possibly unbounded. While collections have a finite size, streams need not. Short-circuiting operations such as limit(n) or findFirst() can allow computations on infinite streams to complete in finite time.
•Consumable. The elements of a stream are only visited once during the life of a stream. Like an Iterator, a new stream must be generated to revisit the same elements of the source. 
Streams can be obtained in a number of ways. Some examples include: •From a Collection via the stream() and parallelStream() methods;
•From an array via Arrays.stream(Object[]);
•From static factory methods on the stream classes, such as Stream.of(Object[]), IntStream.range(int, int) or Stream.iterate(Object, UnaryOperator);
•The lines of a file can be obtained from BufferedReader.lines();
•Streams of file paths can be obtained from methods in Files;
•Streams of random numbers can be obtained from Random.ints();
•Numerous other stream-bearing methods in the JDK, including BitSet.stream(), Pattern.splitAsStream(java.lang.CharSequence), and JarFile.stream().

Additional stream sources can be provided by third-party libraries using these techniques. 

Stream operations and pipelines
Stream operations are divided into intermediate and terminal operations, and are combined to form stream pipelines. A stream pipeline consists of a source (such as a Collection, an array, a generator function, or an I/O channel); followed by zero or more intermediate operations such as Stream.filter or Stream.map; and a terminal operation such as Stream.forEach or Stream.reduce. 
Intermediate operations return a new stream. They are always lazy; executing an intermediate operation such as filter() does not actually perform any filtering, but instead creates a new stream that, when traversed, contains the elements of the initial stream that match the given predicate. Traversal of the pipeline source does not begin until the terminal operation of the pipeline is executed. 
Terminal operations, such as Stream.forEach or IntStream.sum, may traverse the stream to produce a result or a side-effect. After the terminal operation is performed, the stream pipeline is considered consumed, and can no longer be used; if you need to traverse the same data source again, you must return to the data source to get a new stream. In almost all cases, terminal operations are eager, completing their traversal of the data source and processing of the pipeline before returning. Only the terminal operations iterator() and spliterator() are not; these are provided as an "escape hatch" to enable arbitrary client-controlled pipeline traversals in the event that the existing operations are not sufficient to the task.

Parallelism

Processing elements with an explicit for-loop is inherently serial. Streams facilitate parallel execution by reframing the computation as a pipeline of aggregate operations, rather than as imperative operations on each individual element. All streams operations can execute either in serial or in parallel. The stream implementations in the JDK create serial streams unless parallelism is explicitly requested. For example, Collection has methods Collection.stream() and Collection.parallelStream(), which produce sequential and parallel streams respectively; other stream-bearing methods such as IntStream.range(int, int) produce sequential streams but these streams can be efficiently parallelized by invoking their BaseStream.parallel() method. To execute the prior "sum of weights of widgets" query in parallel, we would do: 

     int sumOfWeights = widgets.parallelStream()
                               .filter(b -> b.getColor() == RED)
                               .mapToInt(b -> b.getWeight())
                               .sum();


Reduction operations
A reduction operation (also called a fold) takes a sequence of input elements and combines them into a single summary result by repeated application of a combining operation, such as finding the sum or maximum of a set of numbers, or accumulating elements into a list. The streams classes have multiple forms of general reduction operations, called reduce() and collect(), as well as multiple specialized reduction forms such as sum(), max(), or count(). 
Of course, such operations can be readily implemented as simple sequential loops, as in: 

    int sum = 0;
    for (int x : numbers) {
       sum += x;
    }
 
However, there are good reasons to prefer a reduce operation over a mutative accumulation such as the above. Not only is a reduction "more abstract" -- it operates on the stream as a whole rather than individual elements -- but a properly constructed reduce operation is inherently parallelizable, so long as the function(s) used to process the elements are associative and stateless. For example, given a stream of numbers for which we want to find the sum, we can write: 
    int sum = numbers.stream().reduce(0, (x,y) -> x+y);
 
or: 
    int sum = numbers.stream().reduce(0, Integer::sum);
 

These reduction operations can run safely in parallel with almost no modification: 

    int sum = numbers.parallelStream().reduce(0, Integer::sum);


Mutable reduction
A mutable reduction operation accumulates input elements into a mutable result container, such as a Collection or StringBuilder, as it processes the elements in the stream. 
If we wanted to take a stream of strings and concatenate them into a single long string, we could achieve this with ordinary reduction: 

     String concatenated = strings.reduce("", String::concat)

The mutable reduction operation is called collect(), as it collects together the desired results into a result container such as a Collection. A collect operation requires three functions: a supplier function to construct new instances of the result container, an accumulator function to incorporate an input element into a result container, and a combining function to merge the contents of one result container into another. The form of this is very similar to the general form of ordinary reduction: 

 <R> R collect(Supplier<R> supplier,
               BiConsumer<R, ? super T> accumulator,
               BiConsumer<R, R> combiner);

