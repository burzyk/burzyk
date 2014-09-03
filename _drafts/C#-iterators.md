---
layout: post
title:  "C# iterators"
date:   2014-09-03 22:38:58
categories: C# iterators
---

In .NET [version 2.0](http://link-to-iterators) among generics which we all know and love there was another interesting construct - [iterators](http://link-to-iterators-on-wiki).

They may not be used as often as generics and may not be so fundamental, but there is still a lot of situations when they come in handy. The following example shows a very basic usage of iterators.

{% highlight csharp %}


// this method is an iterator ...
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

As we can see the first method returns an ```IEnumerable<string>``` which will then be upon enumeration return two strings. The second method displays the content of the iterator.

This example, as simple as it may be shows some important features of iterators:

* Iterators are constructed using ```yield``` keyword
* The easiest way of accessing iterator values is through ```foreach(...)``` loop
* Iterator values are retrieved upon request, some even may say they are _lazy-loaded_

I would like to focus on the last attribute. If we modify our example a little we will see exactly what I mean.

{% highlight csharp %}


// this method is an iterator ...
public IEnumerable<string> GetSongs()
{
    yield return "all along the watchtower";

    // this will never be returned and therefore called
    yield return "purple haze"; 
}

// ... and this one consumes it
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

The example makes use of [linq](http://link-to-linq) ```Take()``` method. Please remember that **Iterators works best with LINQ**.

Because the values from an iterator are only retrieved when needed we could write ourselves a nice incremental numbers generator using an infinite loop.

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

Yes, there **is** an infinite loop, but because we have ```Take(10)``` the loop will not cause our application to crash. But if we took ```yield``` or ```Take(10)``` out then it **would**.

This example shows that we can easily use iterators to create infinite collections of objects. This construct can be very useful when writing LINQ queries.

// 
Write about Enumerable.Range and Enumerable.Repeat. 
Give example of while to Linq rewrite.