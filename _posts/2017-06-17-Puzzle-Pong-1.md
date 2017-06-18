---
layout: post
title: Puzzle Pong - part 1
---

[James Buckland](jbuckland.com) and I have decided to start a game of Puzzle
Pong. These puzzles are more about the design of the program than about the
actual answer; so I've decided to cover some of my inferior designs in this blog
post. If you are only interested in the best solution, see the subsection titled
"Third Approach".

He challenged me to solve the following puzzle in his [blog
post](http://jbuckland.com/crossword/) about enumerating crossword puzzles. I
recommend you give it a read.

The Puzzle
==========

For every subset of the 26 character English alphabet, there is a set of words
which can be spelled using only characters from that subset. For example, given
the set {a, b, c, d}, /usr/share/dict/american-english says there are 24 words
which can be spelled - {A, Ac, Ada, B, Ba, C, Ca, Cd, D, Dacca, Dada, a, ad,
add, b, baa, bad, c, cab, cad, d, dB, dab, dad}. 

Which such subset has the highest words to characters ratio?

First Approach
==============

(Spoiler - this approach is fundamentally wrong.)

My first idea was to encode every word in the dictionary as a set of characters.
Then construct a map from set-of-characters to the count of how many times I've
seen that set. Lastly I'd iterate over the map and pick the set with the best
ratio.

### Implementation details

The representation of character sets is critical to making this program
performant. I chose to implement them in C++ as bit sets inside a uint32\_t. I
chose C++ because wanted to use a C like language so I could easily do efficient
bitwise operations. Since there are only 26 characters in the alphabet, 32 bits
is plenty to encode each word. The encoding function is pretty straightforward.
I made the decision to ignore characters outside [a-zA-Z] for simplicity.

     1  uint32_t WordToSet(const std::string& word) {
     2    uint32_t ret = 0;
     3    for (const char c : word) {
     4      if (c >= 'a' && c <= 'z') {
     5        ret |= (1 << (c - 'a'));
     6      } else if (c >= 'A' && c <= 'Z') {
     7        ret |= (1 << (c - 'A'));
     8      }
     9    }
    10    return ret;
    11  }

While I was at it, I wrote the decoding function. This will be used at the end
for printing the winning set.

     1  std::string SetToWord(uint32_t set) {
     2    std::string ret = "";
     3    uint32_t mask = 1;
     4    for (int i = 0; i < 25; i++) {
     5      if (set & (1 << i)) {
     6        ret += ('a' + i);
     7      }
     8    }
     9    return ret;
    10  }

I usually try to avoid string appending but this function is only called once so
it didn't seem like a big deal.

I also needed a function that told me the size of each set. The [Hacker's
Delight](http://www.hackersdelight.org/hdcodetxt/pop.c.txt) gave me several
promising choices. I initially  went with pop4() since it supposedly has the
best performance when the number of bits is small.

I was planning to bechmark this choice against some others until I found out
about
[__builtin_popcount()](https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html).
This lets the compiler use CPU specific instructions when counting the number of
1-bits in an int. 

In practice, this program spends a minority of its time counting the sizes of
sets so the benefit is minimal.

### Putting it all together

     1  int main() {
     2    // Key is a bit set encoding the characters of a string.
     3    // Value is the number of times that key has been seen so far.
     4    static uint32_t counts [(1 << 26) - 1] = {0};
     5   
     6    std::ifstream infile("/usr/share/dict/american-english");
     7    std::string word;
     8    // Populate counts.
     9    while (infile >> word) {
    10      ++counts[wordtoset(word)];
    11    }
    12   
    13    // Find best ratio in counts.
    14    uint32_t best_chars = 0;
    15    int best_num_chars = 0;
    16    int best_count = 0;
    17    for (auto it = counts.begin(); it != counts.end(); ++it) {
    18      int num_chars = Pop(it->first);
    19      int count = it->second;
    20      // Equivelent to (count / num_chars) > (best_count / best_num_chars)
    21      // but avoids floats.
    22      if (count * best_num_chars >= best_count * num_chars) {
    23        best_num_chars = num_chars;
    24        best_count = it->second;
    25        best_chars = it->first;
    26      }
    27    }
    28    
    29    std::cout << best_num_chars << "  " << best_count << "  " 
    30                                << SetToWord(best_chars) << std::endl;
    31   
    32    return 0;
    33  }

There is not much to comment on here. I enjoyed my method of fraction comparison
that avoided floats. This made me comfortable using -Ofast (which is faster than
O3 but might have out of spec behavior on float math in some
situations).

### Performance (and Correctness)

This program is quite speedy! On my 6 year old laptop, it ran in about a 20th of
a second. It says the best set {'a', 'e', 'r', 's', 't'} which contains 60
words:

60 seems like a surprisingly small number for such common characters so I asked
the program to print me which words it though were in this set. It retuned back
to me with the following:

{"Astarte", "Astarte's", "Easter", "Easter's", "Easters", "Sartre", "Teresa",
"Teresa's", "Terra's", "aerates", "arrest", "arrest's", "arrests", "assert",
"asserts", "aster", "aster's", "asters", "eater's", "eaters", "errata's",
"erratas", "rarest", "raster", "rate's", "rates", "reassert", "reasserts",
"restart", "restart's", "restarts", "restate", "restates", "retreat's",
"retreats", "stare", "stare's", "stares", "starter", "starter's", "starters",
"stater", "tare's", "tares", "tartest", "taster", "taster's", "tasters",
"tatter's", "tatters", "tear's", "tears", "teaser", "teaser's", "teasers",
"treat's", "treats"}

Now I could see that there was an error in my design; I being far to strict. I
was counting words that could be spelled using the characters in {'a', 'e', 'r',
's', 't'} but I was also requiring that all those characters be used. And
likewise for every other set of characters. This result was interesting but
ultimately not what I wanted.

Second Approach
===============

My second idea was to solve this problem by brute force. As before, encode the
dictionary 32 bit ints, but this time, loop over the all 2^26 sets of characters
and see what their ratio is. This is not elegant but I was confident it would
get me the correct answer.

### Implementation details

This approach mostly reused helper functions from the previous one. The only new
function I needed was for determining if one bit set was a subset of another.

My first idea was 

     1  // Args: two bit sets stored in uint32_t's
     2  // Returns true if the first set is a subset of the second.
     3  bool Subset(uint32_t first, uint32_t second) {
     4    return ~(~first | second) == 0;
     5  }

A coworker pointed out the much more elegant:

     1  bool Subset(uint32_t first, uint32_t second) {
     2    return (first | second) == second;
     3  }

Both of these only use a handful of assembly instructions.

### Putting it all together


     1  int main() {
     2    std::vector<uint32_t> char_sets;
     3  
     4    std::ifstream infile("/usr/share/dict/american-english");
     5    std::string word;
     6    while (infile >> word) {
     7      char_sets.emplace_back(WordToSet(word));
     8    }
     9  
    10    int best_num_chars = 0;
    11    int best_num_words = 0;
    12    int best_char_set = 0;
    13    for (uint32_t i = 0; i < ((1 << 26) - 1); ++i) {
    14      int num_words = 0;
    15      int num_chars = __builtin_popcount(i);
    16  
    17      // Count the number of words in the dictionary which can be spelled using
    18      // only the letters represented by i.
    19      for (const uint32_t char_set : char_sets) {
    20        if (Subset(char_set, i)) {
    21          ++num_words;
    22        }
    23      }
    24  
    25      // Equivalent to if(num_words/num_chars >= best_num_words/best_num_chars)
    26      // but without needing to use floating point numbers
    27      if (num_words * best_num_chars >= best_num_words * num_chars) {
    28        best_num_words = num_words;
    29        best_num_chars = num_chars;
    30        best_char_set = i;
    31      }
    32    }
    33  
    34    std:: cout << "num_words: " << best_num_words << std::endl
    35               << "set was: " << SetToWord(best_char_set) << std::endl;
    36  
    37    return 0;
    38  }

### Performance (and Correctness)

This code runs in about 25 minutes at produces the answer:
abcdefghiklmnorstuvwy. That is to say, all characters except {'j', 'q', 'x',
'y'}. Upon reflection, this isn't such a surprising answer.

Performancewise, this solution leaves a lot to be desired. For each of 2^26 sets
of characters, I am looping for ~100,000 words in the dictionary. What if there
was a way to do less work for each character set...?


Third Approach
==============

What if instead of looping over ~100k words, I did subset lookups in the map
from the first approach? So for a set like {'a, 'e', 'r'}, sum the values at {},
{'a'}, {'e'}, {'r'}, {'a', 'e'}, {'a', 'r'}, {'e', 'r'} and {'a', 'e', 'r'}. If
the initial set has n elements, then it has 2^n subsets. When n is small, this
is less work than looping over the whole dictionary; when n is large, this is
more work than looping over the whole dictionary.

This felt like a promising strategy but I didn't have an good ideas how to
efficiently iterate over all subsets of a bit set. I mentioned this to a
coworker and he suggested I check out the chess programming wiki. Apparently,
chess programs use bit sets for internal representations so this wiki has lots of
helpful functions for doing computations in bit sets. Indeed, it has a page on
[Traversing Subsets of a Set](https://chessprogramming.wikispaces.com/Traversing+Subsets+of+a+Set).

Armed with this, I modified my solution from approach 2 into the following:

### Putting it all together

     1  int main() {
     2    // For sets with many characters.
     3    std::vector<uint32_t> char_sets;
     4    // For sets with not many characters.
     5    static uint32_t counts [(1 << 26) - 1] = {0};
     6  
     7    std::ifstream infile("/usr/share/dict/american-english");
     8    std::string word;
     9    while (infile >> word) {
    10      char_sets.emplace_back(WordToSet(word));
    11      ++counts[WordToSet(word)];
    12    }
    13  
    14    int best_num_chars = 0;
    15    int best_num_words = 0;
    16    int best_char_set = 0;
    17    for (uint32_t i = 0; i < ((1 << 26) - 1); ++i) {
    18      int num_chars = __builtin_popcount(i);
    19  
    20      // Compute how many words can be spelled with those letters.
    21      int num_words = 0;
    22      if (num_chars <= 14) {
    23        uint32_t n = 0;
    24        do {
    25          num_words += counts[n];
    26          n = (n - i) & i;
    27        } while ( n );
    28      } else {
    29        for (const uint32_t char_set : char_sets) {
    30          if (Subset(char_set, i)) {
    31            ++num_words;
    32          }
    33        }
    34      }
    35  
    36      if (num_words * best_num_chars >= best_num_words * num_chars) {
    37        best_num_words = num_words;
    38        best_num_chars = num_chars;
    39        best_char_set = i;
    40    }
    41  
    42    std:: cout << "num_words: " << best_num_words << std::endl
    43               << "set was: " << SetToWord(best_char_set) << std::endl;
    44  
    45    return 0;
    46  }

### Performance

This program runs in about 16 minutes and produces the same answer as my second
one. I'm sure more improvements are possible but overall I'm much happier with
this design.

One interesting feature I'd like to point out is choice of 14 in line 22. Log
base 2 of 100,000 is around 16.6 so my initial guess for this value was 16. This
is because for sets of size 15 and 16, it costs fewer than 100k lookups in
counts to compute how many words can be spelled with those characters. In
practice, I get best performance with this threshold at 14. My suspicion is
that this is a cache locality issue. Looping over a vector probably has
favorable cache behavior but indexing into a table probably has more cache
misses.

## Conclusion

I challenge James to solve the following puzzle:

> Given the list of numbers [1, 2, 3, 4, 5] unlimited parentheses, and the
> operations {+, -, *, /, ^, !}, it is possible to construct many integers. For
> example, 
>
> 1 = 1 + 2 - 3 - 4 + 5
>
> 2 = 1 + (((2 + 3) * 4!) / 5!)
> ...
>
> What is the smallest positive integer that cannot be written this way?
>
> For some inspiration, please enjoy:
> -  [https://www.youtube.com/watch?v=ukUkVaOyI0o](https://www.youtube.com/watch?v=ukUkVaOyI0o)
> -  [https://www.youtube.com/watch?v=-ruC5A9EzzE](https://www.youtube.com/watch?v=-ruC5A9EzzE)
>
> To formalize the rules slightly.
> 1. Numbers must appear in order, each exactly once.
> 2. You may not take the factorial of a factorial (i.e. 3!! = 720).
> 3. Concatination is forbidden (i.e. 1 + 23 + 4 + 5).


> Bonus problem - For what list of five integers (all less than ten), is the
> answer to this puzzle the smallest? If this is not unique, please provide all
> lists which have the lowest non-expressable value.
