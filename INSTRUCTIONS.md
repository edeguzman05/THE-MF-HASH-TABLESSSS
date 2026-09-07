
# Project 4 - Hash Tables!

In this project, you'll do three major things:

1. Practice writing Hashing Functions

2. Practice writing a Hash Table

3. Implement a digital Rolodex (yesssss!)

4. Compete for the top spot on a Gradescope Leaderboard!

## Source Files

You'll implement code in several files:

* MyHashTable.hpp
* MyRolodex.hpp
* MyRolodex.cpp
* sandbox.cpp (optional)

You may add helper functions to classes within the above source files, but do not remove or modify any existing functions. Unit tests to determine your grade will be run against the code in the above files.

The *Makefile*, *main.cpp*, *Address.hpp*, *CPP_Tests.cpp*, and *words.txt* files should not be edited.

The *main.cpp* file should not be edited. It is just a very simple interface to help you play around with your Rolodex. It will not be graded. If you wish to make a custom program to help your debugging/development, you should edit the *sandbox.cpp* file instead.

As before, you can inspect the source files to hints and instructions written as comments, and a skeleton structure of your implementation files. Use those comments to guide you. You won't need to change arguments, return types, or method names; All you need to do is complete the body of each function.

## Implementing a Simple Hash Function

You'll remember from lecture that a hash function is simply a function that compresses and mangles some input data down to a fixed size range of numbers. In this project, we'll use a modified version of something called a "Mid Square" hash. But we also need to convert a string of characters to a single number, first. The algorithm here is fairly simple:

1. Define a reasonably high limit for a running `sum` so its value doesn't fly out of control. Since the maximum value of an `unsigned long long int` is `18446744073709551615`, we'll estimate our limit to be just a little lower than its square root, or `18446744073709551615/4294967297`, or roughly `4294967295`. You'll find this in your starter code available as a static member variable `MyHashTable::ULLONG_WRAP_AT`.

2. Start with a `sum` of 1.

3. Iterate over each character in the `std::string`. For each character, do the following:

    1. Cast the character to an `unsigned long long int`.

    2. Multiply the `sum` by the value of the current character.

    3. Keep the `sum` under control by applying a modulo against `MyHashTable::ULLONG_WRAP_AT`.

4. You should now have a value for `sum`, derived from the input string.

5. Multiply `sum` by itself, to get its square. Call the new variable `sum_squared`.

6. Keep the "middle digits" of `sum_squared`, and discard the rest (more explanation on this later). We'll call this new value `sum_squared_middle`.

7. Make sure the `sum_squared_middle` value is in-bounds for the current size of your table, by applying a final modulo against the current capacity of your table. Assign the result to a new variable called `hash_code`.

8. This final value of `hash_code` is the return result of your hashing function.

### Keeping the Middle Digits of a Number

So how will we "keep the middle digits"?

For now, let's practice on a 16-bt integer, also known as a [short](https://en.cppreference.com/w/cpp/language/types). Imagine we represent the integer as a sequence of bits (a binary string). Imagine then, we keep some bits in the middle and discard those to the left and right. Since this is a 16-bit number, we'll discard 8 bits (4 from each side). Then, we'll shift the remaining bits all the way to the right and pad the left with zeros.

Suppose we had the number 38647. We can represent this in binary as:

  * `1001011011110111`

If we wanted to "keep the middle 8 bits", we would:

  * Discard the 4 bits on the left: `1001`
  * Discard the 4 bits on the right: `0111`
  * Keep the 8 bits in the middle: `01101111`

Since our result is supposed to be 16-bits, we might imagine the kept bits should be padded with zero's to the left:

  * `0000000001101111`

We could then double check our conversion by lining the original number up against the final number, with some spaces to show their alignment:

```
    1001011011110111
0000000001101111
```

