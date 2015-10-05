---
layout: doc
title: "From ES6 to Scala: Collections"
---

In JavaScript there are basically two kinds of collections you have used to store your data: the `Array` for sequential
data and `Object` (aka dictionary or hash map) for storing key-value pairs. Furthermore both of these are mutable by
default, so if you pass them to a function, that function might go and modify them without your knowledge.

ES6 extends your options with four new collection types `Map`, `Set`, `WeakMap` and `WeakSet`. Of these the `WeakMap`
and `WeakSet` are for special purposes only, so in your application you would typically use only `Map` and `Set`.

## Scala collection hierarchy

Unlike JavaScript, the Scala standard library has a huge variety of different collection types to choose from.
Furthermore the collections are organized in a type hierarchy, meaning they share a lot of common functionality and
interfaces. The high-level hierarchy for the abstract base classes and traits is shown in the image below.

![Scala collection hierarchy](http://docs.scala-lang.org/resources/images/collections.png)

Scala provides _immutable_ and _mutable_ implementations for all these collection types.

<table class="table table-bordered">
<tr><th colspan="2"><h6>Common <i>immutable</i> collections</h6></th></tr>
<tr><td>Seq</td><td>List, Vector, Stream, Range</td></tr>
<tr><td>Map</td><td>HashMap, TreeMap</td></tr>
<tr><td>Set</td><td>HashSet, TreeSet</td></tr>
<tr><th colspan="2"><h6>Common <i>mutable</i> collections</h6></th></tr>
<tr><td>Seq</td><td>Buffer, LinkedList, Queue</td></tr>
<tr><td>Map</td><td>HashMap, LinkedHashMap</td></tr>
<tr><td>Set</td><td>HashSet</td></tr>
</table>

## Comparing to JavaScript

Let's start with familiar things and see how Scala collections compare with the JavaScript `Array` and `Object` (or
`Map`). The closest match for `Array` would be the mutable `Buffer` since arrays in Scala cannot change size after
initialization. For `Object` (or `Map`) the best match is the mutable `HashMap`.

A simple example of array manipulation.

{% columns %}
{% column 6 ES6 %}
{% highlight javascript %}
const a = ["Fox", "jumped", "over"];
a.push("me"); // Fox jumped over me
a.unshift("Red"); // Red Fox jumped over me
const fox = a[1];
a[a.length - 1] = "you"; // Red Fox jumped over you
console.log(a.join(" ")); 
{% endhighlight %}
{% endcolumn %}
        
{% column 6 Scala %}
{% highlight scala %}
import scala.collection.mutable
val a = mutable.Buffer("Fox", "jumped", "over")
a.append("me") // Fox jumped over me
a.prepend("Red") // Red Fox jumped over me
val fox = a(1)
a(a.length - 1) = "you" // Red Fox jumped over you
println(a.mkString(" "))
{% endhighlight %}
{% endcolumn %}
{% endcolumns %}


Working with a hash map (or Object).

{% columns %}
{% column 6 ES6 %}
{% highlight javascript %}
const p = {first: "James", last: "Bond"};
p["profession"] = "Spy";
const name = `${p.first} ${p.last}`
{% endhighlight %}
{% endcolumn %}
        
{% column 6 Scala %}
{% highlight scala %}
import scala.collection.mutable
val p = mutable.HashMap("first" -> "James", 
  "last" -> "Bond")
p("profession") = "Spy"
val name = s"${p("first")} ${p("last")}"
{% endhighlight %}
{% endcolumn %}
{% endcolumns %}

Even though you can use Scala collections like you would use arrays and objects in JavaScript, you really shouldn't,
because you are missing a lot of great functionality.

## Common collections `Seq`, `Map`, `Set` and `Tuple`

For 99% of the time you will be working with those four common collection types in your code. You will instantiate
implementation collections like `Vector` or `HashMap`, but in your code you don't really care what the implementation is,
as long as it behaves like a `Seq` or a `Map`.


#### Tuple

You may have noticed that `Tuple` is not shown in the collection hierarchy above, because it's a very specific
collection type of its own. Scala tuple combines a fixed number of items together so that they can be passed around as a
whole. A tuple is immutable and can hold different types, so it's quite close to an anonymous case class in that sense.
Tuples are used in situations where you need to group items together, like key and value in a map, or to return multiple
values. In JavaScript you can use a fixed size array to represent a tuple.

{% columns %}
{% column 6 ES6 %}
{% highlight javascript %}
const t = ["James", "Bond", 42];
const kv = ["key", 42];

