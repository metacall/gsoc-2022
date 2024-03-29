# Google Summer of Code 2022: Integrate Rust into other languages using Metacall.

I am really honored to participate in GSOC 2022, and add Rust support for [Metacall](https://github.com/metacall/core). 
It is super enjoyable for me to cooperate with the community and I learned a lot from other developers. I really enjoy the excellent experience.

I will explain my work in this post.
 
## What is Metacall?

MetaCall is a polyglot runtime environment that lets developers call functions and methods between different programming languages. 
For example, we can call a Python function from Node.js or we can create a Python object within Ruby. This is done by following the Meta Object Protocol which represents primitive types as an object.
It is super useful when you want to handle different tasks with different programming languages.

## Why we need a Rust Loader for Metacall?

Rust is a very interesting and popular programming language and it has a huge ecosystem which grows fast. 
You can find many useful tools and libraries written in Rust, and maybe you want to use them in other languages for your special use case. 
Further more, if we have a rust loader, we can write functions in Rust on the [FaaS platform](https://metacall.io/).

However, as you may know, rust is a compiled language, which means you need to compile the source code into a executable in order to run it, making it hard to follow the Meta Object Protocol.

## What can we do with rust and Metacall?

Basically, with rust loader, we have three ways to load rust types into Metacall:

1. Load from a rust file.
2. Load from memory (string).
3. Load from a pre-compiled rust project.

Here's [the repository](https://github.com/metacall/python-rust-melody-example) that contains all the code listed below.

### Load from file

First, prepare a rust script contains several functions, here we put the following codes into `basic.rs`.

```rust 
pub fn add(num_1: i32, num_2: i32) -> i32 {
    num_1 + num_2
}
```

Then we can load it within a Python script like this:

```python
from metacall import metacall, metacall_load_from_file
metacall_load_from_file("rs", ["./basic.rs"])
assert metacall("add", 2, 7) == 9
```

We support common primitive types like:

1. Numeric types
2. String
3. Vec
4. HashMap

For more examples, check out [the repository](https://github.com/metacall/python-rust-melody-example).

### Load from memory 

Loading from memory is pretty similar to loading from file, so we can just read the `basic.rs` and do some tests just like before.

```python
with open("./basic.rs", "r") as f:
    content = f.read()
metacall_load_from_memory("rs", content)
```
    
### Load from pre-compiled project

You may find that we cannot use other libraries from `crates.io` just as we do with cargo, this is because when loading a script, we are just using `rustc` to compile the code and it would be hard to manage the dependencies with `rustc` only. The good news is, if we load a pre-compiled package, we can let cargo handles these dependencies and as a result, we can use other crates.

The basic idea is, we create a new rust project using cargo which serves as a wrapper, exposing all functions we need from other crates, and it's a pretty standard way of creating a rust library. After that, cargo will give us `lib<name>.rlib`, which will be loaded into metacall later.

Then we can use the following Python script to load it:

```python
metacall_load_from_package("rs", "./melody_wrapper/target/debug/libmelody.rlib")
```

And now you can use all exported functions in Metacall!

For a fully working example, check out [the repository](https://github.com/metacall/python-rust-melody-example).

## How do we implement a rust loader for metacall?

But how do we implement the rust loader? First, we need to know the information of objects contained in the loaded stuff, like the signature of functions and definition of classes. Second, we need to extract them into Meta Objects which can be recognized by Metacall.

### Parsing 

Depending on the loading method, we have different parsing scheme.

When loading from file or memory, what we get is plain source code, so we choose to hook into the `rustc` and utilize the information generated by `rustc` for compilation including AST and intermediate representation of codes.

When loading from package, we don't have source code, but luckily, compiled `rlib`s contain all necessary information for parsing. For those who are interested in how to parse `rlib`, check out [this repository](https://github.com/MediosZ/meta-decoder) for a minimal but detailed example.

After parsing, we obtain the information contained in the script or package, next step is loading them into metacall.

### Loading

Since metacall is written in `c` mainly and we want to call `rust` functions in `c`, it's intuitive to use `ffi` to do this. The hard part of this how do we handle parameters. Every type in metacall is a piece of data, and it's referenced by a void pointer `void*`, this means we must handle the transformation between metacall value and rust types. A feasible option is generating some wrapper for rust code, which does the transformation automatically through code generation, for example:

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

The wrapper would look like something like this:
```rust 
#[no_mangle]
extern "C" pub fn wrapper_add(a: *mut c_void, b: *mut c_void) -> *mut c_void {
    let arg1 = metacall_value_to_int(a);
    let arg2 = metacall_value_to_int(b);
    let res = add(arg1, arg2);
    metacall_value_create_int(res)
}
```

This works well, we have everything needed to generate them, except that it's very easy to make some mistake when generating these wrappers and ruin the program, since they are generated following simple rules, and we cannot rely on compiler to check these wrappers when writing them. 
On the other hand, it would hard to deal with structs because they can not cross the ffi bound.

When I try to support loading structs, I try to search for rust reflection, and I found a series of [blog posts](https://www.osohq.com/post/rust-reflection-pt-1) about this. The basic idea is that we can use ` Arc<dyn Fn(Vec<MetacallValue>)>` to wrap a function into a object. And then we can treat the function or struct as a normal object. This is similar to the Meta Object Protocol, and with the help of this, not only do we have support for `struct`, but we don't need to generate those wrappers by hand. We can utilize `From` and `Into` trait to transform between metacall value and rust types.

After loading these functions and structs, we can use them in other languages.

## Future works

So far, we still have a lot of interesting things to do, first thing is that we can passing complex parameters like functions, and objects, which will be useful when dealing with callbacks.

Second thing is that we really want to support async rust, like awaiting a rust function in other languages. It helps when dealing with IO and networks.


## Contribution logs

Here's the link of [my contributions to the core](https://github.com/metacall/core/pulls?q=is%3Apr+author%3AMediosZ) if needed.
I also prepared [the example I have been showcasing during this post](https://github.com/metacall/python-rust-melody-example).
