Platform for Situated Intelligence provides a set of stream operators that allow the developer to easily implement a variety of stream windowing operations. This document describes these operators.

__Note__: Prior to reading this document, we strongly recommend that you familiarize yourself with the core concepts of the \\psi framework, by reading the [Brief Introduction](Brief-Introduction) tutorial. The concepts of streams and originating times are a prerequisite for the documentation below.

The document is structured as follows:

1. [**Windows Defined by Relative Message Index**](#RelativeMessageIndex) - describes constructing windows of messages defined by message count (e.g., a window containing the last 4 messages).
2. [**Windows Defined by Relative Time Intervals**](#RelativeTimeInterval) - describes constructing windows of messages defined by a relative time interval (e.g., a window containing all messages from the last 10 seconds).
3. [**Windows Defined Dynamically**](#Dynamic) - describes constructing dynamic windows, that are driven by information from a different stream (e.g., a window containing all messages over the time period when another specified stream has a value of `true`).
4. [**Window Statistics**](#WindowStatistics) - describes how to construct various window statistics.


\\psi enables developers to construct sliding windows of messages over streams of data via a set of `Window()` stream operators. Window extents may be specified by either a message count relative to the current message, by a time interval relative to the current message, or by information provided by a separate stream. 

__Note__: The various overloads of the `Window()` operator described below take an optional `DeliveryPolicy` argument which allows the developer to control how the operators keep up with the incoming flow of messages when not enough computational resources are available to process them. More information is available in the [Delivery Policies](Delivery-Policies) in-depth topic. Below, for improved readability, we simply omit this optional parameter from the operator descriptions.

<a name="RelativeMessageIndex"></a>

# 1. Windows Defined by Relative Message Index

A set of `Window()` operators enable the construction of sliding windows of a given size (in number of messages) over a stream of messages. For example, the code below constructs a stream of sliding windows containing the last 3 messages, including the current one: 

```csharp
    // sliding window of the last 3 messages (including current)
    source.Window(-2, 0)
```

In this case, the operator takes two parameters that describe the index of the start and end message to be included in the window, relative to the current message (which is considered at index 0). The operator returns a stream of `T[]`, where each message contains an array of three elements from the original stream corresponding to the specified window.

The functioning of the operator is illustrated below:

![Results of applying the .Window(-2, 0) operator](Window.CountPast.PNG)

The `Window()` operator based on relative message indexes attempts to publish a window for every incoming message, but does so only if it has formed a complete window. For instance, when doing `source.Window(-2, 0)` above, we are asking for a window of the last 3 messages (including the current one). This means that, at the beginning of the stream, as the first and second messages are received nothing is produced, since a complete window of the specified parameters cannot yet be constructed. Only when the third message arrives, the first output is produced, containing the past 3 elements. From then on, for every incoming message, a new window is produced. 

An alternative overload of the `Window()` operator takes an `IntInterval` which basically defines the two relative indices as the left and right endpoints. The code below computes therefore the same window of 3 past messages (including the current):

```csharp
    // sliding past windows 3 messages (including current)
    source.Window(IntInterval.LeftBounded(-2))
```

The windows do not have to necessarily be backward looking. For instance, the examples below build forward looking windows, or windows that extend both forward and backward in time:

```csharp
    // sliding future window of 3 messages (including current)
    source.Window(0, 2)
    source.Window(IntInterval.RightBounded(2))

    // sliding window of 4 messages (the previous, current, and 2 future messages)
    source.Window(-1, 2)
    source.Window(new IntInterval(-1, 2))
```

![Results of applying the `.Window(-1, 2) operator`](Window.CountAround.PNG)

The originating time for the output messages correspond to the originating time of the incoming message at relative index 0 - we refer to this as the _window anchor message_. So, for a backward-looking window like `Window(-2, 0)`, the anchor message and originating time correspond to the _last_ message in the window (that one has the relative index 0), whereas for a forward-looking window like `Window(0, 2)`, the anchor message and originating time correspond to the _first_ message in the window (that one has a relative index 0). In this latter case, since the operator has to wait until the future messages that complete the window arrive, a latency is introduced. For instance, to output the window [3, 4, 5, 6] corresponding to input 4 in the exmaple above, the operator needs to wait until message 6 has arrived.

Finally, before moving on to time-based windows, we note that the windowing operators also take as an optional parameter a selector function of type `Func<IEnumerable<Message<T>, U>`. When this function is provided, it can be used to transform the constructed window of messages (represented by the input to the function) into another output data type. This selector function is given windows of complete `Messages` including the `Envelope` information from which originating time information may be useful. The originating time of the resulting value (of type `U`) is taken from the _origin_ message as usual.
For example a `Slope` operator may be implemented in terms of these by computing a true slope of a window of numeric types, taking into account not just the values but also the temporal placement (originating-times) of each message in the window.

<a name="RelativeTimeInterval"></a>

# 2. Windows Defined by Relative Time Interval

In addition to windows containing a specified number of messages, windows can be also constructed based on a specified `RelativeTimeInterval` for which to include messages. In this case, the output contains _all the messages in a specified time window relative to the current message_. While the output is still an array, this means that in general, these windows may contain a variable number of messages. The originating time of the output window messages in this case corresponds to the originating time of the anchor message, which sits at relative time zero. The figure below illustrates the use of the operator to generate windows looking 2 seconds in the past and 1 second in the future:

```csharp
    // sliding window of 2 seconds in the past and one seconds in the future
    source.Window(TimeSpan.FromSeconds(-2), TimeSpan.FromSeconds(1))
```

![Results of applying the `.Window(TimeSpan.FromSeconds(-2), TimeSpan.FromSeconds(1))` operator.](Window.RelativeTime.PNG)

Two important observations must be made here: first, in contrast to the windows with a specific message index, when constructing windows based on a relative time interval, a result is always published, for every incoming message. For instance, we have seen above that the message index based `Window(-2, 0)` will not output a result for the first two incoming messages, since no window of 3 last messages can be formed in these cases. In constrast, `Window(TimeSpan.FromSeconds(-2), TimeSpan.FromSeconds(1))` will output an array for every incoming message, even if the message is less than 2 seconds from the start of the stream: the operator simply collects all message in the specified relative time interval with respect to its originating time.

The second observation is that the output messages are however still driven by the messages on the input stream. This means that if the input stream has large gaps between messages (in originating time), these gaps may be reflected in the output. In other words, the window anchors still correspond to each incoming message, which means the window anchors do not advance (slide) uniformly over time, but rather follow the pace of the incoming messages. 

\\psi does however also allow the construction of windows whose anchors are not driven by the source stream, via a dynamic window operator, which we describe below.

# 3. Windows Defined Dynamically, via a Windowing Stream

In the previous cases the windows were defined relative to each incoming message on the source stream, either in terms of number of messages, or in terms of time. Another version of the `Window()` operator enables us to drive the location of the window anchors with a different stream, and to create variable size windows. This operator has a more complex signature, defined like below:

```csharp
    public static IProducer<TOutput> Window<TSource, TWindow, TOutput>(
        this IProducer<TSource> source,
        IProducer<TWindow> window,
        Func<Message<TWindow>, (TimeInterval, DateTime)> windowCreator,
        DeliveryPolicy<TSource> sourceDeliveryPolicy = null,
        DeliveryPolicy<TWindow> windowDeliveryPolicy = null)
```

Apart from the source stream, this operator takes an additional stream as input (the `window` parameter), which will be used to anchor and generate the windows. The window anchors will be incoming messages on this window stream (rather than the `source` stream), and the actual temporal windows will be generated by a developer-specified function `windowCreator`, based on the incoming messages on the window stream. For each message in the `window` stream, this function needs to generate a tuple containing the actual originating-time window, specified in the first `TimeInterval` item of the tuple, and an _obsolete_ `DateTime` indicating _the originating-time prior to which none of the future windows will extend_. The other parameters are an `outputCreator` function, as we have seen in the previous operators, and optional delivery policies for the `source` and `window` streams.

To better understand why this _obsolete_ time is required to be generated by the `windowCreator` function, let's consider how the dynamic window operator functions. Since the size of the windows is not known apriori, but rather depends on each incoming message on the `window` stream, the operator needs to buffer the incoming messages on the `source` stream, as they might be required to be part of future windows. The _obsolete_ time therefore enables the operator not to buffer indefinitely, i.e. the operator can discard all `source` messages from its buffer that have an originating-time previous to this obsolete time. 

It is the responsibility of the user of the operator to correctly generate the obsolete times. Note that, if future windows can extend indefinitely in the past, or if it is not known how far in the past future windows may extend, the developer can always return `DateTime.MinValue` as the obsolete time in the `windowCreator` result, and the operator will buffer indefinitely (this of course, may lead to increase memory usage as all messages on the `source` stream will be retained by the operator.)

This dynamic version of the `Window()` operator can be used in a variety of ways. For instance, it can be used to simply drive a constant window by a uniformly sliding anchor. We use as a `window` stream a simple clock produced by a `Generator.Repeat()` (see more documentation on [Stream Generators](Stream-Generators)), and return in the `windowCreator` function the desired time interval:

```csharp
    anchors = Generators.Repeat(p, 0, TimeSpan.FromSeconds(0.5));
    source.Window(
        anchors,
        m => (
            new TimeInterval(m.Envelope.OriginatingTime - TimeSpan.FromSeconds(0.5), m.Envelope.OriginatingTime + TimeSpan.FromSeconds(0.5)),
            m.Envelope.OriginatingTime - TimeSpan.FromSeconds(0.5)
        ));
```

In this case we are sliding a window based on the `anchors` stream, advancing it by .5 seconds at a time, independent of the cadence of the messages in the `source` stream. The window generated has the same size for all anchors [-.5 sec, +.5sec], and only depends on the originating-time of this message (they are anchored in it). The functioning of this operator is illustrated in the image below.

![Results a dynamic `Window` operator.](Window.Dynamic.PNG)

In the more general case however, the windows generated may depend on the actual contents of the anchor message.

<a name="WindowStatistics"></a>

# 4. Window Statistics

It is very common to perform statistical operations over windows. This may be done by composing \\psi operators with those from LINQ with a \\psi `Select`:

```csharp
    source.Window(-9, 0).Select(xs => xs.Average())
```

or

```csharp
    source.Window(RelativeTimeInterval.Past(TimeSpan.FromSeconds(10))).Select(xs => xs.Average())
```

In this case, `Window()` is the \\psi operator that generates an `IProducer<double[]>` (assuming `source` is an `IProducer<double>`). The `Select()` is also a \\psi operator that internally uses the LINQ `Average()` operator to compute the average of the messages in each window. We can, of course, transform the windows of data this way using _any_ LINQ operation we like.

In the case of the most common statistical operations such as `Average()`, `Min()`, `Max()`, `Sum()`, ... we have operations directly on `IProducer<T>` where `T` is a numeric type. The following are two ways to compute the same averages we have computed above:

```csharp
    source.Average(10)
```

or

```csharp
    source.Average(TimeSpan.FromSeconds(10))
```

Note that these statistical operators are backward-looking. That is, `Average(10)` uses `Window(-9, 0)` internally and `Average(TimeSpan.FromSeconds(10))` uses `Window(RelativeTimeInterval.Past(TimeSpan.FromSeconds(10)))`.

Note also that, without a size (`int`) or `TimeSpan` given, these statistical operators instead give you a _running_ result (e.g. running average).