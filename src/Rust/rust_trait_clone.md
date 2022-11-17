# Rust的Trait clone

## 背景

## 示例
```rust
use std::fmt::Debug;

pub trait DynClone {
    fn clone_box(&self) -> Box<dyn AbsClone>;
}

pub trait AbsClone: Send + Sync + DynClone + Debug {}

impl Clone for Box<dyn AbsClone> {
    fn clone(&self) -> Self {
        self.clone_box()
    }
}

#[derive(Clone)]
struct Foo {
    bar: Box<dyn AbsClone>,
}

impl<T> DynClone for T
where
    T: Clone + AbsClone + 'static,
{
    fn clone_box(&self) -> Box<dyn AbsClone> {
        Box::new(self.clone())
    }
}

#[derive(Clone, Debug)]
struct Bar {
    name: &'static str,
}

impl AbsClone for Bar {}

fn main() {
    let bar = Box::new(Bar { name: "hello bar" });
    let foo = Foo { bar };
    // expect print: Bar { name: "hello bar" }
    println!("{:?}", foo.bar);
}

```

## 参考
* https://github.com/nushell/nushell/blob/1784b4bf50/crates/nu-protocol/src/engine/command.rs#L103-L120
* https://github.com/dtolnay/dyn-clone/blob/1.0.9/src/lib.rs

## 延伸
* https://crates.io/crates/typetag