# 数据类型

In this post I'll discuss Rust's data types. These are roughly equivalent to
classes, structs, and enums in C++. One difference with Rust is that data and
behaviour are much more strictly separated in Rust than C++ (or Java, or other
OO languages). Behaviour is defined by functions and those can be defined in
traits and `impl`s (implementations), but traits cannot contain data, they are
similar to Java's interfaces in that respect. I'll cover traits and impls in a
later post, this one is all about data.

下面我们要讨论Rust中的数据类型。它们大致对应于C++中的class, struct和enum类型。与C++（Java以及大多数面向对象的语言）相比，Rust的数据和行为分离的更为严格。在Rust中，行为可以在traits或impl中由函数定义，其中traits不能包含数据。他们的意义接近于Java中的interface。我将在后面的章节讨论impl和traits，本章则只讨论数据。

## 结构体

A rust struct is similar to a C struct or a C++ struct without methods. Simply a
list of named fields. The syntax is best seen with an example:

Rust中的结构体(struct)与C中的struct或C++中不带方法的struct相似，只需要列出域的名字。他的语法是：

```rust
struct S {
    field1: int,  
    field2: SomeOtherStruct  
}
```

Here we define a struct called `S` with two fields. The fields are comma
separated; if you like, you can comma-terminate the last field too.

这里我们定义了一个名叫S的struct，并定义了两个域。域之间用逗号分割。如果你喜欢，你也可以在最后一个域后面加逗号。

Structs introduce a type. In the example, we could use `S` as a type.
`SomeOtherStruct` is assumed to be another struct (used as a type in the
example), and (like C++) it is included by value, that is, there is no pointer
to another struct object in memory.
结构体定义一个类型。在这个例子中，我们使用S作为类型。并假设SomeOtherStruct是另一个类型并且以值的形式包含在struct中。所谓值的形式，意思是不使用指向其他结构的指针。

Fields in structs are accessed using the `.` operator and their name. An example
of struct use:
要访问结构体中的域，需要使用`.`运算符和域的名字。下面的例子说明了对struct的使用方法

```rust
fn foo(s1: S, s2: &S) {
    let f = s1.field1;
    if f == s2.field1 {
        println!("field1 matches!");
    }
}
```

Here `s1` is struct object passed by value and `s2` is a struct object passed by
reference. As with method call, we use the same `.` to access fields in both, no
need for `->`.
这里 `s1`是一个按值传递的struct对象，而`s2`是一个引用传递的struct对象。在函数中，我们都使用`.`来访问域，而不需要使用`->`

Structs are initialised using struct literals. These are the name of the struct
and values for each field. For example,
结构体由结构自面量(struct literals)初始化。需要指定结构体的名字以及每个域的值。下面是一个结构体初始化的例子

```rust
fn foo(sos: SomeOtherStruct) {
    let x = S { field1: 45, field2: sos };  // initialise x with a struct literal
    println!("x.field1 = {}", x.field1);
}
```

Structs cannot be recursive, that is you can't have cycles of struct names
involving definitions and field types. This is because of the value semantics of
structs. So for example, `struct R { r: Option<R> }` is illegal and will cause a
compiler error (see below for more about Option). If you need such a structure
then you should use some kind of pointer; cycles with pointers are allowed:
由于结构体定义使用值语义，结构体不能递归定义，这意味着struct内域的类型不能包含自己。像`struct R { r: Option<R> }`这样的定义是非法的，并且会触发编译错误（后面将会详细说到Option）。如果你需要递归引用的结构体，你应该使用某种形式的指针，因为指针中允许存在环。下面的例子就是一个合法的自我引用结构体

```rust
struct R {
    r: Option<Box<R>>
}
```

If we didn't have the `Option` in the above struct, there would be no way to
instantiate the struct and Rust would signal an error.
如果上例中没有Option，我们就无法初始化这个结构体，Rust会因此报错。

