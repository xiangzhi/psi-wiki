The ability to manipulate streams of data plays a central role in the Platform for Situated Intelligence (\psi) framework. _Stream operators_ are simple \psi components that transform an input stream into an output stream. The framework provides a variety of such operators, and the document below provides a brief introduction to some of the most frequently used ones.  

A [quick reference summary](#ReferenceTable) that briefly summarizes the operators discussed below is available at the end this tutorial. Further reading and pointers to more complex stream operators are also available at [the end of the document](#FurtherReading).

__Note__: Prior to reading this document, it would be helpful that you familiarize yourself with the core concepts of the \\psi framework, by reading the [Brief Introduction](Brief-Introduction) tutorial.

# The Basic Stream Operators

## `Select`, `Where` and `Do`

Perhaps one of the simplest and most often used stream operators is `Select`. The `Select` operator allows us to simply transform the values appearing on the input stream through a function, producing a stream of resulting values. For those of you familiar with [LINQ](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/), the name `Select` comes from LINQ; you will notice that in fact a number of the \psi stream operators have names similar to their LINQ counterparts (those operate on enumerables instead of streams). 

Suppose that we have a stream of integers, generated like:

```csharp
var naturals = Generators.Sequence(p, 1, x => x + 1, 100);
```

This stream will end up containing the values 1, 2, 3, ..., 100 (don't worry for now about the whole `Generators.Sequence` business -- we'll come back later to how to generally construct streams). We can transform this stream of natural numbers into their negative counterparts by writing:

```csharp
var negatives = naturals.Select(x => -x);
```

Each incoming message on the `naturals` stream is transformed through the function specified as a parameter to `Select`, in this case taking the negative value. The resulting `negatives` stream will contain the values -1, -2, -3, ...

Since a stream operator returns a new (resulting) stream, multiple stream operators can be chained together. For instance, we can write:

```csharp
var doubleneg = naturals.Select(x => -x).Select(x => (x * 2).ToString());
```

and the resulting `doubleneg` stream in this case will be a stream of `string`, and will contain the sequence "-2", "-4", "-8", ..., "-200".

Before moving on to describe some of the other useful operators in \psi, let's make one additional observation about `Select`. Like many other (but not all) stream operators, `Select` takes as a parameter a delegate - in this case a function that transforms the message. Another overload of the select stream operator is available that takes a function of two parameters, and provides access not only to the message data, but also to the message envelope; the latter contains information such as the _originating time_ of the message - see more in the [Brief Introduction tutorial](Brief-Introduction):

```csharp
naturals.Select((x, e) =>
{
    Console.WriteLine($"Message: {x} ({e.OriginatingTime})");
    return -x;
});
```

The code above will transform the `naturals` stream. Each time the function is called, `x` contains the value of the incoming message and `e` is the message envelope. This operator still transforms the stream into the corresponding negative values, but also, as a side effect, prints the incoming messages and their corresponding originating times.

Besides `Select`, two other very frequently used stream operators are `Where` and `Do`. The `Where` operator selectively picks some of the messages from the incoming stream, and copies them on the output stream. The parameter to this operator is a predicate, i.e. a boolean function that specifies which messages should be copied to the output. For instance:

```csharp
var even = naturals.Where(x => x % 2 == 0);
```

defines a stream which will contain only the even numbers, so 2, 4, 6, ..., 100. Like with `Select`, an overload exists which gives access to the message envelope. 

The `Do` operator allows us to perform a certain action for every message on the input stream, while copying all the messages from the input to the output (this way `Do` stays technically a stream operator and allows for chaining other operators downstream.) 

Here is one example, where we first pick only the even numbers and then we print them:

```csharp
naturals
    .Where(x => x % 2 == 0)
    .Do((x, m) => Console.WriteLine($"Value: {x} at time {e.OriginatingTime}"));
```

## `Process`

The `Process` operator allows for writing more general stream processors. The operator takes as a parameter an action, that takes as parameters the incoming value and its envelope from the input stream, as well as an output emitter. You can choose whether and what to post on this output emitter. As an example, consider the code snippet below:

```csharp
var output = naturals.Process<int, int>(
    (x, e, o) =>
    {
        if (x % 2 == 0)
        {
            o.Post(x * 2, e.OriginatingTime);
        }
    });
```

In this case the action delegate given to `Process` checks if the input value is even, and if so doubles the value and posts the result. The resulting stream is 4, 8, 12, ... We could have also accomplished this with a `Where` and `Select`, like:

```csharp
var output = naturals.Where(m => m % 2 == 0).Select(m => m * 2);
```

but `Process` does all this in a single stream operator. While the example above is perhaps contrived, the `Process` operator is useful for more complex cases, as it allows for writing generic stream processors.

## `Aggregate`

The `Aggregate` operator allows us to implement stateful computation, where we can accumulate a value (or maintain state information) over a stream. The `Aggregate` operator takes in a function with two parameters: the first parameter corresponds to the accumulated value so far, while the second corresponds to the message from the input stream; the function produces as a result the next accumulated value, and this is posted on the output stream.

For instance, the code below accumulates the values on the input stream: 

```csharp
var sum = naturals.Aggregate(0, (acc, x) => acc + x);
```

The first parameter, `0`, specifies the initial value for the accumulator. The second parameter is the function: `(acc, x) => acc + x`. When the first message, i.e. `1` arrives, `acc` has the specified initial value, i.e. `0`, and the result will be `1`, which gets posted on the output stream and becomes the current value of `acc`. When the next message, `2`, arrives, the function will return `3` which will be posted on the output stream and becomes the new accumulated state, etc. 

The `Aggregate(...)` operator can be used to write a variety of other stateful operations on the stream. As another quick example, the code below produces streams containing the minimum value seen so far:

```csharp
var min = naturals.Aggregate((acc, x) => x < acc ? x : acc);
```

Interestingly, this lambda delegate is essentially taking the accumulated value (`acc`) and the current message (`x`) and returning the minimum of the two. `Math.Min(...)` is already such a function, so the following would work as well:

```csharp
var min = myStream.Aggregate(Math.Min);
```

The `Aggregate` operator is very general and can be used to accomplish many operations depending of accumulated state. For convenience, the following common specializations are also provided for numerical streams: `Count`, `LongCount`, `Sum`, `Min`, `Max`, `Average`, `Std`, `Log`, `Abs`, `Delta`. Each of these can be applied directly on a stream, e.g.:

```csharp
var avg = naturals.Average();
```

In addition an overload that takes a predicate which specifies a condition can also be applied, e.g.:

```csharp
var evenAvg = naturals.Average(x => x % 2 == 0);
```

# Mathematical and Statistical Operators

A number of mathematical and statistical operators are also available in the \psi runtime, summarized in the table below:

| Operator | Stream Type | Description |
| :---- | :----------------- | :----------------- |
| `Count` | any | Counts the number of messages. |
| `LongCount` | any | Counts the number of messages (returns a `long`). |
| `Sum` | numerical | Sums messages. |
| `Min` | any | Outputs the minimum value (uses default comparer). |
| `Max` | any | Outputs the maximum value (uses default comparer). |
| `Average` | numerical | Averages the messages. |
| `Std` | numerical | Computes the standard deviation. |
| `Abs` | numerical | Computes the absolute value. |
| `Log` | numerical | Computes the logarithm (uses optional base). |
| `Delta` | numerical | Computes the difference between two consecutive messages on a stream. |

The `Count` and `LongCount` operators apply to streams of _any_ type; producing a running count of messages. The `Min` and `Max` use the default comparer `Comparer<T>.Default`, or optionally allow the user to specify a custom `IComparer`. Each of the other operators apply to streams of _numerical_ types; `int`, `long`, `float`, `double`, `decimal`.

Additionally, each operator applies to streams of `IEnumerable<_>`. This is commonly used to aggregate "windows" of data, e.g.:

```csharp
myNumericStream.Window(TimeSpan.FromSeconds(1)).Average()
```

This example produces a stream of sliding one-second windows represented as `IProducer<IEnumerable<_>>` (see [Windowing](Windowing-Operators)). The stream of windows is then consumed by the `.Average()` operator in this example to produce a stream of averages of each window.

As this usage pattern is common, overloads of each operator are also provided that accept windowing parameters (`size` or `timeSpan`) directly as an equivalent shorthand, e.g.:

```csharp
myNumericStream.Average(TimeSpan.FromSeconds(1))
```

Note: One small exception is a `Count()`/`LongCount()` given a window _size_. This would merely always return the given size as the count and so this variant is not provided.

# Time-related Operators

## `TimeOf`

A few operators simplify access to timing information on the streams. The `TimeOf()` operator returns a stream that contains the originating times of the messages on the input stream. For example:

```csharp
var times = source.TimeOf();
```

This operator is simply implemented based on a `Select` that picks up the originating time from the message envelope.

## `Latency`

The `Latency()` operator computes the latency on a given stream. The messages on the output stream are of type `TimeSpan` and correspond to the difference between the time the message was created (captured by the `Time` member of the message envelope) and the originating time of the message. 

## `Delay`

Finally, the `Delay(timeSpan)` operator produces a "delayed" stream by shifting the originating times by a specified time-span. For example, the code below returns a stream where the messages are offset by 200 ms:

```csharp
var delayed = source.Delay(TimeSpan.FromMilliseconds(200));
```

# Other Miscellaneous Operators

## `ToEnumerable` and `ToObservable`

In order to facilite bridging to other stream-like systems, \psi streams may be converted to lazy `IEnumerable` or `IObservable`. For example, simply:

```csharp
var enumerable = myStream.ToEnumerable();
```

construct an enumerable that can be traversed while the pipeline runs asynchronously, or can accumulate values to be consumed after the pipeline ends running.

Similarly, \psi streams can also be converted into observables, by using the `ToObservable` operator.

Bridging to events is also possible, but [a bit more involved](Event-Sources), and requires instantiating an `EventSource` component and providing lambdas to subscribe/unsubscribe (this is due to events not being first class values in C#).

## `NullableSelect`

The `NullableSelect` operator is similar to `Select`, but simplifies transforming streams of nullable values. Nulls are passed through, but when messages have an actual value, the value is unpacked, the transform is applied and the result is repacked as a nullable. For example:

```csharp
var squared = nullableIntStream.NullableSelect(x => x * x);
```

will generated a corresponding stream of squared values, but when the input messages are null, a corresponding null message appears on the `squared` result stream.

## `First`

The `First` operator selects the first message, or a specified count of messages from the beginning of a stream. For instance, in the example below:

```csharp
var first = myStream.First();
var firstThree = myStream.First(3);
```

the resulting `first` stream contains only a single message (the first message from `myStream`), whereas `firstThree` contains the first three messages from `myStream`.

## `Item1`, `Item2` and `Flip`

These three operators act on streams of tuples. The `Item1` and `Item2` operators return a stream that contains only the first or the second item of the tuple. The `Flip` operator returns a tuple in which the two items are flipped.

## `Name`

The `Name` operator allows for naming streams, which can be helpful when debugging. For instance:

```csharp
myStream.Name("My stream");
```

## `PipeTo`

The `PipeTo` operator can be used to connect emitters (streams) to various component receivers. For example:

```csharp
image.PipeTo(croppingComponent.ImageInput);
```

If the component is an `IConsumer<T>`, then the `PipeTo` operator can take as a paramater just the component, and will directly connect the given stream to the `In` receiver of the component. For instance, in the example below, assuming the `toGrayScaleComponent` is an `IConsumer<Image>` (and hence has an `In` receiver of type `Receiver<Image>`), we can pipe an image stream as follows:

```csharp
image.PipeTo(toGrayScaleComponent);
```

In this case, `PipeTo` returns the component that was passed in, e.g. `toGrayScaleComponent` in the case above. If this component is not only a consumer, but also a producer, i.e. `IProducer<TOut>`, then the `PipeTo` operator can be further invoked on this result to connect to another component, allowing us to chain multiple components together. For instance:

```csharp
image.PipeTo(toGrayScaleComponent).PipeTo(toOpticalFlowComponent);
```

will connect `image` to the input of the `toGrayScaleComponent`, and will take the output of this component, i.e. a gray scale image, and further connect that to the input of the `toOpticalFlowComponent` component.


<a name="ReferenceTable"></a>

# Delivery Policies

Most stream operators discussed above accept as an optional parameter a [delivery policy](Delivery-Policies), which allows the developer to control whether and when the messages flowing from the streams to the operator get dropped. To read more on delivery policies and their use, please see [this tutorial](Delivery-Policies).

# Reference Table of Basic Stream Operators

| Operator | Description | 
| :---- | :----------------- | 
| _Basic_ | | 
| `Select`| Applies a specified transform to messages on a stream. | 
| `Where` | Filters messages on a stream based on a predicate. | 
| `Do` | Performs an action for each message on a stream. | 
| `Process` | Allows generic processing of messages on a stream. | 
| `Aggregate` | Allows for stateful operations on a stream. | 
| _Time-related_ | | 
| `TimeOf` | Outputs the originating time of the messages on a stream. | 
| `Latency` | Outputs the latency of the messages on a stream. | 
| `Delay` | Shifts the messages on a stream forward or backward in originating time. | 
| _Mathematical_ | | 
| `Count` | Counts the number of messages. |
| `LongCount` | Counts the number of messages (returns a `long`). |
| `Sum` | Sums messages. |
| `Min` | Outputs the minimum value (uses default comparer). |
| `Max` | Outputs the maximum value (uses default comparer). |
| `Average` | Averages the messages. |
| `Std` | Computes the standard deviation. |
| `Abs` | Computes the absolute value. |
| `Log` | Computes the logarithm (uses optional base). |
| `Delta` | Computes the difference between two consecutive messages on a stream. |
| _Miscellaneous_ | | 
| `ToEnumerable` | Constructs an `IEnumerable` from a stream. | 
| `ToObservable` | Constructs an `IObservable` from a stream. | 
| `NullableSelect` | Similar to `Select`, but handles nullable types, allowing pass-through of null values. | 
| `First` | Gets the first message (or first n messages) in a stream. |
| `Item1`, `Item2` | Generates a stream of sub-items from a stream of tuples. |
| `Flip` | Flips the items in a stream of tuple. |
| `Name` | Names a stream. |
| `PipeTo` | Allows for connecting components (i.e. connecting an emitter of one component to a receiver of another). | 

<a name="FurtherReading"></a>

# Summary and further reading

We have introduced in this tutorial some of the most basic and frequently used stream operators in the \psi framework. The framework provides however a number of more advanced stream operators that deal with specific streaming aspects, such as windowing, sampling, synchronization, etc., These topics and the corresponding stream operators are introduced in more detail in additional tutorials:

* [Stream Fusion and Merging](Stream-Fusion-and-Merging): introduces operators for stream fusion, merging and synchronization.
* [Interpolation and Sampling](Interpolation-and-Sampling): introduces operators for interpolation and sampling.
* [Windowing Operators](Windowing-Operators): introduces operators that allow buffering over streams of data.
* [Stream Generators](Stream-Generators): explains stream generators and timers.
* [Parallel Operator](Parallel-Operator): introduces `Parallel`, an operator that enables vector-based parallel computation and dynamic pipelines.
