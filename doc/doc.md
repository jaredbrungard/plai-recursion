Recursion
=========

Copy your `tc` and `interp` functions into `interp.rs` and update
them to support recursion. Note that you should not make changes
anywhere except `interp.rs`.


What has changed
----------------

Here are the changes to the code that is given to you:


### In `main.rs`

*   A new `Token::Rec` token (the word `rec` in source code) to
    support recursive declarations

*   A new `Expression::Rec` expression type

*   The `Environment` type is updated to store values as type
    `Rc<RefCell<Option<Value>>>` (see discussion below)


### In `parse.rs`

*   A new parse rule for recursive expressions and parsing code to
    go with it


Support for recursion
---------------------

Be sure to refer to the discussion of recursion in the book and
lecture notes.

The big change here is in the implementation of environments. Before
we used a `HashMap` that maps a variable name (`String`) to a value
(`Value`). That gave us immutable values that had to be fully
computed before they could be added to an environment. In addition,
we clone entire environments, so even if we could change a
variable's value it would not be updated across all clones of the
environment.

In the new version we use a *reference counter* (`std::rc::Rc`) to
link to the `Value`, which lets us have multiple references to the
same value (across our clones). When the last reference is dropped,
the value is dropped.

Stop and go look up `std::rc::Rc` and how it works. There is a
discussion in Chapter 4 of the “Programming Rust, 2nd edition” book
as well as in the official documentation.

For recursive definitions, a key idea is that the name of a variable
needs to exist in environment while we are defining it. Consider
this code:

    rec sumn: (int -> int) = fn (n: int) {
        if n < 1 {
            n
        } else {
            n + sumn(n + -1)
        }
    } {
        sumn(10)
    }

A recursive definition follows the same form as `let` except that we
require a type annotation as well. Following the form of `let`, we
see that `sumn` is being set to:

    fn (n: int) {
        if n < 1 {
            n
        } else {
            n + sumn(n + -1)
        }
    }

So that it can be used in the expression:

    sumn(10)

But the lambda expression being assigned to `sumn` needs to be able
to refer to `sumn`. So we need `sumn` to exist in the environment
*before* we have fully calculated the value it will be assigned.

This implies that we must be able to change the environment after it
is created, and also that environment entries with no value may
exist. For the latter problem we use `Option` (so an entry that has
been created but not assigned can be `None` and later be updated to
`Some`).

To allow us to change the value after it is created, we store the
value in a `std::cell:RefCell`. Chapter 10 of the book introduces
this type under the section on “interior mutability”. Go read up on
that. In short, it pushes borrow checking to runtime instead of
compile time.

So our final environment type is a `HashMap` from `String` (as
before) to `Rc<RefCell<Option<Value>>>`.

The good news is that relatively little code is impacted by this
change since we already use a `lookup` helper to resolve variables.

You will also need to implement type checking and interpretation for
`Rec` expressions. Use the description in the textbook and carefully
plan how type checking and evaluation need to work. It may be
helpful to discuss this with other students and do some whiteboard
work before writing code.
