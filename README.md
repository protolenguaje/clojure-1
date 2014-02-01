*Written in January 2014, with Clojure 1.5.1 and Leiningen 2.3.4 on Java 1.7.0_45.*  
_Made possible by Aphyr's excellent, Clojure from-the-ground-up tutorial and "The joy of Clojure" book._

> I write this as I'm going through the book myself, so bear with me :) 
Open PRs please.

To set up your Clojure environment I suggest you use an excellent automation tool (dependency mgr, builder, test runner, packager, all-in-one) [Leiningen](http://leiningen.org/). 

> Notice there's no "installer" in the list of features. That's because Clojure is just another _dependency_ of "our project".

![lein](http://leiningen.org/img/leiningen.jpg)

For your convenience, here's a dead simple, step-by-step environment setup for Ubuntu ([other OS, sir?](http://leiningen.org/#install)):

```sh
cd /usr/bin
sudo curl -O https://raw.github.com/technomancy/leiningen/stable/bin/lein
sudo chmod a+x lein
 ```

Let me explain what we just did. 
First we fetched `lein` script with `curl`° into `/usr/bin`°°.
We made sure it's executable with `chmod`.

°  Use `sudo apt-get install curl`, if you haven't installed it yet (doesn't come OOTB with Ubuntu).  
°° You can use any other dir, as long as you make sure it's in your PATH (check with `echo $PATH`), which makes possible to run it from anywhere.

Let's kick things off:

```sh
cd
lein new scratch
 ```

Here we used `lein new` to create a fresh Clojure project, based on the `scratch` template (_scratch_ - as in _scratch the surface_, I guess).
Lein, in turn, installed _rest of itself_ into `~/.lein/self-installs`.

In case you get stuck, visit [lein install instructions](http://leiningen.org/#install).  
If you want to know more, the official Leiningen tutorial can be found [right here](https://github.com/technomancy/leiningen/blob/master/doc/TUTORIAL.md).

Lovely! Now fire along:

```sh
cd scratch
lein repl
```

Boom! We have our own working Clojure environment.

![repl](https://github.com/mbonaci/clojure/raw/master/resources/repl.png)

Now when that's all sorted and you're eager to learn, let's get things started...

# Basics

Valid Clojure expressions consist of _numbers_, _symbols_, _keywords_, _booleans_, _characters_, _functions_, _function calls_, _macros_ (what?), _strings_, _literal maps_, _vectors_, and _sets_. 

Here's how a function looks like (`inc` is short for _increment_):

```clojure
user=> inc
<core$inc clojure.core$inc@d6206b5>
```
 
Numbers, strings and keywords evaluate to themselves.  

```clojure
;  skipping the 'user=>' prompt from now on
;  oh, and BTW, that was a comment, because it begins with a semicolon
;  so was that, a one-line comment
;; this one also, from that first semicolon to the end of the line. All the way here ->

(inc 2)      ;"increment 2"
3            ;this is REPL output
```

## Syntax

```clojure
(+ 1 (- 5 2) (+ 3 4))
11
```

> Uh, that looks silly, right?

This type of notation, called _prefix notation_ (inherited from _Lisp_), may look weird at the first glance.  

Let's back up a bit.  
All programming languages, in order to execute the source code, need to parse it first. The product of this code parsing is a so called _abstract syntax tree_ (_AST_).  

Let's see how that tree looks for the example at hand.  
In Java, the expression above would be written like this:

```java
1 + (5 - 2) + (3 + 4);   // parentheses left for clarity
// 1 + 3 + 7
// 11
```

... and the _AST_ would look like this:

![ast](https://github.com/mbonaci/clojure/raw/master/resources/ast.png)

Looks familiar?
By doing in-order (depth-first) traversal of this tree, you end up with Java syntax.  
By doing pre-order traversal, you get the Clojure syntax.  

> Now forget all this nonsense, cuz' it's really not important at all (unless you'll be writing your own compiler or something).

# Scalar data types

A scalar data type is the one that can only hold one value at a time (value of a number, symbol, keyword, string or character).

## Numeric types

The following examples of numeric expressions are trivial, so I suggest you try them out, in your REPL.

```clojure
(type 3)
java.lang.Long

Long/MAX_VALUE
9223372036854775807

(inc (bigint Long/MAX_VALUE))
9223372036854775808N

(type 5N)
clojure.lang.BigInt

(type (int 0))
java.lang.Integer

(type (short 0))
java.lang.Short

(type (byte 0))
java.lang.Byte

;; decimal, hexadecimal, octal, radix-32, and binary literals:
[127 0x7F 0177 32r3V 2r01111111]
[127 127 127 127 127]
;; radix supports up to base 36

(Short/MAX_VALUE)
32767

(Integer/MAX_VALUE)
2147483647

Byte/MAX_VALUE
127

(type 1.23)
java.lang.Double

(type (float 1.23))
java.lang.Float

366e3
366000.0
```



```clojure
(+ 1 2.0)
3.0

(class (+ 1 2.0))
java.lang.Double

(= 3 3.0)
false

(== 3 3.0)
true

(* 2 3 1/5)
6/5

(- 5 1 1 1)
2

(- 2)
-2

(* 4)
4

(/ 4)
1/4

(+)
0     ;neutral for addition

(*)
1     ;neutral for multiplication

(<= 1 2 3)
true

(<= 1 3 2)
false

(= 2 2 3)
false

(= 2 2 2)
true
```

To avoid having to round floating-point numbers (e.g. result of `2 / 3`), Clojure provides a rational number type `clojure.lang.Ratio`, thus maintaining absolute precision when dealing with floating point arithmetic:

```clojure
(type 1/3)
clojure.lang.Ratio

;; rational numbers are automatically simplified, if possible:
100/25
4

(rational? 2)   ;is rational or may be rational
true

(rational? 2.1)
false

(rationalize 2.1)
21/10

(numerator (/ 21 10))
21

(denominator (/ 21 10))
10
```

> The calculation of rational math, though accurate, isn’t nearly as fast as with floats or doubles (due to overhead cost of e.g. finding the least common denominator).

**Truncation** 

Truncation is a process of limiting the accuracy of floating-point numbers, due to deficiencies in its representation. When a number is truncated, its precision is limited such that the maximum number of digits of accuracy is bound by the number of bits that can fit into the storage space allowed by its representation.

```clojure
(let [pi-constant 3.14159265358979323846264338327950288419716939937M] ;notice the M
  (println (class pi-constant))
  pi-constant)
java.math.BigDecimal
3.14159265358979323846264338327950288419716939937M

(let [pi-trunc 3.14159265358979323846264338327950288419716939937]
  (println (class pi-trunc))
  pi-trunc)
java.lang.Double
3.141592653589793
```

`M`, at the end of a floating-point number literal is used to tell Clojure to keep the number in its full precision.  
`N` is used for the same thing when dealing with longs.

> Clojure truncates floating point numbers by default

**Promotion**

Clojure is able to detect when overflow occurs. Then, it automatically promotes the value to a numerical representation that can accommodate larger values

```clojure
(def small 9)
(class small)
java.lang.Long

(class (+ small 90000000000000000000))
clojure.lang.BigInt

(class (+ small 9.0))
java.lang.Double

(+ Integer/MAX_VALUE Integer/MAX_VALUE)
4294967294

(class (+ Integer/MAX_VALUE Integer/MAX_VALUE))
java.lang.Long
```

## Strings and Characters

String is any sequence of characters enclosed in double quotes, including newlines:

```clojure
"this is a string
on two lines"
"this is a string\non two lines"  ;REPL output includes newline escape

(type "a")
java.lang.String

(str nil)    ;'nil' is Clojure's 'null'
""
```

> single quote, backtick and `quote` (though with slightly different properties) are used to include literals in a program without evaluating them

```clojure
(str 'cat)
"cat"

(str 'a')
"a'"

(str 1)
"1"

(str '1)
"1"

(str true)
"true"

(str '(1 2 3))
"(1 2 3)"

(str "meow " 3 " times")
"meow 3 times"

;characters are denoted by backslash
\a
\a

\u0042
\B

\\
\\
```

## Regular expressions

`#"..."` is Clojure’s way of writing a regular expression.  
Clojure and Java have very similar Regex syntax.

```clojure
(re-find #"cat" "mystic cat mouse")
"cat"

(re-find #"cat" "only dogs here")
nil
```

The parentheses, i.e. _capturing group_ means that the regular expression should capture that part of the match. We get back a _list_ containing the part of the string that matched the first parentheses, followed by the part that matched the second parentheses, etc:

```clojure
(re-matches #"(.+):(.+)" "mouse:treat")
["mouse:treat" "mouse" "treat"]

;; capturing group in the regex causes each returned item to be a vector:
(re-seq #"\w*(\w)" "one-two/three")
(["one" "e"] ["two" "o"] ["three" "e"])
```

Java's regex `Pattern` class has several methods that can be used directly, but only `split` is used regularly to split a string into an array of Strings:

```clojure
(seq (.split #"," "1,2,3,4"))  ;this is how you call Java methods
("1" "2" "3" "4")
```

> [More on Java interoperability](#java-interop)

The `re-seq` function returns a lazy sequence of all matches in a string:

```clojure
(re-seq #"\w+" "one-two/three")
("one" "two" "three")
```

> Java's regex engine includes a `Matcher` object which mutates in a non-thread-safe way as it walks through a string finding matches. This object is exposed in Clojure through the `re-matcher` function and can be used in combination with `re-groups` and `re-find`.
It's recommended that a direct usage of all of these three functions should be avoided.

## Booleans

**The only negative values** are `false` and `nil`, all others are `true`:

```clojure
(boolean nil)
false
(boolean 0)
true
(boolean "hi")
true
(boolean str)
true

(and true false true)
false
(and true true true)
true
```

... and returns the first _falsy_ value, or the last value if all are _truthy_:
```clojure
(and 1 2 3)
3
(and -1 nil 2)
nil
```

or returns the first _truthy_ value, or the last value if all are _falsy_:
```clojure
(or false 2 3)
2
(or false false nil)
nil
```

`not` inverses the _truthiness_ of the expression:
```clojure
(not nil)
true
```

**Symbols** can have either short or full names. The short name is used to refer to things locally, and the _fully qualified name_ is used to refer unambiguously to a symbol (from anywhere).
Symbol names are separated with a `/`. For instance, the symbol `str` is also present in a namespace called `clojure.core` and the corresponding full name is `clojure.core/str`
The main purpose of symbols is to refer to _things_, i.e. to point to other values. When evaluating a program, symbols are looked up and replaced by their corresponding values. 

> [more about symbols](#symbols)

```clojure
(= str clojure.core/str)
true

(name 'clojure.core/str)
"str"
```


# Keywords

Closely related to symbols and strings are keywords, which begin with a `:`.  
Keywords are like strings in that they are made up of text, but are specifically intended for use as labels or identifiers. These are not labels in the sense of symbols, keywords are not replaced by any other value, they are just names, by themselves:

```clojure
(type :cat)
clojure.lang.Keyword

(str :cat)
":cat"

(name :cat)
"cat"
```

**Using keywords as map keys**

> since keywords are self-evaluating and provide fast equality checks, they are almost always used as map keys

Another important property of keywords, when used as map keys, is that they can be used as functions, taking a map as an argument to perform value lookups:

```clojure
(def mouse-planet {:cats 180, :mice 9})   ;define a map
#'user/mouse-planet

(:cats mouse-planet)                      ;lookup by keyword
180

(println (/ (:cats mouse-planet)          ;much more useful example
            (:mice mouse-planet))
         "cats per capita")
20 cats per capita
```

**As enumerations**

Since their value doesn't change, convenient keyword use case is enumeration. E.g. `:mouse`, `:rat` and `:x-rat` provide a nice visual delineation (for mouse types) within a source code.

> there are other useful things we can do with keywords. We can use them **As multimethod dispatch values** and **As directives**, but we'll deal with that later

**Qualifying your keywords**

Keywords don't belong to any specific namespace, although it's a good practice to define them as if they do, because that way you provide a context:

```clojure
user=> :not-in-ns
:not-in-ns

user=> ::not-in-ns          ;fully qualified keyword
:user/not-in-ns

user=> (ns another)
nil

another=> :user/in-another  ;"fully qualified" keyword
:user/in-another

another=> :haunted/name     ;namespace doesn't have to exist
:haunted/name
```

> double colon is used to fully qualify a keyword by prepending the current namespace name to the keyword name

> equally named keywords are the same object in memory

# Collections

A collection is a group of values. It’s a container which provides some structure, some framework, for the things that it holds. We say that a collection contains elements, or members.

> when a collection is evaluated, each of its contained items is evaluated first

## Lists

Literal lists are written with parentheses: `(yankee hotel foxtrot)`.

When a list is evaluated, the first item of the list, `yankee`, will be resolved to a _function_, _macro_, or _special form_. 
If `yankee` is a _function_, the remaining items in the list will be evaluated in order, and the results will be passed to `yankee` as its parameters.
If `yankee` is a _macro_ or a _special form_, the remaining items in the list
aren’t necessarily evaluated, but are processed as defined by the _macro_ or _operator_.

> **special form** is a form with special syntax or special evaluation rules that are typically not implemented using the base Clojure forms. An example of a special form is
the `.` (dot) operator used for Java interoperability purposes.

```clojure
(cons 1 (2 3))

ClassCastException java.lang.Long cannot be cast to clojure.lang.IFn
;which basically means that the number 2 cannot be used as a function

(cons 1 '(2 3))
(1 2 3)
```

Remember, we quote lists (any everything else) with a `'` (or `quote`) to prevent them from being evaluated.  

```clojure
'(1 2 3)
(1 2 3)

(type '(1 2 3))
clojure.lang.PersistentList
```

There's also a **syntax-quote** (back-tick), which automatically qualifies all unqualified symbols in its argument:

```clojure
`map
clojure.core/map

`(map even? [1 2 3])
(clojure.core/map clojure.core/even? [1 2 3])
```

You can also construct a list using `list`:

```clojure
(list 1 2 3)
(1 2 3)
(= (list 1 2) (list 1 2))
true
```

You can modify a list by _conjoining_ an element onto it (notice it goes to the beginning of the list):

```clojure
(conj (list 1 2 3) 4)
(4 1 2 3)

(first (list 1 2 3))
1
(second (list 1 2 3))
2
(nth (list 1 2 3) 2)
3
```

Unlike some _Lisps_, the empty list in Clojure, `()`, isn't the same as `nil`.
Lists are well-suited for small collections, or collections which are read in linear order, but are slow when you want to get arbitrary elements from later in the list. 

> Calling `seq` on a list returns the list itself, but more often, calling `seq` on a collection returns a new `seq` object for navigating that collection.

## Vectors

For fast access to every element, we use a __vector__.
Vectors are surrounded by square brackets, just like lists are surrounded by parentheses. Because vectors aren’t evaluated like lists are, there is no need to quote the vector literal:

```clojure
[1 2 3]
[1 2 3]

(type [1 2 3])
clojure.lang.PersistentVector
```

> Vectors are similar to arrays, but are immutable and persistent

You can also create vectors with `vector`, or change other structures into vectors with `vec`:

```clojure
(vector 1 2 3)
[1 2 3]

(vec (list 1 2 3))
[1 2 3]
```

`conj` on a vector adds to the end, not the start, like with lists:

```clojure
(conj [1 2 3] 4)
[1 2 3 4]
```

Our friends `first`, `second`, and `nth` work here too; but unlike lists, `nth` is fast on vectors. That’s because internally, vectors are represented as a very broad tree of elements, where each part of the tree branches into 32 smaller trees. Even very large vectors are only a few layers deep, which means getting to elements only takes a few hops, which is why retrieval operation from the interior of a vector takes essentially a constant time.

The important difference, when compared to lists, is that vectors evaluate each item in order. No function or macro is performed on a vector itself.

`rest` and `next` both return _everything but the first element_. They differ only by what happens when there are no remaining elements:

```clojure
(rest [1 2 3 4])
(2 3 4)

(next [1 2 3 4])
(2 3 4)
```

`rest` returns logical `true`, `next` returns logical `false`. Each has their uses, but in almost every case they’re equivalent:

```clojure
(next [1])
nil

(rest [1])
()
```

`last`, surprisingly, returns the last element:

```clojure
(last [1 2 3])
3
```

`count`, shockingly, returns element count:

```clojure
(count [1 2 3 4])
4
```

You can use element's index to access it:

```clojure
([1 2 3] 1)
2
```

Vectors and lists containing the same elements are considered equal:

```clojure
(= [1 2] (list 1 2))
true
```

If you already have a vector but want to "pour" several values into it, use `into`:

```clojure
(let [m-vector [:a :b :c]]
  (into m-vector (range 10)))
[:a :b :c 0 1 2 3 4 5 6 7 8 9]

;;if you want to return a vector the first arg to 'into' must be a vector
;;the second argument must be anything that works with 'seq' function
```

### Primitive vectors

You can store primitive types inside of vectors using the `vector-of` function, which takes any of `:int`, `:long`, `:float`, `:double`, `:byte`, `:short`, `:boolean` and `:char` and returns an empty vector. This returned vector will act just like any other vector, except that, internally, it'll store its contents as primitives.  
The new vector will try to coerce any additions into its internal type when they are being added:

```clojure
(into (vector-of :int) [Math/PI 2 1.3])
[3 2 1]

(into (vector-of :char) [100 101 102])
[\d \e \f]

(into (vector-of :int) [1 2 609812734019519652839108477134])
IllegalArgumentException Value out of range for long: 609812734019519652839108477134
clojure.lang.RT.longCast (RT.java:1134)
```

### Large vectors

Vectors are particularly efficient at three things (relative to lists): 
 - adding or removing things from the right end of the collection
 - accessing or changing items in the interior of the collection by numeric index
 - walking in reverse order

```clojure
(def a-to-e (vec (map char (range 65 91))))
#'mbo/a-to-z

a-to-e
[\A \B \C \D \E]

(nth a-to-e 4)   ;like with a map
\E

(get a-to-e 4)   ;like with a map
\E

(a-to-e 4)       ;invoking a vector as a function
\E
```

![vector-lookup-options](https://github.com/mbonaci/clojure/raw/master/resources/vector-lookup-options.png)

Since vectors are indexed, they can be efficiently walked in either direction. The `seq` and `rseq` return sequences that do exactly that:

```clojure
(seq a-to-e)
(\A \B \C \D \E)

(rseq a-to-e)
(\E \D \C \B \A)
```

Any item in a vector can be "changed" using the `assoc` function (in a constant time). Clojure does this using _structural sharing_ between the old and the new vectors, hence avoiding having to copy all the elements.

```clojure
(assoc a-to-e 3 "former D")
[\A \B \C "former D" \E]
```

The `assoc` function works only on indices that either already exist or, as a special case, exactly one step past the end of a vector (returned vector becomes one item larger).

Function `replace` uses `assoc` internally:

```clojure
(replace {2 :a, 4 :b} [1 2 3 2 3 4])
[1 :a 3 :a 3 :b]
```

The functions `assoc-in`, `update-in` and `get-in` are used to work with nested structures of vectors and/or maps. These functions take a series of indices to pick items from each more deeply nested level:

```clojure
(def matrix
  [[1 2 3]
   [4 5 6]
   [7 8 9]])
#'mbo/matrix

(get-in matrix [1 2])   ;second row, third column
6

(assoc-in matrix [1 2] 'x)
[[1 2 3] [4 5 x] [7 8 9]]
```

`update-in` works the similar way, but instead of taking a value to "overwrite" an existing value, it takes a function to _apply_ to an existing value. It'll replace the value at the given coordinates with the return value of the function:

```clojure
(update-in matrix [1 2] * 100)
[[1 2 3] [4 5 600] [7 8 9]]

;; the coordinates refer to the value 6 and the function is * taking an argument 100
;; so the slot becomes the return value of (* 6 100)
```

Remember? `matrix` itself never changed:

```clojure
matrix
[[1 2 3] [4 5 6] [7 8 9]]
```

### Vectors as stacks

Clojure vectors uses `conj` as `push` and `pop` as `pop` to add and remove elements from the right side. Since vectors are immutable, `pop` returns a new vector with the rightmost element dropped, which is different from many mutable stack APIs, which generally return the dropped item. Consequently, `peek` becomes more important as the primary way to get an item from the top of the stack:

```clojure
(peek m-stack)
3

(pop m-stack)
[1 2]

(conj m-stack 4)
[1 2 3 4]

(+ (peek m-stack) (peek (pop m-stack)))
5
```

> If you're using a collection (any implementor of `clojure.lang.IPersistentStack`) as a stack, it's idiomatic to use `conj` instead of `assoc`, `peek` instead of `last` and `pop` instead of `dissoc`. That avoids unnecessary confusion about how the collection is being used
  
> Lists also implement `IPersistentStack`, but the functions there operate on the left side, i.e. beginning of the list

### Using vectors instead of reverse

When processing a list, it's pretty common to want to produce a new list in the same order. But if all you have is list, then you're left with backward list that needs to be reversed:

```clojure
(defn m-map1 [f coll]
  (loop [coll coll, acc nil]
    (if (empty? coll)
      (reverse acc)
      (recur (next coll) (cons (f (first coll)) acc)))))
#'mbo/m-map1

(m-map1 - (range 5))
(0 -1 -2 -3 -4)
```

After the entire list has been walked once to produce the desired values, `reverse` walks it again to get them in the right order, which is inefficient and non-idiomatic.
One way to avoid `reverse` is to use a vector as the accumulator, instead of a list:

```clojure
(defn m-map2 [f coll]
  (loop [coll coll, acc []]
    (if (empty? coll)
      acc
      (recur (next coll) (conj acc (f (first coll)))))))
#'mbo/m-map2

(m-map2 - (range 5))
[0 -1 -2 -3 -4]
```

### Subvectors

Provide a fast way to take a slice of an existing vector based on start and end indices.
To produce a subvector we use the `subvec` function:

```clojure
(subvec a-to-e 2 4)
[\C \D]

;;first index is inclusive, but the second is exclusive
```

> There's a special logic for taking a `subvec` of a `subvec`, in which case the newest subvector keeps a reference to the original vector, not the intermediate subvector, which keeps both the creation and use of sub-subvectors fast and efficient.

### Vectors as MapEntries

Clojure's hash map, just like hash tables and dictionaries in many other languages, has a mechanism to iterate through the entire collection. Clojure's solution for this iterator is, you guessed it - `seq`.  
Each item in this seq needs to include both the key and the value, hence they are wrapped in a `MapEntry`. When printed, each entry looks like a vector:

```clojure
(first {:width 10 :height 20 :depth 15})
[:depth 15]

;;not only does it look like a vector, it really is one:
(vector? (first {:width 10 :height 20 :depth 15}))
true

;;which means you can use all the regular vector functions on it
;;it even supports destructuring*
```

[destructuring?](#destructuring)

```clojure
;;e.g. the following locals, 'dimension' and 'amount', will take on the value of each
;;key/value pair in turn:
(doseq [[dimension amount] {:width 10, :height 20, :depth 15}]
  (println (str (name dimension) ":") amount " inches"))
depth: 15  inches
width: 10  inches
height: 20  inches
nil

(doc doseq)
-------------------------
clojure.core/doseq
([seq-exprs & body])
Macro
  Repeatedly executes body (presumably for side-effects) with
  bindings and filtering as provided by "for".  Does not retain
  the head of the sequence. Returns nil.
```

A `MapEntry` is its own type and has two functions for retrieving its contents, `key` and `val`, which do exactly the same thing as `(nth m-map 0)` and `(nth m-map 1)`.

### What vectors aren't?

Vectors are versatile, but there are some useful patterns where they might seem like a good fit, but in fact the aren't.

**Vectors aren't sparse**

If you have a vector of length _n_, the only position where you can insert a value is at index _n_, appending to the far right end. You can't skip some indices and insert at a higher index number (use hash map or sorted map).  
Although you can replace values within a vector, you can't insert or delete items such that indices are adjusted (use finger trees for that).

**Vectors aren't queues**

Use a `PersistentQueue` for that.

**Vectors aren't sets**

If you want to find out whether a vector contains a particular value, you might be tempted to use the `contains?` function, but that function checks whether a specific _key_, not _value_ is in a collection, which is really not useful for a vector.

> Vectors are probably the most frequently used collection type in Clojure. They are used as literals for argument lists and `let` bindings, for holding large amounts of application data and as stacks and map entries
  
> In almost all contexts, you can consider vectors, lists, and other sequences as interchangeable. They only differ in their performance characteristics, and in a few data-structure-specific operations.

## Sets

Sets are surrounded by `#{...}`. In general, the order of sets can shift at any time. If you want a particular order, you can ask for it as a list or a vector:

```clojure
#{1 2 3}
#{1 2 3}
```

To ask for elements in a sorted order:

```clojure
(sort #{2 4 1})
(1 2 4)
```

`conj` adds an element to the set:

```clojure
(conj #{:a :b :c} :d)
#{:a :c :b :d}
```

`disj` removes an element:

```clojure
(disj #{1 2} 2)
#{1}
```

`contains?` checks for existence of an element:

```clojure
(contains? #{1 2 3} 3)
true
```

Like vectors, you can use the set itself as a verb. Unlike `contains?`, this expression returns the element itself (if it was present), or `nil`:

```clojure
(#{1 2 3} 3)
3
(#{1 2 3} 4)
nil
```

You can make a set out of any other collection with `set`:

```clojure
(set [2 5 1])
#{1 2 5}
```

## Maps

Maps are represented as braces `{...}` filled by alternating keys and values.  

In the following map, the three keys are `:name`, `:color`, and `:weight`, and their values are `"mittens"`, `"black"`, and `9`, respectively. We can look up the corresponding value for a key with `get`:

```clojure
{:name "luka", :weight 3, :color "white"}
{:weight 3, :name "luka", :color "white"}

(get {"cat" "meow", "dog" "woof"} "cat")
"meow"
```

`get` can also take a default value to return instead of `nil`, if the key doesn’t exist in that map:

```clojure
(get {:glinda :god} :wicked :not-here)
:not-here
```

We can use maps as verbs, directly:

```clojure
({"a" 12, "b" 24} "b")
24
```

Keywords can also be used as verbs, which look themselves up:

```clojure
(:raccoon {:weasel "queen", :raccoon "king"})
"king"
```

`assoc` adds an element to a map: 

```clojure
(assoc {:bolts 1088} :camshafts 3)
{:camshafts 3, :bolts 1088}
```

`assoc` adds keys if they aren’t present, and replaces values if they’re already there. If you associate a value onto `nil`, it creates a new map

```clojure
(assoc nil 5 2)
{5 2}
```

Combine maps together using `merge`. It yields a map containing all the elements of all given maps, preferring the values from later ones:

```clojure
(merge {:a 1, :b 2} {:b 3, :c 4})
{:c 4, :a 1, :b 3}
```

Remove map element with `dissoc`:

```clojure
(dissoc {:a 1, :b 2, :c 4} :c)
{:a 1, :b 2}
```

Like vectors, any item in a map literal is evaluated before the result is stored in the map. Unlike vectors, the order in which they are evaluated isn't guaranteed.

## Persistence

> Persistent collection is immutable, in-memory (not related to disk persistence) collection that allows you to preserve historical versions of its state.

Since arrays are mutable, any changes happen in-place:

```clojure
(def ds (into-array [:frane :luka :glupaca]))  ;mutable
#'mbo/ds

;; btw Glupaca is the name of our cat (which means something like "stupid women")

(seq ds)     ;only for nice REPL printout
(:frane :luka :glupaca)

(aset ds 2 :suzi)   ;replace third element of the array
:suzi

(seq ds)
(:frane :luka :suzi)
```

Using one of Clojure's persistent data structures shows the difference:

```clojure
mbo=> (def ds [:frane :luka :suzi])  ;init persistent collection
#'mbo/ds

ds                    ;print ds to REPL
[:frane :luka :suzi]

(def ds8 (replace {:suzi :glupaca} ds))   ;replace third element and bind the
#'mbo/ds8                                 ;whole collection to ds8

ds                    ;ds did not change
[:frane :luka :suzi]

ds8                   ;the newly created collection
[:frane :luka :glupaca]
```

# Symbols

Closest thing to a variable in Clojure. Primarily used to provide a name for a given value, i.e. to refer to function parameters, local variables, globals, and Java classes.

Unlike keywords, symbols are not unique based solely on their names:

```clojure
(identical? 'node 'node)
false

;; identical? returns true only if symbols are the same object
(let [x 'node y x] (identical? x y))
true
```

Each `node` symbol is a discrete object that only happens to share a name. Though name is the basis for symbol equality:

```clojure
(= 'node 'node)
true

(name 'node)
"node"
```

We can define a meaning for a symbol within a specific expression, using `let`.
The `let` expression first **takes a vector of bindings**: alternating symbols and values that those symbols are bound to within the remainder of the expression.  

“Let the symbol `cats` be `5`, and construct a string composed of `"I have "`, `cats`, and `" cats"`:

```clojure
(let [mice 5] (str "I have " mice " mice."))
"I have 5 mice."
```

Let bindings, also called **locals**, apply only within the `let` expression itself. They also override any existing definitions for symbols at that point in the program. For example, we can redefine addition to mean subtraction, for the duration of a `let`:

```clojure
(let [+ -] (+ 2 3))
-1
```

That definition does not apply outside the `let`:

```clojure
(+ 2 3)
5
```

We can also provide multiple bindings. Since Clojure doesn’t care about spacing, alignment, or newlines, we’ll write this on multiple lines, for clarity:

```clojure
(let [person "joseph"
       num-cats 186]
   (str person " has " num-cats " cats!"))  ;the body
"joseph has 186 cats!"
```

When multiple bindings are given, they are evaluated in order. Later bindings can use previous bindings:

```clojure
(let [cats 3
       legs (* 4 cats)]
 (str legs " legs all together"))
"12 legs all together"
```

> a symbol whose name is prefixed with a namespace, followed by a slash, is called **fully qualified symbol**:

```clojure
clojure.core/map
#<core$map clojure.core$map@2a0406c4>

clojure.set/union
#<set$union clojure.set$union@1be2bcc8>
```

The body is sometimes described as an _implicit do_ (see [blocks bellow](#blocks)) because it follows the same rules: you may include any number of expressions and all will be evaluated, but only the value of the last one is returned.

Because they’re immutable, _locals_ can’t be used to accumulate results. Instead,
you'd use a high level function or loop/recur form.

To summarize, `let` defines the meaning of symbols within an expression. When Clojure evaluates a `let`, it replaces all occurrences of those symbols in the rest of the `let` expression with their corresponding values, then evaluates the rest of the expression.

**Metadata**

Clojure allows the attachment of metadata to various objects, including symbols. `with-meta` function takes an object and a map and returns another object of the same type with the metadata attached:

```clojure
(let [x (with-meta 'node {:js true})    ;attach :js to 'node and assign to x
      y (with-meta 'node {:js false})]
  [(= x y)            ;true because they both hold the same symbol, 'node
   (identical? x y)   ;false because they are different instances
   (meta x)
   (meta y)])
[true false {:js true} {:js false}]
```

**Symbols and namespaces**

Like keywords, symbols don't belong to any specific namespace:

```clojure
user=> (ns what-where)
nil

what-where=> (def one-simbol 'where-is-it)
#'what-where/one-simbol

what-where=> one-simbol
where-is-it

what-where=> (resolve 'one-simbol)
#'what-where/one-simbol  ;looks like namespace-qualified symbol
                         ;but it's just a characteristic of symbol evaluation

what-where=> `one-symbol   ;back tick
what-where/one-symbol
```

# Functions

Functions are a first-class type in Clojure. They can be used the same as any value (stored in Vars, held in collections and passed as arguments and returned as a result of other functions).

> prefix notation allows any number of arguments (infix only two) and completely eliminates the problem of operator precedence.

```clojure
let( [x] (+ x 1))
```

We can’t actually evaluate this program, because there’s no value for `x` yet. It could be `1`, or `4`, or `1453`. We say that `x` is _unbound_, because it has no binding to a particular value. This is the nature of the __function__: an expression with unbound symbols.

Function definition:

```clojure
(fn [x] (+ x 1))
#<user$eval1487$fn__1488 user$eval1487$fn__1488@6b7d28db>
```

Named function definition:

```clojure
(let [twice (fn [x] (* 2 x))]
   (+ (twice 1)
   (twice 3)))
8
```

`let` bindings describe a similar relationship, but with a specific set of values for those arguments. `let` is evaluated immediately, whereas `fn` is evaluated later, when bindings are provided.

## Vars

Once a `let` is defined, there’s no way to change it. If we want to redefine symbols for everyone, even code that we didn’t write, we need a new construct, a mutable variable.

```clojure
(def cats 5)
#'user/cats

(type #'user/cats)
clojure.lang.Var

cats
5
```

`def` defines a type of value we haven’t seen before: a _Var_. _Vars_, like symbols, are references to other values. When evaluated, a _Var_ is replaced by its corresponding value.  

`def` also binds the symbol `cats` (and its globally qualified equivalent `user/cats`) to that _Var_.  

The symbol `inc` points to the _Var_ `#'inc`, which in turn points to the function `#<core$inc clojure.core$inc@16bc0b3c>`.  

We can see the intermediate Var with `resolve`:

```clojure
'inc     ;symbol
inc

(resolve 'inc)
#'clojure.core/inc    ;variable

(eval 'inc)
#<core$inc clojure.core$inc@d6206b5>  ;value
```

Why those two levels of indirection? Unlike with symbol, we can change the meaning of a Var for everyone, globally, at any time.  

Vars don't require a value. Instead we can just declare them and, by doing so, defer the binding of value.

```clojure
(def y)
#'user/y

;; if we try to use it:
y
java.lang.IllegalStateException: Var user/y is unbound
```

## Named functions

```clojure
(def half 
  (fn [number] (/ number 2)))
#'user/half

(half 8)
4
```

Creating a function and binding it to a variable is so common that it has its own form: `defn`, which is a _macro_ that is short for `def fn`:

```clojure
(defn half [number] (/ number 2))
#'user/half

(half 8)
4
```

Functions don’t have to take an argument. We’ve seen functions which take zero arguments, like `(+)`:

```clojure
(defn half [] 1/2)
#'user/half
```

But if we try to use our earlier form with one argument, Clojure complains that the *arity* (the number of arguments to a function) is incorrect:

```clojure
(half 8)

ArityException Wrong number of args (1) passed to: user$half
clojure.lang.AFn.throwArity (AFn.java:437)
```

To handle multiple arities, functions have an alternate form, instead of an argument vector and a body, one provides a series of lists, each of which starts with an argument vector, followed by the body:

```clojure
(defn half
   ([] 1/2)
   ([x] (/ x 2)))
#'user/half

(half)
1/2

(half 8)
4
```

Multiple arguments work just like you expect. Just specify an argument vector of two, or three, or however many arguments the function takes.  
Some functions can take any number of arguments. For that, Clojure provides `&`, which slurps up all remaining arguments as a list:

```clojure
(defn vargs [x y & more-args]
   {:x x
    :y y
    :more more-args})
#'user/vargs

(vargs 1 2)
{:x 1, :y 2, :more nil}

(vargs 1 2 3 4 5)
{:x 1, :y 2, :more (3 4 5)}
```

`x` and `y` are mandatory, though there don’t have to be any remaining arguments.  

To keep track of what arguments a function takes, why the function exists, and what it does, we usually include a **docstring**. Docstrings help fill in the missing context around functions, to explain their assumptions, context, and purpose to the world:

```clojure
(defn launch
   "Launches a spacecraft into the given orbit by initiating a
    controlled on-axis burn. Does not automatically stage, but
    does vector thrust, if the craft supports it."
   [craft target-orbit]
   "OK, we don't know how to control spacecraft yet.")
#'user/launch
```

Docstrings are used to automatically generate documentation for a Clojure programs, but you can also access them from the **REPL** (The `user=>` prompt refers to the top-level namespace of the default REPL):

```clojure
(doc launch)
-------------------------
user/launch
([craft target-orbit])
  Launches a spacecraft into the given orbit by initiating a
   controlled on-axis burn. Does not automatically stage, but
   does vector thrust, if the craft supports it.
nil
```

`doc` tells us the full name of the function, the arguments it accepts, and its docstring. This information comes from the `launch` Var’s metadata, and is saved there by `defn`. We can inspect metadata directly with the `meta` function:

```clojure
(meta #'launch)
{:arglists ([craft target-orbit]), :ns #<Namespace user>, :name launch, :column 1, :doc "Launches a spacecraft into the given orbit by initiating a\n   controlled on-axis burn. Does not automatically stage, but\n   does vector thrust, if the craft supports it.", :line 1, :file "/tmp/form-init523009510157887861.clj"}
```

There’s some other juicy information in there, like the file the function was defined in and which line and column it started at, but that’s not particularly useful since we’re in the _REPL_, not a file. However, this does hint at a way to answer our motivating question: how does the `type` function work?

## Blocks

When you have a series or block of expressions that need to be treated as one, use `do`. All the expressions will be evaluated, but only the last one will be returned:

```clojure
(do
   6
   (+ 5 4)
   3)
3
```

The expressions `6` and `(+ 5 4)` are perfectly legal. The addition in `(+ 5 4)` is even done, but the value is thrown away, only the final expression `3` is returned

## How does `type` work?

`type`, like all functions, is a kind of object with its own unique type:

```clojure
type
#<core$type clojure.core$type@2761df2a>

(type type)
clojure.core$type
```

This tells us that `type` is a particular instance, at memory address `39bda9b9`, of the type `clojure.core$type`. 
`clojure.core` is a namespace which defines the fundamentals of the Clojure language, and `$type` tells us that it’s named type in that namespace. None of this is particularly helpful, though. Maybe we can find out more about the `clojure.core$type` by asking what its supertypes are:

```clojure
(supers (type type))
#{java.io.Serializable java.lang.Runnable clojure.lang.AFunction clojure.lang.IMeta clojure.lang.AFn java.lang.Object clojure.lang.IObj java.util.Comparator clojure.lang.Fn java.util.concurrent.Callable clojure.lang.IFn}
```

This is a set of all the types that include `type`. We say that `type` is an instance of `clojure.lang.AFunction`, or that it implements or extends `java.util.concurrent.Callable`, and so on. Since it’s a member of `clojure.lang.IMeta` it has metadata, and since it’s a member of `clojure.lang.AFn`, it’s a function. Just to double check, let’s confirm that `type` is indeed a function:

```clojure
(fn? type)
true
```

`type` can take a single argument, which it calls `x`. If it has `:type` metadata, that’s what it returns. Otherwise, it returns the class of `x`. Let’s take a deeper look at `type`’s metadata for more clues:

```clojure
(doc type)
-------------------------
clojure.core/type
([x])
  Returns the :type metadata of x, or its Class if none
nil
```

This function was first added to Clojure in version `1.0`, and is defined in the file `clojure/core.clj`, on line `3109`:

```clojure
(meta #'type)
{:ns #<Namespace clojure.core>, :name type, :arglists ([x]), :column 1, :added "1.0", :static true, :doc "Returns the :type metadata of x, or its Class if none", :line 3109, :file "clojure/core.clj"}
```

We could go dig up the Clojure source code and read its definition there, or we could ask Clojure to do it for us. Aha! Here, at last, is how `type` works. It’s a function which takes a single argument `x`, and returns either `:type` from its metadata, or `(class x)`.

```clojure
(source type)
(defn type 
  "Returns the :type metadata of x, or its Class if none"
  {:added "1.0"
   :static true}
  [x]
  (or (get (meta x) :type) (class x)))
nil
```

# Sequences

> a **sequential** collection is one that holds a series of values without reordering them

> a **sequence** is a sequential collection that represents a series of values that may or may not exist yet (may have concrete values, may be lazy or empty). Few composite types are actually _sequences_, though several such as vectors are _sequential_. All an object needs to do to be a sequence is to support two core functions, `first` and `rest`

> a `seq` is a simple API for navigating collections which consists of two functions, `first` and `rest`

If two sequentials have the same values in the same order, `=` will return `true` for them, even if their concrete types are different:

```clojure
(= [1 2 3] '(1 2 3))
true
```

Conversely, even if two collections have the same values in the same order, if one is a sequential collection and the other isn't, `=` will return `false`:

```clojure
(= [1 2 3] #{1 2 3})
false
```

If the collection is empty, `seq` always returns `nil` and never an empty sequence (that goes for all other functions that return a `seq`, e.g. `next`).

> Clojure classifies each composite data type into 3 partitions: _sequentials_, _maps_ and _sets_. Everything that implements `java.util.List` is included in the sequential partition. Generally things that fall into other two partitions include set or map in their name

Every collection type provides at least one kind of `seq` object for walking through its elements

## Recursion

`cons`, makes a list beginning with the first argument, followed by all the elements in the second argument:

```clojure
(cons 1 [2 3 4])
(1 2 3 4)
```

Problem of incrementing all elements of a vector:

```clojure
(defn inc-first [nums]
   (if (first nums)
     ; If there's a first num, build a new list with cons
     (cons (inc (first nums))
           (rest nums))
     ; If there's no first num, return an empty list
     (list)))
#'user/inc-first

(inc-first [])
()

(inc-first [1 2 3])
(2 2 3)
```

What if we called our function on `rest`?

```clojure
(defn inc-all [nums]
   (if (first nums)
     (cons (inc (first nums))
           (inc-all (rest nums)))
     (list)))
#'user/inc-all

(inc-all [1 2 3 4])
(2 3 4 5)
```

This technique is called _recursion_, and it is a fundamental principle in working with collections, sequences, trees, graphs or any problem which has small parts linked together. There are two key elements in a recursive program:
 - Some part of the problem which has a known solution
 - A relationship which connects one part of the problem to the next

Incrementing the elements of an empty list returns the empty list. This is our **base case**, the ground to build on. Our **inductive case**, also called the _recurrence relation_, is how we broke the problem up into incrementing the first number in the sequence, and incrementing all the numbers in the rest of the sequence. The `if` expression bound these two cases together into a single function, a function defined in terms of itself.
Let’s parameterize our `inc-all` function to use any transformation of its elements:

```clojure
(defn transform-all [f xs]
   (if (first xs)
     (cons (f (first xs))
           (transform-all f (rest xs)))
     (list)))
#'user/transform-all

(transform-all inc [1 2 3 4])
(2 3 4 5)
```

## Loop

When using recursion, you sometimes want to loop back not to the top of the function, but to somewhere inside the function body.  
The `loop` acts exactly like `let` but provides a target for `recur` to jump to:

```clojure
(defn sum-down-from [initial-x]
   (loop [sum 0, x initial-x]
     (if (pos? x)
       (recur (+ sum x) (dec x))
       sum)))
#'user/sum-down-from
```

Upon entering the `loop`, the locals `sum` and `x` are initialized (like in `let`). A `recur` always loops back to the closest enclosing `loop` or `fn`. The `loop` locals are rebound to the values given in `recur`.  
`recur` works only from the tail position.  

`keyword` transforms a string to keyword:

```clojure
(transform-all keyword ["aa" "bb" "cc"])
(:aa :bb :cc)
```

To wrap every element in a list:

```clojure
(transform-all list ["aa" "bb" "cc"])
(("aa") ("bb") ("cc"))
```

We basically implemented `map` function:

```clojure
(map inc [1 2 3 4])
(2 3 4 5)
```

The function `map` relates one sequence to another. The type `map` relates keys to values. There is a deep symmetry between the two: maps are usually sparse, and the relationships between keys and values may be arbitrarily complex. The `map` function, on the other hand, usually expresses the same type of relationship, applied to a series of elements in a fixed order.

Clojure has a _special form_ called `recur` that's specifically used for tail recursion:

```clojure
(defn print-down-from [x]
   (when (pos? x)  ;return when x is no longer positive
     (println x)
     (recur (dec x))))
#'user/print-down-from

(print-down-from 5)
5
4
3
2
1
nil
```

> `when` is same as `if`, except it doesn't have the `else` part and it requires an implicit `do` in order to perform side-effects

This is nearly identical to how you’d structure a while loop in an imperative language.One significant difference is that the value of `x` isn’t decremented somewhere in the body of the loop. Instead, a new value is calculated as a parameter to `recur`, which immediately does two things: rebinds `x` to the new value and returns control to the top of `print-down-from`.  
If the function has multiple arguments, the `recur` call must as well, just as if you were calling the function by name instead of using the `recur` special form. And just as with a function call, the expressions in the `recur` are evaluated in order first and only then bound to the function arguments simultaneously.

## Building Sequences

We can use recursion to expand a single value into a sequence of values, each related by some function. For instance (`pos?` returns `true` if `num` is greater than zero, else `false`):

```clojure
(defn expand [f x count]
   (if (pos? count)
     (cons x (expand f (f x) (dec count)))))
#'user/expand

(expand inc 0 10)
(0 1 2 3 4 5 6 7 8 9)
```

Our base case is `x` itself, followed by the sequence beginning with `(f x)`. That sequence in turn expands to `(f (f x))`, and then to `(f (f (f x)))`, and so on. Each time we call `expand`, we count down by one using `dec`. Once the `count` is zero, `if` returns `nil`, and evaluation stops.

Clojure has a more general form of this function, called `iterate`:

```clojure
(take 10 (iterate inc 0))
(0 1 2 3 4 5 6 7 8 9)
```

Since this sequence is infinitely long, we’re using `take` to select only the first 10 elements. We can construct more complex sequences by using more complex functions:

```clojure
(take 10 (iterate (fn [x] (if (odd? x) (+ 1 x) (/ x 2))) 10))
(10 5 6 3 4 2 1 2 1 2)
```

`repeat` constructs a sequence where every element is the same:

```clojure
(take 10 (repeat "a"))
("a" "a" "a" "a" "a" "a" "a" "a" "a" "a")

(repeat 5 "b")
("b" "b" "b" "b" "b")
```

`repeatedly` simply calls a function `(f)` to generate an infinite sequence of values, over and over again, without any relationship between elements. For an infinite sequence of random numbers:

```clojure
(rand)
0.6934524557647231

(rand)
0.1355414232605504

(take 3 (repeatedly rand))
(0.18806021884865332 0.5231673860825672 0.38244349544358525)
```

`range` generates a sequence of numbers between two points. 
`(range n)` gives `n` successive integers starting at 0. 
`(range n m)` returns integers from `n` to `m-1`. 
`(range n m step)` returns integers from `n` to `m`, separated by `step`:

```clojure
(range 5)
(0 1 2 3 4)

(range 5 8)
(5 6 7)

(range 5 25 5)
(5 10 15 20)
```

To extend a sequence by repeating it forever, use `cycle`:

```clojure
(take 6 (cycle (range 5 50 5)))
(5 10 15 20 25 30)
```

## Transforming Sequences

`map` applies a function to each element, but it has a few more tricks up its sleeve:

```clojure
(map (fn [n vehicle] (str "I've got " n " " vehicle "s"))
   [0 200 9]
   ["car" "train" "kiteboard"])
("I've got 0 cars" "I've got 200 trains" "I've got 9 kiteboards")
```

If given multiple sequences, `map` calls its function with one element from each sequence in turn. So the first value will be `(f 0 "car")`, the second `(f 200 "train")`, and so on. Like a zipper, map folds together corresponding elements from multiple collections. To sum three vectors, column-wise:

```clojure
(map + [1 2 3]
        [4 5 6]
        [3 2 1])
(8 9 10)
```

If one sequence is bigger than another, `map` stops at the end of the smaller one. We can exploit this to combine finite and infinite sequences. For example, to number the elements in a vector:

```clojure
(map (fn [index element] (str index ". " element))
      (iterate inc 0)
      ["erlang" "scala" "haskell"])
("0. erlang" "1. scala" "2. haskell")
```

Transforming elements together with their indices is so common that Clojure has a special function for it: `map-indexed`:

```clojure
(map-indexed (fn [index element] (str index ". " element))
              ["erlang" "scala" "haskell"])
("0. erlang" "1. scala" "2. haskell")
```

You can also tack one sequence onto the end of another, like so:

```clojure
(concat [1 2 3] [:a :b :c] [4 5 6])
(1 2 3 :a :b :c 4 5 6)
```

Another way to combine two sequences is to riffle them together, using `interleave`:

```clojure
(interleave [:a :b :c] [1 2 3])
(:a 1 :b 2 :c 3)
```

And if you want to insert a specific element between each successive pair in a sequence, try `interpose`:

```clojure
(interpose :and [1 2 3 4])
(1 :and 2 :and 3 :and 4)
```

To reverse a sequence, use ... you guessed it, `reverse`:

```clojure
(reverse [1 2 3])
(3 2 1)

(reverse "woolf")
(\f \l \o \o \w)
```

Strings are sequences too! Each element of a string is a character, written `\f`. You can rejoin those characters into a string with `apply str`:


```clojure
(apply str (reverse "woolf"))
"floow"
```

…and break strings up into sequences of chars with `seq`:

```clojure
(seq "sato")
(\s \a \t \o)
```

To randomize the order of a sequence, use `shuffle`:

```clojure
(shuffle [1 2 3 4 5])
[4 3 5 1 2]

(apply str (shuffle (seq "abracadabra")))
"raradbabaac"
```

## Subsequences

`take` selects the first `n` elements
`drop` removes the first `n` elements
`take-last` and `drop-last` operate on the last `n` elements:

```clojure
(take 3 (range 10))
(0 1 2)

(drop 3 (range 10))
(3 4 5 6 7 8 9)

(take-last 3 (range 10))
(7 8 9)

(drop-last 3 (range 10))
(0 1 2 3 4 5 6)
```

`take-while` and `drop-while` work just like `take` and `drop`, but use a function to decide when to stop:

```clojure
(take-while pos? [3 2 1 0 -1 -2 10])
(3 2 1)
```

In general, one can cut a sequence in twain by using `split-at` with a particular index. There’s also `split-with`, which uses a function to decide when to cut:

```clojure
(split-at 4 (range 10))
[(0 1 2 3) (4 5 6 7 8 9)]

(split-with number? [1 2 3 :mark 4 5 6 :mark 7])
[(1 2 3) (:mark 4 5 6 :mark 7)]
```

Notice that because indexes start at zero, sequence functions tend to have predictable number of elements.`(split-at 4)` yields four elements in the first collection, and ensures the second collection begins at index four. `(range 10)` has ten elements, corresponding to the first ten indices in a sequence. `(range 3 5)` has two (as in `5 - 3 = 2`) elements. These choices simplify the definition of recursive functions as well.
We can select particular elements from a sequence by applying a function. To pull up all positive numbers in a list, use `filter`:

```clojure
(filter pos? [1 5 -4 -7 3 0])
(1 5 3)
```

`filter` looks at each element in turn, and includes it in the resulting sequence only if `(f element)` returns a _truthy_ value. Its complement is `remove`, which only includes those elements where `(f element)` is `false` or `nil`:

```clojure
(remove string? [1 "tur" :apple])
(1 :apple)
```

One can group a sequence into chunks using `partition`, `partition-all`, or `partition-by`. For instance, one might group alternating values into pairs like this:

```clojure
(partition 2 [:cats 5 :bats 27 :crocs 0])
((:cats 5) (:bats 27) (:crocs 0))
```

Separate a series of numbers into negative and positive subsequences:

```clojure
(partition-by neg? [1 2 3 2 1 -1 -2 -3 -2 -1 1 2])
((1 2 3 2 1) (-1 -2 -3 -2 -1) (1 2))
```

`partition-all n collection` may include partitions with fewer than `n` items at the end:

```clojure
(partition-all 3 [1 2 -5 3 2 1 -1 -2 -3 -2 -1 1 2])
((1 2 -5) (3 2 1) (-1 -2 -3) ( -2 -1 1) ( 2))
```

while `partition` may not:

```clojure
(partition 3 [1 2 -5 3 2 1 -1 -2 -3 -2 -1 1 2])
((1 2 -5) (3 2 1) (-1 -2 -3) (-2 -1 1))
```

### Collapsing subsequences

After transforming a sequence, we often want to collapse it in some way, in order to derive some smaller value. For instance, we might want the number of times each element appears in a sequence:

```clojure
(frequencies [:meow :mrrrow :meow :meow])
{:meow 3, :mrrrow 1}
```

To group elements by some function:

```clojure
(pprint (group-by :first [{:first "Li"    :last "Zhou"}
                           {:first "Sarah" :last "Lee"}
                           {:first "Sarah" :last "Dunn"}
                           {:first "Li"    :last "O'Toole"}]))
{"Li" [{:last "Zhou", :first "Li"} {:last "O'Toole", :first "Li"}],
 "Sarah" [{:last "Lee", :first "Sarah"} {:last "Dunn", :first "Sarah"}]}
```

Here we’ve taken a sequence of people with first and last names, and used the `:first` keyword (which can act as a function!) to look up those first names. `group-by` used that function to produce a map of first names to lists of people, kind of like an index.
In general, we want to combine elements together in some way, using a function. Where `map` treated each element independently, reducing a sequence requires that we bring some information along. The most general way to collapse a sequence is `reduce`:

```clojure
(doc reduce)
-------------------------
clojure.core/reduce
([f coll] [f val coll])
  f should be a function of 2 arguments. If val is not supplied,
  returns the result of applying f to the first 2 items in coll, then
  applying f to that result and the 3rd item, etc. If coll contains no
  items, f must accept no arguments as well, and reduce returns the
  result of calling f with no arguments.  If coll has only 1 item, it
  is returned and f is not called.  If val is supplied, returns the
  result of applying f to val and the first item in coll, then
  applying f to that result and the 2nd item, etc. If coll contains no
  items, returns val and f is not called.

(reduce + [1 2 3 4])
10

(reduce + 1 [1 2 3 4])
11
```

To see the reducing process in action, we can use `reductions`, which returns a sequence of all the intermediate states:

```clojure
(reductions + [1 2 3 4])
(1 3 6 10)
```

Oftentimes we include a default state to start with. For instance, we could start with an empty set, and add each element to it as we go along:

```clojure
(reduce conj #{} [:a :b :b :b :a :c])
#{:a :c :b}
```

Reducing elements into a collection has its own name: `into`. We can `conj [key value]` vectors into a map, for instance, or build up a list:

```clojure
(into {} [[:a 2] [:b 3]])
{:a 2, :b 3}

(into (list) [1 2 3 4])
(4 3 2 1)
```

Because **elements added to a list appear at the beginning**, not the end, this expression **reverses the sequence**. Vectors `conj` onto the end, so to emit the elements in order, using reduce, we might try:

```clojure
(reduce conj [] [1 2 3 4 5])
[1 2 3 4 5]
```

Remember?

```clojure
(conj [-1 0] [1 2 3 4 5])
[-1 0 [1 2 3 4 5]]               ; not [-1 0 1 2 3 4 5]
```

This looks like a `map` function. All that’s missing is some kind of transformation applied to each element:

```clojure
(defn my-map [f coll]
   (reduce (fn [output element]
             (conj output (f element)))
           []
           coll))
#'user/my-map

(my-map inc [1 2 3 4])
[2 3 4 5]
```

So `map` is just a special kind of `reduce`. What about, say, `take-while`?

```clojure
(defn my-take-while [f coll]
   (reduce (fn [out elem]
             (if (f elem)
               (conj out elem)
               (reduced out)))
           []
           coll))
```

We’re using a special function here, `reduced`, to indicate that we’ve completed our reduction early and can skip the rest of the sequence.

```clojure
(my-take-while pos? [2 1 0 -1 0 1 2])
[2 1]
```

Most of Clojure’s **sequence functions are lazy**. For instance, we can increment every number from zero to infinity:

```clojure
(def infseq (map inc (iterate inc 0)))
#'user/infseq

(realized? infseq)
false
```

That function returned immediately. Because it hasn’t done any work yet, we say the sequence is unrealized. It doesn’t increment any numbers at all until we ask for them:

```clojure
(take 10 infseq)
(1 2 3 4 5 6 7 8 9 10)

(realized? infseq)
true
```

**Lazy sequences also remember their contents, once evaluated, for faster access.**

Find the sum of the products of consecutive pairs of the first 1000 odd integers.

```clojure
(reduce +
         (take 1000
               (map (fn [pair] (* (first pair) (second pair)))
                    (partition 2 1
                      (filter odd?
                        (iterate inc 0))))))
1335333000
```

__Homework:__

1. Write a function to find out if a string is a palindrome.

```clojure
(defn palindrome? [word] 
  (== 0 (compare word (apply str (reverse word)))))
```

2. Find the number of `c`s in `“abracadabra”`.

```clojure
(defn occurs-count [c word]
         (get (frequencies (seq word)) c 0))
```

3. Write your own version of `filter`.

```clojure

```

# Java interop

Clojure is _symbiotic_ with its host, providing its rich and powerful features, while Java provides an object model, libraries and runtime support.

**Accessing static class members**

... is trivial:

```clojure
java.util.Locale/JAPAN
#<Locale ja_JP>
```

Idiomatic (best practice) Clojure prefers that you access static class members using a syntax like accessing a namespace-qualified Var:

```clojure
(Math/sqrt 9)  ; the same as (java.lang.Math/sqrt 9)
3.0
```

**Creating Java class instances**

```clojure
(new java.util.HashMap {"foo" 42 "bar" 8 "baz" "beep boop"})
{"baz" "beep boop", "foo" 42, "bar" 8}

; idiomatic:
(java.util.HashMap. {"foo" 42 "bar" 8 "baz" "beep boop"})  ;the dot = constructor call
{"baz" "beep boop", "foo" 42, "bar" 8}
```

**Accessing Java instance members with the dot operator**

To access instance properties, precede the property or method name with a dot:

```clojure
(.x (java.awt.Point. 10 20))  ;create a new Point and access its member x
10
```

To access instance methods, the dot form allows an additional argument to be passed to the method:

```clojure
(.divide (java.math.BigDecimal. "42") 2K)
21M

; what's this M for?
(type 2M)
java.math.BigDecimal
```

**Setting Java instance properties**

In the absence of mutator methods, in the form of `setX`, Java instance properties can be set using `set!` function:

```clojure
(let [origin (java.awt.Point. 0 0)]
  (set! (.x origin) 15)     ;(set! <instance_member_access> <value>)
  (str origin))
"java.awt.Point[x=15,y=0]"
```

**The `..` macro**

For now, think of a macro as a convenience function. We'll meet them [later](#macros).  
In Java, it's a common practice to chain together a sequence of method calls, e.g:

```java
new java.util.Date().toString().endsWith("2014")  ;java
```

... which is equivalent to this Clojure expression:

```clojure
(.endsWith (.toString (java.util.Date.)) "2014")
true
```

Well, anyone would agree that this is rather difficult to read.  
To remedy this, Clojure provides us with `..` macro:

```clojure
(.. (java.util.Date.) toString (endsWith "2014"))
true
```

**The `doto` macro**

In Java, it's also common to initialize a fresh instance by calling a set of mutators:

```java
java.util.HashMap props = new java.util.HashMap();  /* java */
props.put("HOME", "/home/myself");
props.put("SRC",  "src");
props.put("BIN",  "classes");
```

That's obviously overly verbose, but it can be streamlined using the `doto` macro:

```clojure
(doto (java.util.HashMap.)
  (.put "HOME" "/home/myself")
  (.put "SRC"  "src")
  (.put "BIN"  "classes"))
{"HOME" "/home/myself", "BIN" "classes", "SRC" "src"}
```

# Exceptions

Like Java, Clojure also provides `try`, `catch`, `finally` and `throw` forms:

```clojure
(throw (Exception. "Thrown"))

Exception Thrown  user/eval1201 (form-init8547084957850583270.clj:1)

(defn throw-catch [f]
  [(try
    (f)
    (catch ArithmeticException e "You did not? Not by zero! Noooooooooooo...")
    (catch Exception e (str "You blew it " (.getMessage e)))
    (finally (println "returning... ")))])
#'user/throw-catch

(throw-catch #(/ 10 5))
returning... 
[2]

(throw-catch #(/ 10 0))
returning... 
["You did not? Not by zero! Noooooooooooo..."]

(throw-catch #(throw (Exception. "dawg!")))
returning... 
["You blew it dawg!"]
```

> Clojure doesn't adhere to checked exception requirements, like Java does.

> When an exception is thrown in REPL, the result is stored in a Var named `*e`, which allows you to get more detail about the expression, such as the stack trace:

```clojure
(.printStackTrace *e)
```

# Namespaces

Provide a way to bundle related functions, macros and values.

**Creating namespaces using `ns` macro**

```clojure
(ns mbo.core.strings)
nil
mbo.core.strings=>    ;from now on, this is our REPL prompt
```

There's also a _Var_ `*ns*`, which holds the value of the current namespace.  
We know from before that any _Var_ created will be a member of the current namespace.

```clojure
mbo.core.strings=> (defn report-ns [] (str "The current namespace is " *ns*))
#'mbo.core.strings/report-ns

mbo.core.strings=> (report-ns)
"The current namespace is mbo.core.strings"

mbo.core.strings=> (defn sing [] (println "Marjane Marjane, Marjane Marjane, ..."))
#'mbo.core.strings/sing

mbo.core.strings=> sing    ;Clojure looks it up in the current namespace
#<strings$sing mbo.core.strings$sing@3987b05>

mbo.core.strings=> (ns mbo.core.compat)   ;create another namespace
nil

mbo.core.compat=> (report-ns)             ;try invoking function from another ns

mbo.core.compat=> CompilerException java.lang.RuntimeException: 
Unable to resolve symbol: report-ns in this context, 
compiling:(/tmp/form-init8547084957850583270.clj:1:1)

mbo.core.compat=> (mbo.core.strings/report-ns) ;fully qualified name works as expected
"The current namespace is mbo.core.compat"
```

> referring to a namespace symbol using fully qualified name will only work for namespaces created locally or those previously loaded. Read on, it'll become clear...

**Using `:require` directive to load other namespaces**

```clojure
mbo.core.compat=> (ns mbo.core.set
             #_=>   (:require clojure.set))
nil
mbo.core.set=>  ;changed to new ns

mbo.core.set=> (clojure.set/intersection #{1 2 3} #{2 3 4})
#{2 3}

mbo.core.set=> (intersection #{1 2 3} #{2 3 4})  ;invoke a clojure.set function directly

CompilerException java.lang.RuntimeException: Unable to resolve symbol: 
intersection in this context, compiling:(/tmp/form-init8547084957850583270.clj:1:1)
```

This construct indicates that we want the `clojure.set` namespace loaded, but we don't want the mappings of that namespace's symbols to `mbo.core.set` functions.

We can also use `:as` directive to create an additional alias to `clojure.set`:

```clojure
mbo.core.set=> (ns mbo.core.set-alias
                    #_=>   (:require [clojure.set :as s]))
nil

mbo.core.set-alias=> (s/intersection #{1 2 3} #{2 3 4})
#{2 3}
```

The qualified namespace form (e.g. `clojure.set`) looks the same as a call to a _static class method_. The difference is that a _namespace symbol_ can only be used as a qualifier, whereas a _class symbol_ can also be referenced independently:

```clojure
mbo.core.set-alias=> clojure.set

mbo.core.set-alias=> CompilerException java.lang.ClassNotFoundException: 
clojure.set, compiling:(/tmp/form-init8547084957850583270.clj:1:691) 

mbo.core.set-alias=> java.lang.Object
java.lang.Object
```

> That vagaries of namespace mappings from symbols to _Vars_, both qualified and unqualified, have the potential for confusion between _class names_ and _static methods_. In the beginning, that is. The differences will begin to feel natural as we progress (at least that's what _The joy of Clojure_ book promises :)
> One of the Clojure idioms is to use `my.Class` and `my.ns` for naming classes and namespaces, to help eliminate potential confusion.

**Loading and creating mappings with `:use`**

`:use`, unlike `:require`, maps Vars in another namespace to names in your own. That is typically used to avoid calling each function or macro with the qualifying namespace symbol:

```clojure
mbo.core.set-alias=> (ns mbo.test
                #_=>   (:use [clojure.string :only [capitalize]]))
nil

mbo.test=> (map capitalize ["one" "two"])
("One" "Two")
mbo.test=>
```

`:only` is used to indicate that only the listed functions should be mapped in the new namespace (good practice). The `:exclude` directive does the opposite.  
`:use`, besides creating mappings, implicitly invokes `:require`.

> the idiomatic strategy for avoiding conflicts is to use `:require` with `:as` to create a namespace alias

**Create mappings with `:refer`**

`:refer` is a directive that works almost exactly like `:use`, except that it only creates mappings for libraries that have already been loaded (by being previously defined, by being one of Clojure's core namespaces or by having been explicitly loaded using `:require`).

**Loading Java classes with `:import`**

```clojure
mbo.test=> (ns mbo.java
      #_=>   (:import [java.util HashMap]
      #_=>            [java.util.concurrent.atomic.AtomicLong]))
nil

mbo.java=> (HashMap. {"happy?" true})
{"happy?" true}
```

> Any classes in the `java.lang` package are implicitly imported when namespaces are created

**`nil` punning**

Since, in Clojure, everything except `nil` and `false` is `true`, empty collections evaluate to `true` in boolean context. We need a way to test whether a collection is empty or not.  

This is where _nil punning_ comes in:

```clojure
(seq [1 2 3])
(1 2 3)

(seq [])
nil
```

`seq` function returns a sequence view of a collection or `nil` if the collection is empty.

```clojure
(defn print-seq [s]
  (when (seq s)
    (prn (first s))     ;prn prints each object in a newline (to the output stream)
    (recur (rest s))))

(print-seq [1 2])
1
2
nil
```

The use of `seq` as a terminating condition is the idiomatic way of testing whether a sequence is empty

# Destructuring

Allows you to place a collection of names in a binding form where normally you'd put just a single name.  

> Destructuring is loosely related to _pattern matching_ (found in Haskell or [Scala](https://github.com/mbonaci/scala#case-classes-and-pattern-matching)), but much more limited in scope. For full-featured pattern matching in Clojure use [matchure](http://github.com/dcolthorp/matchure).

Perhaps the simplest form of destructuring is picking apart a sequential thing (e.g. a vector or a list), giving each item a name:

```clojure
(let [[fname mname lname] ["Frane" "Luka" "Bonaci"]]
  (str lname ", " fname " " mname))
"Bonaci, Frane Luka"

;; although this is syntactically correct, it isn't factually accurate
;; my month and a half old twins are named Frane and Luka :)
;; what can I say, my wife is a hero (that's what everybody's telling me these days)
;; hmmm, I wonder why?
```

This was a so called _positional destructuring_, which, as you might expect, doesn't work on maps and sets, because they are not logically aligned sequentially. But it does work on `java.util.regex.Matcher` and anything implementing `CharSequence` and `java.util.RandomAccess` interfaces.  
  
**Destructuring with a vector**
  
We can also use an ampersand in a destructuring vector to indicate that any remaining values of the input should be collected into a (possibly lazy) `seq`:

```clojure
(let [[a b c & more] (range 10)]
  (println "a b c are: " a b c)
  (println "the rest is: " more))
a b c are:  0 1 2
the rest is:  (3 4 5 6 7 8 9)
```

A useful feature of vector destructuring is `:as`, which is used to bind a local to the entire collection. It must be placed at the end, even after the `&` local (if it exists):

```clojure
(let [range-vec (vec (range 10))
  [a b c & more :as all] range-vec]
  (println "a b c are: " a b c)
  (println "the rest is " more)
  (println "all is: " all))
a b c are:  0 1 2
the rest is  (3 4 5 6 7 8 9)
all is:  [0 1 2 3 4 5 6 7 8 9]
```

> Notice the difference between `&` and `:as`. While `:all` produces a vector, `&` results with a `seq`.

**Destructuring with a map**

```clojure
(def full-name-map
  {:fname "Frane" :mname "Luka" :lname "Bonaci"})

(let [{fname :fname, mname :mname, lname :lname} full-name-map]
  (str lname ", " fname " " mname))
"Bonaci, Frane Luka"
```

Here, the `:keys` feature might come in handy:

```clojure
(let [{:keys [fname mname lname]} full-name-map]
  (str lname ", " fname " " mname))
"Bonaci, Frane Luka"
```

By using `:keys`, we're telling Clojure that the next form will be a vector of names that it should convert to keywords (e.g. `:fname`), in order to look up their values in the input map.  

We can do the same thing with `:strs` and `:syms`, except that the first one would look for string keys in the map (e.g. `"fname"`), and the latter one would indicate we're looking for symbol keys.  

Sometimes you'll want to get the keys that you didn't name individually by any of the previously described methods. In that case, we use `:as`, which works just like it does with vector:

```clojure
(let [{fname :fname, :as whole-name} full-name-map]
  whole-name)
{:mname "Luka", :lname "Bonaci", :fname "Frane"}
```

If the destructuring map looks up a key that's not in the source map, it's bound to `nil`, but it's possible to provide different defaults with `:or`:

```clojure
(let [{:keys [title fname mname lname], :or {title "Mr."}} full-name-map]
  (println title fname mname lname))
Mr. Frane Luka Bonaci
```

**Associative destructuring**

Using a map to define bindings isn't limited to maps, we can destructure a vector by providing a map declaring the local name as vector indices:

```clojure
(let [{first-thing 0, last-thing 3} [1 2 3 4]]
  [first-thing last-thing])
[1 4]
```

**Destructuring in function parameters**

All previous examples use `let` to do destructuring, but exactly the same features are available in function parameters. Each function parameter can destructure a map or a sequence:

```clojure
(defn print-last-name [{:keys [lname]}]
  (println lname))

(print-last-name full-name-map)
Bonaci
```

> When function arguments include an ampersand, that's not destructuring, but part of support for multiple function bodies, each with its own number of parameters.

**Destructuring versus accessor methods**

It's idiomatic in Clojure to build your application objects by composing maps and vectors, instead of using multiple objects with getters and setters, as Java does.
This makes destructuring natural and straightforward. So anytime you find yourself calling `nth` on the same collection multiple times, or looking up constants in a map, or using `first` or `next`, consider using destructuring instead.

# Composite data types











[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/mbonaci/clojure/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

