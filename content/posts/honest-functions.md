+++
title = "(Dis)Honest Functions"
date = 2021-03-03
+++

After a recent debugging session, discovering I had once again been the victim of a dishonest function signature, I was... Well, let’s just say I was unimpressed. Two thoughts popped up in my head – the first one was «ahh.. this thing again..», and the second was «wait, why is this still even a thing?». It left me in a state of frustration.

<!-- more -->

The language was JavaScript, and the function was [`Array.prototype.sort`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort). Now, before we go on, let me just say that this is far from yet another JavaScript rant. Enough of those exists on the internet already. In fact, I don’t have a lot of problems with the language itself – this exact issue probably exists in most languages. I would like to say _all_, but I’m sure I would be proven wrong by someone on the internet.

But enough of that. As said, my problem was with `Array.prototype.sort`. It’s a sorting function. It sorts arrays. You give it an array and a comparator function, and get a sorted array back. Should be easy enough?

Let’s see an example.

```JavaScript
const unsorted = [0, 99, 50]
const sorted = unsorted.sort((a, b) => a - b)
console.log(sorted)
// -> [0, 50, 99]
```

Now, if we glance past the comparator function (which I dislike, but that’s the story of another blog post), there’s not too much going on here. The TypeScript definition makes a lot of sense as well:

```TypeScript
Array<T>.sort(compareFn?: ((a: T, b: T) => number) | undefined): T[]
```

`sort` is a function on `Array`s. We can even see that if we don’t supply a comparator function it should work anyway, as the return type says we always receive an array.

Now, let’s alter our example just a bit

```JavaScript
const unsorted = [0, 99, 50]
const sorted = unsorted.sort((a, b) => a - b)
console.log(unsorted) // <-- this is the altered bit – we print the original array instead of the sorted one
// [0, 50, 99]
```

Wait a minute... What? Our unsorted array is now sorted, even though we never sorted it? Something weird is definitely going on here. Or, weird might be the wrong word – surprising and confusing are better alternatives.

Once you read the documentation over at [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) the surprise starts to make sense quickly. `Array.prototype.sort` actually sorts the array _in place_. It’s even there in the absolute _first sentence_. There are at least two conclusions we can draw from this fact

- TypeScript definitions cannot tell the whole truth of how functions should be used
- It’s the first thing mentioned in the documentation – I am probably not the first one to make this mistake

Now, errors like these annoys me. It makes my day a lot harder than what it needs to be. I like making things. I don’t like wasting time when it could have easily been avoided.

## Dishonest Functions

It’s easy to put the blame on me as a developer. I should have read the documentation, there’s no denying that. But, what if... what if I didn’t have to read the full documentation to understand this single function?

`Array.prototype.sort` is an example of what I like to call dishonest functions. Functions that for some reason likes to hide who they really are, and surprise you when you least expect it. A function signature acts as a promise (an _actual_ promise, not the programming construct) – a promise from the writer of the function to me as a user that this function will behave in a certain way. When a function breaks that promise, it acts dishonestly. Our example with `Array.prototype.sort` is a bit sneaky, though. The type system can’t tell me whether our unsorted array is mutated in any way, so you could argue that it doesn’t break any promises. But there is a catch here – by returning an array, there is an implicit expectation that it leaves the input alone. No matter how hard I try, I can’t see any valid case where mixing these two behaviors makes sense.

My biggest problem with `Array.prototype.sort` is that there is no single best way of using it. The [examples at MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) seems to favor ignoring the return value. This works, but ignoring return values from functions is a code smell. Return values are there for a reason – ignoring them is just as bad as catching and swallowing exceptions. There are even linting rules to stop us from making such mistakes. But as we have seen, _not_ ignoring the return value gives us another problem. In our first example, our array named `unsorted` is in fact _sorted_, so our code communicates the wrong thing. This is a sleeping bug just waiting to make your day really bad.

Let’s look at a different example – this time from Kotlin

```Kotlin
fun String.toInt(): Int
```

A straight forward function signature. You have a String, you call `toInt`, you receive an `Int`. You might already have guessed where this is going – let’s see what happens if we do something completely unreasonable, such as calling it with something that is not a number.

```Kotlin
val myString = "not-a-number"
val myNumber = myString.toInt()
// --> java.lang.NumberFormatException: For input string: "not-a-number"
```

Oops! That’s not cool at all. You don’t want this in your production code. You might be thinking «yeah, as if I would ever do such a mistake...» right now, and I’m not here to argue with that. But what about your teammate who started programming professionally two months ago? Would they be able to catch these errors just as easy? I know _I_ have done this mistake myself – several times.

The important part here is not the examples themselves, but rather that it’s impossible to know about these inconsistencies unless you have encountered them before, or happen to read the full documentation about every function that you use.

## Honest Functions

Unlike dishonest functions, an honest function is, in lack of better words: honest. It tells you exactly how it behaves through its function signature, and possibly other, established norms. These norms might differ from language to language, but they should be consistent. In our world of honest functions, neither `Array.prototype.sort` or `String.toInt()` would exist in their current forms. They both hide their actual self, posing as something they’re not.

There are two important properties an honest function should honor: it has to be _pure_, and it has to be _total_. In essence, this means three things:

1. Given the same input (or same arguments, if you will), it should always produce the same output (or return value)
2. There should be no side effects.
3. The function has to be defined for all possible values of its input type(s)

If a function follows these three properties, there’s a good chance it is honest. Well, unless you intentionally name your function something completely unrelated to the implementation, but let’s try to keep our intentions good here. There is one problem, though. IO is a side effect, and not allowing IO is taking things a bit too far. Some languages have explicit constructs to indicate on a type level if a function needs to perform IO, but you don’t _need_ this to have honest functions. If a function has to print or log something in addition to its main purpose, just name it accordingly! This won’t tick the "no side effects" checkbox, but I will argue that it makes your function honest.

## Functions are like friends

Let me end this blog post with something that’s really cheesy, but that also somewhat works. I will compare functions to friendship. Yeah. Sorry about that.

Just like we all favor honest friends, we should also favor honest functions. We can most likely tolerate the odd lie here and there as long as it’s small and doesn’t affect us too much, but once the lies grows too big, it starts to have real consequences. Programming is really not too different. If anything, it might be even less forgiving. We can tolerate small amounts of inconsistencies, but too much and it hurts both your ablity to deliver, and ability to reason about your code.

There is no reason we should tolerate dishonest functions when a clear and honest alternative version could be just as easily be implemented. And just to be clear: it _always_ can. Don’t lie to your friends – stay honest, peeps!
