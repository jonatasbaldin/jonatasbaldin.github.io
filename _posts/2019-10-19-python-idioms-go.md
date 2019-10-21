---
layout: post
title: "Python Idioms in Go"
date: 2019-10-21 05:00:00
description: "Python has ton of sugar and Go tons of simplicity. In this article I'll go over some Go implementation of Python idioms."
image: "/img/prism.jpg"
---

In the development world, Python is my native language. It all started in 2015 with a little project called [itopy](https://github.com/jonatasbaldin/itopy). Python is my favorite way to translate thought into applications due to its minimal learning curve, simplicity and elegance.

But like in real life, learning a second language has tons of benefits. You become able to navigate within other countries, talk to strangers across borders, understand their culture and way of living. It opens your mind to think differently about things and to construct new knowledge on top of established ideas. The last point is very important: naturally, we learn new concepts by comparing them to the ones we already know. 

Studying a second programming language is no different. We compare their syntax, semantics, patterns and idioms. If something is the same, it just clicks! Otherwise, time is needed to build up knowledge. 

In 2019 I choose Golang for my next adventure. It is vastly used in infrastructure projects (which I love!), has a community full of Pythonistas and is becoming very very popular. My path to _get it_ was full of: "I can do X in Python, how do I do it in Go"? This article is a collection of those moments.

_PS1: I won't go in detail for the examples. A quick Googling will help you to understand them in depth._  
_PS2: I didn't include `import` statements in Go for the sake of brevity._

## Popping from a List
List are beautifully managed in Python:

```python
words = ["guess", "whos", "back"]
word = words.pop()

print(word)
# back
print(words)
# ['guess', 'whos']
```

Slices – Go's lists – are a bit more complicated, but the [SliceTricks guide](https://github.com/golang/go/wiki/SliceTricks) will help out.
```go
words := []string{"guess", "whos", "back"}
word, words := words[len(words)-1], words[:len(words)-1]

fmt.Println(word)
// back
fmt.Println(words)
// [guess whos]
```

---

## Pushing to a List
Pushing – or appending – is very similar:

🐍:
```python
words = ["mangoes", "peaches", "limes"]
words.append("the sweet life")

print(words)
# ['mangoes', 'peaches', 'limes', 'the sweet life']
```

❗️🐍:
```go
words := []string{"mangoes", "peaches", "limes"}
words = append(words, "the sweet life")

fmt.Println(words)
// [mangoes peaches limes the sweet life]
```

---

## List Comprehension
Personally, the most beautiful sugary diet of Python:

```python
numbers = [1,2,3,4]                        
double = [number * 2 for number in numbers]

print(double)
# [2, 4, 6, 8]
```

Unfortunately, in Go, more work is needed:
```go
numbers := []int{1, 2, 3, 4}
var double []int

for _, number := range numbers {
    double = append(double, number*2)
}

fmt.Println(double)
// [0 2 4 6]
```

---

## Dictionary Comprehension
Like a list comprehension, but iterating over values and keys in dictionary:
```python
numbers = {'a': 1, 'b': 2, 'c': 3}
double = {key: value * 2 for key, value in numbers.items()}

print(double)
# {'a': 2, 'b': 4, 'c': 6}
```

Again, with Golang, more work:
```golang
numbers := map[string]int{"a": 1, "b": 2, "c": 3}
double := make(map[string]int)

for key, value := range numbers {
    double[key] = value * 2
}

fmt.Println(double)
// map[a:2 b:4 c:6]
```

---

## Sets (and intersection)
Sets are like lists, but they have no duplicated items and some special properties. One of them is called _intersection_, which returns items showing up in both sets.

Python has a _set_ data structure with all these features:
```python
dna1 = {"loyalty", "royalty", "dna"}
dna2 = {"power", "poison", "pain", "dna"}

print(dna1 & dna2)
# {'dna'}
```

Sometimes Golang is _too_ simple. It has no concept of `set` nor intersection. However, the same can be accomplish using a `map` and two loops. Not as elegant as Python, that's for sure.
```golang
dna1 := make(map[string]bool)
dna1["loyalty"] = true
dna1["royalty"] = true
dna1["dna"] = true

dna2 := make(map[string]bool)
dna2["power"] = true
dna2["poision"] = true
dna2["pain"] = true
dna2["dna"] = true

intersection := make(map[string]bool)
for d1, _ := range dna1 {
    for d2, _ := range dna2 {
        if d1 == d2 {
            intersection[d1] = true
        }
    }
}

fmt.Println(intersection)
// map[dna:true]
```

---

## Enumerating
In Python, we use `enumerate()` to loop over a list with its index:

```python
words = ["i'll", "tell", "what", "I", "want"]

for index, word in enumerate(words):
    print(index, word)
```

In Go, this is the default behavior:
```go
words := []string{"i'll", "tell", "what", "I", "want"}

for index, word := range words {
    fmt.Println(index, word)
}
```

Output:
```
0 i'll
1 tell
2 what
3 I
4 want
```

---

## x in y
Python's `in` operator is used to tell if something is _inside_ another thing, like in a `list`:

```python
words = ["i'm", "a", "bad", "guy"]
if "bad" in words:
    print("duh!")

# duh!
```

Or to verify if a key exists in a `dict`:
```python
words = {"tough": "guy", "rough": "guy", "enough": "guy"}
if "tough" in words:
    print("billie is the best")

# billie is the best
```

Go has no `in` operator 😢 The verbose road needs to be taken:

```golang
words := []string{"i'm", "a", "bad", "guy"}

for _, word := range words {
    if word == "bad" {
        fmt.Println("duh!")
    }
}

// duh!
```

A bit of sugar is available to check keys in a map:
```golang
words := map[string]string{"tough": "guy", "rough": "guy", "enough": "guy"}

_, ok := words["tough"]
if ok {
    fmt.Println("billie is the best")
}

// billie is the best
```

---

## Lambda Functions
Lambdas, or anonymous functions, are nameless functions usually doing only one very simple thing. In Python, a lambda function any number of arguments but is restricted to a single expression – just one thing can happen after the `:`:

```python
multiply = lambda x: x*x

print(multiply(2))
# 4
```

Golang also has lambda functions, but they can spawn multiple statements:
```golang
double := func(x int) int { return x * 2 }

fmt.Println(double(2))
// 4

printAndDouble := func(x int) int { fmt.Println("one thing"); return x * 2}

fmt.Println(printAndDouble(2))
// one thing
// 4
```

---

## Context Manager
A context manager is a very powerful pattern in Python. It's usually used to manage resources, like closing a file automatically so you don't forget it. This is a very simplified explanation, it can do _a lot more_. [Here](https://book.pythontips.com/en/latest/context_managers.html) is a link for further explanation.
```python
with open('/etc/passwd', 'r') as file:
    print(file.read())

# all your passwords 👺

file.read()
# ValueError: I/O operation on closed file.
```

Golang has a kinda similar concept called `defer`. It ensures that a function call is performed at the end of the current function, so you don't forget to close your file after you've opened it.
```golang
func main() {
    file, _ := os.Open("/etc/passwd")
    // called when this function block finishes 
    defer file.Close()

    data, _ := ioutil.ReadAll(file)

    fmt.Println(data)
    // all your passwords, in bytes 👺
}
```

---

## Assignment Expressions
In Python 3.8, [assignment expressions](https://www.python.org/about/apps/) define a new notation: `NAME := expr`, where you can assign a value to a variable during an expression. An example is using the `if` statement:
```python
def x():
    return 10

if (number := x()) is number == 10:
    print(10)

# 10
```

Go also supports it:

```golang
func x() int { return 10 }

func main() {
    if x := x(); x == 10 {
        fmt.Println(x)
    }
}

// 10
```

---

## Classes with initialization and methods
Python has `Classes` to create a new _type_ with properties and methods, which can be used to create _instances_ of that type:
```python
class Double: 
    def __init__(self, x): 
        self.x = x 
 
    def double(self): 
        return self.x * 2 
 
instance = Double(2) 
 
print(instance.double())     
# 4
```

You can accomplish basically the same with Golang's `struct`:
```golang
type Double struct {
    x int
}

func (d *Double) double() int {
    return d.x * 2
}

func main() {
    instance := Double{x: 2}

    fmt.Println(instance.double())
    // 4
}
```

---

## Sugary Python, Simple Go
Even though I don't add sugar to my coffee, I'd pour some over Go. Learning Go after Python was full of surprises: I was used to a lot of tricks and one-liners. To this day, I read the SliceTricks page more than I'd be proud of.

Besides frustrations here and there, I love Go. Statically typed, easy coroutines, single binary, multi-architecture compilation and more! It has so many great features!

Both languages have their place in my tool belt and my heart ❤️

_Thanks to the people from [Twitter](https://twitter.com/jonatasbaldin/status/1185661547789594625) and [DEV](https://dev.to/jonatasbaldin/what-s-your-favorite-python-idiom-1f7p) to share their favorite idioms 💙_

Share your favorite idioms in the comments below and let's try to find its Go-like implementation ✨
