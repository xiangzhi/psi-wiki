Most often source streams are produced by various sensor components, like cameras or microphones. However, several generic stream-producing "generators" are also available and can be used, via static methods on the `Generators` class. 

# Basic Generators

## `Sequence`

One of the simplest stream generators is `Generators.Sequence(p, ...)`, which can be used to produce a stream in pipeline `p`, containing a sequence of values. For instance: 

```csharp
IProducer<int> sequence = Generators.Sequence(p, new int[] { 1, 2, 3, 7, 42 }, TimeSpan.FromMilliseconds(10));
```

produces a stream of integers called `sequence` on which the values 1, 2, 3, 7, 42 will be posted at 10 millisecond intervals once the pipeline `p` starts. 

In the example above, `Generators.Sequence` uses an `IEnumerable<T>` as a parameter. A number of other overloads are available, including one that uses an initial value and a function to produce the next value. For instance: 

```csharp
IProducer<int> sequence = Generators.Sequence(p, 0, x => x + 1, TimeSpan.FromMilliseconds(10));
```

will produce an infinite stream containing the messages 0, 1, 2, ... at 10 millisecond intervals. Whereas the line below: 

```csharp
IProducer<int> sequence100 = Generators.Sequence(p, 0, x => x + 1, 100, TimeSpan.FromMilliseconds(10));
```

will produce the finite sequence of 100 messages: 0, 1, 2, ..., 99. 

The versions above produce messages at a constant, specified cadence. Alternatively a couple of overloads allow the user to specify as input an enumerable of tuples containing both the value and the originating time at which the message should be posted (i.e., `IEnumerable<(T, DateTime)>`). In this case, the times should be in strictly increasing order (as messages can only be posted on a stream with strictly increasing originating times.)

__A (more advanced) side note on finite versus infinite streams__: By default, `Generators.Sequence` produces a source stream that closes when the sequence is finished. So in the case above, the `sequence100` stream will close after message 99. As a result if this stream is added to a pipeline that contains other infinite sources, the pipeline will shut down after message 99 (pipelines containing both finite and infinite sources shut down when all the finite sources have completed). This behavior of `Generators.Sequence` can be modified with an optional parameter called `keepOpen`. So, for instance:

```csharp
IProducer<int> sequence100_Open = Generators.Sequence(p, 0, x => x + 1, 100, TimeSpan.FromMilliseconds(10), keepOpen: true);
```

generates a similar sequence containing the messages 0, 1, 2, ..., 99 at 10 millisecond intervals, but does not close the stream after the last message is published.

## `Repeat` and `Range`

Besides sequence, two other useful generators are `Repeat` and `Range`. `Generators.Repeat(p, ...)` produces a stream of repeated values (of any type) at a given interval. For example, the code below produces an stream of ten `"foo"` string values, 10 milliseconds apart:

```csharp
var repetition = Generators.Repeat(p, "foo", 10, TimeSpan.FromMilliseconds(10));
```

Or, for an infinite repetition of messages, we can omit the third (`count`) parameter, and write:

```csharp
var repetition = Generators.Repeat(p, "foo", TimeSpan.FromMilliseconds(10));
```

The `Range` operator allows for constructing a finite range of integer values published at a regular interval. So the code:

```csharp
var range = Generators.Range(p, 0, 10, TimeSpan.FromMilliseconds(10));
```

generates the sequence 0, 1, 2, ..., 9. Note that the third parameter above (i.e., `10`) represents the number of messages to post, not the end of the range. 

Like with `Sequence`, the `keepOpen` parameter is available on the `Repeat` and `Range` generators.

## `Return` and `Once`

Finally, the `Return` and `Once` generators produce a stream that contains a single message. The difference between the two is that `Return` closes the stream after the message is posted (so equivalent to doing a `keepOpen: true`), whereas `Once` leaves the stream open. 

```csharp
var life = Generators.Return(pipeline, 42);
```

# Time Alignment

We have seen above that typically generators can be used to produce messages on a specified cadence. In addition, generators may be also instructed to produce _time-aligned_ messages. That is, a sequence of messages may be generated at a given interval (e.g., 100ms) but may be additionally required to align those intervals on a particular time (e.g. the start of a particular second). This is sometimes a useful guarantee when later joining many generated streams that may otherwise be misaligned by some fraction of time.

The `Range`, `Sequence` and `Repeat` generators described above take an optional `DateTime?` parameter called `alignDateTime` that specifies which date-time the messages should align to. To illustrate this consider the code below:

```csharp
using (var p = Pipeline.Create())
{
    var repetition = Generators.Repeat(p, "foo", 10, TimeSpan.FromMilliseconds(10));
    repetition.Do((x, e) => Console.WriteLine($"{x} @ {e.OriginatingTime.TimeOfDay}"));
    p.Run();
}
```

In this case we have constructed a stream that repeats the message "foo" 10 times at 10 millisecond intervals. The corresponding output, from the `Do` operator shows that indeed the messages have originating times that are 10 milliseconds apart, like below:

```text
foo @ 18:26:04.0337482
foo @ 18:26:04.0437482
foo @ 18:26:04.0537482
foo @ 18:26:04.0637482
foo @ 18:26:04.0737482
foo @ 18:26:04.0837482
foo @ 18:26:04.0937482
foo @ 18:26:04.1037482
foo @ 18:26:04.1137482
foo @ 18:26:04.1237482
```

If we want however the timing of these messages to align for instance to the second, i.e. to be an integral number of 10 millisecond intervals away from the second boundary, we can do so by aligning the generator. Since `DateTime.MinValue` has an integral number of seconds (0) in it, we can align with `DateTime.MinValue`:

```csharp
using (var p = Pipeline.Create())
{
    var alignedRepetition = Generators.Repeat(p, "foo", 10, TimeSpan.FromMilliseconds(10), DateTime.MinValue);
    alignedRepetition.Do((x, e) => Console.WriteLine($"{x} @ {e.OriginatingTime.TimeOfDay}"));
    p.Run();
}
```

When running this code, the messages will still be 10 milliseconds away from each other, but will also align on the second, and end in .xxx00000, like below:

```text
foo @ 18:24:57.5900000
foo @ 18:24:57.6000000
foo @ 18:24:57.6100000
foo @ 18:24:57.6200000
foo @ 18:24:57.6300000
foo @ 18:24:57.6400000
foo @ 18:24:57.6500000
foo @ 18:24:57.6600000
foo @ 18:24:57.6700000
foo @ 18:24:57.6800000
```