Structs with no fields do not use braces in either their definition or literal
use. Definitions do need a terminating semi-colon though, presumably just to
facilitate parsing.
没有域的结构体无论在定义还是字面量中都不使用大括号。大概是为了加速语法分析，在定义中还是需要有一个分号结尾。下例定义了一个空结构体

```rust
struct Empty;

fn foo() {
    let e = Empty;
}
```

## Tuples

Tuples are anonymous, heterogeneous sequences of data. As a type, they are
declared as a sequence of types in parentheses. Since there is no name, they are
identified by structure. For example, the type `(int, int)` is a pair of
integers and `(i32, f32, S)` is a triple. Enum values are initialised in the
same way as enum types are declared, but with values instead of types for the
components, e.g., `(4, 5)`. An example:
元组(Tuple)是匿名、异构的数据序列。元组由括号中的类型名来定义。因为元组没有名字，他们是由它们的结构定义的。比如类型`(int,int)`是整数而`(i32, f32, S)`是一个三元组。enum值可以用其声明的enum类型来初始化，而不能用类型以外的值(如`(4,5)`)作为元素。下面是一个例子

```rust
// foo takes a struct and returns a tuple
fn foo(x: SomeOtherStruct) -> (i32, f32, S) {
    (23, 45.82, S { field1: 54, field2: x })
}
```

Tuples can be used by destructuring using a `let` expression, e.g.,
元组可以用在let表达式之后用于解构。

```rust
fn bar(x: (int, int)) {
    let (a, b) = x;
    println!("x was ({}, {})", a, b);
}
```

We'll talk more about destructuring next time.
我们讲在后面详细说解构的问题。

## 元组结构体

Tuple structs are named tuples, or alternatively, structs with unnamed fields.
元组结构体是有名字的元组，或者说是没有域名字的结构体。
They are declared using the `struct` keyword, a list of types in parentheses,
元组结构体使用`struct`关键字声明，接着一些逗号分割的类型并用括号包裹，最后跟着一个分号。
and a semicolon. Such a declaration introduces their name as a type.
这样就声明了一个具有指定名字的类型。
 Their
fields must be accessed by destructuring (like a tuple), rather than by name.
其中的域只能像元组一样通过解构操作访问，而不能通过域的名字访问。
Tuple structs are not very common.
元组结构体不是很常用。下面是一个元组结构体的例子。

```rust
struct IntPoint (int, int);

fn foo(x: IntPoint) {
    let IntPoint(a, b) = x;  // Note that we need the name of the tuple
                             // struct to destructure.
    println!("x was ({}, {})", a, b);
}
```

## Enums

Enums are types like C++ enums or unions, in that they are types which can take
multiple values. 
枚举(Enum)是类似于C++中enum和union的类型，其中可以放置多种类型的值。The simplest kind of enum is just like a C++ enum:
最简单的一种枚举类型就像C++中的enum：

```rust
enum E1 {
    Var1,
    Var2,
    Var3
}

fn foo() {
    let x: E1 = Var2;
    match x {
        Var2 => println!("var2"),
        _ => {}
    }
}
```

However, Rust enums are much more powerful than that. Each variant can contain
data. Like tuples, these are defined by a list of types. In this case they are
more like unions than enums in C++. Rust enums are tagged unions rather untagged
(as in C++), that means you can't mistake one variant of an enum for another at
runtime. An example:
然而，Rust中的枚举比C++中的要强大的多：枚举的每个变体都可以包含数据。他们可以像元组一样通过类型列表来定义。在这种情况下enum更像是C++中的union。Rush的enum可使用带标记的union而不像C++中一样不带标记。着意味着你永远不会在运行时把一个变体当作另一个使用。举例来说：

```rust
enum Expr {
    Add(int, int),
    Or(bool, bool),
    Lit(int)
}

fn foo() {
    let x = Or(true, false);   // x has type Expr
}
```

Many simple cases of object-oriented polymorphism are better handled in Rust
using enums.
在Rust中使用enum可以实现大多数简单的面向对象多态性。

