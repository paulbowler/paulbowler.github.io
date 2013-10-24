---
layout: post
title: "Merging sequences of maps by key values"
date: 2013-10-25 15:43
comments: true
categories: [clojure, maps, merge]
---

A school has data about its students split in different database and needs to combine them into one to supply an important report to the governors. The databases have a common key of :name, and are made up of:

Class data:
``` clj Class Data
(def class-data [
         {:name "Tom Thumb",  :class :twyford}
         {:name "Bob Smith",  :class :marwell}
         {:name "Jim Brown",  :class :owslebury}
         {:name "Kate Jones", :class :littleton}
         {:name "Helen Pain", :class :alresford}])
```

...house data:
``` clj House Data
(def house-data [
         {:name "Tom Thumb",  :house :byron}
         {:name "Bob Smith",  :house :shelley}
         {:name "Jim Brown",  :house :auden}
         {:name "Kate Jones", :house :byron}
         {:name "Helen Pain", :house :blake}])
```

...and school meal data:

``` clj School Meal Data
(def meal-data [
         {:name "Tom Thumb",  :meal :vegetarian}
         {:name "Bob Smith",  :meal :lunchbox}
         {:name "Jim Brown",  :meal :hot}
         {:name "Kate Jones", :meal :lunchbox}
         {:name "Helen Pain", :meal :hot}])
```

Combining maps using the 'merge' function seems the obvious place to start, but this will only merge sequences of maps, not
sequences of sequences of maps.

Using a bit of hammock-driven thinking time and functinal decomposition, we can break a possible solution down to:

	1 Find all the unique values in :name across the databases
	2 For each unique :name, get the maps containing that key/value pair from each database
	3 Merge the maps into new maps

To find the unique key values we can map the :key across all databases, then remove duplicates:

``` clj Unique key-values across databases
(defn unique-key-values [key colls] (distinct (flatten (map #(map key %) colls))))
```

Calling this with a key of :name and a collection of our databases:
``` clj
(unique-key-values :name [class-data house-data meal-data])
```
gives us:
``` clj Unique key values
("Tom Thumb" "Bob Smith" "Jim Brown" "Kate Jones" "Helen Pain")
```
For each :name value we now need to merge all the maps from each database that contain this key/value combination:
``` clj Merge maps containing key/value pair
(defn merge-key-val [key val colls]
  (apply merge (filter #(= (key %) val)(apply concat colls))))
```
For :name as the key and 'Tom Thumb' as the value, this gives us results:
```
{:meal :vegetarian, :house :byron, :name "Tom Thumb", :class :twyford}
```
Excellent! Now we need to bring it all together apping the 'merge-key-val' function over all :name values:
``` clj Final code
(defn key-merge [key & colls]
  (map #(merge-key-val key % colls) (unique-keys key colls)))
```
We can now get the final merged database:
``` clj
(key-merge :name class-data house-data meal-data)
```
``` clj
({:meal :vegetarian, :house :byron, :name "Tom Thumb", :class :twyford}
 {:meal :lunchbox, :house :shelley, :name "Bob Smith", :class :marwell}
 {:meal :hot, :house :auden, :name "Jim Brown", :class :owslebury}
 {:meal :lunchbox, :house :byron, :name "Kate Jones", :class :littleton}
 {:meal :hot, :house :blake, :name "Helen Pain", :class :alresford})
```
Using functional decomposition we have created a simple solution to the problem that is generic enough to be used again.
