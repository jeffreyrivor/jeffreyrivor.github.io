---
date: 2023-08-11 17:45:00 -0700
---
# FizzBuzz Generics

Last time (again, quite a while ago) I ended with a generalization of the ["FizzBuzz"](https://wiki.c2.com/?FizzBuzzTest)
problem that parameterized the divisibility rules and cached constant results for inputs using a function cache the size of the
[least common multiple](https://en.wikipedia.org/wiki/Least_common_multiple), which is the count at which the rules that generate
the output will cycle.

```csharp
// data structure representing constant output result when an input is divisible by the divisor
public readonly struct DivisibleOutputRule
{
  public int Divisor { get; } // assumes integer divisors
  public string Output { get; } // assumes string output

  public DivisibleOutputRule(int divisor, string output)
  {
    Divisor = divisor;
    Output = output;
  }
}

// class encapsulating the function cache based on the DivisibleOutputRule
public class FizzBuzz
{
  private Func<int, string>[] Generators { get; }

  // for containers that can bind multiple configured instances to IEnumerable<T>
  public FizzBuzz(IEnumerable<DivisibleOutputRule> rules) => Generators = GenerateFunctionCache(rules);

  public FizzBuzz(params DivisibleOutputRule[] rules) => Generators = GenerateFunctionCache(rules);

  public string Generate(int i) => Generators[i % Generators.Length](i);

  private static Func<int, string>[] GenerateFunctionCache(this IEnumerable<DivisibleOutputRule> rules)
  {
    var length = LeastCommonMultiple(rules.Select(r => r.Divisor).ToArray());
    var cache = new Func<int, string>[length];

    for (var i = 0; i < length; i++)
    {
      // assumes rules are divisibility rules
      var matches = rules.Where(r => i % r.Divisor == 0).Select(r => r.Output).ToList();
      if (matches.Length > 0)
      {
        // assumes that any input that fulfills multiple rules have their output concatenated
        var result = string.Concat(matches);
        cache[i] = i => result;
      }
      else
      {
        // assumes that any input does not fulfill any rules output their string representation
        cache[i] = Convert.ToString;
      }
    }
  }

  private static int LeastCommonMultiple(params int[] divisors)
  {
    // implementation of an LCM algorithm
  }
}
```

## Generalization with .NET 7 System.Numerics

.NET 7 adds support of [generic math](https://learn.microsoft.com/en-us/dotnet/standard/generics/math) via
[System.Numerics](https://learn.microsoft.com/en-us/dotnet/api/system.numerics?view=net-7.0), making it possible to use a
generic number type in place of `int` as long as the number type has the concepts of zero, modulo (division), and addition.
Furthermore, the output type can be made generic as well, forcing parameterization of the "default" output and the
aggregation of outputs when multiple divisibility rules match.

```csharp
public class FizzBuzz<TNumber, TOutput>
  where TNumber : IBinaryInteger<TNumber>
  where TOutput : IEquatable<TOutput>
{
  private TNumber LeastCommonMultiple { get; }
  private IDictionary<TNumber, Func<TNumber, TOutput>> Generators { get; }

  // for containers that can bind multiple configured instances to IEnumerable<T>
  public FizzBuzz(
    Func<TNumber, TOutput> unmatchedNumberOutputGenerator,
    Func<IEnumerable<TOutput>, TOutput> outputAggregator,
    IEnumerable<(TNumber Divisor, TOutput Output)> rules)
    : this(unmatchedNumberOutputGenerator, outputAggregator, rules.ToArray())
  { }

  public FizzBuzz(
    Func<TNumber, TOutput> unmatchedNumberOutputGenerator,
    Func<IEnumerable<TOutput>, TOutput> outputAggregator,
    params (TNumber Divisor, TOutput Output)[] rules)
  {
    // small revision to ensure divisors used for LCM computation are distinct since LCM(a, b, b) == LCM(a, b)
    LeastCommonMultiple = GetLeastCommonMultiple(rules.Select(r => r.Divisor).Distinct().ToArray());
    Generators = new Dictionary<TNumber, Func<TNumber, TOutput>>();

    for (var i = TNumber.Zero; i < LeastCommonMultiple; i++)
    {
      var matchedOutputs = rules.Where(r => i % r.Divisor == TNumber.Zero).Select(r => r.Output).ToList();

      if (matchedOutputs.Count == 0)
      {
        Generators[i] = unmatchedNumberOutputGenerator;
      }
      else
      {
        var result = outputAggregator(matchedOutputs);
        Generators[i] = (_) => result;
      }
    }
  }

  public TOutput Generate(TNumber number) => Generators[i % LeastCommonMultiple](number);

  private static TNumber GetLeastCommonMultiple(params TNumber[] distinctDivisors)
  {
    // implementation of an LCM algorithm
  }
}

class Program
{
  static Main(string[] args)
  {
    // example usage
    var fizzBuzz = new FizzBuzz<int, string>(
      Convert.ToString, // numbers not divisible by 3 or 5 output the number as a string
      string.Concat, // numbers divisible by 3 and 5 output "Fizz" and "Buzz" concatenated together
      (3, "Fizz"),
      (5, "Buzz"));

    Console.WriteLine(fizzBuzz.Generate(30));
  }
}
```

One of the most noticable side-effects of switching to a generic number type from `int` is that it is no longer possible
to create an array the size of the least common multiple, since arrays can't be constructed with length from a generic number.
As a result:
- `Dictionary<TNumber, Func<TNumber, TOutput>>` is used as the function cache instead.
- The least common multiple is used for the modulo operation to determine the `Dictionary` lookup instead.

I also eliminated the `struct DivisibleOutputRule` in favor of the built-in
[ValueTuple](https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple?view=net-7.0). To ensure the field equality works
as expected, `TOutput` is constrained to `IEquatable<TOutput>` to ensure equality (not just referential equality) is implemented.

## Next time: extreme enterprise generalization

It's not enterprise until you take it to the extreme and add so many generics you can't tell it's "FizzBuzz" anymore.
We're pretty close where we are now, but what else can be done?
