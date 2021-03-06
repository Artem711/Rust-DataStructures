# Bank of Resources for `Rust Fundamentals`

### General Information

### Rust Memory layout

### Expressions vs Statements

### Constants
Values that are available throughout the application which never change. (You must specify the Type)
```rust
const SOME_CONSTANT: u32 = 100;
```
At a technical level constant is not treated as variable during compilation.
Meaning that a compiled program won't have a memory allocation for your constant.
What happens is that during the compilation, the compiler will replace all instances of where you are using it, with the actual value directly in the compiled code.
This creates some runtime speed efficiencies, because the running program doesn't have to go looking for the memory and retrieves its value.
The is directly stored in the compiled code.

### Macros
Rust has excellent support for rust_fundamentals.macros. Macros enable you to write code that writes other code, which is known as metaprogramming.

Sources:
- Overview: https://www.youtube.com/watch?v=dZiWkbnaQe8
- Syntax Fragments (and more): https://danielkeep.github.io/tlborm/book/mbe-macro-rules.html
- Explanation: https://blog.logrocket.com/rust_fundamentals.macros-in-rust-a-tutorial-with-examples/#whatarerustmacros
- Workshop: https://github.com/dtolnay/proc-macro-workshop

```rust
macro_rules! vector {
    ($elem: expr; $n: expr) => { }
}
```
 Declaration of the macro (declares the name of the macro)
 Patterns for rust_fundamentals.macros (arguments to the macro (not like to the function))
 Instead of having a variable followed by a type, we have a `Syntax Pattern`
 The type is kind of a `Syntax Type` (eg: identifier, expression, block...)


Declarative Macros can be invoked in following ways (different delimiters):
```rust
vector!();
vector![];
vector!{};
```

Note: When returning anything from the declarative macro, we add an additional scope `{}` around everything.
This also allows us to store variables within that scope.
Below two examples are shown with test implementations:

The reason for this is that the output of a macro (if there is one) must be a valid expression syntax.
Therefore, if return a sequence without any block scope, it isn't a valid expression.
Examples:
```rust
#[cfg(test)]
mod tests {
 use super::*;

 #[test]
 fn vector_something() {
  let something_vector = something!(1,2,3,4,5);
  assert_eq!(something_vector, vec![1,2,3,4,5]);
 }

 #[test]
 fn printer() {
  let x = 32;
  something!(x);
 }

 #[test]
 fn result() {
  let v: Vec<i32> = something!();
 }
}

#[macro_export]
macro_rules! something {
 () => {
  Vec::new()
 };
 ($elem: ident) => {
  println!("Good morning");
  let s = Vec::<i32>::new(); // just to show case the ability that we can define variables without the block scope
                             // as long as we don't return and try to recieve any data from the macro
  for i in 0..10 {
   println!("Happy coding!, {}", $elem);
  }
 };
 ($($elem: expr), *) => {
  {
   let mut temp_vec = Vec::new();
   $(
     temp_vec.push($elem);
   )*
   temp_vec
  }
 }
}
```

```rust
// Simple sum of all passed values (variable number of arguments) and returning the result
#[macro_export]
macro_rules! simple_sum {
    ( $($x: expr), +) => {
        { // <- this block scope is added
            let mut acc = 0;
            $( acc += $x; )+
            acc
        } // <- this block scope is added
    }
}

// Sum by mutating the reference to `x` that gets passed into the macro (as mutable reference), & without a return value
#[macro_export]
macro_rules! assignment_sum {
    ($x: ident; $($num: expr), *) => {
        $(
            $x += $num;
        )*
    }
}

// With a return value (we are creating a new vector, manipulating and returning it)
#[macro_export]
macro_rules! vector {
    ( $($x: expr), *) => {
        { // <- this block scope is added
            let mut temp_vec = Vec::new();
            $( temp_vec.push($x); )*
            temp_vec
        } // <- this block scope is added
    }
}
```
Here we have the tests for these rust_fundamentals.macros: 
```rust
#[test]
fn simple_sum_test() {
 let result = simple_sum!(2, 4, 6);
 assert_eq!(result, 12);
}

#[test]
fn assignment_sum_test() {
 let mut x = 1;
 assignment_sum![x; 2, 34, 5];
 assert_eq!(x, 42);
}

#[test]
fn vector_test() {
 let result = vector!(1, 2, 3, 4, 5);
 assert_eq!(result, vec![1, 2, 3, 4, 5]);
}
```