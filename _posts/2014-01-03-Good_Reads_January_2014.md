---
layout: default
title: "Good Reads January 2014"
date: 2014-01-03
tags: links reads
---

# Good Reads January 2014

## Stroustrap: A Tour of C++

### Chapter 1

- Core language features: built-in types, loops, etc.
- Standard-library components: containers, I/O operations
- Every C++ program must have exactly one function named `main()`
- read `<<` as *put to*
- standard output stream `std::cout`
- `std::` signifies that the name `cout` is declared in the standard-library
  namespace
- `using namespace std;`: make names from `std` visible
- *defining a function / variable*: specifying how an operation is done
- *declaring a function / variable*: making the name and type
  of a variable or function known to the program
- function declarations can contain argument names but unless the
  declaration is also a definition the argument name is ignored
- if two functions have the same name but different numbers of arguments,
  the **compiler** will choose the most appropriate function to invoke
  for each call (*function overloading*)
- if choosing between two functions is ambiguous, the compiler throws an error
- *type*: defines possible values and allowed operations
- *object*: memory that holds a value of some type
- *value*: set of bits interpreted according to type
- *variable*: a named object
- *fundamental types*: `bool`, `int`, etc.
- *initializer lists*: `double a {1.0};` or `complex<double> z {1.0, 2.0};`
  (note that `=` is optional with `{ }`)
- prefer `{}` initialization over C-style `=`
- use `auto` when stating type is redundant in an initialization:
  `auto b = true;` or `auto i = 123;`
- with `auto` it is fine to use `=` because C-style type conversion impossible
- mention type specifically when:
    - want to make type clear specifically
    - be specific about range or precision (`double` v. `float`)
- *scopes:
    - local: function, lambda
    - class: member names (class member names)
    - namespace: namespace member names (outside function, lambda, class)
    - global: a name defined outside any of the above (in global namespace)
- unnamed objects are created with `new Class{"Initial Value"};`
- objects constructed with `new` live until destroyed with `delete`
- *immutable objects*:
    - `const` promises that object does not change
    - `constexpr`: placed in read-only memory (faster), expression will be
      evaluated at compile time, e.g. `constexpr int i{value};` is okay if
      `value` is a constant expression itself, e.g. `const int value {1};`
- `constexpr` declared functions must be simple functions with just a return
  value and need to be called with constant expression as argument, otherwise
  the resultant function will not be a constant expression
- read prefix `\*` as *contents of*
- read prefix `&` as *address of*
- `for (auto x : v)`: for every element `x` in `int v[] = {1,2,3,4}`
- `for (auto i : {1,2,3,4})`
- In declarations, the prefix `&` does not mean *address of* but
  *reference to*. References are like pointers except that we do not need to
  use the `\*` prefix to get the value of the object they refer to.
  A reference cannot be made to point to a different object after its
  initialization.
- `for (auto &i : v) i++;` changes `v` to `{2, 3, 4, 5}`
- use references in when specifying function arguments: avoids copying
  the argument when the function is called and we know we are operating on
  the argument that was passed to the function:
- `const` before argument type and name also prevents copying and does not
  allow the function to modify the passed argument
- *declarator operators*:
    - `T a[n];`: array of `n` `T`'s
    - `T* p`: pointer to `p`
    - `T& p`: reference to `p`
- `nullptr`: represents notion of *no object available*
- note that `nullptr` is an actual pointer instead of C-style `NULL` which
  is the preprocessor-defined integer `0`
- testing pointer `while(p)` equivalent to `while(p!=nullptry)`
- `<<` is *put to*, `>>` is *get from*
- `cin`: standard input stream

## Git: moving files between repositories

[Blog post by Greg Bayer](http://gbayer.com/development/moving-files-from-one-git-repository-to-another-preserving-history/).

## Recursive Neural Networks

[Socher *et al.*](http://nlp.stanford.edu/pubs/SocherLinNgManning_ICML2011.pdf)

[Yoshua Bengio](http://nlp.stanford.edu/pubs/SocherLinNgManning_ICML2011.pdf)

* Recursive structure in scenes and natural language
  (Principle of Compositionality)

    * The 'whole' (sentence, image) can be split into hierarchical regions
      (noun phrases, words, objects) that may occur in different 'whole'
      objects
    * Meaning of a sentence is given by its words and the rules that combine
      these words.
      
* Composite vector representation

    * Representation of a sentence in a vector space that represents
      the vocabulary.
      
* Recursive neural networks jointly learn compositional vector representations
  and parse trees

* Generate a parse tree of the sentence using a standard tokenizer,
  the tokens are the leaves of the parse tree
* Iteratively, choose two child nodes and decide if you want to combine them
  into their parent node which is the semantic representation of the child
  nodes - also produce a score of how plausible the new parent node is
  
## Unsupervised Feature Learning - Amgad Muhammed

[http://www.slideshare.net/AmgadMuhammad/unsupervised-feature-learning](http://www.slideshare.net/AmgadMuhammad/unsupervised-feature-learning)

- Autoencoders

    - These are neural networks whose weights are trained (using backpropagation)
      to reproduce the input in the output (the weights are trained so that the
      final network is a good representation of the input)

    - A sparsity condition, enforced with the Kullback-Leibler divergence is
      used to constrain the activity of the hidden neurons

- Principal Component Analysis

- Whitening

    - Scale features so that they all have unit variance

- Self-Taught Learning and Unsupervised Feature Learning

    - Train autoencoder on unlabeled data to get condensed representation of
      the data

    - Condensed because the number of hidden nodes is chosen to be less than
      the number of input (and equally output) nodes?

    - Feed labeled data to trained autoencoder and replace input with
      activation of hidden nodes - this condenses our 
      representation of the input
      to the (fewer) hidden neurons