function sumProduct(s) {
  let sum = 0;
  let product = 1;
  for(let i of s) {
    sum += i;
    product += i;
  }
  return [sum, product];
}
{% endhighlight %}
{% endcolumn %}
{% column 6 Scala %}
{% highlight scala %}
val t = ("James", "Bond", 42)
val kv = "key" -> 42 // same as ("key", 42)

def sumProduct(s: Seq[Int]):(Int, Int) = {
  var sum = 0
  var product = 1
  for(i <- s) {
    sum += i
    product +=i1
  }
  (sum, product)
}
{% endhighlight %}
{% endcolumn %}
{% endcolumns %}

To access values inside a tuple, use the `tuple._1` syntax, where the number indicates position within the tuple
(starting from 1, not 0). Quite often you can also use _destructuring_ to extract the values.

{% columns %}
{% column 6 ES6 %}
{% highlight javascript %}
const sc = sumProduct([1, 2, 3]);
const sum = sc[0];
const product = sc[1];

// with destructuring
const [sum, product] = sumProduct([1, 2, 3]);
{% endhighlight %}
{% endcolumn %}
{% column 6 Scala %}
{% highlight scala %}
val sc = sumProduct(Seq(1, 2, 3))
val sum = sc._1
val product = sc._2

// with destructuring
val (sum, product) = sumProduct(Seq(1, 2, 3))
{% endhighlight %}
{% endcolumn %}
{% endcolumns %}

#### Seq

`Seq` is an ordered sequence. Typical implementations include `List`, `Vector`, `Buffer` and `Range`. Although Scala
`Array` is not a `Seq`, it can be wrapped into a `WrappedArray` to enable all `Seq` operations on arrays. In Scala this
is done automatically through an implicit conversion, allowing you to write code like following.

{% columns %}
{% column 9 Scala %}
{% highlight scala %}
val ar = Array(1, 2, 3, 4)
val product = ar.foldLeft(1)((a, x) => a * x) // foldLeft comes from WrappedArray
{% endhighlight %}
{% endcolumn %}
{% endcolumns %}

The `Seq` trait exposes many methods familiar to the users of JavaScript arrays, including `foreach`, `map`, `filter`,
`slice`, `find` and `reverse`. In addition to these, there are several more useful methods shown with examples in the
code block below.

{% columns %}
{% column 9 Scala %}
{% highlight scala %}
val ar = Seq(1, 2, 3, 4, 5)
ar.isEmpty == false
ar.forall(x => x > 0) == true // JS Array.every()
ar.exists(x => x % 3 == 0) == true // JS Array.some()
ar.head == 1
ar.tail == Seq(2, 3, 4, 5)
ar.last == 5
ar.init == Seq(1, 2, 3, 4)
ar.drop(2) == Seq(3, 4, 5)
ar.dropRight(2) == Seq(1, 2, 3)
ar.count(x => x < 3) == 2
ar.groupBy(x => x % 2) == Map(1 -> Seq(1, 3, 5), 0 -> Seq(2, 4))
ar.sortBy(x => -x) == Seq(5, 4, 3, 2, 1)
ar.partition(x => x > 3) == (Seq(4, 5), Seq(1, 2, 3))
ar :+ 6 == Seq(1, 2, 3, 4, 5, 6)
ar ++ Seq(6, 7) == Seq(1, 2, 3, 4, 5, 6, 7) // JS Array.concat()
{% endhighlight %}
{% endcolumn %}
{% endcolumns %}

JavaScript `Array.reduce` functionality is covered by separate `reduceLeft` and `foldLeft` methods. The difference is
that in `foldLeft` you provide an initial ("zero") value (which is optional parameter to `Array.reduce`) and in 
`reduceLeft` you don't. Also note that in `foldLeft` the type of the accumulator can be something else, for example a
tuple, but in `reduceLeft` it must always be a supertype of the value. 

{% columns %}
{% column 6 ES6 %}
{% highlight javascript %}
function sumProduct(s) {
  // destructuring works in the function argument
  return s.reduce(([sum, product], x) => 
    [sum + x, product * x], 
    [0, 1] // use an array to represent a tuple
  );
}
{% endhighlight %}
{% endcolumn %}
{% column 6 Scala %}
{% highlight scala %}
def sumProduct(s: Seq[Int]):(Int, Int) = {
  // use a tuple accumulator to hold sum and product
  s.foldLeft((0, 1)){ case ((sum, product), x) => 
    (sum + x, product * x) 
  }
}
{% endhighlight %}
{% endcolumn %}
{% endcolumns %}


## Map

A `Map` consists of pairs of keys and values. Both keys and values can be of any valid Scala type, unlike in JavaScript
where an `Object` may only contain `string` keys (the new ES6 `Map` allows using other types as keys, but supports only
referential equality for comparing keys). As keys must be unique, a `Map` is a great way to store information you need
to look up later.

