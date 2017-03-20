title: Notes of Hacking with Swift
date: 2016-05-31 13:24:03
tags: [Swift,学习笔记]
---
## Variables and constants
- A **variable** is a data store that can have its value changed whenever you want.
- A **constant** is a data store that you set once and can never change.
- In Swift, you make a variable using the `var` keyword, and a constants using the `let` keyword:
``` Swift
var name = "Brand Lee"
// you shouldn't use the var keyword each time – that's only used when you're declaring new variables
name = "Romeo"

let name = "Brand Lee"
// Cannot assign to 'let' value 'name'
// name = "Romeo"
```

- **Important note**: variable and constant names must be unique in your code.

## Types of Data
``` Swift
var name: String
name = "Tim McGraw"

var age: Int
age = 25

var latitude: Double
latitude = 36.166667

var longitude: Float
longitude = -86.783333

var stayOutTooLate: Bool
stayOutTooLate = true
```

- Two options to create your variable:
  1. create your variable and give it an initial value on one line of code;
  2. **type annotations**,like below:
- **Note**: some people like to put a space before and after the colon, making `var name : String`, but they are wrong and you should try to avoid mentioning their wrongness in polite company.
- The lesson here is that Swift always wants to know what type of data every variable or constant will hold,called **type safety**.
- Note how both `String` and `Int` have capital letters at the start, whereas name and age do not – this is the standard coding convention in Swift.
- `Float` and `Double`:the official Apple recommendation is always to use `Double` because it has the highest accuracy(`Double` has twice the accuracy of `Float`).
- it's possible to specify a data type and provide a value at the same time:

``` Swift
var name: String = "Tim McGraw"
```

## Operators
- In Swift strings are **case-sensitive**.

## String interpolation
- fancy name but a simple thing:combining variables and constants inside a string.

``` Swift
var name = "Tim McGraw"
var age = 25
var latitude = 36.166667
"Your name is \(name)"
// output: Your name is Tim McGraw

// Doing that using + is much more difficult, because Swift doesn't let you add integers and doubles to a string
"Your name is \(name), your age is \(age), and your latitude is \(latitude)"

"Your name is " + name
```
- One of the powerful features of string interpolation is that everything between \( and ) can actually be a full Swift expression.

## Arrays
- Swift uses `type inference` to figure out what type of data your array holds
``` Swift
var oddNumbers = [2, 4, 6, 8]
var songs = ["Shake it Off", "You Belong with Me", "Back to December"]
// Swift starts counting at 0
songs[0]
songs[1]
songs[2]
// print out the data type of variable
songs.dynamicType // output: Array<String>.Type

// Swift is behind the scenes converting the array to hold the data type NSObject
var songs = ["Shake it Off", "You Belong with Me", "Back to December", 3]
songs.dynamicType // output: Array<NSObject>.Type

// use type annotations to specify exactly what type of data you want an array to store
var songs: [String] = ["Shake it Off", "You Belong with Me", "Back to December"]
```

- Creating arrays

``` Swift
// uses a type annotation to make it clear we want an array of strings, and it assigns an empty array to it
var songs: [String] = []

// the () tells Swift we want to create the array in question, which is then assigned to songs using type inference
var songs = [String]()

var songs = ["Shake it Off", "You Belong with Me", "Love Story"]
var songs2 = ["Today was a Fairytale", "White Horse", "Fifteen"]
var both = songs + songs2

both += ["Everything has Changed"]
```
## Dictionaries
- Dictionaries are another common type of collection, but they differ from arrays because they let you access values based on a key you specify.
``` Swift
var person = [
                "first": "Taylor",
                "middle": "Alison",
                "last": "Swift",
                "month": "December",
                "website": "taylorswift.com"
            ]

person["middle"]
person["month"]
```

## Conditional statements

``` Swift
var action: String
var person = "hater"

if person == "hater" {
    action = "hate"
} else if person == "player" {
    action = "play"
} else {
    action = "cruise"
}
```

- Evaluating multiple conditions
  Swift uses something called `short-circuit` evaluation to boost performance: if it is evaluating multiple things that all need to be true, and the first one is false, it doesn't even both evaluating the rest.

## Loops

``` Swift
for i in 1...10 {
    print("\(i) x 10 is \(i * 10)")
}

// If you don't need to know what number you're on, you can use an underscore instead.

var str = "Fakers gonna"

for _ in 1 ... 5 {
    str += " fake"
}

print(str)
```

- the half open range operator: `..<`
  counts from one number up to and excluding another
- Looping over arrays