To use enums we usually use a match expression. Remember that these are similar
to C++ switch statements. I'll go into more depth on these and other ways to
destructure data next time. Here's an example:
我们通常将match表达式于enum一起使用。之前说过match表达式与C++中的switch语句相似。之后我们将深入的讨论match表达式及其与数据解构结合的高级用法。这是一个例子

```rust
fn bar(e: Expr) {
    match e {
        Add(x, y) => println!("An `Add` variant: {} + {}", x, y),
        Or(..) => println!("An `Or` variant"),
        _ => println!("Something else (in this case, a `Lit`)"),
    }
}
```

Each arm of the match expression matches a variant of `Expr`. All variants must
be covered. The last case (`_`) covers all remaining variants, although in the
example there is only `Lit`. Any data in a variant can be bound to a variable.
In the `Add` arm we are binding the two ints in an `Add` to `x` and `y`. If we
don't care about the data, we can use `..` to match any data, as we do for `Or`.
match的每一个分支匹配了`Expr`的一个变体，而且必须涵盖所有变体。最后一个`_`分支可以涵盖所有没有列出的变体（这个例子中只有`Lit`）。每个变体中的数据都被绑定在一个变量中，如在`Add`分支中我们绑定了`Add`的两个int到`x`和`y`中。如果我们不关心数据，可以像例子中的`Or`分支一样，使用`..`来匹配任意数据。

## Option

One particularly common enum in Rust is `Option`. This has two variants - `Some`
and `None`. `None` has no data and `Some` has a single field with type `T`
(`Option` is a generic enum, which we will cover later, but hopefully the
general idea is clear from C++). Options are used to indicate a value might be
there or might not. Any place you use a null pointer in C++ to indicate a value
which is in some way undefined, uninitialised, or false, you should probably use
an Option in Rust. Using Option is safer because you must always check it before
use; there is no way to do the equivalent of dereferencing a null pointer. They
are also more general, you can use them with values as well as pointers. An
example:
Rust中一个尤其常用的enum类型是`Option`。它有两个变体:`Some`和`None`。`None`不包含数据，而`Some`有一个类型为`T`的数据。`Option`是一个范型enum，我们将在后面详细说明，他的基本思想来源于C++。Option常常表示一个值可以存在也可以不存在。在C++中使用nullptr的地方（可能用于表示不可用、未定义、未初始化或者false），在Rust中都应使用Option。使用Option将会保证在使用值之前对它进行检查，从而避免出现对null指针解引用之类的操作，因而更加安全。同时它更具一般性，既可以与指针一起使用，也可以与值一起使用。下面是一个例子：

```rust
use std::rc::Rc;

struct Node {
    parent: Option<Rc<Node>>,
    value: int
}

fn is_root(node: Node) -> bool {
    match node.parent {
        Some(_) => false,
        None => true
    }
}
```

Here, the parent field could be either a `None` or a `Some` containing an
`Rc<Node>`. In the example, we never actually use that payload, but in real life
you usually would.
这里，`parent`域既可以使`None`也可以是包含一个`Rc<Node>`类型的`Some`。这个例子中通过使用`_`忽略了`Some`的载荷，因为我们并不需要它，在实际工作中有可能需要对其进行绑定。

There are also convenience methods on Option, so you could write the body of
`is_root` as `node.is_none()` or `!node.is_some()`.
Option包含了很多方法方便使用，因此你可以将刚才`is_root`函数的函数体写为`node.is_none()`或者`!node.is_some()`。

## Inherited mutabilty and Cell/RefCell

Local variables in Rust are immutable by default and can be marked mutable using
`mut`. We don't mark fields in structs or enums as mutable, their mutability is
inherited. This means that a field in a struct object is mutable or immutable
depending on whether the object itself is mutable or immutable. Example:

