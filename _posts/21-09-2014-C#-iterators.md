---
layout: post
title:  "C# iterators"
date:   2014-09-21 22:38:58
categories: C# iterators
---

In .NET version 2.0 among generics, partial classes and nullable types which we all know and love there was another interesting construct - [iterators](http://msdn.microsoft.com/en-us/library/dscyy5s0(v=vs.80).aspx).

They may not be used as often as generics and may not be so fundamental, but there is still a lot of situations when they come in handy. The following example shows a very basic usage of iterators.

{% highlight csharp %}


// this method returns an iterator ...
public IEnumerable<string> GetSongs()
{
    yield return "all along the watchtower";
    yield return "purple haze";
}

// ... and this one consumes it
public void DisplaySongs()
{
    var songs = GetSongs();

    foreach (var s in songs)
    {
        Console.WriteLine(s);
    }
}

{% endhighlight %}

Official documentation for iterators says:

> When the compiler detects your iterator, it will automatically generate the Current, MoveNext and Dispose methods of the IEnumerable or IEnumerable<T> interface.

This example, as simple as it may be shows some important features of iterators:

* Iterators are constructed using ```yield``` keyword
* The easiest way of accessing iterator values is through ```foreach(...)``` loop
* Iterator values are retrieved upon request, some even may say they are _lazy-loaded_

I would like to focus on the last attribute. Let's modify the example a little bit.

{% highlight csharp %}

public IEnumerable<string> GetSongs()
{
    yield return "all along the watchtower";

    // this will never be returned and therefore never called
    yield return "purple haze"; 
}

public void DisplaySongs()
{
    var songs = GetSongs();

    // We take only one element from the iterator
    foreach (var s in songs.Take(1))
    {
        Console.WriteLine(s);
    }
}

{% endhighlight %}

The example makes use of [LINQ Take()](http://msdn.microsoft.com/en-us/library/vstudio/bb503062(v=vs.100).aspx) method.

Because the values from an iterator are only retrieved when needed we could write a nice incremental numbers generator using an infinite loop.

{% highlight csharp %}


// this method is an iterator ...
public IEnumerable<int> GetSongs()
{
    var i = 0;

    while (true)
    {
        yield return i++;
    }
}

// ... and this one consumes it
public void DisplaySongs()
{
    var songs = GetSongs();

    // We take only one element from the iterator
    foreach (var s in songs.Take(10))
    {
        Console.WriteLine(s);
    }
}

{% endhighlight %}

Yes, there **is** an infinite loop, but because we have ```Take(10)``` the loop will not cause our application to crash.

This example shows that we can easily use iterators to create infinite collections of objects. This construct can be very useful when writing LINQ queries.

Under the hood
--------------

The code from the previous example seems very useful and reusable. There are in fact two ```Enumerable``` methods that do very similar thing: ```Enumerable.Range()``` and ```Enumerable.Repeat()```

Let's have a look under the hood. I used [dotPeek](http://www.jetbrains.com/decompiler/) to get the decompile ```System.Linq.dll``` but I guess Reflector should return similar results

{% highlight csharp %}

[__DynamicallyInvokable]
public static IEnumerable<int> Range(int start, int count)
{
  long num = (long) start + (long) count - 1L;
  if (count < 0 || num > (long) int.MaxValue)
    throw Error.ArgumentOutOfRange("count");
  else
    return Enumerable.RangeIterator(start, count);
}

private static IEnumerable<int> RangeIterator(int start, int count)
{
  for (int i = 0; i < count; ++i)
    yield return start + i;
}

[__DynamicallyInvokable]
public static IEnumerable<TResult> Repeat<TResult>(TResult element, int count)
{
  if (count < 0)
    throw Error.ArgumentOutOfRange("count");
  else
    return Enumerable.RepeatIterator<TResult>(element, count);
}

private static IEnumerable<TResult> RepeatIterator<TResult>(TResult element, int count)
{
  for (int i = 0; i < count; ++i)
    yield return element;
}

{% endhighlight %}

As we can see both methods use iterators and 95% of the time these two methods are all you need.

Real life example
-----------------

Let's assume that we are building a system which generates unique random strings. 'Classical' approach would be to use while loop and generate codes until we find a unique one.

{% highlight csharp %}

public string GetRandomString()
{
    var code = string.Empty;

    while (string.IsNullOrWhiteSpace(code))
    {
        code = this.GetRandomString(5);

        if (Context.Codes.Any(o => o.Value == code))
        {
            code = string.Empty;
        }
    }

    return code;
}

{% endhighlight %}

The same code but using iterators will look like that.


{% highlight csharp %}

public string GetRandomString()
{
    return Enumerable
        .Repeat(this.GetRandomString(5), int.MaxValue)
        .First(x => context.Codes.All(o => o.Value != x));
}

{% endhighlight %}

Summary
-------

Using declarative ```LINQ``` instead of imperative ```while``` has many benefits, it is easier to parallelize code execution, it is more readable and in some cases produces more efficient code. Iterators are a very nice bridge that brings LINQ into problems that normally would be only solved using imperative code.
