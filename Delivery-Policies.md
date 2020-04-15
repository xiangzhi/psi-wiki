__Note__: Prior to reading this document, we strongly recommend that you familiarize yourself with the core concepts of the \\psi framework, by reading the [Brief Introduction](Brief-Introduction) tutorial.

__Delivery policies__ allow the developer to control whether and when the messages flowing over streams in a Platform for Situated Intelligence application get dropped. This allows developers to configure how an application may keep up with the incoming streams when not enough computational resources are available to process every message. The document below provides an introduction to the various types of delivery policies available and their operation. It is structured as follows

1. [**Delivery Policy Basics**](#Basics) - introduces the concept and basic set of \psi delivery policies.
2. [**Typed Delivery Policies and Message Delivery Guarantees**](#TypedPolicies) - describes delivery policies that offer message guarantees.
3. [**Default Pipeline Policies**](#PipelinePolicies) - describes how pipeline-level defaults for delivery policies may be specified.
4. [**Summary of Available Delivery Policies**](#Summary) - summarizes the set of available delivery policies.

<a name="Basics"></a>

# 1. Delivery Policy Basics

To introduce the concept of _delivery policies_, we will start with a simple example. Suppose we are processing a stream of data, but that the computation we are performing on each message is time-consuming, and takes more time than the interval between two consecutive messages on the stream. As a concrete example, here is a class representing a simple consumer-producer component (for more info see [__Writing Components__](Writing-Components) topic). In the receiver we sleep to simulate a time-consuming operation, which takes about 1 second for every message received.

```csharp
using Microsoft.Psi.Components;

public class IntensiveComponent : ConsumerProducer<int, int>
{
    public IntensiveComponent(Pipeline pipeline)
        : base(pipeline)
    {
    }

    protected override void Receive(int message, Envelope envelope)
    {
        // do some expensive computation in here. To simulate, we 
        // simply sleep for one second
        System.Threading.Thread.Sleep(1000);

        // post the same message as result
        this.Out.Post(message, envelope.OriginatingTime);
    }
}
```

Let us now connect this component to a source stream that generates messages with a faster cadence -- in this example every 100 milliseconds:

```csharp
using (var p = Pipeline.Create())
{
    // generate a sequence of 50 integers starting at zero, one every 100 milliseconds
    var source = Generators.Sequence(p, 0, x => x + 1, 50, TimeSpan.FromMilliseconds(100));

    // instantiate the intensive component and connect the source to its input
    var intensiveComponent = new IntensiveComponent(p);
    source.PipeTo(intensiveComponent.In);

    // output the results of the intensive component
    intensiveComponent.Do((m, e) => Console.WriteLine($"{m} @ {(e.Time - e.OriginatingTime).TotalSeconds:0.00} seconds latency"));

    // run the pipeline
    p.Run();
}
```

Because the component is not able to keep up with the source stream, the messages are queued in front of the component's receiver. The component processes one message per second, but during that time 10 more messages arrive and are queued at the receiver, waiting to be processed (recall that the receivers for a component execute exclusively, so a receiver will not execute again until it has completed). The component will keep dequeuing and processing messages one by one. Because the component is not able to keep up with the rate at which messages are produced on the incoming stream, the latency of each output from the component will keep increasing, as shown in the results written to the console (subsequent messages have to wait longer and longer in the queue until they are being processed):

```text
0 @ 1.04 seconds latency
1 @ 1.94 seconds latency
2 @ 2.84 seconds latency
3 @ 3.74 seconds latency
4 @ 4.64 seconds latency
5 @ 5.54 seconds latency
6 @ 6.44 seconds latency
7 @ 7.34 seconds latency
...
```

This default behavior, in which all messages are queued and eventually processed, corresponds to a subscription delivery policy called `DeliveryPolicy.Unlimited`. We can modify however this default behavior by specifying an alternative delivery policy when connecting the source stream to the component's receiver. Another commonly used policy is `DeliveryPolicy.LatestMessage`. So, for example:

```csharp
    source.PipeTo(intensiveComponent.In, DeliveryPolicy.LatestMessage)
```  

In this case, the maximum queue size for the `In` receiver is set to size one. If a new message arrives while an existing one is already queued, the older message will essentially be dropped and only the most recent message will be delivered to the receiver. If we re-run the program with this change, the results look like:

```text
0 @ 1.04 seconds latency
10 @ 1.04 seconds latency
20 @ 1.05 seconds latency
30 @ 1.05 seconds latency
40 @ 1.05 seconds latency
...
```

The latency at the output is now maintained constant, but a significant number of messages from the incoming stream are being dropped and not processed. For instance the messages for 1, 2, 3, .. 9 were dropped from the delivery queue and the second message processed by the `Do` operation was message 10. 

In the example above, we have specified the delivery policy on the `PipeTo` operator that connects an emitter to a receiver. We note that the various stream operators available in the \\psi framework, such as `Select()`, `Where()`, `Process()` etc., also take an optional delivery policy parameter. This allows developers to specify inline for each stream operation, which delivery policy should be used. For example:

```csharp
source.Select(x => timeIntensiveCompute(x), DeliveryPolicy.LatestMessage)
```

In general, the same holds true for all stream operators that follow the [__recommended design pattern__](Writing-Components#StreamOperators) for writing operators.


## Throttling Delivery
While dropping messages can avoid ever increasing queues and latencies, we may sometimes want to signal to the source to slow down to match the speed of the time-consuming operation. The `DeliveryPolicy.Throttle` delivery policy notifies the upstream component once there is a pending message for a receiver that it is no longer able to accept further messages. It does this by locking or throttling the upstream component, such that it too may no longer accept messages at its receivers. It is possible to create chains of components with throttling back to the originating source(s) of messages. In this way, we can ensure that all messages are eventually processed without queueing up messages due to slow operations.

## Synchronous Delivery
The default behavior is to queue messages at receivers then service these queues and deliver messages asynchronously. In certain situations we may prefer to deliver and process messages synchronously as soon as they are posted. The `DeliveryPolicy.SynchronousOrThrottle` delivery policy will attempt to do this for as long as it is possible, otherwise it throttles its source. This allows producer-consumer links to behave as a single receiver and can reduce the delivery overhead, especially when chaining together many trivial operations. Note that there are times when synchronous delivery may not always be possible, for example if the component is locked to ensure mutual exclusivity across its receivers. In such situations, messages will be queued and the source throttled.

## Queue- and Latency-Constrained Delivery
Apart from the static delivery policies we have described above, the \\psi framework enables the construction of queue- or latency-constrained delivery policies. 

The `DeliveryPolicy.LatencyConstrained(TimeSpan maximumLatency)` factory method enables the developer to define a latency-constrained policy. When using this policy, messages will be queued and are delivered to the receiver if their latency is below the specified `maximumLatency`. As time elapses, messages that exceed that maximum latency will be discarded. 

Similarly, the `DeliveryPolicy.QueueSizeConstrained(int maximumQueueSize, bool throttleWhenFull = false, bool attemptSynchronous = false)` factory method allows developers to define a queue-size constrained policy. In this case, only the most recent messages up to the `maximumQueueSize` will be delivered to the receiver. The optional flag `throttleWhenFull` controls whether the source component is throttled in an attempt to slow it down when the queue is full. Synchronous delivery may also be attempted where possible in an effort to bypass the queue by setting the optional `attemptSynchronous` flag to `true`.

<a name="TypedPolicies"></a>

# 2. Typed Delivery Policies and Message Delivery Guarantees

The `Unlimited`, `LatestMessage`, `Throttle`, `SynchronousOrThrottle` and other delivery policies described above are all untyped (they are instances of the class `DeliveryPolicy`), and control how message delivery happens without taking into account the type, or the contents of the message in question. They only take into account latencies and size of the receiver queue.

In some cases, it is useful to be able to construct delivery policies which pay attention to the contents of the message. Specifically, \\psi enables the construction of typed delivery policies (instances of the class `DeliveryPolicy<T>`) which can _provide guarantees that certain messages will always be delivered_. This can be useful when streaming data where some of the messages in the stream are more important than others, and should never be dropped.

Such typed policies, with guaranteed delivery, can be constructed by calling the `WithGuarantees<T>(Func<T, bool> guaranteeDelivery)` method on an existing, untyped delivery policy. This method takes as a parameter the `guaranteeDelivery` predicate which describes which messages _cannot_ be dropped. For instance: 

```csharp
    DeliveryPolicy.LatestMessage.WithGuarantees<int>(i => i % 5 == 0)
```

is a typed delivery policy for a stream of integers that operates like `LatestMessage` (so uses a queue size of 1 on the receiver), but guarantees that all integer multiples of 5 will not be dropped. This means that in fact, if a message containing a multiple of 5 arrives while the queue is full, the message will not be dropped, and the queue in this case will grow past the maximum size (in this case 1). Using this delivery policy in the example above leads to an output of the form:

```text
0 @ 1.07 seconds latency
5 @ 1.57 seconds latency
10 @ 2.07 seconds latency
15 @ 2.57 seconds latency
20 @ 3.07 seconds latency
25 @ 3.57 seconds latency
30 @ 4.07 seconds latency
35 @ 4.57 seconds latency
...
```

When the message containing `5` arrives, the receiver from our `intensiveComponent` is still busy processing message `0`. However, instead of being dropped `5` is actually queued and processed because the policy guarantees its delivery. The same holds true for `10`, `15`, `20`, and so on.

The `WithGuarantees(...)` method can also be called on an existing typed delivery policy, allowing us to create delivery policies that chain multiple guarantees. For instance: 

```csharp
    DeliveryPolicy.LatestMessage.WithGuarantees<int>(i => i % 5 = 0).WithGuarantees(i => i % 2 = 0)
```

does not allow messages containing multiples of five _or_ multiples of two to drop. (In this case the first application of `WithGuarantees<int>` returns a typed `DeliveryPolicy<int>` and hence the <int> specifier is no longer required on the second application.)

Finally, we note that the `PipeTo()` operator, as well as all stream operators that respect the recommended design pattern take in as parameter a _typed_ delivery policy, i.e., `DeliveryPolicy<T>`since streams are strongly typed in \\psi. However, when calling these operators, a simple, untyped policy such as `DeliveryPolicy.LatestMessage` or `DeliveryPolicy.Unlimited` can be used because \\psi defines an implicit conversion operator from `DeliveryPolicy` to `DeliveryPolicy<T>`.

<a name="PipelinePolicies"></a>

# 3. Default Pipeline Policies

In the previous section we have seen how delivery policies can be specified when connecting emitters to receivers via the `PipeTo()` operator, or when applying stream operators directly, such as `Select()`, `Do()`, etc. If a delivery policy is not specified, the runtime configures the connection to automatically use a default delivery policy, specified by the pipeline in which the given emitter and receiver belong. 

This default delivery policy is computed by a virtual method on the `Pipeline` class, named `GetDefaultDeliveryPolicy<T>()`. When calling `Pipeline.Create()` to construct a pipeline, the developer can specify a default untyped delivery policy (if no default policy is specified upon the construction of the pipeline, `DeliveryPolicy.Unlimited` is assumed.) The pipeline's `GetDefaultDeliveryPolicy` method implementation simply returns this user-specified default. This mechanism allows developers to simply specify a default policy at the pipeline level: 

```charp
using (var p = Pipeline.Create(deliveryPolicy: DeliveryPolicy.LatestMessage))
{
    // this connection automatically uses the LatestMessage policy specified on the pipeline
    componentA.Output1.PipeTo(componentB.Receiver1);

    // whereas this connection overrides with an Unlimited policy
    componentA.Output2.PipeTo(componentB.Receiver2, DeliveryPolicy.Unlimited);
}
```

The virtual `GetDefaultDeliveryPolicy<T>()` method allows for an even finer-grained control of the default delivery policy, by type. A developer could write their own pipeline class, derived from `Pipeline` and overwrite the `GetDefaultDeliveryPolicy<T>` method to return different default policies for different types `T`. This would enable for instance the connections in a pipeline to use a certain default policy for streams of images, versus streams of audio, versus streams of speech recognition results, etc. 


<a name="Summary"></a>

# 4. Summary of Available Delivery Policies

In the discussion above we have highlighted several untyped delivery policies, implemented as static members on the `DeliveryPolicy` class:

* `DeliveryPolicy.Unlimited`: queues and delivers all messages at the receiver (up to int.MaxValue messages).
* `DeliveryPolicy.LatestMessage`: only delivers the most recent message to the receiver, and discards all other ones.
* `DeliveryPolicy.Throttle`: delivers one message at a time to the receiver and throttles its source for as long as there is a pending queued message.
* `DeliveryPolicy.SynchronousOrThrottle`: attempts to deliver and process each message synchronously at the receiver. If the receiver is busy, the message is queued and the source is throttled until the receiver is able to process it per `DeliveryPolicy.Throttle`.

Besides these predefined policies, additional policies may be created using two available static factory methods:
* `DeliveryPolicy.LatencyConstrained(...)`: defines a latency-constrained delivery policy, where messages are delivered if their latency is below a specified `maximumLatency`. 
* `DeliveryPolicy.QueueSizeConstrained(...)`: defines a queue-size constrained delivery policy, where only the most recent messages up to a specified `maximumQueueSize` will be delivered to the receiver. 

Finally, typed delivery policies which guarantee delivery for specific messages can be constructed by calling the `WithGuarantees<T>(Func<T, bool> guaranteeDelivery)` method on an existing typed or untyped delivery policy. In this case, the `guaranteeDelivery` predicate specifies which messages should have guaranteed delivery.