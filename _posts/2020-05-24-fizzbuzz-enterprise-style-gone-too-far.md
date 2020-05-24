---
title: "Fizzbuzz enterprise-style gone too far"
date: 2020-05-24 01:30:00 -0700
---
"Enterprise-class" code design can very easily degenerate into what I will call "interface obsession"
(named after the ["primitive obsession" code smell](https://blog.codinghorror.com/code-smells/)). Let's
take a look at various strategies of abstraction and over-obessions in the context of the classic
["FizzBuzz"](https://wiki.c2.com/?FizzBuzzTest) interview question.

> Write a program that prints the numbers from 1 to 100.
> But for multiples of three print “Fizz” instead of the number and for the multiples of five print “Buzz”.
> For numbers which are multiples of both three and five print “FizzBuzz”.

## Naïve implementation
Usually, given the time constraints of an interview, a simple loop with case branches is a good first pass.

```csharp
static void Main(string[] args)
{
  for (var i = 1; i <= 100; i++)
  {
    if (i % 15 == 0)
    {
      Console.WriteLine("FizzBuzz");
    }
    else if (i % 3 == 0)
    {
      Console.WriteLine("Fizz");
    }
    else if (i % 5 == 0)
    {
      Console.WriteLine("Buzz");
    }
    else
    {
      Console.WriteLine(i);
    }
  }
}
```

Some may find enough spare time to get clever about the case of numbers divisible by both 3 and 5, or take
advantage of nice features of higher-order runtimes like .NET or Java.

```csharp
static void Main(string[] args)
{
  foreach (var i in Enumerable.Range(1, 100))
  {
    if (i % 3 == 0)
    {
      Console.Write("Fizz");
    }

    if (i % 5 == 0)
    {
       Console.Write("Buzz");
    }

    if (Console.CursorLeft == 0)
    {
      Console.Write(i);
    }

    Console.WriteLine();
  }
}
```

## Going enterprise
Now let's get this ready for enterpise use. Let's say another codebase wants to take a dependency on this
functionality, so console output isn't suitable (and the console writing actually becomes the performance
bottleneck at large iteration counts).

The simplest start would be to ship a static class with a method.

```csharp
public static class FizzBuzz
{
  public static string Generate(int i)
  {
    string output = string.Empty;

    if (i % 3 == 0)
    {
      output += "Fizz";
    }

    if (i % 5 == 0)
    {
      output += "Buzz";
    }

    if (string.IsNullOrEmpty(output))
    {
      output = i.ToString();
    }

    return output;
  }
}
```

While it gets the job done, the engineers for the consuming code want to wire this up to their
[dependency injection container](https://martinfowler.com/articles/injection.html). They don't want to just
hard-code a call to a static method in the middle of their code. Cool, makes sense.

```csharp
public interface IFizzBuzz
{
  string Generate(int i);
}

public class FizzBuzz : IFizzBuzz
{
  public string Generate(int i)
  {
    // omitted for brevity (see previous snippet)
  }
}
```

### More parameterization
With this interface being versatile and library use proliferating, now you're getting requests for more
configurability. The biggest request is for different combinations of outputs for different integer
multiples. Sounds like it's time to parameterize.

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

public class FizzBuzz : IFizzBuzz
{
  private DivisibleOutputRule[] Rules { get; }

  // for containers that can bind multiple configured instances to IEnumerable<T>
  public FizzBuzz(IEnumerable<DivisibleOutputRule> rules) : this(rules.ToArray()) { }

  public FizzBuzz(params DivisibleOutputRule[] rules) => Rules = rules;

  public string Generate(int i) => Rules
    .Where(r => r.Divisor % i == 0)
    .Select(r => r.Output)
    .DefaultIfEmpty(i.ToString())
    .Aggregate(string.Empty, (output, current) => output + current);
}
```

### More abstraction
At this point, you could easily get requests for more abstractions. Here's some examples and potential
solutions to consider.

#### Adding an object to contain the rules
A single object is easier to configure in most dependency injection containers than sets, so an object that
contains all the `DivisibleOutputRule` values makes sense.

```csharp
public class DivisibleOutputRules : IEnumerable<DivisibleOutputRule>
{
  public IEnumerable<DivisibleOutputRule> Rules { get; set; }

  public IEnumerator<DivisibleOutputRule> GetEnumerator() => Rules.GetEnumerator();

  public static implicit operator DivisibleOutputRules(
    IEnumerable<DivisibleOutputRule> rules) => new DivisibleOutputRules { Rules = rules; }
}
```

#### Adding a class instance factory
Some would argue that a factory interface should be create for creating `FizzBuzz` instances. Maybe they're
using ASP.NET Core and want the `DivisibleOutputRule` to come from request-lifetime
[IOptionsSnapshot\<TOptions\>](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options?view=aspnetcore-3.1#reload-configuration-data-with-ioptionssnapshot)
to make the configuration refreshable.

```csharp
public interface IFizzBuzzFactory
{
  IFizzBuzz Create();
}
```

It makes more sense to leave these meta-responsibilities outside the `FizzBuzz` library. An easy way to
see this is to look at an implementation of a factory for the ASP.NET Core options scenario, combined with
the rule container `DivisibleOutputRules` class above.

```csharp
public class OptionsSnapshoptFizzBuzzFactory : IFizzBuzzFactory
{
  private IEnumerable<DivisibleOutputRule> Rules { get; }

  public OptionsSnapshoptFizzBuzzFactory(IOptionsSnapsnot<DivisibleOutputRules> rulesOptions)
  {
    Rules = rulesOptions.Value;
  }

  public IFizzBuzz Create() => return new FizzBuzz(Rules);
}
```

Again, this factory is not needed in the `FizzBuzz` library. Additionally, adding the `IOptions`
concrete factory would introduce another dependency to the `FizzBuzz` library
([Microsoft.Extensions.Options](https://www.nuget.org/packages/Microsoft.Extensions.Options))
and increase possibility of
[diamond dependency problems](https://docs.microsoft.com/en-us/dotnet/standard/library-guidance/dependencies#diamond-dependencies)
in the future. Instead, if many consumers would end up writing a factory, consider placing this in a
separate `FizzBuzz.Options` library.

Another option that avoids an explicit factory would be for consumers to use the factory delegate
configuration methods provided by the dependency injection framework being used. For example,
this is available in the default container in ASP.NET Core.

```csharp
public void ConfigureServices(IServiceCollection services)
{
  services.AddScoped<IFizzBuzz>(serviceProvider =>
  {
    var options = serviceProvider.GetRequiredService<IOptionsSnapshot<DivisibleOutputRules>>();
    return new FizzBuzz(options.Value);
  });
}
```

#### Interfaces for the divisibility rule objects
Consumers may clamor for abstractions for `DivisibleOutputRule`, arguing for the ability to mock the
object or replace it with their own implementations.

```csharp
public interface IDivisibleOutputRule
{
  int Divisor { get; }
  string Output { get; }
}
```

It's fairly easy to see that this provides no additional value compared to the concrete `struct`.
- `DivisibleOutputRule` can be constructed by anything because it has no dependencies
- `DivisibleOutputRule` is basically a data-transfer object or a [POCO](https://en.wikipedia.org/wiki/Plain_old_CLR_object)

To further drive the point home, I chose to use a `readonly struct` with no methods instead of a `class`
to make it immutable, which makes it clear that it is for configuration and is not business logic that
requires interchangability with other classes/structs.

Do you think that this kind of interface obsession doesn't happen? I've seen it firsthand: POCOs that
implement an interface that was literally just extracted from said POCO, just to change some method
arguments from the concrete to the interface.

## Less abstraction: radical reductionism
Those more inclined toward functional-style programming have probably noticed this by now: why is there an
interface or class at all? The `Generate` method is just LINQ! And you are absolutely right.

```csharp
public static class EnumerableDivisibleOutputRuleExtensions
{
  public static string Generate(this IEnumerable<DivisibleOutputRule> rules, int i) => rules
    .Where(r => r.Divisor % i == 0)
    .Select(r => r.Output)
    .DefaultIfEmpty(i.ToString())
    .Aggregate(string.Empty, (output, current) => output + current);
}
```

So what was all that nonsense for? To demonstrate (albeit through a fairly contrived problem) what happens
in enterprises all the time. However, there are benefits of having a class that is not yet utilized, which
will be covered in a future post.

## Next time: improving FizzBuzz performance for real enterprise scale
There's still one big problem with all the designs for FizzBuzz above: they're computationally expensive per
call. The LINQ version always loops through all the divisor checks and computes the default `int.ToString()`
(even if it's unused). It's elegant and succinct, but intolerably slow when called in a hot path. The next
post will go down the rabbit hole of making a blazing fast implementation.