If you're unfamiliar with how to manipulate the bits in an integer, here are some research areas to help:

  * [Arithmetic Operators](https://en.cppreference.com/w/cpp/language/operator_arithmetic)

  * [std::bitset](https://en.cppreference.com/w/cpp/utility/bitset)

***Hint***: You can easily convert an integer to a string representation of its bits with the *bitset* class, to help your debugging.

### Full Example

In this example, we'll convert the string `Hello` to a hash code from start to finish.

#### Convert the String to a Number

Perform the following steps on the input string:

1. Start `sum` with a value of `1`

2. Iterate over each character's numeric representation (hint: *static casting*) (hint: *ascii*). You might think of this like breaking down the string to the following array: `[72, 101, 108, 108, 111]`.

3. Process `sum` with the first letter's value: `1 * 72 == 72` and apply a modulo of `MyHashTable::ULLONG_WRAP_AT` to make sure our sum stays within a reasonable bound. The sum ends up remaining at `72`.

4. Process the next digit: `(72 * 101) % ULLONG_WRAP_AT == 7272`

5. Process the next digit: `(7272 * 108) % ULLONG_WRAP_AT == 785376`

6. Process the next digit: `(785376 * 108) % ULLONG_WRAP_AT == 84820608`

7. Process the last digit: `(84820608 * 111) % ULLONG_WRAP_AT == 825152898`.

    * The result of `84820608 * 111` was `9415087488` and beyond `MyHashTable::ULLONG_WRAP_AT`. Thus, this is the first time the modulo operator actually did anything (it wrapped `9415087488` to `825152898`).

At this point, we've successfully converted our `std::string` key to a number.

#### Square and Keep the Middle Bits

All that's left now is to square it, then keep the middle bits. We can perform the following steps:

1. Square the number: `825152898 * 825152898 == 680877305077798404`

2. Convert or imagine the number as its binary representation. We'll write it out in chunks of 8-bits (1 byte) to make it easier to read:

    * `00001001 01110010 11110110 01001101 00110000 11001001 10010110 00000100`

3. Since this number is 64-bits long, we'll keep only the 32 bits in the middle:

    * `11110110 01001101 00110000 11001001`
    * (we discarded 16 bits on the left and 16 bits on the right)

4. Because our target data type is still a 64-bit integer, we'll pad the left side of our bits with 0's:

    * `00000000 00000000 00000000 00000000 11110110 01001101 00110000 11001001`

Don't forget to wrap the result of the "mid square" by the number of rows in your hash table, so it doesn't go out of bounds. In the current example, our final bit sequence works out to the number `4132253897`:

* `4132253897` wrapped to a table of size 100000 would be `53897`

* `4132253897` wrapped to a table of size 1000 would be `897`

* `4132253897` wrapped to a table of size 1024 would be `201`

## Implementing the Hash Table

For the most part, your hash table will follow the form presented in lecture. Some of the function prototypes are a bit different to make things interesting, but the general concept is the same.

### Difference Between Size and Capacity

For the purposes of this project, we'll say that `size` is the number of items currently inside our hash table, while `capacity` is the number of rows in our hash table. This may be a little misleading at first, because technically our hash table can store more than its capacity, through the use of collisions.

(also remember that collisions are handled through the use of a singly linked list sitting in each row)

## Implementing the Rolodex

Once again, we're going to use the wrapper pattern. You'll see the `MyRolodex` class contains all the functions you'd probably expect from a Rolodex, which are very similar to the functionality your hash table already provides. For the most part, you'll simply wrap each function of the `MyRolodex` class around a call to your `MyHashTable`. Easy as Pi.

## Gradescope Leaderboard

You may notice several empty functions inside `MyHashTable`:

* `myCustomHashFunction1`

* `myCustomHashFunction2`

* `myCustomHashFunction3`

* `myCustomHashFunction4`

Although the unit tests will primarily grade your submission against the `midSquareHash` function, you can also play around with the four custom functions above. None of the four custom hash functions will improve your test scores, BUT .... implementing something really good ***might*** result in bonus points to your overall projects category score!

Try to use each of the four functions to experiment with making your very own custom hashing functions. You may search the internet for formulas and algorithms, but don't copy any code (else you may risk plagiarism). The unit tests will determine how many collisions your hashing functions cause. Remember: One property of a good hashing function is that it produces the fewest amount of collisions as possible.

Once finished, the unit tests will take your *best* hashing function (read as: The one that produces the least number of collisions), and upload the number of collisions it produces, to Gradescope. Each student/group's best hashing function will be placed onto a Leaderboard hosted by Gradescope, for everyone to see. When this assignment is finished, the ***top submissions*** (including any ties) will earn a small bonus to each student's semester grade, placed in the project category. The number of *top submissions* and the project category grade boost will be announced in class.

It should go without saying that you shouldn't try to cheat the system by using any pre-made hashing functions that may or may not be available within C++ or other libraries. Doing so may be considered academic dishonesty. And yes, trying to use a random number generator with a fixed seed is also cheating. You must write the hash function yourself!

***Just to clarify***: Gradescope won't show your entire project grade to everyone; Only your custom hashing function's performance will be shown. Also, you can choose any name alias for the leaderboard you like - other students will only see your alias, but your professor will still see your real name.

## Hints and Tips

This section contains hints and tips that may help you along your way.

### Using std::forward_list::erase_after

We handle collisions by making each row of your hash table an `std::forward_list`. But you may notice that an `std::forward_list` doesn't have an `erase` function! Instead, you'll have to use a function called `erase_after`, which takes an Iterator to an element, and erases the element *after* that iterator.

This doesn't sound too hard at first. You could easily erase any element by simply iterating through the list, making a copy of the iterator at each step that can be incremented to peek at the next element. Once your peek-ahead iterator see's the element to be deleted, you just call `std::forward_list::erase_after` on the original iterator.

But what if the element to be erased is at the beginning of your list? In this case, you can grab an iterator that starts *before* the beginning of your list by calling `std::forward_list::before_begin` instead of `std::forward_list::begin`.

See the following example for more information: [before_begin](https://www.cplusplus.com/reference/forward_list/forward_list/before_begin/)

## Execution and Testing

Execution and testing are controlled with a *Makefile* written for [GNU Make](https://www.gnu.org/software/make/). The included Makefile has several targets you can use during development. As mentioned earlier, do not modify the *Makefile* file.

See a help menu with available commands:
```console
$ make help
```

## Submission

As before, we'll be using [Github](https://github.com/) to push code, but [Gradescope](https://www.gradescope.com/) to submit for grading. Do not only submit to Github. If you forget to submit through Gradescope, you will receive a zero grade.

Please note that all grades are subject to further deductions via manual grading, as needed.

## Copyright Notice

This assignment and its content are Copyright 2024 Mike Peralta, all rights reserves unless otherwise specified. No content may not be shared, uploaded, or distributed unless otherwise noted or permission given.

Authorization is given to students enrolled in this course to reproduce this material exclusively for their own personal academic use.

If you are an instructor wishing to use this assignment for your own course, send me an email :)


