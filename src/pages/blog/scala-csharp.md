---
layout: "../../layouts/BlogPost.astro"
title: "Scala and C# comparison"
description: "Lorem ipsum dolor sit amet"
pubDate: "2017-07-27 16:00:04 +0200"
---

So why do we need another "Scala for C# developers" or vice versa? Well, at this moment I didn't find any post which would bite deeper in the comparison. 
I - having .NET background and jumping on Scala - might have a slightly different perspective. Also I've met lots of people in both camps which aren't aware how the other camp operates.
The short answer is - both platforms are very similar, which I want to show in this series.

 So lets get started.

C# (pronounced c-sharp) is an object oriented language with some features which are attributed to functional programming running on CLR.
Scala is an object oriented language with some features which are attributed to functional programming running on JVM.

So first thing might be surprising is both descriptions are similar, the difference is that C# emphasizes OOP more then its FP features.
Scala emphasizes FP slightly more from OOP, but not as strongly as C# does it for OOP - but this is my subjective remark.

Both languages implement concepts of a `class`, inheritance, interfaces, polymorphism and other things which are attributed to be pillars of OOP, and similarly they do it for FP even thought in C# it's a little shunned - like immutability and purity, which are considered in the C# environment as plain old "good practices".

I will skip the OOP basics and language syntax differences as there are many other resources which are talking about it.

So lets start will some meat.
I will focus on Scala 2.12 and C# 7.
I will default to Scala syntax as most of the time its more terse.

One of the more interesting things I heard about Scala is that its collections API is the best thing the person ever saw.
Ommiting the fact that the design of collections itself is heavily criticized in the community, but still is focused on the the developer experience.
While C# have similarly good experience, it achieves it differently - with the so call Language Integrated Query(LINQ).

Lets assume we have a collection of people(Person) and we want to operate on that collection.
Both languages provide unified APIs on many different collection types. ~~Scala has a quite convoluted hierarchy of traits and other tricks~~(more to read [here](https://www.scala-lang.org/blog/2017/05/30/tribulations-canbuildfrom.html)), and C# achieves a lot through extension methods(a gross over simplification).

Having a person
```scala
case class Person(name: String, surname: String, age: Int)
```
This is equivalent with having a class in C# of the same name with single constructor having all 3 parameters on it and providing backing fields and property getters for read-only access. There are talks to introduce Record syntax to C# in version 8. So fret not!


So common operations in scala would look like
```scala
val people = List(Person(.....),....)
people.filter(p => p.age > 18)
people.map(p => p.copy(age => p.age + 1)) //copies the class with changing the value
//getGroceries(p:Person): List[Grocery]
people.flatMap(p => getGroceries(p)) //returns List[Grocery]
```
This is also a very verbose way of declaring things in Scala.
`p => p.age > 18` can be replace with `_.age > 18` and `flatMap(p => getGroceries(p))` to `flatMap(getGroceries)`.

In C# it looks quite similar

```csharp
var people =  new List<Person> {
  new Person(.....)......
}
people.Where(p => p.age > 18)
//needed to introduce some function as classes don't have copy function
people.Select(f => incrementAge(f))
//you can also pass in the function directly
people.Select(incrementAge)
//List<Grocery> GetGroceries(Person p)
people.SelectMany(p => GetGroceries(p)) //returns List<Grocery>
```

Eerily similar isn't it?

There is also another interesting property emerging here.
In Scala there are so called `for-comprehensions`.

```scala
for {
  p <- people
  if p.age > 18
  groceries <- getGroceries(p)
} yield groceries
```
```
This will take all people, filter out only people having more than 18 years old and find their groceries and return them as a single list.
For-comprehension is a sugar syntax on `map`, `flatMap`, `foreach` and `withFilter`.

It will "lower" to

```scala
p.withFilter(_.age > 18).flatMap(getGroceries)
```

Why `withFilter`?
```
Note: the difference between c filter p and c withFilter p is that the former
creates a new collection, whereas the latter only restricts the domain of
subsequent map, flatMap, foreach, and withFilter operations.
```

In C# with LINQ, there is the Query syntax

```csharp
from p in people
where p.age > 18
from g in GetGroceries(p)
select g
```

In addition LINQ provides methods like `orderby` which makes it is a breeze to sort data.

In both cases the "lowering" or "desugaring" can be done for whenever the object has specific methods accessible, example via extension methods or implicit classes.

```scala
case class Query()

object QueryOps {
  implicit class RichQuery(query: Query) {
    def map[B](f: Query => B): B = f(query)
    def flatMap[B] ...
    def withFilter....
  }
}

val q = Query()

val result = for{
  q1 <- q
} yield q1
```

For C#

```csharp
public static class LINQQueryExtension
{
   public static T Select<T>(this Query query, Func<Query, T> foo) => foo(query);
   public static T SelectMany<T>(this Query query, Func<Query, T> foo) => ...
   public static ??? Where(this Query query, Func<Query, bool> predicate) =>...
} 

var q = new Query();

var result = from z in q
             select z;
```

I omit flatMap/SelectMany and withFilter/Where for now as it gets more complex and requires special overloads to make it work with many different collection types.
It is not necessary to have all these methods always available, but then it won't compile when your `for-comprehension` or `query` will try to resolve them - compile time.