JavaScript `Object` doesn't really have methods for using it as a map, although you can iterate over the keys
with `Object.keys`. When using `Object` as a map, most developers use utility libraries like
[lodash](https://lodash.com/docs#mapValues) to get access to suitable functionality. The ES6 `Map` object contains
`keys`, `values` and `forEach` methods for accessing its contents, but all transformation methods are missing.

You can build a map directly or from a sequence of key-value pairs.

{% columns %}
{% column 6 ES6 %}
{% highlight javascript %}
// object style map
const m = { first: "James", last: "Bond" };
// ES6 Map
const data = [["first", "James"], ["last", "Bond"]];
const m2 = new Map(data);
{% endhighlight %}
{% endcolumn %}
        
{% column 6 Scala %}
{% highlight scala %}
val m = Map("first" -> "James", "last" -> "Bond")
val data = Seq("first" -> "James", "last" -> "Bond")
val m2 = Map(data:_*)
{% endhighlight %}
{% endcolumn %}
{% endcolumns %}

In Scala when a function expects a variable number of parameters (like the `Map` constructor), you can destructure a
sequence with the `seq:_*` syntax.

Accessing `Map` contents can be done in many ways.

{% columns %}
{% column 6 ES6 %}
{% highlight javascript %}
// object syntax
const name = `${m.last}, ${m.first} ${m.last}`
// ES6 Map syntax
const name2 = `${m2.get("last"), m2.get("first") m2.get("last")`
// use default value when missing
const age = m.age === undefined ? "42" : m.age;
// check all fields are present
const person = m.first !== undefined && 
  m.last !== undefined && 
  m.age !== undefined ? `${m.last}, ${m.first}: ${m.age}` :
  "missing";
{% endhighlight %}
{% endcolumn %}
        
{% column 6 Scala %}
{% highlight scala %}
val name = s"${m("last")}, ${m("first")} ${m("last")}"
// use default value when missing
val age = m.getOrElse("age", "42")
// check all fields are present
val person = (for {
    first <- m.get("first")
    last <- m.get("last")
    age <- m.get("age")
  } yield s"$last, $first: $age"
  ).getOrElse("missing")
{% endhighlight %}
{% endcolumn %}
{% endcolumns %}

In the previous example `m.get("first")` returns an `Option[String]` indicating whether the key is present in the map
or not. By using for comprehension we can easily extract three separate values from the map and use them to build the
result. The result from `for {} yield` is also an `Option[String]` so we can use `getOrElse` to provide a default value.

Let's try something more complicated. Say we need to maintain a collection of players and all their game scores. This
could be represented by a `Map[String, Seq[Int]]`

{% columns %}
{% column 6 ES6 %}
{% highlight javascript %}
const scores = {};

function addScore(player, score) {
  if (scores[player] === undefined) 
    scores[player] = [];
  scores[player].push(score);
}

function bestScore() {
  let bestScore = 0;
  let bestPlayer = "";
  for(let player in scores) {
    const max = scores[player].reduce( (a, score) =>
      Math.max(score, a)
    );
    if( max > bestScore ) {
      bestScore = max;
      bestPlayer = player;
    }
  }
  return [bestPlayer, bestScore];
}

function averageScore() {
  let sum = 0;
  let count = 0;
  for(let player in scores) {
    for(let score of scores[player]) {
      sum += score;
      count++;
    }
  }
  if( count == 0 )
    return 0;
  else 
    return Math.round(sum / count); 
}
{% endhighlight %}
{% endcolumn %}
        
{% column 6 Scala %}
{% highlight scala %}
var scores = Map.empty[String, Seq[Int]]

def addScore(player: String, score: Int) {
  val prev = scores.getOrElse(player, Seq())
  scores = scores.updated(player, prev :+ score)
}

def bestScore: (String, Int) = {
  val all = scores.flatMap { 
    case (player, pScores) =>
      pScores.map(s => (player, s))
  }
  if (all.isEmpty)
    ("", 0)
  else
    all.maxBy(_._2)
}

def averageScore: Int = {
  val allScores = scores.flatMap(_._2)
  if (allScores.isEmpty)
    0
  else
    allScores.sum / allScores.size
}
{% endhighlight %}
{% endcolumn %}
{% endcolumns %}



-------------


{% columns %}
{% column 6 ES6 %}
{% highlight javascript %}
{% endhighlight %}
{% endcolumn %}
        
{% column 6 Scala %}
{% highlight scala %}
{% endhighlight %}
{% endcolumn %}
{% endcolumns %}