``` Swift
var songs = ["Shake it Off", "You Belong with Me", "Back to December"]

for song in songs {
    print("My favorite song is \(song)")
}

var people = ["players", "haters", "heart-breakers", "fakers"]
var actions = ["play", "hate", "break", "fake"]

// half open range operator
for i in 0 ..< people.count {
    print("\(people[i]) gonna \(actions[i])")
}
```
- One important note: although programmers conventionally use i, j and even k for loop constants, you can name them whatever you please: for personNumber in 0 ..< people.count is perfectly valid
- While loops

``` Swift
var counter = 0

while true {
    print("Counter is now \(counter)")
    counter += 1

    if counter == 556 {
        break
    }
}
```

## Switch case
- Swift will ensure your cases are exhaustive.That is, if there's the possibility of your variable having a value you don't check for, Xcode will refuse to build your app.
- Swift can apply some evaluation to your case statements in order to match against variables.

``` Swift
let studioAlbums = 5

switch studioAlbums {
case 0...1:
    print("You're just starting out")

case 2...3:
    print("You're a rising star")

case 4...5:
    print("You're world famous!")

default:
    print("Have you done something new?")
}
```

- use the `fallthrough` keyword to make one case fall into the next

## Functions
- Functions let you define re-usable pieces of code that perform specific pieces of functionality.
- an old Apple naming convention: function names should be designed to be read, including their first parameter. 
``` Swift
func printAlbumRelease(name: String, year: Int) {
    print("\(name) was released in \(year)")
}

printAlbumRelease("Fearless", year: 2008)
printAlbumRelease("Speak Now", year: 2010)
printAlbumRelease("Red", year: 2012)
```
- Swift functions can return a value by writing `->` then a data type. 
``` Swift
func albumsIsTaylor(name: String) -> Bool {
    if name == "Taylor Swift" { return true }
    if name == "Fearless" { return true }
    if name == "Speak Now" { return true }
    if name == "Red" { return true }
    if name == "1989" { return true }

    return false
}
```
## Optionals
- An optional value is one that might have a value or might not.

``` Swift
// question mark: that means "optional string."
func getHaterStatus(weather: String) -> String? {
    if weather == "sunny" {
        return nil
    } else {
        return "Hate"
    }
}

// when you declare a value as being optional, Swift will make sure you handle it safely.
// make status a optional String
var status: String?
status = getHaterStatus("rainy")
// or
var status = getHaterStatus("rainy")
```

- `optional unwrapping`:checks whether an optional has a value, and if so unwraps it into a non-optional type then runs a code block.
``` Swift
if let unwrappedStatus = status {
    // unwrappedStatus contains a non-optional value!
} else {
    // in case you you want an else block, here you go…
}
```

When you work with these "might be there, might not be" values, Swift forces you to unwrap them before using them, thus acknowledging that there might not be a value. That's what `if let` syntax does: **if the optional has a value then unwrap it and use it, otherwise don't use it at all**.
- `Force unwrapping optionals`:Swift lets you override its safety by using the exclamation mark character: !

``` Swift
func yearAlbumReleased(name: String) -> Int? {
    if name == "Taylor Swift" { return 2006 }
    if name == "Fearless" { return 2008 }
    if name == "Speak Now" { return 2010 }
    if name == "Red" { return 2012 }
    if name == "1989" { return 2014 }

    return nil
}

var year = yearAlbumReleased("Red")

if year == nil {
    print("There was an error")
} else {
// Note the exclamation mark: it means "I'm certain this contains a value, so force unwrap it now."
    print("It was released in \(year!)")
}
```

- `Implicitly unwrapped optionals`
   1. A regular variable must contain a value. Example: String must contain a string, even if that is string empty, i.e. "".
   2. An optional variable might contain a value, or might not. It must be unwrapped before it is used. Example: String? might contain a string, or it might contain nil. The only way to find out is to unwrap it.
   3. An implicitly unwrapped optional might contain a value, or might not. But it does not need to be unwrapped before it is used. Swift won't check for you, so you need to be extra careful. Example: String! might contain a string, or it might contain nil – and it's down to you to use it appropriately.
   
## Optional chaining
- `optional chaining`, which lets you run code only if your optional has a value.

``` Swift
// Note that there's a question mark in there, which is the optional chaining: everything after the question mark will only be run if everything before the question mark has a value.
let album = albumReleasedYear(2006)?.someOptionalValue?.someOtherOptionalValue?.whatever
```
- `The nil coalescing operator`
it effectively stops them from being optional because you provide a non-optional value B

``` Swift
let album = albumReleasedYear(2006) ?? "unknown"
print("The album is \(album)")
```