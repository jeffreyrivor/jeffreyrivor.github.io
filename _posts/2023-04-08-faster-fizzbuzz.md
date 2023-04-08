---
date: 2023-04-08 02:35:00 -0700
---
# Faster FizzBuzz

Last time (quite a while ago) I talked about "enterprise" code design with the ["FizzBuzz"](https://wiki.c2.com/?FizzBuzzTest)
problem. I ended with the shortest implementation in LINQ, which (not-so-coincidentally) is the lightest of the "enterprise"
patterns (in terms of number of abstractions) while having a level of parameterization not in the standard "first-draft"
procedural method.

```csharp
public readonly struct DivisibleOutputRule
{
  public int Divisor { get; }
  public string Output { get; }

  public DivisibleOutputRule(int divisor, string output)
  {
    Divisor = divisor;
    Output = output;
  }
}

public static class EnumerableDivisibleOutputRuleExtensions
{
  public static string Generate(this IEnumerable<DivisibleOutputRule> rules, int i) => rules
    .Where(r => r.Divisor % i == 0)
    .Select(r => r.Output)
    .DefaultIfEmpty(i.ToString())
    .Aggregate(string.Empty, (output, current) => output + current);
}
```

There are a few assumptions in this implementation worth noting:
1. The default output is `int.ToString()`
2. The parameters can only set the divisor and the output string for that divisor
3. When multiple divisors match the output string is a concatenation of all the matches in the order that they were specified

Besides the restrictions imposed by these assumptions, it is also not very performant. I'll focus on this aspect today.

## Result caching

With the assumptions currently in place, one interesting fact that can be leveraged is that the generator is a
[pure function](https://en.wikipedia.org/wiki/Pure_function). As a pure function, the output can be cached or pre-computed
for a given input, and in pratical applications manifests as memory or distributed caches with eviction policies (usually
leveraging non-random disproportiate demand to keep the outputs for frequently used inputs).

This pre-computed result cache can be taken one step further in FizzBuzz: the output for the specified divisors is always the
same, so several inputs map to the same output. The outputs cycle on the
[least common multiple](https://en.wikipedia.org/wiki/Least_common_multiple) of the divisors. For example, in the specific
FizzBuzz problem the LCM 15, so the input space can be reduced to an array of 15 functions that generate the output.

```csharp
public static class FizzBuzz
{
  // cache of output functions, indexed based on i % LCM(3, 5)
  private static Func<int, string>[] Generators = new Func<int, string>[]
  {
    i => "FizzBuzz", // divisible by 15
    Convert.ToString,
    Convert.ToString,
    i => "Fizz", // divisible by 3
    Convert.ToString,
    i => "Buzz", // divisible by 5
    i => "Fizz", // divisible by 6
    Convert.ToString,
    Convert.ToString,
    i => "Fizz", // divisible by 9
    i => "Buzz", // divisible by 10
    Convert.ToString,
    Convert.ToString,
    Convert.ToString,
    Convert.ToString,
  };

  public static string Generate(int i) => Generators[i % Generators.Length](i);
}
```

### Parameterizing the result cache

To generalize the result cache, the array of functions need to be generated based on the `IEnumerable<DivisibleOutputRule>`.

```csharp
public static class EnumerableDivisibleOutputRuleExtensions
{
  // use the generated set to compute the output for a given input
  public static string Generate(this Func<int, string>[] functions, int i) => functions[i % functions.Length](i);

  // generate the set from the rules, keep for the result reuse
  public static Func<int, string>[] GenerateFunctionCache(this IEnumerable<DivisibleOutputRule> rules)
  {
    var length = LeastCommonMultiple(rules.Select(r => r.Divisor).ToArray());
    var cache = new Func<int, string>[length];

    for (var i = 0; i < length; i++)
    {
      var matches = rules.Where(r => i % r.Divisor == 0).Select(r => r.Output).ToList();
      if (matches.Length > 0)
      {
        var result = string.Concat(matches);
        cache[i] = i => result;
      }
      else
      {
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

Then with the extension methods starting to leak abstractions (the function cache array), bringing back a class to
encapsulate the functionality starts to make more sense.

```csharp
public class FizzBuzz
{
  private Func<int, string>[] Generators { get; }

  // for containers that can bind multiple configured instances to IEnumerable<T>
  public FizzBuzz(IEnumerable<DivisibleOutputRule> rules) => Generators = GenerateFunctionCache(rules);

  public FizzBuzz(params DivisibleOutputRule[] rules) => Generators = GenerateFunctionCache(rules);

  public string Generate(int i) => Generators[i % Generators.Length](i);

  private static Func<int, string>[] GenerateFunctionCache(this IEnumerable<DivisibleOutputRule> rules)
  {
    // omitted for brevity (see previous snippet)
  }

  private static int LeastCommonMultiple(params int[] divisors)
  {
    // implementation of an LCM algorithm
  }
}
```

## Next time: more enterprise in the more performant FizzBuzz

With the runtime performance largely addressed, the next post will go through various generics strategies to reduce
hard-coded assumptions and extract a set of generalized base functionality.
