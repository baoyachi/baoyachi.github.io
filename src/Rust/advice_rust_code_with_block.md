# Rust代码优化-闭包惰性求值
![code_practice]()
## 背景
写代码时，经常会遇到从`Option`或`Result`中取值得问题，为了避免使用`unwrap()`,使用了`if let Some(xxx)`或`if let Ok(xxx)`
的代码表达。之前的文章也介绍了这种写法，惯用法也自以为习以为常。

* 1.最近一次代码在发PR时，下面这段代码作者建议，又有了写收获：
```rust
struct Message(usize);

fn get_msg(msg: &Option<Message>) -> String {
    let mut rep_msg = "Ok".to_string();
    if let Some(msg) = msg {
        rep_msg = format!("{}", msg.0)
    }
    rep_msg
}
```
* 2.为了避免`resp_msg`是`mut`，建议改使用下面这种方式：
```rust
fn get_msg(msg: &Option<Message>) -> String {
    let rep_msg = if let Some(msg) = msg {
        format!("{}", msg.0)
    } else {
        "Ok".to_string()
    };
    rep_msg
}
```
通过赋值方式传递`rep_msg`,避免了`rep_msg`的可变性。

* 3.本身上面代码看起来还是有点绕口，为了表达直观，通过代码块`{}`，可表达为下面方式：
```rust
fn get_msg(msg: &Option<Message>) -> String {
    let rep_msg = {
        if let Some(msg) = msg {
            format!("{}", msg.0)
        } else {
            "Ok".to_string()
        }
    };
    rep_msg
}
```

* 4.但是上面代码看起来还是有点啰嗦，我们继续优化，可以采用`Option`的组合子方法，链式表达，修改如下
```rust
fn get_msg(msg: &Option<Message>) -> String {
    msg.as_ref().map(|msg| format!("{}", msg.0)).unwrap_or("Ok".to_string())
}
```
是不是使用combinator的模式，代码更简单、直观了。但是，这里还是有点缺陷，我们继续往下看：

* 5.如果使用`cargo clippy`的使用，`clippy`会经常提示我们修改这段逻辑：
```rust
msg.unwrap_or("Ok".to_string())
 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ help: try this: `unwrap_or_else(|| "OK".to_string())`
                                          |
```
为什么呢？对于`unwrap_or`,即使不使用其值，参数`"Ok".to_string()`也将进行求值。对于`unwrap_or_else`，参数接收一个闭包，
当触发`unwrap_or_else`调用时，通过传递函数(闭包)进行惰性求值(lazy evaluation)计算，提升程序性能,那我们改造下
```rust
fn get_msg(msg: &Option<Message>) -> String {
    msg.as_ref().map(|msg| format!("{}", msg.0)).unwrap_or_else(||"Ok".to_string())
}
```

从第一个函数看到现在，有么有收获呢？欢迎大家在评论区留言，一起交流。

## 最后
推荐大家在Rust项目中，将 [clippy](https://github.com/rust-lang/rust-clippy) 这个插件使用起来，优化程序的结构。让优秀成为一种习惯。

## 参考链接
* [https://github.com/rust-lang/rust/pull/55014](https://github.com/rust-lang/rust/pull/55014)
* [https://stackoverflow.com/questions/56726571/why-choosing-unwrap-or-else-over-unwrap-or](https://stackoverflow.com/questions/56726571/why-choosing-unwrap-or-else-over-unwrap-or)
* [https://stackoverflow.com/questions/45547293/why-should-i-prefer-optionok-or-else-instead-of-optionok-or](https://stackoverflow.com/questions/45547293/why-should-i-prefer-optionok-or-else-instead-of-optionok-or)