```rust
struct S1 {
    field1: int,
    field2: S2
}
struct S2 {
    field: int
}

fn main() {
    let s = S1 { field1: 45, field2: S2 { field: 23 } };
    // s is deeply immutable, the following mutations are forbidden
    // s.field1 = 46;
    // s.field2.field = 24;

    let mut s = S1 { field1: 45, field2: S2 { field: 23 } };
    // s is mutable, these are OK
    s.field1 = 46;
    s.field2.field = 24;
}
```

Inherited mutability in Rust stops at references. This is similar to C++ where
you can modify a non-const object via a pointer from a const object. If you want
a reference field to be mutable, you have to use `&mut` on the field type:

```rust
struct S1 {
    f: int
}
struct S2<'a> {
    f: &'a mut S1   // mutable reference field
}
struct S3<'a> {
    f: &'a S1       // immutable reference field
}

fn main() {
    let mut s1 = S1{f:56};
    let s2 = S2 { f: &mut s1};
    s2.f.f = 45;   // legal even though s2 is immutable
    // s2.f = &mut s1; // illegal - s2 is not mutable
    let s1 = S1{f:56};
    let mut s3 = S3 { f: &s1};
    s3.f = &s1;     // legal - s3 is mutable
    // s3.f.f = 45; // illegal - s3.f is immutable
}
```

(The `'a` parameter on `S2` and `S3` is a lifetime parameter, we'll cover those soon).

Sometimes whilst an object is logically mutable, it has parts which need to be
internally mutable. Think of various kinds of caching or a reference count
(which would not give true logical immutability since the effect of changing the
ref count can be observed via destructors). In C++, you would use the `mutable`
keyword to allow such mutation even when the object is const. In Rust we have
the Cell and RefCell structs. These allow parts of immutable objects to be
mutated. Whilst that is useful, it means you need to be aware that when you see
an immutable object in Rust, it is possible that some parts may actually be
mutable.

RefCell and Cell let you get around Rust's strict rules on mutation and
aliasability. They are safe to use because they ensure that Rust's invariants
are respected dynamically, even though the compiler cannot ensure that those
invariants hold statically. Cell and RefCell are both single threaded objects.

Use Cell for types which have copy semantics (pretty much just primitive types).
Cell has `get` and `set` methods for changing the stored value, and a `new`
method to initialise the cell with a value. Cell is a very simple object - it
doesn't need to do anything smart since objects with copy semantics can't keep
references elsewhere (in Rust) and they can't be shared across threads, so there
is not much to go wrong.

Use RefCell for types which have move semantics, that means nearly everything in
Rust, struct objects are a common example. RefCell is also created using `new`
and has a `set` method. To get the value in a RefCell, you must borrow it using
the borrow methods (`borrow`, `borrow_mut`, `try_borrow`, `try_borrow_mut`)
these will give you a borrowed reference to the object in the RefCell. These
methods follow the same rules as static borrowing - you can only have one
mutable borrow, and can't borrow mutably and immutably at the same time.
However, rather than a compile error you get a runtime failure. The `try_`
variants return an Option - you get `Some(val)` if the value can be borrowed and
`None` if it can't. If a value is borrowed, calling `set` will fail too.

Here's an example using a ref-counted pointer to a RefCell (a common use-case):

```rust
use std::rc::Rc;
use std::cell::RefCell;

Struct S {
    field: int
}

fn foo(x: Rc<RefCell<S>>) {
    {
        let s = x.borrow();
        println!("the field, twice {} {}", s.f, x.borrow().field);
        // let s = x.borrow_mut(); // Error - we've already borrowed the contents of x
    }

    let s = x.borrow_mut(); // O, the earlier borrows are out of scope
    s.f = 45;
    // println!("The field {}", x.borrow().field); // Error - can't mut and immut borrow
    println!("The field {}", s.f);
}
```

If you're using Cell/RefCell, you should try to put them on the smallest object
you can. That is, prefer to put them on a few fields of a struct, rather than
the whole struct. Think of them like single threaded locks, finer grained
locking is better since you are more likely to avoid colliding on a lock.
