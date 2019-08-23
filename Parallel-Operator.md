
The `Parallel` operator enables parallel execution over dense or sparse vectors (i.e., arrays or dictionaries).

## Dense vector parallel processing

When using parallel over a stream of dense vectors, i.e. a stream of arrays, a separate pipeline is instantiated for each element in the vector. Each element is processed in parallel and the results are merged at the end back into a vector of the output type. 

```csharp
Parallel<TIn, TOut>(
    this IProducer<TIn[]> source,
    int vectorSize,
    Func<int, IProducer<TIn>, IProducer<TOut>> streamTransform,
    bool outputDefaultIfDropped = false,
    TOut defaultValue = default,
    DeliveryPolicy deliveryPolicy = null)
```

In this case, we need to specify the vector size and a function that given an index in the array and an input stream of type `TIn` produces an output stream of type `TOut`. This function essentially allows us to  specify the sub-pipeline that will be created for the input index.

For instance, consider the example below:

```csharp
var streamOfArray = Generators.Sequence(p, new[] { 0, 1, 2 }, r => new[] { r[0] + 1, r[1] + 1, r[2] + 1 }, 100);
streamOfArray.Parallel(3, (int index, IProducer<int> s) => {
    if(index == 0)
    {
        return s;
    }
    else if(index == 1)
    {
        return s.Select(m => m * 2);
    }
    else
    {
        return s.Select(m => m * 3);
    }
});

```

Here, the `streamOfArray` looks like this:

```
0  1  2  3  4
1  2  3  4  5
2  3  4  5  6
```

The `Parallel` operator defines a index-specific pipeline, which keeps the values for index 1 (we return the original stream as the output stream; doubles the values for index 2 via a `Select` operator, and triples the values for index 3 again via a `Select` operator. The results will look like this:

```
0  1  2  3  4
2  4  6  8 10
6  9 12 15 18
```

## Sparse vector parallel processing

A similar `Parallel` operator is available to process sparse vectors, i.e. dictionaries `Dictionary<TKey, TValue>`. The signature is:

```csharp
Parallel<TIn, TKey, TOut>(
    this IProducer<Dictionary<TKey, TIn>> source,
    Func<TKey, IProducer<TIn>, IProducer<TOut>> streamTransform,
    bool outputDefaultIfDropped = false,
    TOut defaultValue = default,
    DeliveryPolicy deliveryPolicy = null,
    Func<TKey, Dictionary<TKey, TIn>, DateTime, (bool, DateTime)> branchTerminationPolicy = null)
    ) -> IProducer<Dictionary<TKey, TOut>>
```

Internally, sparse vector parallel processing creates and runs instances of [`Subpipeline`](Writing-Components#SubPipelines) dynamically and stops them based on the `branchTerminationPolicy`. The default policy is to stop and remove subpipelines when when the corresponding key is no longer available in the message.

A good example usage is when _tracking_ multiple entities (e.g. face tracking). When a set of tracked entities are represented as a stream of dictionaries, `Parallel` will create multiple `Subpipeline` instances, each processing a particular entity independently, and gather the results. Each `Subpipeline` will terminate when the tracked entity is no longer found (e.g. a particular face moves out of frame).
