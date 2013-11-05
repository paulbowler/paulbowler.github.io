---
layout: post
title: "Sorting maps by multiple keys"
date: 2013-10-24 11:34
comments: true
categories: [clojure, sorting, maps]
---
Several hundred school students have taken an exam and the Headmistress wants a list of results ordered by score. Pretty simple ask.

Here is a representative sample of the students, their age and score as a sequence of maps:
``` clj
(def results [{:name "Fred" :age 10 :score 9}
              {:name "Jim"  :age 15 :score 8}
              {:name "Bill" :age 15 :score 9}
              {:name "Emma" :age 7  :score 7}
              {:name "Jane" :age 9  :score 8}])
```

As keywords are also Clojure functions, it is very simple to sort maps by a specific key:

``` clj Sorting map by single key
(sort-by :score results)
```
``` clj
({:name "Emma", :age 7,  :score 7}
 {:name "Jim",  :age 15, :score 8}
 {:name "Jane", :age 9,  :score 8}
 {:name "Fred", :age 10, :score 9}
 {:name "Bill", :age 15, :score 9})
```
This is fine, although the scores are in lowest-first order. We can fix that by reversing the sequence:

``` clj Reversing the order of results
(reverse (sort-by :score results))
```
... or adding a second parameter to 'sort-by':
``` clj Specifying the order of results
(sort-by :score > results)
```
``` clj
({:name "Bill", :age 15, :score 9}
 {:name "Fred", :age 10, :score 9}
 {:name "Jane", :age 9,  :score 8}
 {:name "Jim",  :age 15, :score 8}
 {:name "Emma", :age 7,  :score 7})
```
The headmistress is happy, although now she also wants the results weighted by age so that of students with
equal marks the younger will come above the older.

To do this we need to be able to sort by multiple keys such as you would in a relational database using 'order by score, age'. In clojure we can't just do this by adding the second keyword to sort-by. Instead, what we need is to be able to apply each sort in turn. Looking through the docs it appears that [juxt](http://clojuredocs.org/clojure_core/clojure.core/juxt) does exactly what we need:

{% blockquote %}
Takes a set of functions and returns a fn that is the juxtaposition
of those fns. The returned fn takes a variable number of args, and
returns a vector containing the result of applying each fn to the
args (left-to-right).
{% endblockquote %}

Adding in juxt to the code gives:

``` clj Sorting map by multiple keys
(sort-by (juxt :score :age) results)
```
``` clj
({:name "Emma", :age 7,  :score 7}
 {:name "Jane", :age 9,  :score 8}
 {:name "Jim",  :age 15, :score 8}
 {:name "Fred", :age 10, :score 9}
 {:name "Bill", :age 15, :score 9})
```
Unfortunately we now have both age and score going from lowest to highest. We need score ranking from highest to lowest again as we had in the previous example. To use the SQL example again, we need 'order by score <em>desc</em>, age'. Clearly this time we can't just reverse the resulting sequence as student's ages would then be in the wrong order. Hmmm...

Under the covers Clojure sorts integers using a comparison of two values. This returns -1 for 'a < b', 0 for 'a = b' or +1 for 'a > c'. It can then use a sort algorithm to put them all in the correct place. The comparison function itself is written in the base class, in this instance 'Integer' which implements the 'Comparable' interfaces 'compareTo' method.

We cannot overide the 'compareTo' method, so we have to find other creative ways. One way is to change the score value to a negative by wrapping the map 'get' function (which is effectively what the keyword does as a function) and performing the transformation there. The scores can then be sorted in the correct order as before:

``` clj Reversing the score order
(sort-by (juxt #(* -1 (:score %)) :age) results)
```
This gives us exactly what we want:
``` clj
({:name "Fred", :age 10, :score 9}
 {:name "Bill", :age 15, :score 9}
 {:name "Jane", :age 9,  :score 8}
 {:name "Jim",  :age 15, :score 8}
 {:name "Emma", :age 7,  :score 7})
```
The Headmistress is very please and she can now process the hundreds of test results in no time at all. Result!
