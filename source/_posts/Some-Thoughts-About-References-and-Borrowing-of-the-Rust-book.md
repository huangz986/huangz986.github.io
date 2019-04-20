---
title: Some Thoughts About References and Borrowing of the Rust book
tags: rust
date: 2019-04-21 00:21:25
---

I've been reading the book, **The Rust Programming Language**, lately, and after reading *CH4.2. References and Borrowing*, I think there is something wrong or misleading in the book. In general, it concluded that the rules of references are:
1.* At any given time, you can have *either* one mutable reference or any number of immutable references.
2.* *References must always be valid.*
These two rules are definitely right of course, but they are not very clear and accurate. Actually, it's not just the book, the rustc's error messages about borrowing are not very clear either.    

## The *REAL* Rules of References 
Here I will state the rules of references in my understanding first, and then explain the details. The rules are:
1. References must be valid while being used.
2. A mutable reference will invalidate all previous references, including mutable and immutable ones.
3. An immutable reference will invalidate the previous mutable reference, if there is one, but don't affect other immutable references.    

*ps: these rules apply to "a particular piece of data in a particular scope"*   

### When I Encounter The First Problem
In the middle of this chapter, It states such a conclusion and gives an example about it:
*But mutable references have one big restriction: you can have only one mutable reference to a particular piece of data in a particular scope. This code will fail:*   
*Filename:src/main.rs*   
```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;

println!("{}, {}", r1, r2);
```
The error message says:
`error[E0499]: cannot borrow 's' as mutable more than once at a time`   

Well, this example doesn't compile indeed. But somehow I accidentally delete the last sentence of the example, and compile it successly without error. How could that happen? `Rustc` just tell me *cannot borrow 's' as mutable more than once at a time*, and there are two references `r1` and `r2`, if I don't `println!` then no error at all? So I CAN borrow 's' more than once? Then I try to print r1 and r2 separately, error occurs. I could print `r2` but not `r1`. 
### And then
As I read on, something similar happens, if combining mutable and immutable references:
```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
let r3 = &mut s; // BIG PROBLEM

println!("{}, {}, and {}", r1, r2 ,r3);
```
Error says:
`error[E0502]: cannot borrow 's' as mutable because it is also borrowed as immutable`     

Actually, there is no problem when declaring r3, but a *BIG PROBLEM* if you try to print all references. I you print them separately, you'll find that r3 is printable, while `r1` and `r2` are not available to print. As the error message says, I *cannot borrow 's' as mutable because it is also borrowed as immutable*, we know it's definitely no true. 
## The Conclusion
From the previous examples, we know that we **could** *borrow 's' as mutable more than once*, but we could only access the last one, and that we **could** *borrow 's' as mutable* after *it is also borrowed as immutable*, but again we could only access the last reference which is mutable. The book says *References must always be valid*, so I assume that there is a validation status for every reference, a reference is accessable only if it's valid. Thus came my version of rules of references:
1. References must be valid while being used.
2. A mutable reference will invalidate all previous references, including mutable and immutable ones.
3. An immutable reference will invalidate the previous mutable reference, if there is one, but don't affect other immutable references.     

#### Something Else
If you run `rustc --explain E0499` and `rustc --explain E0502`, to find the explanations of those examples, you'll find the same mistakes in them. The *wrong* examples they provide actually turn out to be correct. Although we have to handle the `mut` issues which might have been raised due to rust update in those explanations. I had intended to write more details, but I had no spare time to do so. If anyone's reading my post, he should go through these `rustc --explain Exxxx` himself.
