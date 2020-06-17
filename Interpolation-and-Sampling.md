Interpolation and sampling are performed in Platform for Situated Intelligence via the corresponding `Interpolate` and `Sample` operators. These operators rely on a specified interpolator class - please first read the in-depth topic on [Stream Fusion and Merging](Stream-Fusion-and-Merging) to first understand interpolators before proceeding below.

Both `Interpolate` and `Sample` take as a first parameter either a clock stream that drives the interpolation/sampling by providing the interpolation or sampling points, or a `TimeSpan` interval (in which case a generator stream with that cadence is produced and used). 

The `Interpolate` operator is the most general one and accepts any interpolator. 

For example:

```csharp
var interpolated = doubleStream.Interpolate(TimeSpan.FromMilliseconds(100), Reproducible.Linear());
```

will generate an interpolated stream using linear interpolation, at a cadence of 100 milliseconds.


The `Sample` operators are intended for use to perform sampling, i.e. selecting a specific message from the source stream at a given cadence. The `Sample` operator accepts as a second parameter a `RelativeTimeInterval` or a `TimeSpan` tolerance and essentially performs sampling by interpolating with the corresponding `Reproducible.Nearest` interpolator. For instance:

```csharp
var sampled = source.Sample(TimeSpan.FromMilliseconds(100), RelativeTimeInterval.Past());
```

will construct a sampled stream on 100ms cadence that contains at each sampling point the last message on the source stream.

