---
title: Guide
---

Welcome to the Cargo guide. This guide will give you all that you need to know
about how to use Cargo to develop Rust projects.

# Why Cargo exists

Cargo is a tool that allows Rust projects to declare their various
dependencies, and ensure that you'll always get a repeatable build.

To accomplish this goal, Cargo does four things:

* Introduces two metadata files with various bits of project information.
* Fetches and builds your project's dependencies.
* Invokes `rustc` or another build tool with the correct parameters to build your project.
* Introduces conventions, making working with Rust projects easier.

# Converting to Cargo

You can convert an existing Rust project to use Cargo. You'll have to create a
`Cargo.toml` file with all of your dependencies, and move your source files and
test files into the places where Cargo expects them to be. See the [manifest
description](/manifest.html) and the "Cargo Conventions" section below for more
details.

# Creating A New Project

To start a new project with Cargo, use `cargo new`:

```shell
$ cargo new hello_world --bin
```

We're passing `--bin` because we're making a binary program: if we
were making a library, we'd leave it off.

Let's check out what Cargo has generated for us:

```shell
$ cd hello_world
$ tree .
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

This is all we need to get started. First, let's check out `Cargo.toml`:

```toml
[package]

name = "hello_world"
version = "0.0.1"
authors = ["Yehuda Katz <wycats@example.com>"]
```

This is called a **manifest**, and it contains all of the metadata that Cargo
needs to compile your project. 

Here's what's in `src/main.rs`:

```rs
fn main() {
    println!("Hello world!");
}
```

Cargo generated a 'hello world' for us. Let's compile it:

<pre><code class="highlight"><span class="gp">$</span> cargo build
<span style="font-weight: bold"
class="s1">   Compiling</span> hello-world v0.1.0</code></pre>

And then run it:

```shell
$ ./target/hello-world
Hello world!
```

We can also use `cargo run` to compile and then run it, all in one step:

<pre><code class="highlight"><span class="gp">$</span> cargo run
<span style="font-weight: bold"
class="s1">   Fresh</span> hello-world v0.1.0
<span style="font-weight: bold"
class="s1">   Running</span> `target/hello_world`
Hello world!</code></pre>

You'll now notice a new file, `Cargo.lock`. It contains information about our
dependencies. Since we don't have any yet, it's not very interesting.

# Working on an existing Cargo project

If you download an existing project that uses Cargo, it's really easy
to get going.

First, get the project from somewhere. In this example, we'll use `color-rs`:

```sh
$ git clone https://github.com/bjz/color-rs.git
$ cd color-rs
```

To build, just use `cargo build`:

<pre><code class="highlight"><span class="gp">$</span> cargo build
<span style="font-weight: bold" class="s1">   Compiling</span> color v1.0.0 (https://github.com/bjz/color-rs.git)</code></pre>

This will fetch all of the dependencies and then build them, along with the
project.

# Adding Dependencies

To depend on a library, add it to your `Cargo.toml`.

```toml
[package]

name = "hello-world"
version = "0.1.0"
authors = ["Yehuda Katz <wycats@example.com>"]

[dependencies.color]

git = "https://github.com/bjz/color-rs.git"
```

You added the `color` library, which provides simple conversions
between different color types.

Now, you can pull in that library using `extern crate` in
`main.rs`.

```rs
extern crate color;

use color::{RGB, ToHSV};

fn main() {
    println!("Converting RGB to HSV!");
    let red = RGB::new(255u8, 0, 0);
    println!("HSV: {}", red.to_hsv::<f32>());
}
```

Let's tell Cargo to fetch this new dependency and update the `Cargo.lock`:

<pre><code class="highlight"><span class="gp">$</span> cargo update color
<span style="font-weight: bold" class="s1">    Updating</span> git repository `https://github.com/bjz/color-rs.git`</code></pre>

Compile it:

<pre><code class="highlight"><span class="gp">$</span> cargo run
<span style="font-weight: bold" class="s1">   Compiling</span> color v1.0.0 (https://github.com/bjz/color-rs.git#bf739419)
<span style="font-weight: bold" class="s1">   Compiling</span> hello-world v0.1.0
$ ./target/hello-world
Converting RGB to HSV!
HSV: HSV { h: 0, s: 1, v: 1 }</code></pre>

We just specified a `git` repository for our dependency, but our `Cargo.lock`
contains the exact information about which revision we used:

```toml
[root]
name = "hello_world"
version = "0.0.1"
dependencies = [
 "color 0.0.1 (git+https://github.com/bjz/color-rs.git#bf739419e2d31050615c1ba1a395b474269a4)",
]

[[package]]
name = "color"
version = "0.0.1"
source = "git+https://github.com/bjz/color-rs.git#bf739419e2d31050615c1ba1a395b474269a4"
```

Now, if `color-rs` gets updated, we will still build with the same revision, until
we choose to `cargo update` again.

# Cargo Conventions

Cargo uses conventions to make it easy to dive into a new Cargo project. Here
are the conventions that Cargo uses:

* `Cargo.toml` and `Cargo.lock` are stored in the root of your project.
* Source code goes in the `src` directory.
* External tests go in the `tests` directory.
* The default executable file is `src/main.rs`.
* Other executables can be placed in `src/bin/*.rs`.
* The default library file is `src/lib.rs`.

# Cargo.toml vs Cargo.lock

`Cargo.toml` and `Cargo.lock` serve two different purposes. Before we talk
about them, here's a summary:

* `Cargo.toml` is about describing your dependencies in a broad sense, and is written by you.
* `Cargo.lock` contains exact information about your dependencies, and is maintained by Cargo.
* If you're building a library, put `Cargo.lock` in your `.gitignore`.
* If you're building an executable, check `Cargo.lock` into `git`.

Let's dig in a little bit more.

`Cargo.toml` is a **manifest** file. In the manifest, we can specify a bunch of
different metadata about our project. For example, we can say that we depend
on another project:

```toml
[package]

name = "hello-world"
version = "0.1.0"
authors = ["Yehuda Katz <wycats@example.com>"]

[dependencies.color]

git = "https://github.com/bjz/color-rs.git"
```

This project has a single dependency, on the `color` library. We've stated in
this case that we're relying on a particular Git repository that lives on
GitHub. Since we haven't specified any other information, Cargo assumes that
we intend to use the latest commit on the `master` branch to build our project.

Sound good? Well, there's one problem: If you build this project today, and
then you send a copy to me, and I build this project tomorrow, something bad
could happen. `bjz` could update `color-rs` in the meantime, and my build would
include this commit, while yours would not. Therefore, we would get different
builds. This would be bad, because we want reproducible builds.

We could fix this problem by putting a `rev` line in our `Cargo.toml`:

```toml
[dependencies.color]

git = "https://github.com/bjz/color-rs.git"
rev = "bf739419e2d31050615c1ba1a395b474269a4"
```

Now, our builds will be the same. But, there's a big drawback: now we have to
manually think about SHA-1s every time we want to update our library. This is
both tedius and error prone.

Enter the `Cargo.lock`. Because of its existence, we don't need to manually
keep track of the exact revisions: Cargo will do it for us. When we have a
manifest like this:

```toml
[package]

name = "hello-world"
version = "0.1.0"
authors = ["Yehuda Katz <wycats@example.com>"]

[dependencies.color]

git = "https://github.com/bjz/color-rs.git"
```

Cargo will take the latest commit, and write that information out into our
`Cargo.lock` when we build for the first time. That file will look like this:

```toml
[root]
name = "hello_world"
version = "0.0.1"
dependencies = [
 "color 0.0.1 (git+https://github.com/bjz/color-rs.git#bf739419e2d31050615c1ba1a395b47426)",
]

[[package]]
name = "color"
version = "0.0.1"
source = "git+https://github.com/bjz/color-rs.git#bf739419e2d31050615c1ba1a395b47426"
```

You can see that there's a lot more information here, including the exact
revision we used to build. Now, when you give your project to someone else,
they'll use the exact same SHA, even though we didn't specify it in our
`Cargo.toml`.

When we're ready to opt in to a new version of the library, Cargo can
re-calculate the dependencies, and update things for us:

```shell
$ cargo update       # updates all dependencies
$ cargo update color # updtes just 'color'
```

This will write out a new `Cargo.lock` with the new version information.

# Overriding Dependencies

# Tests

Cargo can run your tests with the `cargo test` command. Cargo runs tests in two
places: in each of your `src` files, and any tests in `tests/`. Tests
in your `src` files should be unit tests, and tests in `tests/` should be
integration-style tests. As such, you'll need to import your crates into
the files in `tests`.

To run your tests, just run `cargo test`:

<pre><code class="highlight"><span class="gp">$</span> cargo test
<span style="font-weight: bold"
class="s1">   Compiling</span> hello-world v0.1.0
running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
</code></pre>

Of course, if your project has tests, you'll see more output, with the
correct number of tests.

## Travis-CI

To test your project on Travis-CI, here is a sample `.travis.yml` file:

```
install:
  - curl http://www.rust-lang.org/rustup.sh | sudo sh -
script:
  - cargo build --verbose
  - cargo test --verbose
env:
  - LD_LIBRARY_PATH=/usr/local/lib
```
